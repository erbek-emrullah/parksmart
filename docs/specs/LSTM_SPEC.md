# LSTM Doluluk Tahmin Modeli Spesifikasyonu

## 1. Amaç

Son 24 saatlik otopark doluluk verisinden, gelecek 24 saatin doluluk dağılımını
tahmin eden bir LSTM modeli eğitmek.

## 2. Veri Kaynakları

### Birincil: Birmingham Parking Dataset
- **URL:** https://archive.ics.uci.edu/dataset/482/parking+birmingham
- **İçerik:** 35 otopark, ~6 aylık saatlik doluluk verisi
- **Format:** CSV (timestamp, occupancy, capacity)
- **Kullanım:** LSTM'in ana eğitim verisi

### İkincil: CARLA Simülasyon Verisi
- `carla/timeseries_generator.py` tarafından üretilen sentetik zaman serisi
- 15 dakikalık periyotlarla doluluk kayıtları
- Gerçek veriyi desteklemek için ek veri kaynağı

## 3. Özellik Mühendisliği (Feature Engineering)

### Girdi Özellikleri

| Özellik | Tip | Açıklama |
|---|---|---|
| `occupancy_rate` | float [0,1] | Anlık doluluk oranı |
| `hour_sin` | float [-1,1] | `sin(2π × saat / 24)` |
| `hour_cos` | float [-1,1] | `cos(2π × saat / 24)` |
| `dow_sin` | float [-1,1] | `sin(2π × gün / 7)` — day of week |
| `dow_cos` | float [-1,1] | `cos(2π × gün / 7)` |
| `is_weekend` | binary 0/1 | Cumartesi veya Pazar mı? |
| `is_rush_hour` | binary 0/1 | 08:00-09:30 veya 17:00-18:30 arası mı? |

> Sin-cos kodlama gerekçesi: saat 23 ile 0 arasındaki geçişi modelin doğru
> algılamasını sağlar. Basit integer saat (0-23) kullanmak bu geçişi bozar.

### Normalizasyon
- `occupancy_rate` zaten [0,1] aralığında
- Sin/cos zaten [-1,1] aralığında
- Binary özellikler zaten 0/1
- **Ek normalizasyon gerekmez** (MinMaxScaler opsiyonel)

## 4. Model Mimarisi

```
Input Shape: (96, 7)
│  96 = 24 saat × 4 (15dk periyot)
│  7  = özellik sayısı
│
├── LSTM Layer 1 (64 units, return_sequences=True)
├── Dropout (0.2)
├── LSTM Layer 2 (32 units, return_sequences=False)
├── Dropout (0.2)
├── Dense (64, activation='relu')
├── Dense (96, activation='sigmoid')
│
Output Shape: (96,)
│  96 = gelecek 24 saatin doluluk tahminleri
│  sigmoid → [0,1] aralığında doluluk oranı
```

### Alternatif Mimariler (Karşılaştırma İçin)

**GRU Modeli:**
```
Input → GRU(64) → Dropout(0.2) → GRU(32) → Dropout(0.2) → Dense(64) → Dense(96)
```

**ARIMA Baseline:**
```
ARIMA(p,d,q) — grid search ile en iyi parametreler
statsmodels kütüphanesi
```

**Naive Baseline:**
```
"Yarın bugünle aynı olacak" — son 24 saati olduğu gibi kopyala
```

## 5. Eğitim Detayları

```python
# Hiperparametreler
WINDOW_SIZE = 96          # 24 saat × 4
FORECAST_HORIZON = 96     # 24 saat × 4
BATCH_SIZE = 32
EPOCHS = 200
LEARNING_RATE = 0.001
OPTIMIZER = 'adam'

# Callbacks
EarlyStopping(monitor='val_loss', patience=15, restore_best_weights=True)
ReduceLROnPlateau(monitor='val_loss', factor=0.5, patience=5, min_lr=1e-6)
ModelCheckpoint('best_lstm_model.h5', save_best_only=True)

# Veri Bölme
TRAIN_RATIO = 0.7
VAL_RATIO = 0.15
TEST_RATIO = 0.15
# Zaman serisi olduğu için RASTGELE BÖLME YAPMA
# Kronolojik sırayla böl: ilk %70 train, sonraki %15 val, son %15 test
```

### Framework
- **TensorFlow 2.x / Keras**
- **Eğitim ortamı:** Google Colab

## 6. Metrikler

| Metrik | Formül | Anlamı |
|---|---|---|
| RMSE | √(Σ(ŷ-y)²/n) | Ortalama tahmin hatası (aynı birimde) |
| MAE | Σ|ŷ-y|/n | Mutlak hata ortalaması |
| R² | 1 - SS_res/SS_tot | Açıklanan varyans oranı (1'e yakın = iyi) |
| MAPE | Σ|ŷ-y|/|y| × 100/n | Yüzde cinsinden hata |

## 7. Karşılaştırma Tablosu Formatı

```markdown
| Model       | RMSE   | MAE    | R²     | MAPE   |
|-------------|--------|--------|--------|--------|
| LSTM        | ?      | ?      | ?      | ?      |
| GRU         | ?      | ?      | ?      | ?      |
| ARIMA       | ?      | ?      | ?      | ?      |
| Naive       | ?      | ?      | ?      | ?      |
```

## 8. Görselleştirmeler

1. **Tahmin vs Gerçek grafiği:** X=zaman, Y=doluluk; iki çizgi (gerçek ve tahmin)
2. **Hata dağılımı:** Histogram (tahmin hatası dağılımı)
3. **Haftalık döngü:** Modelin hafta içi/sonu farkını yakalayıp yakalamadığı
4. **Karşılaştırma bar chart:** 4 modelin RMSE değerleri yan yana

## 9. Dosya Çıktıları

```
ml/lstm/
├── prepare_data.py          # Veri indirme, temizleme, özellik mühendisliği
├── train.py                 # LSTM eğitim scripti
├── train_baselines.py       # GRU, ARIMA, Naive eğitimi
├── evaluate.py              # Metrik hesaplama ve karşılaştırma
├── visualize.py             # Grafik oluşturma
├── notebooks/
│   └── lstm_experiment.ipynb  # Colab notebook
└── experiment/
    ├── best_lstm_model.h5
    ├── comparison_table.csv
    ├── prediction_vs_actual.png
    └── error_distribution.png
```
