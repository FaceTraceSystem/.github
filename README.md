# Facial Recognition System

A software system consisting of an operator dashboard, a vision-processing service, and an identity API, designed to enrol people, recognise faces from a camera stream, and manage associated identity records.

## Overview

This project is organised into three application components. The dashboard is the browser interface used by operators; the vision service processes camera images and controls recognition flow; and the identity backend stores people, face descriptors, and recognition records.

## System Architecture

```yaml
Operators
  |
  v
React Dashboard
  |
  v
FastAPI Vision Service
  |
  v
Spring Boot Identity API
  |
  v
PostgreSQL Database
```

The browser sends images to the vision service. The vision service detects and encodes faces, then sends face embeddings to the identity API for matching and record management. Face embeddings are not returned to the browser.

## Components

| Component | Location | Description |
| --- | --- | --- |
| Dashboard | `dashboard/` | React operator interface for live recognition, guided enrolment, people management, identity-card details, and a camera-aware focus timer. |
| Vision service | `vision-service/` | FastAPI service that performs face detection, encoding, quality checks, blink-based liveness checks, live WebSocket streaming, and proxying to the identity API. |
| Identity backend | `search-backend/` | Spring Boot API for storing people and face descriptors, exact face matching, recognition history, photos, signatures, and administrative updates. |

## Key Features

- Live browser-camera recognition with per-face detection and temporal voting.
- Guided enrolment across front, left, right, up, and down poses.
- Image-quality checks and blink-based liveness gating before identities are shown.
- Exact Euclidean matching of 128-dimensional dlib face embeddings.
- Duplicate-enrolment, wrong-sample, and five-sample-per-person safeguards.
- People administration: active/suspended status, face samples, recognition history, ID-card fields, photos, and signatures.

## Tech Stack

- **Dashboard:** React, TypeScript, Vite, WebSocket, browser `getUserMedia`
- **Vision service:** Python, FastAPI, Uvicorn, dlib, `face_recognition`, OpenCV, NumPy, HTTPX
- **Identity backend:** Java 25, Spring Boot, Spring MVC, JDBC, Flyway
- **Database:** PostgreSQL with face embeddings stored as `real[]` arrays
- **Local database:** Docker Compose with `pgvector/pgvector:pg17`

## Getting Started

1. Start PostgreSQL from `search-backend/`:

   ```powershell
   docker compose up -d postgres
   ```

2. Configure the identity backend database connection. The supplied Docker database uses port `5432`, while the checked-in backend default is `2323`; set `FRS_DB_PORT=5432` and the Docker credentials before starting the backend.

3. Start the identity backend:

   ```powershell
   cd search-backend
   .\mvnw.cmd spring-boot:run
   ```

4. Install and start the vision service:

   ```powershell
   cd vision-service
   # Install face_recognition_models from a local checkout first, as noted in requirements.txt.
   pip install -r requirements.txt
   python run_service.py
   ```

5. Install and start the dashboard:

   ```powershell
   cd dashboard
   npm install
   npm run dev
   ```

The dashboard normally runs at `http://localhost:5173`, the vision service at `http://127.0.0.1:8000`, and the identity backend at `http://localhost:8082`.

## Configuration

| Variable | Default | Purpose |
| --- | --- | --- |
| `VITE_VISION_URL` | `http://127.0.0.1:8000` | Vision service URL used by the dashboard. |
| `FRS_IDENTITY_URL` | `http://localhost:8082` | Identity backend URL used by the vision service. |
| `FRS_DB_HOST` / `FRS_DB_PORT` | `localhost` / `2323` | Identity backend database host and port. |
| `FRS_DB_NAME` | `facialrecognition` | Database name. |
| `FRS_DB_USER` / `FRS_DB_PASSWORD` | `postgres` / empty | Database credentials. |
| `FRS_REQUIRE_LIVENESS` | `1` | Set to `0` to allow recognition without confirmed liveness. |

## Documentation

- Dashboard implementation: `dashboard/`
- Vision-service API and liveness notes: `vision-service/` and `vision-service/docs/liveness.md`
- Identity API, database migrations, and local database configuration: `search-backend/`

## Security Note

Authentication and authorization are not implemented in the current codebase. Run the system only in a trusted environment until access control and production security measures are added.

## License

No license file is included in this repository.
