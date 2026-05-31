# MedVision AI SaaS

![Python](https://img.shields.io/badge/Python-3.11-blue) ![FastAPI](https://img.shields.io/badge/FastAPI-0.111-green) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue) ![React](https://img.shields.io/badge/React-18-61DAFB) ![Docker](https://img.shields.io/badge/Docker-Compose-blue) ![License](https://img.shields.io/badge/License-MIT-yellow)

> Medical imaging AI SaaS platform — DICOM ingestion, radiomic feature extraction, multi-tenant dashboards, and clinical ML model serving at scale.

## Vision

MedVision is an end-to-end SaaS platform for clinical AI teams. It handles the full pipeline from raw DICOM upload to interactive radiomic dashboards and ML model inference — all behind a secure, multi-tenant API.

Designed for hospital systems, research consortia, and healthtech startups that need clinical-grade AI tooling without building the infrastructure themselves.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  React + TypeScript UI                  │
│         Dashboard │ DICOM Viewer │ Model Explorer        │
└───────────────────────┬─────────────────────────────────┘
                        │ REST / WebSocket
┌───────────────────────▼─────────────────────────────────┐
│              FastAPI Gateway (Auth + Routing)            │
│   JWT Middleware │ Tenant Isolation │ Rate Limiting       │
└──────┬──────────────┬──────────────────┬────────────────┘
       │              │                  │
┌──────▼──────┐ ┌─────▼──────┐ ┌────────▼────────┐
│  DICOM      │ │ Radiomic   │ │  ML Model       │
│  Ingestion  │ │ Extraction │ │  Serving API    │
│  Service    │ │  Worker    │ │  (ONNX/PyTorch) │
└──────┬──────┘ └─────┬──────┘ └────────┬────────┘
       │              │                  │
┌──────▼──────────────▼──────────────────▼────────┐
│         PostgreSQL │ Redis │ MinIO (DICOM store)  │
└─────────────────────────────────────────────────┘
```

## Tech Stack

| Layer | Technology |
|-------|------------|
| Backend API | FastAPI, Python 3.11 |
| Frontend | React 18, TypeScript, Plotly.js |
| DICOM Processing | Pydicom, SimpleITK, ITK-SNAP |
| Radiomic Features | PyRadiomics, scikit-learn |
| ML Serving | ONNX Runtime, PyTorch |
| Auth | JWT, OAuth2, RBAC |
| Database | PostgreSQL 16, Redis 7, TimescaleDB |
| Object Storage | MinIO (S3-compatible) |
| Container | Docker, Docker Compose, Kubernetes-ready |
| CI/CD | GitHub Actions |

## Key Features

- **Multi-tenant DICOM ingestion** — secure upload with tenant isolation and DICOM metadata parsing
- **Automated radiomic extraction** — PyRadiomics pipeline with 107 standardised IBSI features
- **Interactive dashboards** — Plotly.js visualisations of feature distributions, correlations, and PCA projections
- **ML model serving** — upload ONNX or PyTorch models and serve predictions via REST API
- **Clinical audit trail** — every prediction logged with patient ID, model version, and confidence score
- **Role-based access** — Admin, Radiologist, Researcher, and Viewer roles with JWT enforcement

## Project Structure

```
medvision-ai-saas/
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   ├── routes/
│   │   │   │   ├── dicom.py          # DICOM upload & parsing
│   │   │   │   ├── radiomic.py       # Feature extraction endpoints
│   │   │   │   ├── models.py         # ML model management
│   │   │   │   ├── tenants.py        # Multi-tenant management
│   │   │   │   └── auth.py           # JWT auth endpoints
│   │   ├── core/
│   │   │   ├── config.py
│   │   │   ├── security.py
│   │   │   └── database.py
│   │   ├── models/
│   │   │   ├── patient.py
│   │   │   ├── study.py
│   │   │   ├── radiomic_features.py
│   │   │   └── ml_prediction.py
│   │   ├── services/
│   │   │   ├── dicom_ingestion.py
│   │   │   ├── feature_extractor.py
│   │   │   └── model_server.py
│   │   └── workers/
│   │       └── radiomic_worker.py    # Celery async worker
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── DicomViewer.tsx
│   │   │   ├── FeatureDashboard.tsx
│   │   │   └── ModelExplorer.tsx
│   │   └── pages/
├── infrastructure/
│   ├── docker-compose.yml
│   ├── docker-compose.prod.yml
│   └── k8s/
├── .github/
│   └── workflows/
│       ├── test.yml
│       └── deploy.yml
└── README.md
```

## Database Schema

```sql
-- Multi-tenant schema
CREATE TABLE tenants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE patients (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID REFERENCES tenants(id),
    mrn VARCHAR(100),
    anonymised BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE dicom_studies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    patient_id UUID REFERENCES patients(id),
    study_uid VARCHAR(255) UNIQUE,
    modality VARCHAR(10),
    storage_path TEXT,
    metadata JSONB,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE radiomic_extractions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    study_id UUID REFERENCES dicom_studies(id),
    features JSONB NOT NULL,
    extractor_version VARCHAR(50),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE ml_predictions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    study_id UUID REFERENCES dicom_studies(id),
    model_name VARCHAR(100),
    model_version VARCHAR(50),
    prediction JSONB,
    confidence FLOAT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

## API Endpoints

```
POST   /api/v1/dicom/upload              Upload DICOM files
GET    /api/v1/dicom/{study_id}          Get study metadata
POST   /api/v1/radiomic/extract/{id}     Trigger feature extraction
GET    /api/v1/radiomic/{study_id}       Get extracted features
POST   /api/v1/models/upload             Upload ML model
POST   /api/v1/models/predict/{id}       Run prediction
GET    /api/v1/tenants/                  List tenants (admin)
POST   /api/v1/auth/login               JWT login
POST   /api/v1/auth/refresh             Token refresh
```

## Quick Start

```bash
git clone https://github.com/SylvesterKT/medvision-ai-saas
cd medvision-ai-saas
cp .env.example .env
docker-compose up --build

# API available at http://localhost:8000
# Frontend at http://localhost:3000
# Docs at http://localhost:8000/docs
```

## CI/CD Pipeline

```yaml
# .github/workflows/test.yml
name: Test Suite
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
      redis:
        image: redis:7
    steps:
      - uses: actions/checkout@v4
      - name: Run pytest
        run: pytest tests/ --cov=app --cov-report=xml
      - name: Upload coverage
        uses: codecov/codecov-action@v4
```

## Roadmap

- [x] Multi-tenant DICOM ingestion
- [x] PyRadiomics extraction pipeline
- [x] JWT authentication and RBAC
- [ ] DICOM web viewer (Cornerstone.js)
- [ ] Federated learning across tenants
- [ ] FHIR R4 integration
- [ ] HL7 message processing

## Resume Impact

Built a multi-tenant medical imaging SaaS platform integrating DICOM ingestion, radiomic feature extraction (PyRadiomics), and ONNX model serving — demonstrating end-to-end clinical AI product engineering.

## License

MIT © Sylvester KT
