# CARLA Sahne Kurulumu ve Sentetik Veri Üretim Spesifikasyonu

## 1. CARLA Kurulumu

- **Versiyon:** CARLA 0.9.15 (Windows)
- **İndirme:** https://github.com/carla-simulator/carla/releases/tag/0.9.15/
- **Gerekli dosya:** `CARLA_0.9.15.zip` (Windows)
- **Python paketi:** `carla` (pip ile kurulacak, CARLA sürümüyle uyumlu olmalı)
- **Ek paket:** `carla-0.9.15-cp312-cp312-win_amd64.whl` (Python 3.12 uyumu kontrol edilmeli; yoksa Python 3.10 venv gerekebilir)

> ⚠️ CARLA'nın Python API'si Python 3.12 ile uyumlu olmayabilir. Bu durumda `carla/` altında ayrı bir Python 3.10 venv oluşturulmalı.

## 2. Otopark Sahne Tasarımı

### Harita Seçimi
- **Birincil:** `Town05` — geniş düz alanlar, otopark simülasyonuna uygun
- **Alternatif:** `Town03` — daha kentsel
- Agent haritayı yükleyip uygun düz alan bulmalı

### Park Bölmesi Düzeni
- **Bölme sayısı:** 20-30 arası (basit tutmak için)
- **Düzen:** 2 sıralı, karşılıklı park (gerçek bir açık otopark gibi)
- **Bölme boyutu:** ~5.0m × 2.5m (gerçekçi)
- **Sıralar arası yol:** ~6m

### Kamera Yerleşimi
- **Ana kamera:** Otopark alanının üstünden, ~15-20m yükseklikten, hafif açılı (~60-70°)
- **Çözünürlük:** 1280×720 (720p)
- **FOV:** 90°
- **Opsiyonel 2. kamera:** Farklı açıdan (çoklu kamera desteği için)

### Araç Yerleştirme
- CARLA'nın `vehicle.spawn` API'si kullanılacak
- Araç modelleri rastgele seçilecek (CARLA'nın araç blueprints'leri)
- Doluluk oranı parametre olarak verilecek (ör: %60 → 30 bölmenin 18'i dolu)
- Araçlar park bölmesi koordinatlarına yerleştirilecek (spawn_points değil, manuel koordinat)

## 3. Sentetik Veri Üretim Pipeline'ı

### Çeşitlilik Parametreleri

```python
WEATHER_PRESETS = [
    {"name": "clear_noon",      "cloudiness": 0,   "precipitation": 0,  "sun_altitude": 70},
    {"name": "cloudy",          "cloudiness": 60,  "precipitation": 0,  "sun_altitude": 50},
    {"name": "light_rain",      "cloudiness": 80,  "precipitation": 30, "sun_altitude": 40},
    {"name": "heavy_rain",      "cloudiness": 90,  "precipitation": 80, "sun_altitude": 30},
    {"name": "foggy",           "cloudiness": 70,  "precipitation": 0,  "fog_density": 50},
    {"name": "sunset",          "cloudiness": 20,  "precipitation": 0,  "sun_altitude": 10},
    {"name": "night",           "cloudiness": 10,  "precipitation": 0,  "sun_altitude": -30},
    {"name": "night_rain",      "cloudiness": 90,  "precipitation": 60, "sun_altitude": -20},
]

OCCUPANCY_RATES = [0.0, 0.2, 0.4, 0.6, 0.8, 1.0]

CAMERA_POSITIONS = [
    {"name": "overhead",   "x": 0, "y": 0, "z": 20, "pitch": -70, "yaw": 0},
    {"name": "angled",     "x": -15, "y": 0, "z": 15, "pitch": -45, "yaw": 20},
]
```

### Hedef Veri Miktarı

| Parametre Kombinasyonu | Açıklama |
|---|---|
| 8 hava koşulu × 6 doluluk × 2 kamera × ~30 çeşitleme | ~2880 görüntü minimum |
| Her çeşitlemede araç pozisyonları rastgele değişir | Daha fazla varyasyon |
| **Hedef:** 3000-5000 etiketli görüntü | |

### Otomatik Etiketleme

CARLA'nın semantic segmentation kamerası ve bounding box API'si kullanılacak:

```
Her görüntü için üretilecek:
├── image_XXXX.png          → RGB görüntü
├── image_XXXX.txt          → YOLOv8 format etiket
│   └── Her satır: class_id x_center y_center width height (normalize)
│   └── class_id: 0 = vehicle
└── image_XXXX_meta.json    → Metadata (hava, doluluk, kamera, araç listesi)
```

**Etiketleme yöntemi:**
1. CARLA'nın `get_transform()` ile her aracın dünya koordinatını al
2. Kamera intrinsic/extrinsic matrisleri ile piksel koordinatına dönüştür
3. Bounding box hesapla
4. YOLOv8 formatına normalize et

**Alternatif (daha kolay):**
1. CARLA'nın `semantic segmentation` kamerasını kullan
2. Araç piksellerini tespit et
3. Connected component ile bounding box çıkar

## 4. Zaman Serisi Verisi Üretimi (LSTM için)

### Scripted Senaryolar
```
Bir "gün" simülasyonu:
├── 00:00-06:00 → %10-20 doluluk (gece)
├── 06:00-08:00 → Doluluk artıyor (%20→%60)
├── 08:00-09:00 → Hızlı artış (%60→%90) — sabah pik
├── 09:00-12:00 → Yüksek (%85-95)
├── 12:00-13:00 → Küçük dalgalanma (öğle)
├── 13:00-17:00 → Yüksek (%80-90)
├── 17:00-19:00 → Düşüş (%90→%40) — akşam çıkış
└── 19:00-00:00 → Düşük (%20-30)

Hafta sonu farkı:
├── Genel doluluk %30-50 daha düşük
└── Pik saat yok, düz dağılım
```

### Kayıt Formatı
```csv
timestamp,lot_id,occupied_count,total_slots,occupancy_rate,weather,is_weekend,is_holiday
2024-01-15 08:00:00,1,18,30,0.60,clear,0,0
2024-01-15 08:15:00,1,21,30,0.70,clear,0,0
...
```

### Minimum Veri
- **30 gün** simülasyonu (22 hafta içi + 8 hafta sonu)
- **15 dakika** periyot → günde 96 kayıt → toplam ~2880 kayıt
- Birmingham açık kaynak veri seti ek olarak kullanılabilir

## 5. Dosya Çıktıları

```
carla/
├── scene_setup.py          # Sahne kurulumu (harita, kamera, bölmeler)
├── data_generator.py       # Sentetik görüntü üretimi (batch)
├── timeseries_generator.py # LSTM zaman serisi üretimi
├── realtime_feed.py        # Gerçek zamanlı görüntü akışı (Faz 3 için)
├── utils.py                # Yardımcı fonksiyonlar (koordinat dönüşümü vb.)
├── requirements.txt
└── configs/
    ├── parking_polygons.json    # Park bölmesi koordinatları
    ├── camera_configs.json      # Kamera pozisyonları
    └── weather_presets.json     # Hava koşulları presetleri
```
