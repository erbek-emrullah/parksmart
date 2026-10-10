# Android Mobil Uygulama Spesifikasyonu

## 1. Teknoloji Yığını

| Bileşen | Teknoloji | Versiyon |
|---|---|---|
| Dil | Kotlin | 1.9+ |
| UI | Jetpack Compose | BOM latest |
| Mimari | MVVM | — |
| HTTP | Retrofit2 + OkHttp | 2.9+ |
| JSON | Moshi veya Kotlinx Serialization | — |
| WebSocket | Socket.IO Client (Java) | 2.x |
| Push Bildirim | Firebase Cloud Messaging | — |
| Grafik | MPAndroidChart | 3.1+ |
| Navigasyon | Compose Navigation | — |
| DI | Hilt (Dagger) | — |
| Token Saklama | DataStore (Preferences) | — |

## 2. Mimari — MVVM

```
┌─────────────────────────────────────────┐
│              UI Layer                    │
│  Jetpack Compose Screens                │
│  ├── LoginScreen                        │
│  ├── ParkingMapScreen                   │
│  ├── PredictionScreen                   │
│  ├── ReservationScreen (bonus)          │
│  └── ProfileScreen (bonus)              │
└──────────────┬──────────────────────────┘
               │ observe (StateFlow/LiveData)
┌──────────────▼──────────────────────────┐
│           ViewModel Layer               │
│  ├── AuthViewModel                      │
│  ├── ParkingViewModel                   │
│  ├── PredictionViewModel                │
│  └── ReservationViewModel               │
└──────────────┬──────────────────────────┘
               │ calls
┌──────────────▼──────────────────────────┐
│          Repository Layer               │
│  ├── AuthRepository                     │
│  ├── ParkingRepository                  │
│  └── PredictionRepository              │
└──────────────┬──────────────────────────┘
               │ uses
┌──────────────▼──────────────────────────┐
│         Data Sources                    │
│  ├── ApiService (Retrofit2)             │
│  ├── SocketManager (Socket.IO)          │
│  ├── TokenManager (DataStore)           │
│  └── FCMService                         │
└─────────────────────────────────────────┘
```

## 3. Ekranlar

### Ekran 1: Giriş / Kayıt (LoginScreen)

```
Tasarım:
┌─────────────────────────┐
│     🅿️ ParkSmart        │
│                         │
│  ┌───────────────────┐  │
│  │ E-posta           │  │
│  └───────────────────┘  │
│  ┌───────────────────┐  │
│  │ Şifre             │  │
│  └───────────────────┘  │
│                         │
│  [    Giriş Yap     ]  │
│                         │
│  Hesabın yok mu?        │
│  Kayıt ol               │
│                         │
│  --- Kayıt formu ---    │
│  Araç tipi: [Dropdown]  │
│  sedan|suv|motor|engelli│
└─────────────────────────┘

Davranış:
- POST /auth/login → JWT token al → DataStore'a kaydet
- Token varsa direkt ParkingMapScreen'e yönlendir
- Hata: Snackbar ile göster
```

### Ekran 2: Canlı Otopark Haritası (ParkingMapScreen) ⭐

```
Tasarım:
┌─────────────────────────┐
│ 🅿️ Kampüs Otoparkı  ≡  │
│ Doluluk: 18/30 (%60)    │
│ ┌─────────────────────┐ │
│ │                     │ │
│ │  [A1][A2][A3][A4]   │ │
│ │  [A5][A6][A7][A8]   │ │
│ │                     │ │
│ │  [B1][B2][B3][B4]   │ │
│ │  [B5][B6][B7][B8]   │ │
│ │                     │ │
│ │  🟢 Boş  🔴 Dolu    │ │
│ │  🔵 Senin           │ │
│ └─────────────────────┘ │
│                         │
│ [  Yer Ayırt  ]        │
│                         │
│ 🏠  🅿️  📊  👤         │
└─────────────────────────┘

Davranış:
- Socket.IO ile /ws/occupancy/{lot_id} dinle
- Her güncelleme geldiğinde slot renklerini değiştir
- Boş = yeşil, dolu = kırmızı, kendi rezervasyonu = mavi
- "Yer Ayırt" → POST /reservations → sonucu göster
```

### Ekran 3: Tahmin Grafiği (PredictionScreen)

```
Tasarım:
┌─────────────────────────┐
│ 📊 Doluluk Tahmini      │
│                         │
│ ┌─────────────────────┐ │
│ │  100%│    ╭─╮       │ │
│ │   80%│╭──╯  ╰──╮    │ │
│ │   60%│╯        ╰╮   │ │
│ │   40%│          ╰─╮ │ │
│ │   20%│            ╰╮│ │
│ │     0├──┬──┬──┬──┬─┤│ │
│ │      08 12 16 20 00 │ │
│ │                     │ │
│ │  ── Gerçek  -- Tahmin│ │
│ └─────────────────────┘ │
│                         │
│ En yoğun: 09:00 (%92)   │
│ En boş:   22:00 (%15)   │
│ Önerilen: 14:00 (%45)   │
│                         │
│ 🏠  🅿️  📊  👤         │
└─────────────────────────┘

Davranış:
- GET /predictions/{lot_id} → 24 saatlik veri
- MPAndroidChart LineChart ile çiz
- İki çizgi: geçmiş (gerçek) + gelecek (tahmin)
- Pik ve en boş saatleri metin olarak göster
```

### Ekran 4: Rezervasyon (ReservationScreen) — Bonus

```
Tasarım:
┌─────────────────────────┐
│ 🎫 Rezervasyonlarım     │
│                         │
│ ┌─────────────────────┐ │
│ │ Aktif Rezervasyon    │ │
│ │ Bölme: A3            │ │
│ │ Süre: 42 dk kaldı    │ │
│ │ [   İptal Et    ]    │ │
│ └─────────────────────┘ │
│                         │
│ Geçmiş:                 │
│ • A5 - 12 Ekim 14:30   │
│ • B2 - 11 Ekim 09:15   │
│                         │
│ 🏠  🅿️  📊  👤         │
└─────────────────────────┘
```

### Ekran 5: Profil (ProfileScreen) — Bonus

```
Tasarım:
┌─────────────────────────┐
│ 👤 Profilim              │
│                         │
│ Ad: Emrullah Erbek      │
│ E-posta: emrul@...      │
│ Araç: Sedan             │
│                         │
│ Toplam park: 23         │
│ En sık bölme: A3        │
│                         │
│ [  Çıkış Yap  ]        │
│                         │
│ 🏠  🅿️  📊  👤         │
└─────────────────────────┘
```

## 4. API Bağlantısı

### Retrofit2 Service Interface
```kotlin
interface ParkingApi {
    @POST("auth/login")
    suspend fun login(@Body request: LoginRequest): LoginResponse

    @POST("auth/register")
    suspend fun register(@Body request: RegisterRequest): RegisterResponse

    @GET("parking/lots")
    suspend fun getLots(): List<ParkingLot>

    @GET("parking/lots/{lotId}/slots")
    suspend fun getSlots(@Path("lotId") lotId: Int): List<ParkingSlot>

    @POST("reservations")
    suspend fun createReservation(@Body request: ReservationRequest): ReservationResponse

    @GET("predictions/{lotId}")
    suspend fun getPredictions(@Path("lotId") lotId: Int): PredictionResponse
}
```

### Socket.IO Bağlantısı
```kotlin
// Bağlantı URL: http://<backend_ip>:8000
// Event: "occupancy_update" → ParkingSlot listesi güncelle
// Auth: JWT token header veya query param
```

## 5. Paket Yapısı

```
com.parksmart.app/
├── di/                     # Hilt modules
│   ├── NetworkModule.kt
│   └── DatabaseModule.kt
├── data/
│   ├── api/
│   │   └── ParkingApi.kt
│   ├── socket/
│   │   └── SocketManager.kt
│   ├── local/
│   │   └── TokenManager.kt
│   └── repository/
│       ├── AuthRepository.kt
│       ├── ParkingRepository.kt
│       └── PredictionRepository.kt
├── domain/
│   └── model/
│       ├── User.kt
│       ├── ParkingLot.kt
│       ├── ParkingSlot.kt
│       ├── Reservation.kt
│       └── Prediction.kt
├── ui/
│   ├── theme/
│   │   └── Theme.kt
│   ├── navigation/
│   │   └── NavGraph.kt
│   ├── auth/
│   │   ├── LoginScreen.kt
│   │   └── AuthViewModel.kt
│   ├── parking/
│   │   ├── ParkingMapScreen.kt
│   │   └── ParkingViewModel.kt
│   ├── prediction/
│   │   ├── PredictionScreen.kt
│   │   └── PredictionViewModel.kt
│   ├── reservation/
│   │   ├── ReservationScreen.kt
│   │   └── ReservationViewModel.kt
│   └── profile/
│       ├── ProfileScreen.kt
│       └── ProfileViewModel.kt
└── ParkSmartApplication.kt
```

## 6. Renk Paleti

```kotlin
// Marka renkleri
val PrimaryBlue = Color(0xFF1976D2)
val PrimaryDark = Color(0xFF0D47A1)
val AccentGreen = Color(0xFF4CAF50)

// Slot durumları
val SlotEmpty = Color(0xFF4CAF50)       // Yeşil
val SlotOccupied = Color(0xFFF44336)    // Kırmızı
val SlotReserved = Color(0xFF2196F3)    // Mavi
val SlotDisabled = Color(0xFFFF9800)    // Turuncu
```
