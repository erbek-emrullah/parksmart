# REQUIREMENTS.md — Gereksinimler, Kısıtlar ve Kabul Kriterleri

## Fonksiyonel Gereksinimler

### FR-1: Anlık Doluluk Tespiti
- Kamera görüntüsünden YOLOv8n ile araç tespiti yapılır.
- Tespit edilen bounding box'lar, önceden tanımlanmış park bölmesi poligonlarıyla IoU hesabı ile eşlenir.
- IoU ≥ 0.25 → bölme dolu; IoU < 0.25 → bölme boş.
- Sonuçlar JSON formatında sunucu katmanına iletilir.
- **Kabul Kriteri:** mAP, precision ve recall değerleri belgelenecek; PKLot ve CARLA test setleri üzerinde ayrı ayrı değerlendirilecek.

### FR-2: Doluluk Tahmini (24 Saat)
- LSTM modeli son 24 saatlik doluluk verisiyle gelecek 24 saatin doluluk dağılımını tahmin eder.
- Girdi özellikleri: doluluk oranı, sinüs-kosinüs kodlu saat/gün, hafta sonu/tatil/yoğun saat ikili değişkenleri.
- 15 dakikalık periyotlarla güncellenen zaman serisi verisi kullanılır.
- **Kabul Kriteri:** RMSE, MAE, R² metrikleri; ARIMA ile karşılaştırmalı değerlendirme.

### FR-3: Akıllı Park Yeri Atama (MCDM)
- Boş bölmeler arasında ağırlıklı doğrusal bileşim yöntemiyle puan hesaplanır.
- Kriterler: çıkışa yakınlık, asansöre yakınlık, kat bilgisi, geçmiş tercih skoru.
- Ağırlıklar araç tipine göre dinamik: engelli → asansör yakınlığı ↑, motosiklet → çıkış yakınlığı ↑.
- **Kabul Kriteri:** Farklı araç tipleri için beklenen sıralama davranışı doğrulanacak.

### FR-4: Rezervasyon Sistemi
- Kullanıcı mobil uygulama üzerinden park yeri rezervasyonu yapabilir.
- Eş zamanlı çakışmalar veritabanı düzeyinde kilitleme mekanizmasıyla engellenir.
- **Kabul Kriteri:** Eş zamanlı çakışma testi başarılı geçer.

### FR-5: Bekleme Listesi ve Bildirimler
- Bir park yeri boşaldığında bekleyen kullanıcılara FCM push bildirimi gönderilir.
- FCM Multicast API ile toplu bildirim desteklenir.
- **Kabul Kriteri:** Bildirim gecikme süresi ölçülecek.

### FR-6: Kullanıcı Kimlik Doğrulama
- JWT tabanlı kimlik doğrulama: 15 dakikalık access token + 7 günlük refresh token.
- Kullanıcı kaydı ve giriş API uç noktaları.
- **Kabul Kriteri:** Token süresi dolduğunda yenileme akışı çalışır.

### FR-7: Gerçek Zamanlı İletişim
- WebSocket üzerinden bağlı istemcilere anlık doluluk güncellemesi gönderilir.
- **Kabul Kriteri:** CARLA görüntüsünden mobil ekrana uçtan uca gecikme ölçülecek.

### FR-8: Mobil Uygulama Ekranları
- **Giriş/Kayıt ekranı**
- **Canlı otopark haritası** — WebSocket ile anlık güncellenen
- **Rezervasyon ekranı** — MCDM sonucunu gösteren
- **Tahmin grafiği** — 24 saatlik LSTM tahmin grafiği
- **Profil ekranı**
- **Kabul Kriteri:** SUS kullanılabilirlik anketi skoru ≥ 68 (endüstri ortalaması).

### FR-9: CARLA Dijital İkiz
- CARLA simülatöründe sanal otopark sahnesi kurulur.
- İki işlev: (a) eğitim için sentetik veri üretimi, (b) çalışma zamanında görüntü sağlama.
- Python API ile hava koşulları, araç sayısı, kamera açısı parametrelendirilebilir.
- **Kabul Kriteri:** En az 3 farklı hava koşulunda (güneşli, bulutlu, yağmurlu/gece) sentetik veri üretimi.

### FR-10: Veri Seti Hazırlama
- PKLot veri seti + CARLA sentetik görüntülerle hibrit eğitim havuzu.
- Veri bölme: %70 / %15 / %15, seed=42.
- **Kabul Kriteri:** Veri bölme scripti tekrarlanabilir.

## Fonksiyonel Olmayan Gereksinimler

### NFR-1: KVKK Uyumu
- Görüntüler diske kaydedilmez, yalnızca RAM'de işlenir.
- Plaka/yüz tespiti yapılmaz.
- Sunucuya yalnızca doluluk durumu (0/1) aktarılır.

### NFR-2: Gerçek Zamanlı Performans
- YOLOv8n ile kare başına düşük çıkarım süresi (hedef: gerçek zamanlı, spesifik ms hedefi belgede yok).
- Uçtan uca gecikme ölçülecek (CARLA → Backend → Mobil).

### NFR-3: Ölçeklenebilirlik
- Rate limiting mekanizması kötüye kullanımı engeller.
- Redis önbellek ile veritabanı yükü hafifletilir.

### NFR-4: Düşük Donanım Uyumluluğu
- YOLOv8n seçiminin gerekçesi: NVIDIA Jetson Nano/Xavier NX gibi uç cihazlarda çalışabilirlik.
- ⚠️ *Not: Proje kapsamında gerçek uç cihaz kullanımı kesinleşmiş değil; belgede "plan" olarak geçiyor.*

### NFR-5: Açık Kaynak ve Lisans
- CARLA: MIT lisanslı.
- YOLOv8: AGPL-3.0 (Ultralytics) — ticari kullanım niyeti varsa dikkate alınmalı.

## Kısıtlar

| Kısıt | Detay |
|---|---|
| Platform | Android (iOS desteği yok) |
| Backend dili | Python (FastAPI) |
| Veritabanı | PostgreSQL |
| Önbellek | Redis |
| Mobil dil | Kotlin + Jetpack Compose |
| Simülatör | CARLA |
| Model eğitim ortamı | Google Colab |
| Veri bölme seed | 42 |

## Belirsiz / Henüz Kesinleşmemiş Gereksinimler

> ⚠️ Aşağıdaki maddeler belgede kesin bir karara bağlanmamıştır.

1. **LSTM framework seçimi:** Keras/TensorFlow mi PyTorch mu kullanılacak? Belgede belirtilmemiş.
2. **CARLA versiyonu:** Hangi CARLA sürümü kullanılacak? (0.9.x serisi önerilir ama belgede yok.)
3. **Uç cihaz dağıtımı:** Jetson Nano/Xavier NX üzerinde gerçek dağıtım yapılacak mı yoksa yalnızca simülasyon mu kalacak?
4. **Otopark sahne tasarımı:** CARLA'da kaç bölmeli, kaç katlı bir otopark modeli oluşturulacak?
5. **LSTM açık erişimli veri seti:** Hangi açık erişimli otopark doluluk zaman serisi veri seti kullanılacak?
6. **Rate limiting parametreleri:** Dakikada kaç istek limiti uygulanacak?
7. **Rezervasyon süresi:** Bir rezervasyon ne kadar süre geçerli?
8. **WebSocket vs Socket.IO:** Belgede hem WebSocket hem Socket.IO geçiyor — hangisi kullanılacak?
9. **MCDM ağırlık değerleri (w1–w4):** Başlangıç değerleri nedir?
10. **SUS anketi uygulama detayları:** Kaç kişiyle yapılacak?
11. **Risk yönetimi tablosu:** Belgede tablo başlıkları var ama risk detayları doldurulmamış.
12. **İş takvimi:** Belgede başlık var ama detaylar doldurulmamış.
