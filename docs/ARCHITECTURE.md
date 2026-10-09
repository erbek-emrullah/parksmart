# ARCHITECTURE.md — Sistem Mimarisi, Modüller ve Veri Akışı

## Genel Mimari — Üç Katmanlı Tasarım

```
┌─────────────────────────────────────────────────────────────┐
│                    KATMAN 1: CARLA DİJİTAL İKİZ            │
│                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │ Sanal Otopark│───▶│ Kamera       │───▶│ YOLOv8n      │  │
│  │ Sahnesi      │    │ Görüntüleri  │    │ Tespit       │  │
│  └──────────────┘    └──────────────┘    └──────┬───────┘  │
│                                                  │          │
│  ┌──────────────┐                      ┌────────▼───────┐  │
│  │ Sentetik     │                      │ Poligon-IoU    │  │
│  │ Veri Üretimi │                      │ Eşleme        │  │
│  │ (Eğitim)     │                      │ (Shapely)     │  │
│  └──────────────┘                      └────────┬───────┘  │
│                                                  │          │
│                              Doluluk JSON (0/1)  │          │
└──────────────────────────────────────────────────┼──────────┘
                                                   │
                                          HTTP/REST│
                                                   ▼
┌─────────────────────────────────────────────────────────────┐
│                    KATMAN 2: FASTAPI SUNUCU                 │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │ Auth     │  │ Otopark  │  │ LSTM     │  │ Doluluk   │  │
│  │ (JWT)    │  │ CRUD     │  │ Tahmin   │  │ Güncelle  │  │
│  └──────────┘  └──────────┘  └────┬─────┘  └─────┬─────┘  │
│                                    │              │         │
│  ┌──────────┐  ┌──────────┐  ┌────▼──────────────▼─────┐  │
│  │ Rate     │  │ MCDM     │  │ PostgreSQL              │  │
│  │ Limiting │  │ Atama    │  │ (ORM)                   │  │
│  └──────────┘  └──────────┘  └────┬────────────────────┘  │
│                                    │                        │
│  ┌──────────┐  ┌──────────┐  ┌────▼─────┐                 │
│  │ FCM Push │  │ WebSocket│  │ Redis    │                  │
│  │ Bildirim │  │ Sunucu   │  │ Önbellek │                  │
│  └──────────┘  └────┬─────┘  └──────────┘                  │
└──────────────────────┼──────────────────────────────────────┘
                       │
              WebSocket│
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                    KATMAN 3: ANDROİD MOBİL UYGULAMA        │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │ Giriş/   │  │ Canlı    │  │ Rezerv.  │  │ Tahmin    │  │
│  │ Kayıt    │  │ Harita   │  │ Ekranı   │  │ Grafiği   │  │
│  └──────────┘  └──────────┘  └──────────┘  └───────────┘  │
│                                                             │
│  Mimari: MVVM (Repository → ViewModel → Compose UI)        │
│  Teknolojiler: Kotlin, Jetpack Compose, Retrofit2,          │
│                Socket.IO, FCM, MPAndroidChart               │
└─────────────────────────────────────────────────────────────┘
```

> **Kaynak:** Belge s.14 (Genel Sistem Mimarisi)

## Modül Detayları

### M1: CARLA Dijital İkiz Modülü

| Özellik | Detay |
|---|---|
| **Araç** | CARLA Simulator + Python API |
| **Girdi** | Sahne konfigürasyonu (kamera, hava, araç, ışık) |
| **Çıktı-1** | Etiketli sentetik eğitim görüntüleri (offline) |
| **Çıktı-2** | Gerçek zamanlı kamera kareleri (runtime) |
| **İşlem** | Sahne kurulumu → Kare yakalama → YOLOv8 çıkarım → Poligon IoU eşleme → JSON çıktı |

### M2: YOLOv8 Nesne Tespiti Modülü

| Özellik | Detay |
|---|---|
| **Model** | YOLOv8n (Nano varyant) |
| **Eğitim verisi** | PKLot + CARLA hibrit havuz |
| **Transfer öğrenimi** | COCO pretrained → ince ayar |
| **Girdi** | Kamera karesi (görüntü) |
| **Çıktı** | Bounding box'lar (sınıf: araç) |
| **Poligon eşleme** | Shapely IoU; eşik = 0.25 |
| **Son çıktı** | Her bölme için dolu/boş (0/1) JSON |

### M3: LSTM Tahmin Modülü

| Özellik | Detay |
|---|---|
| **Model** | LSTM (framework belgede belirsiz) |
| **Eğitim ortamı** | Google Colab |
| **Girdi özellikleri** | Doluluk oranı, sin/cos kodlu saat-gün, hafta sonu/tatil/yoğun saat binary |
| **Girdi penceresi** | Son 24 saat (15 dk periyotlarla → 96 zaman adımı) |
| **Çıktı** | Gelecek 24 saat doluluk tahmini |
| **Eğitim teknikleri** | Erken durdurma, LR scheduler, en iyi model kaydetme |
| **Üretim** | FastAPI'de yüklü model; Redis'ten veri alır, tahmin üretir |

### M4: FastAPI Backend Modülü

**API Uç Noktaları (Tasarlanan):**

| Uç Nokta | Metod | Açıklama |
|---|---|---|
| `/auth/register` | POST | Kullanıcı kaydı |
| `/auth/login` | POST | JWT ile giriş |
| `/auth/refresh` | POST | Token yenileme |
| `/parking/lots` | GET | Otopark listesi |
| `/parking/lots/{id}/slots` | GET | Bölme durumları |
| `/reservations` | POST | Rezervasyon oluşturma |
| `/predictions/{lot_id}` | GET | LSTM tahmin sonuçları |
| `/occupancy/update` | POST | CARLA'dan doluluk güncellemesi |

> ⚠️ *Not: API uç noktaları belgeden çıkarılan taslak tasarımdır; isimlendirme ve yapı uygulama sırasında değişebilir.*

**Alt Bileşenler:**
- **JWT Auth:** 15dk access + 7gün refresh token
- **Rate Limiter:** Kötüye kullanım engelleme (parametreler belirsiz)
- **MCDM Scoring:** `Score(s) = w1·(1/d_exit) + w2·(1/d_elevator) + w3·(1/|floor|) + w4·H(s)`
- **FCM Push:** Bekleme listesindeki kullanıcılara toplu bildirim
- **WebSocket:** Anlık doluluk güncellemesi yayını

### M5: Veritabanı Katmanı

**Tablolar (Tasarlanan):**

| Tablo | Açıklama |
|---|---|
| `users` | Kullanıcı bilgileri |
| `parking_lots` | Otopark tesisleri |
| `parking_slots` | Park bölmeleri (poligon koordinatları dahil) |
| `reservations` | Rezervasyonlar (DB-level locking ile çakışma engelleme) |
| `occupancy_history` | 15 dk periyotlarla doluluk geçmişi (LSTM eğitim/girdi verisi) |
| `waitlist` | Bekleme listesi |
| `prediction_cache` | LSTM tahmin önbelleği |

> ⚠️ *Not: Tablo yapısı belgeden çıkarılmış taslaktır. ER diyagramı belgede eksik.*

**Önbellek:** Redis — sık erişilen veriler (anlık doluluk, tahmin sonuçları).

### M6: Android Mobil Uygulama

**Mimari:** MVVM

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐
│ Compose UI  │────▶│  ViewModel   │────▶│  Repository   │
│ (5 Ekran)   │◀────│  (İş mantığı)│◀────│  (Veri kayn.) │
└─────────────┘     └──────────────┘     └───────┬───────┘
                                                  │
                                    ┌─────────────┼─────────────┐
                                    │             │             │
                              ┌─────▼───┐  ┌─────▼───┐  ┌─────▼───┐
                              │Retrofit2│  │Socket.IO│  │  FCM    │
                              │(REST)   │  │(WS)     │  │(Push)   │
                              └─────────┘  └─────────┘  └─────────┘
```

**Ekranlar:**
1. Giriş / Kayıt
2. Canlı otopark haritası (WebSocket ile anlık)
3. Rezervasyon (MCDM sonucu)
4. 24 saatlik tahmin grafiği (MPAndroidChart)
5. Profil

## Veri Akışı — Uçtan Uca

```
CARLA Sahnesi
    │
    ▼ (kamera karesi)
YOLOv8n Tespiti
    │
    ▼ (bounding box'lar)
Shapely IoU Eşleme
    │
    ▼ (bölme durumları JSON)
FastAPI REST Endpoint ──────────▶ PostgreSQL (occupancy_history)
    │                                    │
    ├──▶ WebSocket broadcast             ▼
    │         │                    LSTM Tahmin Servisi
    │         │                          │
    │         ▼                          ▼
    │    Mobil: Canlı Harita      Redis Önbellek
    │                                    │
    │                                    ▼
    │                             Mobil: Tahmin Grafiği
    │
    ├──▶ MCDM Puanlama ──▶ Mobil: Rezervasyon
    │
    └──▶ FCM Push ──▶ Mobil: Bildirim
```
