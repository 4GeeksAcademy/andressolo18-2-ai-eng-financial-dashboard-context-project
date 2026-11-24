# Tech Stack — Financial Metrics Dashboard

> Basado en evidencia directa del código: `package.json`, `requirements.txt`,
> `Dockerfile`s, archivos de configuración y sentencias `import`/`from`
> en todos los módulos del proyecto.

---

## 📦 Resumen por Capa

| Capa | Tecnología | Versión (verificada) |
|---|---|---|
| **Frontend Framework** | React | ^19.2.4 |
| **Lenguaje Frontend** | TypeScript | ~6.0.2 |
| **Bundler / Dev Server** | Vite | ^8.0.4 |
| **Estilos** | Tailwind CSS | ^4.2.2 |
| **UI Component Library** | shadcn/ui (Card, Skeleton) | Personalizado sobre Radix |
| **Gráficos** | Recharts | ^3.8.1 |
| **Iconos** | Lucide React | ^1.8.0 |
| **Backend Framework** | FastAPI (Python) | Última estable (pip) |
| **Lenguaje Backend** | Python | 3.13 (slim) |
| **ASGI Server** | Uvicorn | Última estable (pip) |
| **Testing Backend** | pytest + httpx + pytest-cov | Últimas estables |
| **Testing Frontend** | Vitest | ^4.1.4 |
| **Contenedores** | Docker + Docker Compose | — |
| **Debugging** | debugpy (backdoor en backend) | Última estable (pip) |
| **Linter** | ESLint (flat config) | ^9.39.4 |

---

## 🖥️ Frontend — Dependencias de Producción

| Paquete | Versión | Rol |
|---|---|---|
| `react` | ^19.2.4 | UI framework |
| `react-dom` | ^19.2.4 | Renderizado DOM |
| `recharts` | ^3.8.1 | Gráficos de líneas (income/outcome, profit margin) |
| `lucide-react` | ^1.8.0 | Iconos vectoriales (TrendingUp, TrendingDown, DollarSign, BarChart2, LayoutDashboard) |
| `class-variance-authority` | ^0.7.1 | Variantes de componentes (shadcn/ui) |
| `clsx` | ^2.1.1 | Classnames condicionales |
| `tailwind-merge` | ^3.5.0 | Merge inteligente de clases Tailwind |

### Frontend — Dependencias de Desarrollo

| Paquete | Versión | Rol |
|---|---|---|
| `typescript` | ~6.0.2 | Tipado estático |
| `vite` | ^8.0.4 | Bundler + dev server + proxy |
| `@vitejs/plugin-react` | ^6.0.1 | Plugin React para Vite |
| `@tailwindcss/vite` | ^4.2.2 | Plugin Tailwind para Vite |
| `vitest` | ^4.1.4 | Test runner unitario |
| `@vitest/coverage-v8` | ^4.1.4 | Cobertura de código |
| `eslint` | ^9.39.4 | Linter |
| `@eslint/js` | ^9.39.4 | Config base ESLint |
| `typescript-eslint` | ^8.58.0 | ESLint para TypeScript |
| `eslint-plugin-react-hooks` | ^7.0.1 | Reglas React Hooks |
| `eslint-plugin-react-refresh` | ^0.5.2 | Reglas React Refresh |
| `globals` | ^17.4.0 | Globals de ESLint |
| `@types/react` | ^19.2.14 | Tipos React |
| `@types/react-dom` | ^19.2.3 | Tipos ReactDOM |
| `@types/node` | ^24.12.2 | Tipos Node.js |
| `autoprefixer` | ^10.4.27 | Prefixes CSS (postcss) |
| `postcss` | ^8.5.9 | Procesador CSS |

---

## 🖧 Backend — Dependencias (requirements.txt)

| Paquete | Rol |
|---|---|
| `fastapi` | Framework web ASGI (rutas, validación Pydantic, OpenAPI automática) |
| `uvicorn[standard]` | Servidor ASGI + workers estándar |
| `debugpy` | Debugger remoto (puerto 5678) |
| `pytest` | Test runner |
| `pytest-cov` | Cobertura de tests |
| `httpx` | HTTP client para tests asíncronos (TestClient de FastAPI) |

### Backend — Uso en código

- **Pydantic models:** `FinancialMovement`, `MetricsFacets`, `MetricsSummaryItem`,
  `TopCategoryItem`, `MetricsComparison`, `MetricsAlert` — todos definidos
  con `BaseModel` en `routes.py`.
- **Tipos literales:** `OperationType`, `Category`, `BusinessType`, `GroupBy`
  definidos con `typing.Literal`.
- **Sin base de datos:** todos los datos son generados en memoria con
  `random.seed(42)`.

---

## 🧪 Testing Stack

| Capa | Herramienta | Configuración |
|---|---|---|
| Backend | `pytest` + `httpx.TestClient` | Sin configuración extra. TestClient apunta a `app.main:app`. |
| Frontend | `vitest` + `@vitest/coverage-v8` | Sin archivo de config dedicado; usa defaults de Vite. |

---

## 🐳 Infraestructura / Tooling

### Docker

| Servicio | Imagen Base | Puerto Expuesto | Comando |
|---|---|---|---|
| `frontend` | `node:24-alpine` | 5173 | `npm run dev -- --host 0.0.0.0 --port 5173` |
| `backend` | `python:3.13-slim` | 8000 (app) + 5678 (debug) | `debugpy -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload` |

- **Volúmenes montados:** ambos servicios montan su directorio de código en
  modo bind para hot-reload.
- **Dependencia:** `frontend` tiene `depends_on: backend`.
- **debugpy:** el backend inicia con debugger remoto escuchando en `0.0.0.0:5678`,
  lo que permite attach desde VS Code o similar.

### ESLint — Configuración

- Archivo: `frontend/eslint.config.js` (formato flat config).
- Plugins precargados: `typescript-eslint`, `react-hooks`, `react-refresh`,
  `globals`.
- Script: `npm run lint` → `eslint .`

### TypeScript — Configuración

| Archivo | Propósito |
|---|---|
| `tsconfig.json` | Config raíz (project references) |
| `tsconfig.app.json` | Config para código fuente (`src/`) |
| `tsconfig.node.json` | Config para Node (Vite config) |

### Vite — Configuración

- **Plugins:** `@vitejs/plugin-react`, `@tailwindcss/vite`
- **Proxy:** `/api` → `http://backend:8000`
- **Alias:** `@` → `./src`
- **Host:** `0.0.0.0` (accesible desde Docker)

---

## 📐 Dependencias Clave entre Capas

```
Frontend (React)                    Backend (FastAPI)
     │                                     │
     │  Contrato de tipos                  │
     │  ──────────────────────────          │
     │  FinancialMovement {                 │  FinancialMovement (Pydantic)
     │    create_date: string (ISO date)    │    create_date: date
     │    amount: number                    │    amount: float
     │    operation_type: 'income'/'outcome'│    operation_type: OperationType
     │    category: Category                │    category: Category
     │    business_type: 'B2B'/'B2C'       │    business_type: BusinessType
     │  }                                   │  }
     │                                     │
     │  El frontend solo consume:          │  El backend expone 9 endpoints
     │  GET /api/metrics                    │
     │  y procesa los datos localmente      │
```

**Nota importante:** los tipos están definidos por separado en frontend
(`financial-types.ts`) y backend (modelos Pydantic en `routes.py`).
No hay un contrato compartido automáticamente — depende de consistencia manual.

---

## 🔍 Observaciones del Stack

1. **Sin base de datos:** todo el stack es liviano, sin persistencia.
   Ideal para propósitos educativos/demo.
2. **Sin estado compartido:** el frontend es una SPA sin estado global
   (Redux, Zustand, Context API no se usan).
3. **Sin router:** no hay `react-router`, es una sola página.
4. **Sin ORM:** backend opera solo con listas en memoria.
5. **Debug activo en producción dev:** debugpy escucha en `0.0.0.0:5678`
   dentro del contenedor — adecuado para desarrollo, no para despliegue real.
6. **Tailwind v4 con el nuevo plugin Vite:** usa `@tailwindcss/vite`
   en lugar del clásico PostCSS.
7. **TypeScript 6.0:** versión muy reciente del compilador.
8. **shadcn/ui:** se usan los componentes `Card` y `Skeleton` con el
   patrón de slots `cn()`.
