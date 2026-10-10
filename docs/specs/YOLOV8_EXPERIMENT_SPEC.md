# YOLOv8 Sim-to-Real Deney Spesifikasyonu

## 1. Araştırma Sorusu

> "CARLA simülatöründen üretilen sentetik otopark verileri, YOLOv8 araç tespit
> modelinin gerçek dünya performansını istatistiksel olarak anlamlı biçimde
> artırır mı?"

## 2. Deney Tasarımı

### Test Seti (Sabit — Hiç Değişmez)
- **Kaynak:** PKLot test split (%15)
- **Amaç:** Tüm modeller aynı gerçek dünya görüntüleri üzerinde test edilir
- **Bu seti eğitimde KESİNLİKLE kullanma**

### Eğitilecek Modeller

| Model | Eğitim Verisi | Amaç |
|---|---|---|
| **Model A** | Sadece PKLot train (%70) | Baseline — gerçek veri ile ne kadar başarılı? |
| **Model B** | Sadece CARLA sentetik (~3000-5000) | Sentetik veri tek başına yeterli mi? |
| **Model C** | PKLot train + CARLA sentetik (hibrit) | Sentetik veri eklemenin etkisi |
| **Model D** | Hibrit + güçlü augmentation (domain adaptation) | Domain gap azaltma teknikleri |

### Model D — Domain Adaptation Detayları
Model C ile aynı veri, ama ek augmentation stratejileri:
- CLAHE (Contrast Limited Adaptive Histogram Equalization)
- Gaussian gürültüsü
- Renk jittering (brightness, contrast, saturation)
- Rastgele perspektif dönüşümü
- Bu augmentation'lar YOLOv8'in kendi augmentation'larına ek olarak uygulanır

## 3. Sabit Hiperparametreler (Tüm Modeller)

```yaml
model: yolov8n.pt           # Nano varyant
pretrained: true             # COCO pretrained weights
epochs: 100
patience: 20                 # Early stopping
batch: 16
imgsz: 640
optimizer: AdamW
lr0: 0.001
lrf: 0.01                   # Final learning rate factor
momentum: 0.937
weight_decay: 0.0005
warmup_epochs: 3
warmup_momentum: 0.8
box: 7.5
cls: 0.5
dfl: 1.5

# Augmentation (YOLOv8 varsayılan — Model A, B, C için)
mosaic: 1.0
mixup: 0.0
copy_paste: 0.0
hsv_h: 0.015
hsv_s: 0.7
hsv_v: 0.4
degrees: 0.0
translate: 0.1
scale: 0.5
shear: 0.0
perspective: 0.0
flipud: 0.0
fliplr: 0.5

# Sınıflar
names:
  0: vehicle
```

## 4. Ek Deney: Sentetik Veri Miktarı vs Performans

```
PKLot train SABİT + CARLA miktarı DEĞİŞKEN:

Deney 4a: PKLot + 500 CARLA   → mAP: ?
Deney 4b: PKLot + 1000 CARLA  → mAP: ?
Deney 4c: PKLot + 2000 CARLA  → mAP: ?
Deney 4d: PKLot + 3000 CARLA  → mAP: ?
Deney 4e: PKLot + 5000 CARLA  → mAP: ?

Bu deneyin çıktısı: X ekseni = sentetik veri miktarı, Y ekseni = mAP grafiği
"Diminishing returns" noktasını gösterir.
```

## 5. Ölçülecek Metrikler

Her model için:

| Metrik | Açıklama |
|---|---|
| mAP@0.5 | IoU 0.5 eşiğinde ortalama hassasiyet |
| mAP@0.5:0.95 | IoU 0.5-0.95 aralığında ortalama hassasiyet |
| Precision | Doğruluk |
| Recall | Duyarlılık |
| F1-Score | Precision ve Recall harmonik ortalaması |
| Confusion Matrix | Doğru/yanlış tespitler matrisi |
| Inference Time | ms/kare (RTX 4050 üzerinde) |
| Model Size | MB cinsinden |

## 6. Sonuç Tablosu Formatı

```markdown
| Model | Eğitim Verisi | mAP@0.5 | mAP@0.5:0.95 | Precision | Recall | F1 | Inf. Time |
|-------|--------------|---------|---------------|-----------|--------|-----|-----------|
| A     | PKLot        | ?       | ?             | ?         | ?      | ?   | ? ms      |
| B     | CARLA        | ?       | ?             | ?         | ?      | ?   | ? ms      |
| C     | Hibrit       | ?       | ?             | ?         | ?      | ?   | ? ms      |
| D     | Hibrit+DA    | ?       | ?             | ?         | ?      | ?   | ? ms      |
```

## 7. Beklenen Sonuçlar (Hipotez)

- **Model C > Model A:** Sentetik veri eklemenin olumlu etkisi
- **Model A > Model B:** Sadece sentetik veri, gerçek verinin yerini tutamaz
- **Model D ≥ Model C:** Domain adaptation teknikleri fark yaratabilir
- **Ek deney:** Belirli bir noktadan sonra daha fazla sentetik veri marjinal fayda getirir

## 8. Eğitim Ortamı

- **Platform:** Google Colab (ücretsiz T4 GPU yeterli)
- **Kütüphane:** `ultralytics` (pip install ultralytics)
- **Veri yükleme:** Google Drive üzerinden

## 9. Dosya Çıktıları

```
ml/yolov8/
├── train.py                 # Eğitim scripti (4 model için parametrize)
├── evaluate.py              # Değerlendirme scripti
├── compare_models.py        # Karşılaştırma tablosu ve grafik oluşturma
├── data_split.py            # PKLot veri bölme scripti (seed=42)
├── notebooks/
│   └── sim_to_real_experiment.ipynb  # Colab notebook (tek notebook)
└── experiment/
    ├── model_a/             # PKLot only
    ├── model_b/             # CARLA only
    ├── model_c/             # Hybrid
    ├── model_d/             # Hybrid + DA
    └── results/
        ├── comparison_table.csv
        ├── mAP_comparison.png
        └── synthetic_amount_vs_performance.png
```
