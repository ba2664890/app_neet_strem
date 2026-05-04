# IANOVA NEET — Plateforme Prédictive IA

> Plateforme d'intelligence géospatiale dédiée à l'analyse et au suivi des populations NEET au Sénégal.

![IANOVA](https://img.shields.io/badge/IANOVA-NEET%20Intelligence-1B3A2D?style=for-the-badge)
![Django](https://img.shields.io/badge/Django-4.2-092E20?style=for-the-badge&logo=django)
![React](https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react)
![PostGIS](https://img.shields.io/badge/PostGIS-15-336791?style=for-the-badge&logo=postgresql)

---

## 🏗️ Architecture

```
ianova/
├── backend/                    # Django REST API
│   ├── ianova_project/
│   │   ├── settings/base.py    # Configuration Django
│   │   ├── urls.py             # Routes principales
│   │   ├── celery.py           # Tâches asynchrones
│   │   ├── wsgi.py / asgi.py
│   ├── apps/
│   │   ├── accounts/           # Auth + Utilisateurs (RBAC)
│   │   ├── neet/               # Données NEET, zones, secteurs
│   │   ├── geospatial/         # Cartographie intelligente
│   │   ├── reports/            # Génération rapports (RAG+LLM)
│   │   └── chatbot/            # Assistant NLP Wolof/Français
│   ├── requirements.txt
│   └── Dockerfile
│
├── frontend/                   # React + TypeScript
│   ├── src/
│   │   ├── pages/
│   │   │   ├── LandingPage.tsx         # Page d'accueil (Image 2)
│   │   │   ├── AnalyticsDashboard.tsx  # Dashboard (Image 3)
│   │   │   ├── MapExplorer.tsx         # Carte (Image 1)
│   │   │   ├── ReportsPage.tsx         # Rapports
│   │   │   └── LoginPage.tsx           # Connexion
│   │   ├── components/
│   │   │   └── layout/
│   │   │       ├── AppLayout.tsx
│   │   │       ├── Sidebar.tsx
│   │   │       └── Topbar.tsx
│   │   ├── services/api.ts     # Axios + tous les endpoints
│   │   └── store/authStore.ts  # Zustand auth
│   ├── package.json
│   ├── tailwind.config.js
│   ├── vite.config.ts
│   └── Dockerfile
│
├── nginx/nginx.conf            # Reverse proxy
└── docker-compose.yml          # Stack complète
```

---

## ⚡ Démarrage Rapide

### Prérequis
- Docker & Docker Compose
- Node.js 20+ (développement frontend)
- Python 3.11+ (développement backend)

### 1. Démarrage avec Docker (recommandé)

```bash
# Cloner et démarrer
git clone <repo>
cd ianova
docker-compose up -d

# L'application sera disponible sur :
# Frontend : http://localhost:3000
# Backend API : http://localhost:8000
# Admin Django : http://localhost:8000/admin
# API Docs : http://localhost:8000/api/docs/
# MinIO : http://localhost:9001
```

### 2. Développement local

**Backend Django :**
```bash
cd backend

# Créer l'environnement virtuel
python -m venv venv
source venv/bin/activate  # Linux/Mac
# ou : venv\Scripts\activate  # Windows

# Installer les dépendances
pip install -r requirements.txt

# Variables d'environnement
cp .env.example .env
# Éditer .env avec vos paramètres DB/Redis

# Migrations
python manage.py migrate
python manage.py createsuperuser

# Lancer le serveur
python manage.py runserver
```

**Frontend React :**
```bash
cd frontend
npm install
npm run dev
# → http://localhost:3000
```

**Celery Worker :**
```bash
cd backend
celery -A ianova_project worker -l info
```

**Celery Beat (tâches planifiées) :**
```bash
cd backend
celery -A ianova_project beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler
```

---

## 🔑 Variables d'Environnement

Créer `/backend/.env` :

```env
# Django
DJANGO_SECRET_KEY=your-super-secret-key-change-in-production
DJANGO_SETTINGS_MODULE=ianova_project.settings.base
DEBUG=True

# PostgreSQL + PostGIS
DB_NAME=ianova_db
DB_USER=ianova_user
DB_PASSWORD=ianova_password
DB_HOST=localhost
DB_PORT=5432

# Redis (Celery + Cache)
REDIS_URL=redis://localhost:6379/0

# MinIO (Stockage fichiers)
MINIO_ENDPOINT=http://localhost:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin123
MINIO_BUCKET=ianova-media

# LLM (Chatbot + Rapports)
OPENAI_API_KEY=sk-...    # Ou modèle fine-tuné IANOVA
```

Créer `/frontend/.env` :
```env
VITE_API_URL=http://localhost:8000/api
```

---

## 📡 API REST — Endpoints Principaux

### Authentification
```
POST /api/accounts/login/              → JWT login
POST /api/accounts/register/           → Inscription 3 étapes
POST /api/auth/token/refresh/          → Refresh JWT
GET  /api/accounts/profile/            → Profil utilisateur
PATCH /api/accounts/profile/           → Mise à jour profil
GET  /api/accounts/dashboard-stats/    → Stats dashboard
```

### NEET
```
GET  /api/neet/snapshots/national_summary/   → KPIs nationaux (live)
GET  /api/neet/snapshots/hotspots/           → Zones à risque élevé
GET  /api/neet/snapshots/                    → Liste snapshots (filtrable)
GET  /api/neet/regions/                      → Régions Sénégal
GET  /api/neet/zones/                        → Zones géographiques
GET  /api/neet/opportunities/                → Opportunités sectorielles
GET  /api/neet/opportunities/by_sector_summary/ → Résumé par secteur
GET  /api/neet/analytics-dashboard/          → Données dashboard complet
POST /api/neet/profiles/                     → Créer profil NEET
GET  /api/neet/profiles/{id}/matching/       → Matching IA Jeune↔Opportunité
```

### Géospatial
```
GET  /api/geo/map-data/              → Données carte (layers, NEET, zones industrielles)
GET  /api/geo/layers/                → Couches disponibles
GET  /api/geo/industrial-zones/      → Zones industrielles ANAT
GET  /api/geo/pipeline-projects/     → Projets pipeline
GET  /api/geo/export/                → Export GeoJSON compatible SIG
```

### Rapports
```
GET  /api/reports/                   → Liste rapports
POST /api/reports/                   → Créer rapport (génération async Celery)
GET  /api/reports/{id}/              → Détail rapport
POST /api/reports/{id}/regenerate/   → Régénérer
GET  /api/reports/templates/         → Templates disponibles
GET  /api/reports/scheduled/         → Rapports programmés
```

### Chatbot
```
POST /api/chatbot/chat/              → Envoyer message (Wolof/Français)
GET  /api/chatbot/history/           → Historique conversations
```

---

## 👥 Rôles & Plans

### Rôles
| Rôle | Description | Accès |
|------|-------------|-------|
| `individual` | Jeune NEET / Grand public | Carte, profil, matching, chatbot |
| `ianova_team` | Équipe interne IANOVA | Accès complet + Studio ML, DataOps |
| `partner` | Partenaires / Investisseurs | Dashboard investissement, rapports, API |

### Plans
| Plan | Fonctionnalités |
|------|----------------|
| 🌟 Freemium | Carte basique, profil individu, chatbot limité |
| 💡 Standard | Matching IA complet, rapports auto, carte avancée |
| 🚀 Pro | API, exports, analytics avancés, portail partenaire |
| 🏢 Entreprise | Accès complet, SLA, support dédié, API illimitée |

---

## 🧠 Architecture ML (Interface séparée)

La partie Machine Learning est volontairement séparée selon la note conceptuelle.

Les interfaces sont prêtes dans :
- `apps/neet/tasks.py` — `update_vulnerability_scores()`, `generate_neet_forecasts()`
- `apps/neet/views.py` — endpoints qui appellent les modèles
- `apps/reports/tasks.py` — `_generate_llm_narrative()` pour RAG+LLM

**Pour intégrer vos modèles ML :**

```python
# Exemple d'intégration dans apps/neet/tasks.py
from your_ml_module import XGBoostNEETModel, ProphetForecaster

model = XGBoostNEETModel.load('/app/ml_models/neet_v4.pkl')
score = model.predict(snapshot_features)
```

**Stack ML attendue :** `scikit-learn`, `XGBoost`, `LightGBM`, `Prophet`, `LSTM`, `SHAP`

---

## 🗺️ Pages Frontend

| Page | Route | Description |
|------|-------|-------------|
| Landing | `/` | Page d'accueil + inscription 3 étapes |
| Login | `/login` | Connexion JWT |
| Dashboard | `/dashboard` | Analytics temps réel (Image 3) |
| Map Explorer | `/map` | Carte géospatiale interactive (Image 1) |
| Reports | `/reports` | Génération et téléchargement rapports |

---

## 🔐 Sécurité

- **JWT** : Access token (2h) + Refresh token (7j) avec rotation
- **RBAC** : Contrôle d'accès granulaire par rôle et par plan
- **Audit Logs** : Journalisation complète de toutes les opérations sensibles
- **RGPD** : Anonymisation des données individuelles, consentement granulaire
- **XAI** : Explicabilité SHAP de chaque prédiction ML
- **AES-256** : Chiffrement des données sensibles

---

## 🚀 Roadmap

| Phase | Durée | Livrable |
|-------|-------|----------|
| Phase 0 | Mois 1-2 | Stack technique, BDD PostgreSQL+PostGIS, ingestion ANSD/ANAT |
| Phase 1 | Mois 3-5 | Moteur ML segmentation NEET, pipeline ETL, dashboard interne |
| Phase 2 | Mois 6-8 | Cartographie géospatiale, chatbot NLP v1, portail partenaires |
| Phase 3 | Mois 9-11 | Grand public, PWA, scoring vulnérabilité, notifications IA |
| Phase 4 | Mois 12 | Beta privée, tests, monitoring Prometheus/Grafana, lancement |

---

## 📚 Documentation Swagger

Disponible sur `http://localhost:8000/api/docs/` après démarrage du backend.

---

**© 2025 IANOVA — Tous droits réservés — Document confidentiel**
