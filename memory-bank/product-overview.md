# Product Overview — Financial Metrics Dashboard

> **Basado en:** evidencia verificable del código fuente,
> archivos de configuración, tests y documentación del repositorio
> `andressolo18-2-ai-eng-financial-dashboard-context-project-main`.

---

## 🎯 Propósito

Aplicación **full-stack** tipo dashboard que visualiza métricas financieras
clave de una empresa: ingresos, gastos, beneficio neto y margen de beneficio.
Está concebida como proyecto educativo de **4Geeks Academy** para el programa
de **AI Engineering**, donde los estudiantes practican análisis de código,
documentación asistida por agentes de IA y configuración de reglas para
agentes (`./.agents/rules`).

---

## 📐 Arquitectura General

```
Frontend (React + TypeScript + Vite)         Backend (FastAPI + Python 3.13)
       │                                              │
       │  GET /api/metrics                            │
       │─────────────────────────────────────────────>│
       │<─────────────────────────────────────────────│
       │                                    JSON: FinancialMovement[]
       │
       │  Procesa datos localmente:
       │    computeKPIs()      → 4 tarjetas KPI
       │    computeMonthlyData() → 2 gráficos Recharts
```

- **Frontend:** SPA en `frontend/`, sirve en `:5173` vía Vite dev server.
- **Backend:** API REST en `backend/`, sirve en `:8000` vía Uvicorn.
- **Proxy:** Vite redirige `/api/*` → `http://backend:8000` (configurado en
  `vite.config.ts`).
- **Orquestación:** `docker-compose.yml` levanta ambos servicios con un solo
  comando.

---

## 🧩 Funcionalidades Actuales (verificadas en código)

### Frontend — Componentes

| Componente | Archivo | Propósito |
|---|---|---|
| `DashboardHeader` | `dashboard-header.tsx` | Título "Financial Overview" + badge de período |
| `KPIRow` | `kpi-row.tsx` | Grid responsivo de 4 KPIs |
| `KPICard` | `kpi-card.tsx` | Tarjeta individual con icono, valor, helper text y variante de color |
| `IncomeOutcomeChart` | `income-outcome-chart.tsx` | Gráfico de líneas (Recharts): ingresos vs gastos mensuales |
| `ProfitPercentChart` | `profit-percent-chart.tsx` | Gráfico de líneas: margen de beneficio mensual con referencia en 0 |

### Frontend — Lógica

| Módulo | Archivo | Funciones exportadas |
|---|---|---|
| `financial-types.ts` | Tipos compartidos | `FinancialMovement`, `KPIMetrics`, `MonthlyDataPoint`, `OperationType`, `Category`, `BusinessType` |
| `financial-utils.ts` | Utilidades KPI/chart | `computeKPIs()`, `computeMonthlyData()`, `formatCurrency()`, `formatPercent()` |
| `mock-data.ts` | 58 movimientos mock estáticos (año 2024) | Datos de respaldo/sin backend |
| `utils.ts` | `cn()` | Merge de clases Tailwind vía `clsx` + `tailwind-merge` |

### Backend — Endpoints API (FastAPI)

| Endpoint | Método | Descripción | ¿Usado por frontend? |
|---|---|---|---|
| `/health` | GET | Health check → `{"status": "ok"}` | No |
| `/api/metrics` | GET | Lista de movimientos (filtrable por fecha, categoría, tipo) | **Sí** (único usado) |
| `/api/metrics/facets` | GET | Facetas disponibles para filtros | No |
| `/api/metrics/summary` | GET | Resumen agrupado por day/week/month con income/outcome/net | No |
| `/api/metrics/categories/top` | GET | Top categorías por monto total | No |
| `/api/metrics/comparison` | GET | Comparativa período actual vs anterior (delta absoluto y %) | No |
| `/api/metrics/alerts` | GET | Alertas de anomalías en gastos por umbral | No |
| `/api/metrics/b2b` | GET | Movimientos filtrados solo B2B | No |
| `/api/metrics/b2c` | GET | Movimientos filtrados solo B2C | No |

### Backend — Generación de Datos

- `generate_mock_movements(seed=42)` genera **360 movimientos** (30 por mes × 12 meses).
- Tipos de operación: `income` / `outcome`.
- Categorías: `sales`, `suppliers`, `operational`, `administrative`, `others`.
- Tipo de negocio: `B2B` (~55%) / `B2C` (~45%).
- Datos deterministas gracias a `random.seed(42)`.

---

## 🧪 Tests (verificados)

| Capa | Framework | Archivos | Cantidad |
|---|---|---|---|
| Backend | `pytest` + `httpx.TestClient` | `test_routes.py` | **16 tests** de integración |
| Frontend | `vitest` | `financial-utils.test.ts` | **4 tests** unitarios |

### Backend — Casos cubiertos

- `test_generate_mock_movements_returns_full_year_sorted_data` — 360 movs, orden cronológico
- `test_filter_movements_by_date_includes_range_edges` — filtro por fecha (inclusive)
- `test_health_endpoint_returns_ok` — health check
- `test_metrics_endpoint_respects_date_filters` — filtro date en `/api/metrics`
- `test_b2b_endpoint_only_returns_b2b_records` — solo B2B
- `test_b2c_endpoint_only_returns_b2c_records` — solo B2C
- `test_metrics_endpoint_filters_by_category` — filtro categoría
- `test_metrics_endpoint_filters_by_operation_type` — filtro tipo op.
- `test_b2b_endpoint_combines_new_filters` — filtros combinados
- `test_metrics_facets_returns_filter_options_and_date_range` — facets
- `test_metrics_summary_by_month_returns_balances` — summary mensual
- `test_metrics_summary_by_week_honors_business_type_filter` — summary semanal + B2C
- `test_top_categories_returns_limited_sorted_categories` — top categorías
- `test_metrics_comparison_returns_delta_fields` — comparación períodos
- `test_metrics_alerts_returns_anomaly_candidates` — alertas

### Frontend — Casos cubiertos

- `computeKPIs` — totals + profit (2 tests: normal y sin ingresos)
- `computeMonthlyData` — datos cronológicos multi-año
- `formatCurrency` — formato moneda sin decimales
- `formatPercent` — porcentaje con 1 decimal

---

## 🔗 Flujo de Datos

```
1. App.tsx monta → useEffect llama a fetch(`${API_BASE_URL}/api/metrics`)
2. Backend genera 360 movimientos con seed=42, filtra y devuelve JSON
3. computeKPIs(movements) → totalIncome, totalOutcome, profit, profitPercent
4. computeMonthlyData(movements) → MonthlyDataPoint[] (12 meses)
5. KPIRow recibe KPIMetrics → 4 KPICards
6. IncomeOutcomeChart / ProfitPercentChart reciben MonthlyDataPoint[]
7. Recharts renderiza gráficos con tooltips personalizados
```

---

## 🚀 Ejecución

### Opción 1 — Docker (recomendada)
```bash
docker compose up --build
```

### Opción 2 — Desarrollo manual
```bash
# Backend
cd backend && pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000

# Frontend
cd frontend && npm install && npm run dev
```

### URLs
- Frontend: http://localhost:5173
- Backend (API): http://localhost:8000
- Documentación automática (OpenAPI): http://localhost:8000/docs

---

## 📁 Estructura del Proyecto (verificada)

```
├── docker-compose.yml          ← Orquestación Docker
├── AGENTS.md                   ← Guía para agentes de IA
├── README.md / README.es.md    ← Documentación bilingüe
├── verification.md             ← Resumen de verificación manual
├── .agents/
│   ├── rules/                  ← Reglas para agentes
│   └── skills/                 ← Skills para agentes
├── backend/
│   ├── Dockerfile              ← python:3.13-slim + debugpy
│   ├── requirements.txt        ← fastapi, uvicorn, pytest, httpx, debugpy
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py             ← FastAPI app + CORS
│   │   └── routes.py           ← 9 endpoints + lógica de negocio
│   └── tests/
│       ├── conftest.py         ← Fixtures (sys.path)
│       └── test_routes.py      ← 16 tests de integración
└── frontend/
    ├── Dockerfile              ← node:24-alpine
    ├── package.json            ← React 19, Recharts, Tailwind 4, shadcn/ui
    ├── vite.config.ts          ← Vite 8 + proxy /api + Tailwind plugin
    ├── index.html
    ├── tsconfig*.json
    ├── eslint.config.js
    ├── public/
    │   └── favicon.svg
    └── src/
        ├── main.tsx / App.tsx
        ├── index.css           ← Tailwind + CSS variables
        ├── lib/                ← Tipos, utilidades, mock data
        └── components/
            ├── dashboard/      ← 5 componentes del dashboard
            └── ui/             ← Card, Skeleton (shadcn/ui)
```

---

## 👤 Autoría y Propósito Educativo

- **Creado por:** [@marcogonzalo](https://github.com/marcogonzalo) y
  [contribuidores](https://github.com/4GeeksAcademy/ai-eng-financial-dashboard-context-project/graphs/contributors)
- **Institución:** [4Geeks Academy](https://4geeksacademy.com/)
- **Programa:** AI Engineering
- **Propósito:** Proyecto de práctica para que estudiantes de AI Engineering
  aprendan a inspeccionar código, documentar con agentes de IA y configurar
  reglas de automatización.
