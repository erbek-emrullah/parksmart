# IMPLEMENTATION_STATUS.md — Uygulama Durumu Takibi

> Son güncelleme: 2026-10-13

## Durum Simgeleri

| Simge | Durum |
|---|---|
| ⬜ | Henüz başlanmadı |
| 🟡 | Devam ediyor |
| ✅ | Tamamlandı |
| ⏸️ | Beklemede |

## Faz 0 — Ortam Kurulumu

| # | Görev | Durum | Not |
|---|---|---|---|
| 0.1 | Python venv oluştur | ✅ | Python 3.12.10, pip güncellendi |
| 0.2 | CARLA 0.9.15 indir ve kur | 🟡 | ~13GB manuel indirme rehberi paylaşıldı |
| 0.3 | CUDA/cuDNN kontrol | ✅ | PyTorch CUDA 12.1 + RTX 4050 Laptop GPU |
| 0.4 | PostgreSQL kur | ⏸️ | Faz 3 öncesi kurulacak |
| 0.5 | Android Studio kur | ⏸️ | Faz 4 öncesi kurulacak |
| 0.6 | Proje dizin yapısı | ✅ | Tamamlandı |
| 0.7 | Git repo başlat | ✅ | Private repo (parksmart) + develop branch |

## Faz 1 — Veri Hattı

| # | Görev | Durum | Not |
|---|---|---|---|
| 1.1 | PKLot veri seti indir | ⬜ | |
| 1.2 | PKLot → YOLOv8 formatına dönüştür | ⬜ | |
| 1.3 | Veri bölme (%70/%15/%15, seed=42) | ⬜ | |
| 1.4 | CARLA otopark sahnesi kur | ⬜ | |
| 1.5 | Sentetik veri üretim scripti | ⬜ | Hedef: ~3000-5000 görüntü |
| 1.6 | Otomatik etiketleme (CARLA API) | ⬜ | |
| 1.7 | Hibrit veri seti oluştur | ⬜ | |
| 1.8 | LSTM zaman serisi verisi hazırla | ⬜ | Birmingham dataset + CARLA |

## Faz 2 — Model Eğitimi ve Sim-to-Real Deneyi ⭐

| # | Görev | Durum | Not |
|---|---|---|---|
| 2.1 | Model A: Sadece PKLot ile eğit | ⬜ | Colab |
| 2.2 | Model B: Sadece CARLA ile eğit | ⬜ | Colab |
| 2.3 | Model C: Hibrit (PKLot+CARLA) ile eğit | ⬜ | Colab |
| 2.4 | Model D: Hibrit + domain adaptation | ⬜ | Colab |
| 2.5 | 4 modeli aynı test setinde karşılaştır | ⬜ | mAP, P, R, F1 |
| 2.6 | Sentetik veri miktarı vs mAP grafiği | ⬜ | |
| 2.7 | LSTM model eğitimi | ⬜ | Keras/TF, Colab |
| 2.8 | LSTM vs ARIMA vs GRU karşılaştırma | ⬜ | |

## Faz 3 — Backend + Gerçek Zamanlı Sistem

| # | Görev | Durum | Not |
|---|---|---|---|
| 3.1 | FastAPI proje iskeleti | ⬜ | |
| 3.2 | PostgreSQL şema + ORM modelleri | ⬜ | |
| 3.3 | JWT auth endpoint'leri | ⬜ | |
| 3.4 | Doluluk API endpoint'leri | ⬜ | |
| 3.5 | WebSocket yayın mekanizması | ⬜ | |
| 3.6 | CARLA → Backend gerçek zamanlı akış | ⬜ | |
| 3.7 | LSTM tahmin servisi entegrasyonu | ⬜ | |

## Faz 4 — Mobil + Dashboard + Demo

| # | Görev | Durum | Not |
|---|---|---|---|
| 4.1 | Android proje iskeleti (MVVM) | ⬜ | |
| 4.2 | Giriş/Kayıt ekranı | ⬜ | |
| 4.3 | Canlı otopark haritası ekranı | ⬜ | WebSocket |
| 4.4 | Tahmin grafiği ekranı | ⬜ | MPAndroidChart |
| 4.5 | Web dashboard (Streamlit) | ⬜ | Jüri demo'su için |
| 4.6 | CARLA canlı kontrol (dashboard'dan) | ⬜ | |
| 4.7 | Demo senaryosu prova | ⬜ | |

## Faz 5 — Test, Metrik, Rapor

| # | Görev | Durum | Not |
|---|---|---|---|
| 5.1 | Uçtan uca gecikme ölçümü | ⬜ | |
| 5.2 | SUS anketi (5-10 kişi) | ⬜ | |
| 5.3 | Rapor — deney sonuçları bölümü | ⬜ | |
| 5.4 | Rapor — UML/ER diyagramları | ⬜ | |
| 5.5 | Rapor — boş bölümleri doldur | ⬜ | |

## Kararlar Logu

| Tarih | Karar | Bağlam |
|---|---|---|
| 2026-10-09 | Proje dokümantasyonu oluşturuldu | İlk analiz |
| 2026-10-13 | Odak: Sim-to-Real transfer deneyi | Projeyi güçlendirme |
| 2026-10-13 | LSTM framework: Keras/TensorFlow | Colab uyumluluğu |
| 2026-10-13 | CARLA versiyonu: 0.9.15 | En güncel stabil |
| 2026-10-13 | Uç cihaz dağıtımı yok, sadece simülasyon | Bütçe kısıtı |
| 2026-10-13 | Web dashboard: Streamlit | Hızlı geliştirme |
