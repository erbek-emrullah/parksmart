# Streamlit Web Dashboard Spesifikasyonu

## 1. Amaç

Jüri sunumu ve yönetici kullanımı için büyük ekranda gösterilecek web tabanlı
gerçek zamanlı izleme ve kontrol paneli.

## 2. Teknoloji

- **Framework:** Streamlit
- **Grafik:** Plotly (interaktif) + Streamlit native charts
- **Gerçek zamanlı:** `st.experimental_rerun` + polling veya Streamlit WebSocket
- **Harita:** Matplotlib/Plotly ile 2D otopark görselleştirme

## 3. Sayfa Yapısı

### Ana Sayfa: Canlı İzleme

```
┌─────────────────────────────────────────────────────────────┐
│  🅿️ ParkSmart Dashboard                     [Canlı ●]      │
├─────────────────────────┬───────────────────────────────────┤
│                         │  Anlık Durum                      │
│   OTOPARK HARİTASI      │  ┌──────┐ ┌──────┐ ┌──────┐     │
│   (kuş bakışı 2D)       │  │  18  │ │  12  │ │ %60  │     │
│                         │  │ Dolu │ │ Boş  │ │Doluluk│     │
│  🟢🟢🔴🔴🟢🔴🟢🔴      │  └──────┘ └──────┘ └──────┘     │
│  🔴🟢🟢🔴🔴🟢🔴🟢      │                                  │
│  🟢🔴🔴🟢🟢🟢🔴🔴      │  YOLOv8 Canlı Tespit            │
│                         │  ┌─────────────────────────┐     │
│                         │  │  [kamera görüntüsü      │     │
│                         │  │   bounding box'larla]    │     │
│                         │  └─────────────────────────┘     │
├─────────────────────────┴───────────────────────────────────┤
│  24 Saatlik Tahmin (LSTM)                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  [interaktif çizgi grafik - Plotly]                  │    │
│  └─────────────────────────────────────────────────────┘    │
├─────────────────────────────────────────────────────────────┤
│  Doluluk Isı Haritası (Son 7 Gün)                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  [saat × gün ısı haritası - Plotly heatmap]          │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Sidebar: CARLA Kontrolleri (Demo İçin)

```
┌─────────────────────┐
│  🎮 CARLA Kontrol    │
│                     │
│  Hava Koşulu:       │
│  [Güneşli     ▼]    │
│                     │
│  Doluluk:           │
│  [====|====] %60    │
│                     │
│  [Uygula]           │
│                     │
│  ─────────────────  │
│  📊 Sim-to-Real     │
│  [Sonuçları Göster] │
│                     │
│  ─────────────────  │
│  Son güncelleme:    │
│  14:32:15           │
│  Gecikme: 1.2s      │
└─────────────────────┘
```

### Sim-to-Real Sonuçları Sayfası

```
┌─────────────────────────────────────────────────────────────┐
│  📊 Sim-to-Real Transfer Deney Sonuçları                    │
│                                                             │
│  ┌─────────────────────────────────────────┐                │
│  │ Model    │ mAP@0.5 │ Precision │ Recall │                │
│  │──────────│─────────│───────────│────────│                │
│  │ PKLot    │  0.XX   │   0.XX    │  0.XX  │                │
│  │ CARLA    │  0.XX   │   0.XX    │  0.XX  │                │
│  │ Hibrit   │  0.XX   │   0.XX    │  0.XX  │                │
│  │ Hibrit+DA│  0.XX   │   0.XX    │  0.XX  │                │
│  └─────────────────────────────────────────┘                │
│                                                             │
│  ┌─────────────────────┐ ┌─────────────────────┐           │
│  │ [mAP karşılaştırma  │ │ [sentetik veri      │           │
│  │  bar chart]          │ │  miktarı vs mAP]   │           │
│  └─────────────────────┘ └─────────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────┐           │
│  │ [LSTM tahmin vs gerçek grafiği]              │           │
│  └─────────────────────────────────────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

## 4. CARLA Kontrol Mekanizması

Dashboard → Backend API → CARLA:

```
POST /api/v1/carla/control
  Body: {
    "weather": "heavy_rain",
    "occupancy_rate": 0.8
  }
  Response: { "status": "applied" }

Bu endpoint backend'den CARLA'ya komut gönderir
(HTTP veya shared config dosyası ile)
```

## 5. Dosya Yapısı

```
dashboard/
├── app.py                  # Ana Streamlit uygulaması
├── pages/
│   ├── 1_live_monitoring.py    # Canlı izleme
│   └── 2_experiment_results.py # Sim-to-Real sonuçları
├── components/
│   ├── parking_map.py      # 2D otopark çizimi
│   ├── prediction_chart.py # LSTM tahmin grafiği
│   └── heatmap.py          # Doluluk ısı haritası
├── utils/
│   └── api_client.py       # Backend API çağrıları
├── requirements.txt
└── .streamlit/
    └── config.toml         # Tema konfigürasyonu
```

## 6. requirements.txt

```
streamlit>=1.29.0
plotly>=5.18.0
requests>=2.31.0
pandas>=2.1.0
numpy>=1.26.0
Pillow>=10.0.0
websocket-client>=1.6.0
```
