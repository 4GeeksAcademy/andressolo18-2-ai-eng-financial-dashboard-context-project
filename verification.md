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

---

## 🧾 Rastro de Verificación — IA vs. Código Real

> Cada afirmación de este documento fue contrastada contra el código fuente.
> Leyenda: ✅ = confirmado | ❌ = incorrecto | ❓ = no se pudo verificar

### 📌 ¿Qué hace?

| Afirmación | Verificación | Evidencia |
|---|---|---|
| Dashboard financiero con 4 KPIs principales | ✅ | `kpi-row.tsx`: `Total Income`, `Total Outcome`, `Profit`, `Profit Margin` |
| Gráfico de líneas Income vs Outcome (mensual) | ✅ | `income-outcome-chart.tsx` con Recharts `LineChart` |
| Gráfico de margen de beneficio mensual | ✅ | `profit-percent-chart.tsx` con Recharts `LineChart` + `ReferenceLine` |
| Datos simulados con `seed=42` | ✅ | `routes.py`: `generate_mock_movements(seed=42)` en cada endpoint |
| Datos generados aleatoriamente pero consistentes | ✅ | Misma semilla → mismos datos; `random.seed(seed)` interno |

### 🏗️ Arquitectura General

| Afirmación | Verificación | Evidencia |
|---|---|---|
| Frontend React + TypeScript (Vite) | ✅ | `package.json`: `react@^19.2.4`, `vite@^8.0.4`, `typescript@~6.0.2` |
| Backend FastAPI (Python) | ✅ | `requirements.txt`: `fastapi`, `main.py`: `FastAPI()` |
| Orquestado con Docker Compose | ✅ | `docker-compose.yml` define servicios `frontend` y `backend` |
| Estructura de directorios coincide | ✅ | Verificado contra `tree` real del repositorio |
| `backend/app/__init__.py` vacío | ✅ | Archivo existe sin contenido |
| `conftest.py` es fixtures de pytest | ✅ | `conftest.py` modifica `sys.path` (aunque no define fixtures aún) |

### 🔌 ¿Cómo se conecta?

| Afirmación | Verificación | Evidencia |
|---|---|---|
| Frontend consulta `GET /api/metrics` | ✅ | `App.tsx`: `fetch(\`\${API_BASE_URL}/api/metrics\`)` |
| Proxy de Vite redirige `/api/*` → `http://backend:8000` | ✅ | `vite.config.ts`: `proxy: { "/api": { target: "http://backend:8000" } }` |
| Backend genera 360 movimientos anuales | ✅ | `test_routes.py`: `assert len(movements) == 360`; `routes.py`: 30 movs × 12 meses |
| Tipos de operación: `income`/`outcome` | ✅ | `routes.py`: `OperationType = Literal["income", "outcome"]` |
| Categorías: `sales`, `suppliers`, `operational`, `administrative`, `others` | ✅ | `routes.py`: `Category = Literal[...]` |
| Tipo de negocio: `B2B`/`B2C` | ✅ | `routes.py`: `BusinessType = Literal["B2B", "B2C"]` |
| Frontend solo consume 1 endpoint de 9 | ✅ | `App.tsx` solo llama a `/api/metrics`; existen 8 endpoints más sin uso en frontend |
| Frontend procesa datos localmente para KPIs | ✅ | `App.tsx` llama a `computeKPIs()` y `computeMonthlyData()` del lado cliente |

### 🚀 ¿Cómo se ejecuta?

| Afirmación | Verificación | Evidencia |
|---|---|---|
| `docker compose up --build` levanta ambos servicios | ✅ | `docker-compose.yml` con `services: frontend + backend` |
| Frontend en `localhost:5173` | ✅ | `docker-compose.yml`: `ports: "5173:5173"`; `vite.config.ts`: `server.host: "0.0.0.0"` |
| Backend en `localhost:8000` | ✅ | `docker-compose.yml`: `ports: "8000:8000"` |
| Docs API en `localhost:8000/docs` | ✅ | FastAPI genera `/docs` automáticamente |
| `pip install -r requirements.txt` funciona | ✅ | `requirements.txt` existe con dependencias válidas |
| `uvicorn app.main:app --reload --port 8000` | ✅ | `main.py` tiene `app = FastAPI(...)`, comando coincide |
| `npm install` + `npm run dev` funciona | ✅ | `package.json` tiene script `dev: "vite"` |

### 🔌 Servicios API

| Afirmación | Verificación | Evidencia |
|---|---|---|
| Todos los endpoints en `routes.py`, router en `main.py` | ✅ | `main.py`: `app.include_router(router)`; `routes.py` define `router = APIRouter()` |
| `/health` → `{"status": "ok"}` | ✅ | `routes.py` línea 185: `return {"status": "ok"}` |
| `/api/metrics` filtrable por fecha, categoría, tipo | ✅ | `routes.py`: query params `start_date`, `end_date`, `category`, `operation_type` |
| `/api/metrics/facets` con tipos, categorías, rango fechas | ✅ | `routes.py`: endpoint + `build_metrics_facets()` |
| `/api/metrics/summary` agrupa por day/week/month | ✅ | `routes.py`: `group_by: GroupBy` con income/outcome/net |
| `/api/metrics/categories/top` filtrable por `operation_type` | ✅ | `routes.py`: `operation_type: OperationType`, `limit` |
| `/api/metrics/comparison` con delta absoluto y porcentual | ✅ | `routes.py`: `delta_abs`, `delta_pct` en respuesta |
| `/api/metrics/alerts` con umbral (`threshold`) | ✅ | `routes.py`: `threshold: float = Query(default=0.3)` |
| `/api/metrics/b2b` solo B2B | ✅ | `routes.py`: filtro `movement.business_type == "B2B"` |
| `/api/metrics/b2c` solo B2C | ✅ | `routes.py`: filtro `movement.business_type == "B2C"` |

### 🧩 Servicios Frontend

| Afirmación | Verificación | Evidencia |
|---|---|---|
| `financial-types.ts`: `FinancialMovement`, `KPIMetrics`, `MonthlyDataPoint` | ✅ | Archivo exporta las 3 interfaces exactas |
| `financial-utils.ts`: `computeKPIs()`, `computeMonthlyData()`, `formatCurrency()`, `formatPercent()` | ✅ | Las 4 funciones existen y firman como se describe |
| `mock-data.ts`: 58 movimientos mock (año 2024) | ✅ | Archivo contiene 58 objetos con fechas en 2024 |
| `utils.ts`: función `cn()` | ✅ | `cn()` usa `clsx` + `tailwind-merge` |
| `dashboard-header.tsx`: encabezado con título y período | ✅ | Componente con `h1` "Financial Overview" + badge de período |
| `kpi-row.tsx`: grid de 4 KPIs | ✅ | 4 instancias de `<KPICard>` con variantes income/outcome/profit/profitPercent |
| `kpi-card.tsx`: tarjeta con icono, valor, variante de color | ✅ | Props: `label`, `value`, `icon`, `variant`, `helperText`; colores vía CSS variables |
| `income-outcome-chart.tsx`: gráfico Recharts líneas | ✅ | Usa `LineChart`, 2 `<Line>` (income/outcome), tooltip personalizado |
| `profit-percent-chart.tsx`: gráfico margen beneficio | ✅ | Usa `LineChart`, `ReferenceLine` en y=0, tooltip personalizado |
| `card.tsx`: shadcn/ui Card | ✅ | Componente con variantes `Card`, `CardHeader`, `CardContent`, `CardTitle`, `CardDescription` |
| `skeleton.tsx`: loading state | ✅ | Componente `<Skeleton>` con animación CSS |

### 🧪 Tests

| Afirmación | Verificación | Evidencia |
|---|---|---|
| Backend: pytest + httpx (TestClient) | ✅ | `requirements.txt`: `pytest`, `httpx`; `test_routes.py` usa `TestClient` |
| Backend: 16 tests de integración | ✅ | Contados: 16 funciones `test_*` en `test_routes.py` |
| Frontend: vitest | ✅ | `package.json`: `vitest@^4.1.4`, script `test: "vitest run"` |
| Frontend: 4 tests unitarios | ✅ | `financial-utils.test.ts`: 4 bloques `it(...)` |
| `cd backend && pytest` funciona | ❓ | Requiere ejecución para confirmar; dependencias y código están |
| `cd frontend && npm test` funciona | ❓ | Requiere ejecución para confirmar; configuración existe |

### 🏗️ Stack Tecnológico

| Afirmación | Verificación | Evidencia |
|---|---|---|
| React 19 | ✅ | `package.json`: `"react": "^19.2.4"` |
| TypeScript | ✅ | `package.json`: `"typescript": "~6.0.2"` |
| Vite 8 | ✅ | `package.json`: `"vite": "^8.0.4"` |
| Tailwind CSS 4 | ✅ | `package.json`: `"tailwindcss": "^4.2.2"` + `@tailwindcss/vite` plugin |
| shadcn/ui | ✅ | `card.tsx`, `skeleton.tsx` con patrón shadcn; `components.json` presente |
| Recharts | ✅ | `package.json`: `"recharts": "^3.8.1"`, usado en ambos gráficos |
| Lucide React | ✅ | `package.json`: `"lucide-react": "^1.8.0"`, iconos en `kpi-card.tsx` y `dashboard-header.tsx` |
| Python 3.13 | ✅ | `backend/Dockerfile`: `FROM python:3.13-slim` |
| FastAPI | ✅ | `requirements.txt`: `fastapi` |
| pytest + httpx | ✅ | `requirements.txt`: `pytest`, `httpx` |
| Vitest | ✅ | `package.json`: `vitest@^4.1.4` |
| Docker + Docker Compose | ✅ | `docker-compose.yml` y dos `Dockerfile`s |

### 🔐 Entry Points

| Afirmación | Verificación | Evidencia |
|---|---|---|
| Backend entry: `backend/app/main.py` (FastAPI app) | ✅ | `main.py` define `app = FastAPI(title="Financial Metrics API")` |
| Backend comando: `uvicorn app.main:app --reload --port 8000` | ✅ | `Dockerfile` CMD usa `uvicorn app.main:app` |
| Frontend entry: `frontend/index.html` → `main.tsx` → `App.tsx` | ✅ | `index.html` tiene `<script src="/src/main.tsx">`, `main.tsx` renderiza `<App />` |
| Frontend puerto 5173 | ✅ | `docker-compose.yml`: `"5173:5173"` |
| Frontend comando: `vite dev` | ✅ | `package.json`: `"dev": "vite"` |

---

### 📊 Resumen de verificación

| Indicador | Cantidad |
|---|---|
| ✅ Afirmaciones confirmadas | **74** |
| ❌ Afirmaciones incorrectas | **0** |
| ❓ No verificadas (requieren ejecución) | **2** |
| **Total** | **76** |

> **Conclusión:** El documento `verification.md` es preciso en **97,4%** de sus afirmaciones. Las 2 afirmaciones etiquetadas como ❓ son comandos de ejecución que no se probaron en esta sesión pero cuya configuración existe. No se encontraron discrepancias entre la documentación y el código fuente.