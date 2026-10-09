# MASTER_PLAN.md — Orkestrasyon Belgesi

> Bu belge başka bir AI agent'ın projeyi kodlaması için hazırlanmıştır.
> Her faz için `docs/prompts/` klasöründe detaylı prompt dosyaları,
> `docs/specs/` klasöründe teknik spesifikasyonlar bulunur.

## Proje Özeti (Agent Bağlam Notu)

**Ne:** CARLA simülatöründe dijital ikiz otopark ortamı kurarak YOLOv8 ile anlık
araç tespiti ve LSTM ile 24 saatlik doluluk tahmini yapan, FastAPI backend ve
Android mobil uygulama ile kullanıcıya sunan bütünleşik bir sistem.

**Araştırma Sorusu:** "CARLA'dan üretilen sentetik otopark verileri, YOLOv8'in
gerçek dünya performansını istatistiksel olarak anlamlı biçimde artırır mı?"

**Donanım:** Windows, RTX 4050 6GB VRAM, 32GB RAM, Python 3.12.10

## Fazlar ve Prompt Dosyaları

| Faz | Prompt Dosyası | Bağımlılık |
|---|---|---|
| 0 | `docs/prompts/FAZ0_ORTAM_KURULUMU.md` | Yok |
| 1 | `docs/prompts/FAZ1_VERI_HATTI.md` | Faz 0 tamamlanmış olmalı |
| 2 | `docs/prompts/FAZ2_MODEL_EGITIMI.md` | Faz 1 tamamlanmış olmalı |
| 3 | `docs/prompts/FAZ3_BACKEND.md` | Faz 0 tamamlanmış olmalı (Faz 2 ile paralel olabilir) |
| 4 | `docs/prompts/FAZ4_MOBIL_DASHBOARD.md` | Faz 3 tamamlanmış olmalı |
| 5 | `docs/prompts/FAZ5_TEST_RAPOR.md` | Tüm fazlar tamamlanmış olmalı |

## Teknik Spesifikasyonlar

| Spec Dosyası | İçerik |
|---|---|
| `docs/specs/CARLA_SCENE_SPEC.md` | CARLA sahne kurulumu ve sentetik veri üretim detayları |
| `docs/specs/YOLOV8_EXPERIMENT_SPEC.md` | Sim-to-Real deney tasarımı ve eğitim hiperparametreleri |
| `docs/specs/LSTM_SPEC.md` | LSTM model mimarisi ve eğitim detayları |
| `docs/specs/BACKEND_API_SPEC.md` | FastAPI endpoint'leri, DB şeması, auth akışı |
| `docs/specs/MOBILE_APP_SPEC.md` | Android uygulama ekranları, mimari, bileşenler |
| `docs/specs/DASHBOARD_SPEC.md` | Streamlit web dashboard tasarımı |

## Agent'a Genel Kurallar

1. Her prompt dosyasını **bağımsız** bir görev olarak ele al.
2. Dosya başındaki "Bağlam" bölümünü oku ama detayları spec dosyalarından al.
3. Kod yazarken mevcut dosyaları kontrol et, gereksiz yere silme veya yeniden yazma.
4. Her görevin sonunda "Kabul Kriterleri" bölümündeki tüm maddelerin karşılandığından emin ol.
5. Belirsiz bir şey varsa varsayım yapma, kullanıcıya sor.
6. Türkçe değişken/fonksiyon isimleri kullanma — kod İngilizce, yorumlar Türkçe olabilir.

## Dizin Yapısı (Hedef)

```
bitirme/
├── AGENTS.md
├── carla/
│   ├── scene_setup.py
│   ├── data_generator.py
│   ├── realtime_feed.py
│   ├── requirements.txt
│   └── configs/
│       └── parking_polygons.json
├── ml/
│   ├── yolov8/
│   │   ├── train.py
│   │   ├── evaluate.py
│   │   ├── compare_models.py
│   │   └── notebooks/          # Colab notebook'ları
│   ├── lstm/
│   │   ├── train.py
│   │   ├── evaluate.py
│   │   └── notebooks/
│   └── data/                   # .gitignore'da
│       ├── pklot/
│       ├── carla_synthetic/
│       └── hybrid/
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── api/
│   │   │   ├── auth.py
│   │   │   ├── parking.py
│   │   │   ├── predictions.py
│   │   │   └── websocket.py
│   │   ├── models/
│   │   │   ├── user.py
│   │   │   ├── parking.py
│   │   │   └── occupancy.py
│   │   ├── schemas/
│   │   ├── services/
│   │   │   ├── detection.py
│   │   │   ├── prediction.py
│   │   │   └── scoring.py
│   │   └── core/
│   │       ├── config.py
│   │       ├── database.py
│   │       └── security.py
│   ├── alembic/
│   ├── requirements.txt
│   └── Dockerfile
├── mobile/                     # Android Kotlin projesi
├── dashboard/
│   ├── app.py                  # Streamlit
│   └── requirements.txt
├── scripts/
│   ├── setup_env.ps1
│   └── download_pklot.py
├── configs/
├── docs/
│   ├── prompts/
│   ├── specs/
│   ├── PROJECT_CONTEXT.md
│   ├── REQUIREMENTS.md
│   ├── ARCHITECTURE.md
│   ├── IMPLEMENTATION_STATUS.md
│   ├── SOURCE_TRACEABILITY.md
│   └── MASTER_PLAN.md
├── .gitignore
└── README.md
```
