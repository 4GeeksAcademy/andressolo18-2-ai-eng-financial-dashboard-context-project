# 📊 Financial Dashboard — Verificación y Resumen del Proyecto

## 🎯 ¿Qué hace?

Es un **dashboard financiero** que muestra métricas clave de ingresos y gastos de una empresa. Visualiza:

- **4 KPIs principales**: Ingresos totales, Gastos totales, Beneficio neto y Margen de beneficio (%)
- **Gráfico de líneas (Income vs Outcome)**: Evolución mensual de ingresos vs gastos
- **Gráfico de margen (Profit Margin %)**: Porcentaje de beneficio mes a mes

Los datos son **simulados** (mock), generados aleatoriamente pero consistentes (usa `seed=42` en el backend).

---

## 🏗️ Arquitectura General

Proyecto **Full-Stack** con frontend en **React + TypeScript (Vite)** y backend en **FastAPI (Python)**, orquestado con **Docker Compose**.

```
andressolo18-2-ai-eng-financial-dashboard-context-project-main/
├── docker-compose.yml          ← Orquestación de servicios
├── AGENTS.md                   ← Guía para agentes de IA
├── README.md                   ← Documentación principal
├── README.es.md                ← Documentación en español
├── verification.md             ← Este archivo
│
├── backend/                    ← API REST (Python)
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── app/
│   │   ├── __init__.py         ← Marca el paquete (vacío)
│   │   ├── main.py             ← Entry point de FastAPI
│   │   └── routes.py           ← Endpoints + lógica de negocio
│   └── tests/
│       ├── conftest.py         ← Fixtures de pytest
│       └── test_routes.py      ← Tests de integración
│
└── frontend/                   ← SPA (React + TypeScript)
    ├── Dockerfile
    ├── package.json
    ├── vite.config.ts          ← Configuración de Vite + proxy
    ├── index.html              ← HTML entry point
    ├── tsconfig.json / *.json  ← Config TypeScript
    ├── eslint.config.js        ← Linter
    ├── public/
    └── src/
        ├── main.tsx            ← Entry point React
        ├── App.tsx             ← Componente raíz
        ├── index.css           ← Estilos globales + Tailwind
        ├── lib/
        │   ├── financial-types.ts     ← Tipos compartidos
        │   ├── financial-utils.ts     ← Utilidades KPI/charts
        │   ├── financial-utils.test.ts ← Tests unitarios (Vitest)
        │   ├── mock-data.ts           ← Mock data estática
        │   └── utils.ts               ← Utilidad cn() (clsx+twMerge)
        └── components/
            ├── dashboard/
            │   ├── dashboard-header.tsx       ← Header del dashboard
            │   ├── kpi-row.tsx                ← Fila de 4 KPIs
            │   ├── kpi-card.tsx               ← Tarjeta de KPI individual
            │   ├── income-outcome-chart.tsx   ← Gráfico ingresos/gastos
            │   └── profit-percent-chart.tsx   ← Gráfico margen de beneficio
            └── ui/
                ├── card.tsx                   ← Componente Card (shadcn/ui)
                └── skeleton.tsx               ← Componente Skeleton
```

---

## 🔌 ¿Cómo se conecta?

```
🌐 Navegador
    │
    ▼
[Frontend] ── Vite (puerto 5173)
    │
    │  fetch("/api/metrics")
    │       │
    ▼       ▼
Proxy de Vite (vite.config.ts)
    │  redirige /api/* → http://backend:8000
    ▼
[Backend] ── FastAPI (puerto 8000)
    │
    ▼
Genera datos mock (360 movimientos anuales con seed=42)
```

**Datos mock**: El backend genera 30 movimientos aleatorios por cada uno de los 12 meses (total 360), con tipos `income`/`outcome`, categorías (`sales`, `suppliers`, `operational`, `administrative`, `others`) y tipo de negocio (`B2B`/`B2C`).

**Conexión actual**: El frontend solo consume **1 endpoint** (`GET /api/metrics`) y procesa los datos localmente para calcular KPIs y datos mensuales. El backend tiene **9 endpoints** disponibles que no están siendo usados por el frontend aún.

---

## 🚀 ¿Cómo se ejecuta?

### Opción 1 — Docker (recomendada)

```bash
docker compose up --build
```

Esto levanta ambos servicios:
- **Frontend**: http://localhost:5173
- **Backend**: http://localhost:8000
- **Docs API**: http://localhost:8000/docs

### Opción 2 — Desarrollo manual

**Backend** (desde `backend/`):
```bash
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

**Frontend** (desde `frontend/`):
```bash
npm install
npm run dev
```

---

## 🔌 Servicios API (Backend — FastAPI)

Todos los endpoints definidos en `routes.py`, router montado en `app/main.py`:

| Endpoint | Método | Descripción |
|----------|--------|-------------|
| `/health` | GET | Health check → `{"status": "ok"}` |
| `/api/metrics` | GET | Lista de movimientos financieros (filtrable por `start_date`, `end_date`, `category`, `operation_type`) |
| `/api/metrics/facets` | GET | Facetas disponibles para filtros (tipos de operación, categorías, rangos de fecha) |
| `/api/metrics/summary` | GET | Resumen agrupado por `day`/`week`/`month` con income/outcome/net |
| `/api/metrics/categories/top` | GET | Top categorías por monto total, filtrable por `operation_type` |
| `/api/metrics/comparison` | GET | Comparativa período actual vs anterior (delta absoluto y porcentual) |
| `/api/metrics/alerts` | GET | Alertas de anomalías en gastos por umbral (`threshold`) |
| `/api/metrics/b2b` | GET | Movimientos filtrados solo B2B |
| `/api/metrics/b2c` | GET | Movimientos filtrados solo B2C |

---

## 🧩 Servicios Frontend

| Módulo | Archivo | Función |
|--------|---------|---------|
| **Tipos** | `financial-types.ts` | Interfaces: `FinancialMovement`, `KPIMetrics`, `MonthlyDataPoint` |
| **Utilidades** | `financial-utils.ts` | `computeKPIs()`, `computeMonthlyData()`, `formatCurrency()`, `formatPercent()` |
| **Mock data** | `mock-data.ts` | 58 movimientos mock estáticos (año 2024) |
| **Utils** | `utils.ts` | Función `cn()` para merge de clases Tailwind |
| **Componentes** | `dashboard-header.tsx` | Encabezado con título y período |
| | `kpi-row.tsx` | Grid de 4 KPIs (Income, Outcome, Profit, Profit Margin) |
| | `kpi-card.tsx` | Tarjeta individual con icono, valor y variante de color |
| | `income-outcome-chart.tsx` | Gráfico de líneas (Recharts) ingresos vs gastos mensuales |
| | `profit-percent-chart.tsx` | Gráfico de líneas del margen de beneficio mensual |
| **UI Base** | `card.tsx` | Componentes Card (shadcn/ui) |
| | `skeleton.tsx` | Componente Skeleton para loading states |

---

## 🧪 Tests

| Capa | Framework | Archivos | Cantidad |
|------|-----------|----------|----------|
| **Backend** | `pytest` + `httpx` (TestClient) | `test_routes.py` | 16 tests de integración |
| **Frontend** | `vitest` | `financial-utils.test.ts` | 4 tests unitarios |

```bash
# Backend
cd backend && pytest

# Frontend
cd frontend && npm test
```

---

## 🏗️ Stack Tecnológico

| Capa | Tecnología |
|------|-----------|
| **Frontend** | React 19 + TypeScript + Vite 8 |
| **Estilos** | Tailwind CSS 4 + shadcn/ui |
| **Gráficos** | Recharts |
| **Iconos** | Lucide React |
| **Backend** | Python 3.13 + FastAPI |
| **Testing Backend** | pytest + httpx |
| **Testing Frontend** | Vitest |
| **Infraestructura** | Docker + Docker Compose |

---

## 🔐 Entry Points

| Capa | Archivo | Puerto | Comando |
|------|---------|--------|---------|
| **Backend** | `backend/app/main.py` (FastAPI app) | `8000` | `uvicorn app.main:app --reload` |
| **Frontend** | `frontend/index.html` → `frontend/src/main.tsx` → `App.tsx` | `5173` | `vite dev` |
| **Infraestructura** | `docker-compose.yml` | 8000 y 5173 | `docker compose up --build` |