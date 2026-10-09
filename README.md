# ParkSmart — Akıllı Otopark Yönetim Sistemi

Atatürk Üniversitesi Bilgisayar Mühendisliği Bölümü bitirme projesi kapsamında geliştirilen; CARLA simülatörü üzerinde otopark dijital ikizi oluşturan, YOLOv8 ile araç tespiti ve LSTM ile doluluk tahmini yapan gerçek zamanlı karar destek sistemi.

**Danışman:** Doç. Dr. Ferhat Bozkurt

---

## Proje Hakkında
Bu çalışmanın temel amacı, CARLA simülasyon ortamından elde edilen sentetik otopark görüntüleri ile eğitilen YOLOv8 modelinin, gerçek dünya koşullarındaki (PKLot veri seti) tespit başarımına etkisini (Sim-to-Real transfer) incelemektir. Tespit edilen doluluk verileri FastAPI backend üzerinden işlenerek LSTM modeli ile gelecekteki doluluk oranları tahmin edilir ve mobil/web arayüzlerine aktarılır.

## Mimari ve Teknolojiler
- **Simülasyon:** CARLA 0.9.15 (Dijital ikiz ve sentetik veri üretimi)
- **Görüntü İşleme / Tespit:** YOLOv8n, Shapely (Poligon/Park yeri eşleme)
- **Zaman Serisi / Tahmin:** LSTM (Keras / TensorFlow)
- **Backend:** FastAPI, PostgreSQL, Redis
- **İstemciler:** Android (Kotlin, Jetpack Compose), Web Dashboard (Streamlit)

## Dizin Yapısı
- `carla/`: Simülasyon senaryoları, sensör konfigürasyonları ve veri üretim scriptleri
- `ml/`: YOLOv8 ve LSTM eğitim/deney kodları
- `backend/`: FastAPI servisleri ve veritabanı modelleri
- `mobile/`: Android mobil uygulama kaynak kodları
- `dashboard/`: Yönetim ve izleme web arayüzü
- `docs/`: Proje mimarisi, gereksinimler ve detaylı spesifikasyonlar

## Kurulum ve Başlangıç

Geliştirme ortamını hazırlamak için:

```bash
# Sanal ortam oluşturma ve etkinleştirme
python -m venv .venv
.venv\Scripts\activate   # Windows

# Bağımlılıkların yüklenmesi
pip install -r requirements.txt
```

*Not: CARLA simülatörü ve CUDA kurulumu ile ilgili detaylar için `docs/` dizini altındaki dokümantasyonu inceleyebilirsiniz.*
