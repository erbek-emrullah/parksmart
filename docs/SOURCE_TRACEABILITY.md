# SOURCE_TRACEABILITY.md — Kaynak Belge İzlenebilirlik Matrisi

> Kaynak Belge: `Elective_Design_2_220709030_210709037.docx`
> 
> Not: Sayfa numaraları belgede İÇİNDEKİLER'de verilen sayfa eşlemesine dayanır. DOCX formatında sayfa numaraları render'a göre değişebilir; içindekiler tablosundaki bölüm başlıkları referans alınmıştır.

## Bölüm — Sayfa Eşlemesi (İçindekiler Tablosundan)

| Bölüm | Sayfa |
|---|---|
| Değişiklik Tarihçesi | 3 |
| Kısaltmalar | 4 |
| Şekiller Dizini | 6 |
| Tablolar Dizini | 7 |
| Özet | 8 |
| Giriş | 8 |
| Literatür Araştırması | 11 |
| Proje | 14 |
| Materyal ve Yöntem | 21 |
| Bulgular ve Tartışma | 22 |
| Sonuçlar ve Öneriler | 23 |
| Sürdürülebilir Kalkınma İlişkisi | 24 |
| Etik Değerlendirme | 25 |
| Hukuki Değerlendirme | 26 |
| Sağlık, Çevre ve Güvenlik Etkileri | 27 |
| Kaynakça | 29 |

## Gereksinim → Kaynak Eşlemesi

| Gereksinim ID | Gereksinim | Kaynak Bölüm | Sayfa |
|---|---|---|---|
| FR-1 | Anlık doluluk tespiti (YOLOv8) | Proje > YOLOv8 Model Eğitimi | ~15–16 |
| FR-2 | LSTM 24 saat tahmin | Proje > LSTM Doluluk Tahmin Modeli | ~16 |
| FR-3 | MCDM park yeri atama | Proje > Kuramsal Temeller > MCDM | ~13–14 |
| FR-4 | Rezervasyon sistemi | Proje > Backend Geliştirme | ~16–17 |
| FR-5 | Bekleme listesi ve FCM bildirim | Proje > Backend Geliştirme | ~17 |
| FR-6 | JWT kimlik doğrulama | Proje > Kuramsal Temeller > REST API ve JWT | ~13 |
| FR-7 | WebSocket gerçek zamanlı iletişim | Proje > Genel Sistem Mimarisi | ~14 |
| FR-8 | Mobil uygulama ekranları | Proje > Mobil Uygulama Geliştirme | ~17–18 |
| FR-9 | CARLA dijital ikiz | Giriş > Dijital İkiz ve CARLA | ~9; Proje > Genel Sistem Mimarisi | ~14 |
| FR-10 | Veri seti hazırlama | Proje > Veri Seti Hazırlama | ~14–15 |
| NFR-1 | KVKK uyumu | Proje > Genel Sistem Mimarisi (güvenlik paragrafı) | ~14 |
| NFR-2 | Gerçek zamanlı performans | Proje > YOLOv8 Model Eğitimi | ~15–16 |
| NFR-4 | Düşük donanım uyumluluğu | Literatür > Uçta Hesaplama | ~11 |

## Tasarım Kararı → Kaynak Eşlemesi

| Karar | Gerekçe | Kaynak | Sayfa |
|---|---|---|---|
| YOLOv8**n** varyantı seçimi | Düşük donanımda çalışabilirlik, kısa çıkarım süresi | Proje > YOLOv8 Model Eğitimi | ~15 |
| COCO pretrained + fine-tune | Temel görsel özellikler önceden öğrenilmiş | Proje > YOLOv8 Model Eğitimi | ~15 |
| IoU eşiği = 0.25 | Bölme dolu/boş kararı için | Proje > YOLOv8 Model Eğitimi | ~15 |
| LSTM (RNN yerine) | Uzun vadeli bağımlılıkları yakalama üstünlüğü | Kuramsal Temeller > LSTM; Literatür [5][6][7] | ~12; 11 |
| 15 dk periyotla veri kaydı | LSTM girdi serisi oluşturma | Proje > LSTM Doluluk Tahmin | ~16 |
| Sin-cos zaman kodlaması | Gece yarısı geçiş doğruluğu | Proje > LSTM Doluluk Tahmin | ~16 |
| PKLot + CARLA hibrit eğitim | Domain gap azaltma | Giriş; Proje > Veri Seti Hazırlama | ~9; ~14 |
| Domain randomization | Sim-to-real transfer | Kuramsal Temeller > Dijital İkiz | ~13 |
| FastAPI | Asenkron yapı, eşzamanlı bağlantı verimliliği | Proje > Backend Geliştirme | ~16 |
| PostgreSQL + Redis | İlişkisel veri + önbellek | Proje > Backend Geliştirme | ~16–17 |
| JWT (15dk/7gün) | Stateless auth, oturum kaydı gereksiz | Kuramsal Temeller > REST API ve JWT | ~13 |
| MVVM (Android) | Katmanlı sorumluluk ayrımı | Proje > Mobil Uygulama | ~17 |
| Görüntü RAM'de işlenip silinir | KVKK uyumu | Proje > Genel Sistem Mimarisi | ~14 |
| Seed = 42 | Veri bölme tekrarlanabilirliği | Proje > Veri Seti Hazırlama | ~14 |

## Belgede Eksik/Doldurulmamış Bölümler

| Bölüm | Durum | Sayfa |
|---|---|---|
| Değişiklik Tarihçesi | Boş (tablo başlıkları var, içerik yok) | 3 |
| İş Takvimi | Boş (yalnızca başlık) | ~18 |
| Risk Yönetimi | Tablo başlıkları var ama riskler doldurulmamış | ~18 |
| Projenin Gerçeklenmesi | Boş (henüz uygulama yapılmadığı için bekleniyor) | ~19 |
| Projeden Elde Edilen Sonuçlar | Boş | ~19 |
| Proje Bütçesi | Boş | ~19 |
| Bulgular ve Tartışma | Boş | 22 |
| Sonuçlar ve Öneriler | Boş | 23 |
| Şekiller / Tablolar Dizini | Placeholder — gerçek şekil/tablo referansı yok | 6–7 |
| Etik benzerlik oranları | Doldurulmamış | 25 |
| Hukuki değerlendirme | Doldurulmamış | 26 |
| Sağlık/Çevre/Güvenlik | Doldurulmamış | 27 |

## Kaynakça (14 Referans)

Belgede 14 akademik/teknik kaynak referans verilmiştir. Tam liste belgede s.29'dan itibaren yer almaktadır.
Önemli referanslar:
- [2] PKLot veri seti — Almeida et al. 2015
- [4] YOLOv8 — Jocher et al. 2023
- [5] LSTM karşılaştırma — Greff et al. 2017
- [8] CARLA simülatörü — Dosovitskiy et al. 2017
- [13] Orijinal LSTM — Hochreiter & Schmidhuber 1997
- [14] Cruising for parking — Shoup 2006
