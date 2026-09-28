# PulseOEE - Industrial IoT & Real-Time Manufacturing Intelligence Platform

[![React](https://img.shields.io/badge/Frontend-React%2018%20%2B%20TypeScript-blue.svg)](https://reactjs.org/)
[![Node.js](https://img.shields.io/badge/Backend-Node.js%20%2B%20Express-green.svg)](https://nodejs.org/)
[![Database](https://img.shields.io/badge/Database-PostgreSQL%2016-336791.svg)](https://www.postgresql.org/)
[![Cache](https://img.shields.io/badge/Cache%20%26%20PubSub-Redis%207-DC382D.svg)](https://redis.io/)
[![Docker](https://img.shields.io/badge/Containerization-Docker%20Compose-2496ED.svg)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-purple.svg)](LICENSE)

An enterprise-grade, real-time manufacturing intelligence platform built to monitor **Overall Equipment Effectiveness (OEE)**, production losses, machine downtime, and operator efficiency across manufacturing plants. Features continuous IoT multi-sensor telemetry streaming, automated condition-based predictive maintenance alerts, and Pareto (80/20) downtime root cause analysis.

---

## Architecture Overview

```
                      +------------------------------------------+
                      |         React + TypeScript Frontend       |
                      |   (Vite + Tailwind CSS + Recharts + UI)  |
                      +--------------------+---------------------+
                                           |
                                [ REST API & WebSockets ]
                                           |
                      +--------------------v---------------------+
                      |         Express.js Backend API           |
                      |   (TypeScript + Socket.io Server)        |
                      +----------+--------------------+----------+
                                 |                    |
                [ Persistent Queries ]           [ Pub/Sub & Live Cache ]
                                 |                    |
                      +----------v----------+  +------v----------+
                      | PostgreSQL Database |  |   Redis Cache   |
                      | (OEE, Downtime,     |  | (Live Telemetry,|
                      |  Alerts, Operators) |  |  Shift State)   |
                      +---------------------+  +-----------------+
```

---

## Key Features

1. **Real-Time OEE Monitoring Engine**:
   - Availability = Operating Time / Planned Production Time
   - Performance = (Total Pieces Produced / Operating Time) / Ideal Run Rate
   - Quality = Good Pieces / Total Pieces Produced
   - Overall OEE = Availability × Performance × Quality (%)
   - Visual circular gauges rated against the **World-Class OEE (>85%)** benchmark.

2. **Downtime Tracking & Pareto Diagnostics (80/20 Rule)**:
   - Tracks TPM categories: *Equipment Breakdown*, *Setup & Adjustment*, *Idling & Minor Stops*, *Tool Change*, *Material Shortage*, *Planned Maintenance*.
   - Dual-axis Pareto chart identifying the vital 20% root causes responsible for 80% of factory downtime.
   - Shift incident logging with root causes and corrective action audit trails.

3. **Predictive Maintenance & Condition Monitoring**:
   - Continuous IoT multi-sensor telemetry simulation (Bearing Vibration, Spindle Temperature, Hydraulic Line Pressure, RPM, Power Consumption).
   - Real-time threshold evaluation:
     - Temperature > 80°C (Warning) / > 85°C (Critical)
     - Vibration > 4.5 mm/s (Warning) / > 6.0 mm/s (Critical)
     - Hydraulic Pressure < 2.0 bar (Warning)
   - Interactive alarm acknowledgment and resolution workflow.

4. **Operator Productivity & Shift Quotas**:
   - Shift-level metrics (Shift A, Shift B, Shift C).
   - Realized output vs planned quota attainment.
   - Operator efficiency ratings with performance classification badges.

5. **TPM 6 Big Losses Visualization**:
   - Breakdowns categorized into Availability Losses, Performance Losses, and Quality Losses with Kaizen / SMED lean recommendations.

6. **Interactive What-If OEE Calculator**:
   - Ad-hoc calculator modal allowing engineers to test variables and model production scenarios.

---

## Tech Stack

| Component | Technology | Rationale |
| :--- | :--- | :--- |
| **Frontend** | React 18, TypeScript, Vite | Fast, typed, reactive UI with modern bundling |
| **Styling** | Tailwind CSS | Sleek industrial dark theme and custom data visualizations |
| **Charts** | Recharts, Lucide Icons | Composed dual-axis Pareto charts, live trend lines, and SVG gauges |
| **Backend** | Node.js, Express.js, TypeScript | Clean RESTful API architecture with typed DTOs and services |
| **Real-time** | Socket.io | Bi-directional streaming for live multi-sensor telemetry ticks |
| **Database** | PostgreSQL 16 | ACID-compliant relational persistence with indexed time-series logs |
| **Cache & Pub/Sub** | Redis 7 + In-Memory Fallback | High-speed cache for sensor snapshots and pub/sub event bus |
| **DevOps** | Docker & Docker Compose | Multi-container orchestration (App, Database, Redis, Nginx) |
| **CI/CD** | GitHub Actions | Automated build, type-checking, and docker compose validation |

---

## Project Structure

```
.
├── .github/
│   └── workflows/
│       └── ci.yml               # GitHub Actions CI pipeline
├── backend/
│   ├── src/
│   │   ├── config/              # PostgreSQL & Redis connections + fallback
│   │   ├── controllers/         # OEE, Machine, Downtime, Alert, Operator controllers
│   │   ├── models/
│   │   │   └── schema.sql       # PostgreSQL DDL schema & indexes
│   │   ├── routes/              # Express REST routes
│   │   ├── services/            # OEE math, IoT simulator, Alert engine
│   │   ├── socket/              # Socket.io handlers
│   │   ├── types/               # TypeScript interfaces
│   │   └── index.ts             # Express entry point
│   ├── Dockerfile
│   ├── package.json
│   └── tsconfig.json
├── frontend/
│   ├── src/
│   │   ├── components/          # Gauges, Charts, Modals, Tables, Navbar, Sidebar
│   │   ├── pages/               # Overview, Machines, Downtime, Alerts, Operators
│   │   ├── services/            # API client & Socket.io client
│   │   ├── types/               # Frontend TypeScript interfaces
│   │   ├── App.tsx              # Main application orchestrator
│   │   └── main.tsx
│   ├── Dockerfile
│   ├── nginx.conf               # Nginx reverse proxy configuration
│   ├── package.json
│   └── vite.config.ts
├── docker-compose.yml           # Complete container stack
├── .gitignore
└── README.md
```

---

## REST API Reference

### 1. Machines (`/api/machines`)
- `GET /api/machines` - Retrieve all machines with latest attached telemetry
- `GET /api/machines/:id` - Retrieve machine by ID
- `PATCH /api/machines/:id/status` - Update operational state (`RUNNING`, `IDLE`, `BREAKDOWN`, `MAINTENANCE`, `OFFLINE`)

### 2. OEE Metrics (`/api/oee`)
- `GET /api/oee/summary` - Aggregate factory KPI snapshot (Overall OEE, Availability, Performance, Quality, Units)
- `GET /api/oee/metrics` - Historical shift-level OEE records
- `GET /api/oee/losses` - TPM 6 Big Losses breakdown
- `POST /api/oee/calculate` - Ad-hoc OEE formula calculation

### 3. Downtime & Pareto (`/api/downtime`)
- `GET /api/downtime` - Retrieve all recorded downtime logs
- `GET /api/downtime/pareto` - Calculate Pareto 80/20 root cause distribution
- `POST /api/downtime` - Record a new machine downtime incident

### 4. Predictive Alerts (`/api/alerts`)
- `GET /api/alerts?status=ACTIVE` - Filter alerts by status
- `PATCH /api/alerts/:id/status` - Acknowledge or Resolve alert
- `POST /api/alerts` - Create alert entry

### 5. Operators (`/api/operators`)
- `GET /api/operators` - Retrieve roster with efficiency ratings and shift allocations

---

## Getting Started

### Option A: Running with Docker Compose (Recommended)

Ensure Docker Desktop is running, then run:

```bash
# Clone the repository
git clone https://github.com/<your-username>/pulseoee.git
cd pulseoee

# Build and start all 4 services (PostgreSQL, Redis, Backend, Frontend)
docker compose up --build
```

- **Frontend Application**: `http://localhost:80`
- **Backend REST API**: `http://localhost:5000/api`
- **API Health Check**: `http://localhost:5000/api/health`

---

### Option B: Running Locally (Native Node.js)

The platform is engineered with a **Resilient In-Memory Fallback Engine**. Even if PostgreSQL or Redis are not running locally, the platform boots up seamlessly with full functionality and simulated telemetry!

#### 1. Start the Backend:
```bash
cd backend
npm install
npm run dev
```
*Backend runs on `http://localhost:5000`*

#### 2. Start the Frontend:
```bash
cd ../frontend
npm install
npm run dev
```
*Frontend runs on `http://localhost:5173`*

---

## Git & GitHub Version Control

```bash
# Stage all files
git add .

# Create initial commit
git commit -m "feat: complete PulseOEE manufacturing intelligence platform"

# Push to your GitHub repository
git remote add origin https://github.com/<your-username>/pulseoee.git
git branch -M main
git push -u origin main
```
