# Backend API Spesifikasyonu

## 1. Teknoloji Yığını

| Bileşen | Teknoloji |
|---|---|
| Framework | FastAPI (Python 3.12) |
| ORM | SQLAlchemy 2.x (async) |
| Veritabanı | PostgreSQL 16 |
| Migrasyon | Alembic |
| Auth | JWT (python-jose + passlib[bcrypt]) |
| Validasyon | Pydantic v2 |
| WebSocket | FastAPI native WebSocket |
| Önbellek | Redis (opsiyonel, Faz 5'te eklenebilir) |
| ASGI Server | Uvicorn |

## 2. Veritabanı Şeması

### users
```sql
CREATE TABLE users (
    id              SERIAL PRIMARY KEY,
    email           VARCHAR(255) UNIQUE NOT NULL,
    hashed_password VARCHAR(255) NOT NULL,
    full_name       VARCHAR(100),
    vehicle_type    VARCHAR(20) DEFAULT 'sedan',  -- sedan, suv, motorcycle, disabled
    is_active       BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMP DEFAULT NOW(),
    updated_at      TIMESTAMP DEFAULT NOW()
);
```

### parking_lots
```sql
CREATE TABLE parking_lots (
    id          SERIAL PRIMARY KEY,
    name        VARCHAR(100) NOT NULL,
    total_slots INTEGER NOT NULL,
    location    VARCHAR(255),
    is_active   BOOLEAN DEFAULT TRUE,
    created_at  TIMESTAMP DEFAULT NOW()
);
```

### parking_slots
```sql
CREATE TABLE parking_slots (
    id              SERIAL PRIMARY KEY,
    lot_id          INTEGER REFERENCES parking_lots(id),
    slot_number     VARCHAR(10) NOT NULL,        -- "A1", "A2", "B1"...
    polygon_coords  JSONB NOT NULL,              -- [[x1,y1],[x2,y2],[x3,y3],[x4,y4]]
    slot_type       VARCHAR(20) DEFAULT 'standard', -- standard, disabled, motorcycle
    is_occupied     BOOLEAN DEFAULT FALSE,
    distance_to_exit     FLOAT DEFAULT 0,
    distance_to_elevator FLOAT DEFAULT 0,
    floor_level          INTEGER DEFAULT 0,
    updated_at      TIMESTAMP DEFAULT NOW(),
    UNIQUE(lot_id, slot_number)
);
```

### occupancy_history
```sql
CREATE TABLE occupancy_history (
    id              SERIAL PRIMARY KEY,
    lot_id          INTEGER REFERENCES parking_lots(id),
    timestamp       TIMESTAMP NOT NULL,
    occupied_count  INTEGER NOT NULL,
    total_slots     INTEGER NOT NULL,
    occupancy_rate  FLOAT NOT NULL,              -- occupied_count / total_slots
    weather         VARCHAR(50),                 -- CARLA weather preset
    created_at      TIMESTAMP DEFAULT NOW()
);
-- 15 dakikada bir kayıt eklenir (LSTM girdi verisi)
CREATE INDEX idx_occupancy_lot_time ON occupancy_history(lot_id, timestamp);
```

### reservations
```sql
CREATE TABLE reservations (
    id          SERIAL PRIMARY KEY,
    user_id     INTEGER REFERENCES users(id),
    slot_id     INTEGER REFERENCES parking_slots(id),
    status      VARCHAR(20) DEFAULT 'active',   -- active, completed, cancelled, expired
    created_at  TIMESTAMP DEFAULT NOW(),
    expires_at  TIMESTAMP NOT NULL,
    completed_at TIMESTAMP
);
-- Aynı slot'a eşzamanlı rezervasyonu engellemek için:
CREATE UNIQUE INDEX idx_active_reservation
    ON reservations(slot_id)
    WHERE status = 'active';
```

## 3. API Endpoint'leri

### Auth

```
POST /api/v1/auth/register
  Body: { "email": str, "password": str, "full_name": str, "vehicle_type": str }
  Response: { "id": int, "email": str, "access_token": str }

POST /api/v1/auth/login
  Body: { "email": str, "password": str }
  Response: { "access_token": str, "refresh_token": str, "token_type": "bearer" }

POST /api/v1/auth/refresh
  Body: { "refresh_token": str }
  Response: { "access_token": str }
```

### Parking

```
GET /api/v1/parking/lots
  Auth: JWT required
  Response: [{ "id": int, "name": str, "total_slots": int, "occupied_slots": int, "occupancy_rate": float }]

GET /api/v1/parking/lots/{lot_id}/slots
  Auth: JWT required
  Response: [{ "id": int, "slot_number": str, "is_occupied": bool, "slot_type": str }]

POST /api/v1/parking/lots/{lot_id}/occupancy
  Auth: Internal API key (CARLA → Backend)
  Body: { "slots": [{ "slot_number": str, "is_occupied": bool }], "weather": str }
  Response: { "updated_count": int }
  Side effects:
    - parking_slots tablosunu günceller
    - WebSocket broadcast yapar
    - 15 dk'da bir occupancy_history'ye kayıt ekler
```

### Reservations

```
POST /api/v1/reservations
  Auth: JWT required
  Body: { "lot_id": int }  -- slot otomatik atanır (MCDM)
  Response: { "reservation_id": int, "slot_number": str, "expires_at": str }

DELETE /api/v1/reservations/{id}
  Auth: JWT required (kendi rezervasyonu)
  Response: { "status": "cancelled" }

GET /api/v1/reservations/my
  Auth: JWT required
  Response: [{ "id": int, "slot_number": str, "status": str, "expires_at": str }]
```

### Predictions

```
GET /api/v1/predictions/{lot_id}
  Auth: JWT required
  Query: ?hours=24  (varsayılan 24)
  Response: {
    "lot_id": int,
    "predictions": [
      { "timestamp": str, "predicted_occupancy_rate": float }
    ],
    "model_info": { "model": "LSTM", "rmse": float, "last_trained": str }
  }
```

### WebSocket

```
WS /ws/occupancy/{lot_id}
  Auth: JWT token query param (?token=xxx)
  Server → Client mesajları:
  {
    "type": "occupancy_update",
    "data": {
      "lot_id": int,
      "timestamp": str,
      "slots": [{ "slot_number": str, "is_occupied": bool }],
      "occupancy_rate": float
    }
  }
```

## 4. JWT Konfigürasyonu

```python
SECRET_KEY = "..."           # .env dosyasından
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 15
REFRESH_TOKEN_EXPIRE_DAYS = 7
```

## 5. MCDM Park Yeri Atama

```python
def calculate_slot_score(slot, user):
    """
    Score(s) = w1 * (1/d_exit) + w2 * (1/d_elevator) + w3 * (1/(|floor|+1)) + w4 * H(s)
    """
    weights = get_weights_for_vehicle_type(user.vehicle_type)
    # weights = {"exit": 0.4, "elevator": 0.2, "floor": 0.2, "history": 0.2}

    # Engelli: asansör ağırlığı artır
    # Motosiklet: çıkış ağırlığı artır

    score = (
        weights["exit"]     * (1.0 / max(slot.distance_to_exit, 0.1)) +
        weights["elevator"] * (1.0 / max(slot.distance_to_elevator, 0.1)) +
        weights["floor"]    * (1.0 / (abs(slot.floor_level) + 1)) +
        weights["history"]  * get_history_score(slot.id, user.id)
    )
    return score

# Varsayılan ağırlıklar:
WEIGHT_PRESETS = {
    "sedan":      {"exit": 0.3, "elevator": 0.2, "floor": 0.3, "history": 0.2},
    "suv":        {"exit": 0.3, "elevator": 0.2, "floor": 0.3, "history": 0.2},
    "motorcycle": {"exit": 0.5, "elevator": 0.1, "floor": 0.2, "history": 0.2},
    "disabled":   {"exit": 0.1, "elevator": 0.5, "floor": 0.3, "history": 0.1},
}
```

## 6. Proje Yapısı

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI app, CORS, lifespan
│   ├── api/
│   │   ├── __init__.py
│   │   ├── deps.py             # Dependency injection (get_db, get_current_user)
│   │   ├── auth.py             # /auth/* routes
│   │   ├── parking.py          # /parking/* routes
│   │   ├── reservations.py     # /reservations/* routes
│   │   ├── predictions.py      # /predictions/* routes
│   │   └── websocket.py        # WebSocket handler
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── parking.py
│   │   └── occupancy.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── parking.py
│   │   ├── reservation.py
│   │   └── prediction.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── auth_service.py
│   │   ├── detection_service.py    # YOLOv8 çıkarım + Shapely IoU
│   │   ├── prediction_service.py   # LSTM tahmin
│   │   └── scoring_service.py      # MCDM puanlama
│   └── core/
│       ├── __init__.py
│       ├── config.py           # Pydantic Settings (.env)
│       ├── database.py         # Async SQLAlchemy engine + session
│       └── security.py         # JWT encode/decode, password hash
├── alembic/
│   ├── env.py
│   └── versions/
├── alembic.ini
├── requirements.txt
├── .env.example
└── Dockerfile
```

## 7. requirements.txt

```
fastapi>=0.104.0
uvicorn[standard]>=0.24.0
sqlalchemy[asyncio]>=2.0
asyncpg>=0.29.0
alembic>=1.13.0
pydantic>=2.5.0
pydantic-settings>=2.1.0
python-jose[cryptography]>=3.3.0
passlib[bcrypt]>=1.7.4
python-multipart>=0.0.6
websockets>=12.0
shapely>=2.0
ultralytics>=8.0
tensorflow>=2.15.0
redis>=5.0.0
httpx>=0.25.0
```
