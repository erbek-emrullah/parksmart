# PROJECT_CONTEXT.md — Proje Bağlamı ve Temel Kararlar

## 1. Projenin Amacı

Herhangi bir fiziksel sensör yatırımı gerektirmeksizin, mevcut güvenlik kamerası altyapısı üzerinden çalışan, **bütünleşik bir akıllı otopark yönetim ve karar destek sistemi** geliştirmek.

Sistem şu dört temel yeteneği tek çatı altında birleştirir:

1. **Anlık doluluk tespiti** — Kamera görüntüsünden YOLOv8 ile araç tespiti
2. **Gelecek doluluk tahmini** — LSTM ile sonraki 24 saatin doluluk öngörüsü
3. **Akıllı yer atama** — Çok kriterli puanlama (MCDM) ile en uygun boş bölmeye yönlendirme
4. **Mobil erişim** — Tüm işlevleri Android uygulaması üzerinden kullanıcıya sunma

> **Kaynak:** Belge s.8 (Özet), s.8–9 (Giriş — Projenin Amaçları)

## 2. Çözdüğü Problem

| Problem | Açıklama |
|---|---|
| **Donanım bağımlılığı** | Sensör tabanlı sistemler her bölme için ayrı donanım, kablolama ve bakım gerektiriyor |
| **Gerçek zamanlı veri eksikliği** | Mevcut sistemler yalnızca anlık doluluk gösteriyor; "Saat 10:30'da yer bulabilir miyim?" sorusuna yanıt veremiyor |
| **Veri toplama güçlüğü** | Gerçek kamera görüntüleri plaka/yüz içerdiğinden KVKK sorunları doğuruyor |
| **Bütünleşik sistem eksikliği** | Literatürde tespit, tahmin, akıllı atama ve kullanıcı arayüzünü tek çatıda birleştiren çalışma çok az |

> **Kaynak:** Belge s.8–9 (Problem Tanımı)

## 3. Dijital İkiz Yaklaşımı — CARLA Simülatörü

- CARLA açık kaynaklı (MIT lisanslı) otonom sürüş simülatörüdür.
- Python API'si ile kamera konumu, araç sayısı, hava koşulları, aydınlatma, trafik yoğunluğu kontrol edilebilir.
- İki işlev üstlenir:
  1. **Eğitim verisi kaynağı:** Sentetik etiketli görüntüler üretir, PKLot ile hibrit eğitim havuzu oluşturulur.
  2. **Çalışma zamanı dijital ikiz:** Gerçek otopark kamerasının sanal karşılığı olarak görüntü sağlar.
- Domain gap azaltma stratejileri: domain randomization + hibrit eğitim (CARLA + PKLot).

> **Kaynak:** Belge s.9 (Dijital İkiz ve CARLA), s.13 (Sim-to-Real Transferi)

## 4. Gizlilik ve KVKK Uyumu

Sistem **yapısal olarak KVKK uyumlu** tasarlanmıştır:
- Kameradan alınan görüntüler diske kaydedilmez, yalnızca RAM üzerinde işlenir.
- Plaka veya yüz tespiti yapılmaz.
- Sunucuya yalnızca park bölmelerinin doluluk durumu (0/1 mantıksal veri) aktarılır.

> **Kaynak:** Belge s.14 (Sistem Mimarisi — güvenlik/gizlilik paragrafı)

## 5. Veri Seti Stratejisi

| Kaynak | İçerik | Amaç |
|---|---|---|
| **PKLot** | 12.416 etiketli park yeri görüntüsü (UFPR & PUC-PR, Brezilya) — güneşli, bulutlu, yağmurlu | Birincil eğitim verisi |
| **CARLA sentetik** | Python API ile üretilecek etiketli görüntüler — gece, şiddetli yağmur, farklı araç çeşitliliği | PKLot'un kapsayamadığı senaryolar |

- Veri bölme: %70 eğitim / %15 doğrulama / %15 test
- Random seed: 42
- Augmentation: Mozaik, yatay çevirme (H-Flip), HSV değişimi, CLAHE, Gaussian gürültüsü
- YOLOv8 kendi augmentation motorunu eğitim sırasında otomatik uygular.

> **Kaynak:** Belge s.14–15 (Veri Seti Hazırlama)

## 6. Temel Algoritmik Kararlar

### YOLOv8 Nesne Tespiti
- **Varyant:** YOLOv8n (Nano) — düşük donanım, gerçek zamanlı çıkarım
- **Transfer öğrenimi:** COCO önceden eğitilmiş ağırlıklardan ince ayar (fine-tuning)
- **Park bölmesi eşleme:** Her bölme 4 noktalı poligon → Shapely ile IoU hesabı → IoU ≥ 0.25 → dolu

### LSTM Zaman Serisi Tahmini
- **Girdi:** Son 24 saatlik doluluk oranı + sinüs-kosinüs kodlanmış saat/gün + ikili özellikler (hafta sonu, tatil, yoğun saat)
- **Çıktı:** Gelecek 24 saatlik doluluk tahmini
- **Veri kaynağı:** 15 dakikalık periyotlarla PostgreSQL'e kaydedilen doluluk kayıtları
- **Eğitim:** Google Colab, erken durdurma, öğrenme oranı azaltma, en iyi model kaydetme
- **Üretim:** FastAPI sunucusunda yüklü model, Redis önbelleğinden veri alarak tahmin üretir

### MCDM Park Yeri Atama
- **Formül:** `Score(s) = w1·(1/d_exit) + w2·(1/d_elevator) + w3·(1/|floor|) + w4·H(s)`
- Ağırlıklar araç tipine göre dinamik (engelli → asansör yakınlığı ↑, motosiklet → çıkış yakınlığı ↑)

> **Kaynak:** Belge s.12–13 (Kuramsal Temeller), s.15–16 (YOLOv8, LSTM, Backend)

## 7. Test ve Doğrulama Stratejisi

| Bileşen | Metrikler | Karşılaştırma |
|---|---|---|
| YOLOv8 | mAP, Precision, Recall, Confusion Matrix | PKLot ve CARLA test setleri ayrı ayrı |
| LSTM | RMSE, MAE, R² | ARIMA karşılaştırma modeli |
| Sunucu | Eş zamanlı rezervasyon çakışma testi, uçtan uca gecikme ölçümü | — |
| Mobil | SUS kullanılabilirlik anketi | — |

> **Kaynak:** Belge s.16–17 (Test ve Doğrulama)
