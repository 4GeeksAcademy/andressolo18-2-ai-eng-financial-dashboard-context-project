# Current Status — Financial Metrics Dashboard

> Estado actual del proyecto basado en evidencia verificable del código fuente,
> tests, configuración y documentación.

---

## ✅ Qué Funciona

### Frontend — Componentes Visuales

| Funcionalidad | Evidencia |
|---|---|
| Dashboard con 4 KPIs responsivos | `kpi-row.tsx` grid `1→2→4 columnas`; `kpi-card.tsx` muestra label, valor con formato, helper text, icono y color por variante |
| Gráfico Income vs Outcome (mensual) | `income-outcome-chart.tsx` con Recharts `LineChart`, tooltip personalizado, leyenda, grid, 12 puntos de datos |
| Gráfico Profit Margin % (mensual) | `profit-percent-chart.tsx` con `ReferenceLine` en y=0, tooltip personalizado |
| Loading states (skeleton) | `skeleton.tsx` + `loading` prop en todos los componentes del dashboard |
| Manejo de error (API caída) | `App.tsx` catch → muestra `<div>` con mensaje de error y estilo `destructive` |
| Estado "sin datos" en gráficos | Ambos charts verifican `data.some(...)` y muestran "No data available to display" si está vacío |
| Responsive design | Tailwind breakpoints: `sm:` (2 columnas KPIs), `xl:` (4 columnas KPIs y 2 gráficos lado a lado) |

### Frontend — Lógica de Negocio

| Funcionalidad | Evidencia |
|---|---|
| `computeKPIs()` — 4 métricas desde movimientos | `financial-utils.ts` líneas 20-31 |
| `computeMonthlyData()` — agregación mensual cronológica | `financial-utils.ts` líneas 33-57 |
| `formatCurrency()` — formato USD sin decimales | `financial-utils.ts` líneas 59-65 |
| `formatPercent()` — porcentaje con 1 decimal | `financial-utils.ts` líneas 67-69 |
| Mock data estática (58 movs, año 2024) | `mock-data.ts` — respaldo si no hay backend |

### Backend — API REST

| Funcionalidad | Evidencia |
|---|---|
| `/health` → `{"status": "ok"}` | `routes.py` línea 184-185 |
| `/api/metrics` — 360 movs con seed=42, filtrable | `routes.py` líneas 188-199 |
| `/api/metrics/facets` — tipos, categorías, rango fechas | `routes.py` líneas 201-204, test verifica |
| `/api/metrics/summary` — agrupación day/week/month | `routes.py` líneas 206-223, test verifica |
| `/api/metrics/categories/top` — top N por tipo | `routes.py` líneas 225-238, test verifica |
| `/api/metrics/comparison` — delta absoluto y % | `routes.py` líneas 240-268, test verifica |
| `/api/metrics/alerts` — detección de anomalías | `routes.py` líneas 270-292, test verifica |
| `/api/metrics/b2b` — solo B2B | `routes.py` líneas 294-305, test verifica |
| `/api/metrics/b2c` — solo B2C | `routes.py` líneas 307-318, test verifica |
| CORS abierto (`allow_origins=["*"]`) | `main.py` línea 8 |
| Documentación OpenAPI automática | FastAPI genera `/docs` por defecto |

### Backend — Generación de Datos Mock

- Semilla fija `seed=42` → datos deterministas (mismos datos en cada ejecución).
- 30 movimientos por mes × 12 meses = 360 movimientos.
- Distribución: ~55% B2B, ~45% B2C.
- Income: 90% categoría `sales`, 10% `others`.
- Outcome: equitativo entre `suppliers`, `operational`, `administrative`, `others`.
- Función `ensure_chronological_order()` garantiza orden por fecha.

### Tests

| Suite | Estado esperado | Cantidad |
|---|---|---|
| `pytest test_routes.py` | ✅ Todos pasan | 16 tests |
| `vitest run` (financial-utils.test.ts) | ✅ Todos pasan | 4 tests |
| **Total** | ✅ **20 tests** | |

---

## ⚠️ Gaps Conocidos (evidenciados en código)

### 🔴 Brechas Funcionales

| Gap | Detalle | Evidencia |
|---|---|---|
| **Solo 1 de 9 endpoints usado** | El frontend consume únicamente `GET /api/metrics`. Los otros 8 endpoints (`/summary`, `/comparison`, `/alerts`, `/top`, `/facets`, `/b2b`, `/b2c`) existen pero no tienen UI. | `App.tsx` solo llama a `${API_BASE_URL}/api/metrics` |
| **Sin filtros en UI** | El backend soporta filtros por fecha, categoría, tipo de operación y tipo de negocio, pero el frontend no expone ningún control de filtro. | `App.tsx` no tiene inputs, selects ni date pickers |
| **Sin interactividad en gráficos** | Los gráficos son estáticos (solo tooltips). No hay drill-down, zoom, ni selección de rango. | `income-outcome-chart.tsx` y `profit-percent-chart.tsx` son solo lectura |
| **Período fijo "2024 — Full Year"** | El header muestra un badge estático con el período, sin controles para cambiar año o rango. | `dashboard-header.tsx` prop `period` con default fijo |
| **Mock data duplicada** | Existe `mock-data.ts` (58 movs estáticos) que no se usa en producción; el frontend siempre llama a la API. Podría servir como fallback offline. | `App.tsx` no importa `mock-data.ts` |
| **Sin estado global** | No hay Context API, Redux ni Zustand. Cada llamada re-renderiza todo el árbol. | No hay `useContext`, `Provider` ni store |
| **Sin actualización periódica** | Los datos se cargan una sola vez al montar el componente. No hay polling, WebSocket ni refetch. | `useEffect([], [])` sin intervalo ni eventos |

### 🟡 Brechas Técnicas

| Gap | Detalle | Evidencia |
|---|---|---|
| **Tests frontend limitados** | Solo 4 tests unitarios de utilidades. No hay tests de componentes (sin Testing Library, sin react-test-renderer). | Solo existe `financial-utils.test.ts` |
| **Sin tests de integración E2E** | No hay Cypress, Playwright ni similar. | No hay archivos de configuración ni dependencias E2E |
| **Sin validación de tipos compartida** | Los tipos del frontend (`financial-types.ts`) y backend (Pydantic en `routes.py`) están duplicados manualmente. Un cambio en uno no alerta al otro. | No hay OpenAPI client generation ni schema compartido |
| **Debugpy expuesto en producción** | El backend corre debugpy en `0.0.0.0:5678` dentro del contenedor. Es un riesgo de seguridad si se despliega. | `backend/Dockerfile` línea `CMD [...] "debugpy" "--listen" "0.0.0.0:5678"` |
| **Sin variables de entorno para entorno** | No hay diferenciación entre dev/staging/production. `.env.example` existe pero no se usa en el flujo principal. | `App.tsx` usa `import.meta.env.VITE_API_BASE_URL ?? ""` |
| **Sin CI/CD pipeline** | No hay GitHub Actions, workflow de tests automáticos ni despliegue. | No hay directorio `.github/workflows/` |

### 🟢 Brechas de Calidad de Código

| Gap | Detalle | Evidencia |
|---|---|---|
| **Sin límite de resultados en API** | `GET /api/metrics` devuelve todos los 360 movs siempre. No hay paginación. | `routes.py` endpoint sin parámetros `limit`/`offset` |
| **Side effect en generación mock** | `generate_mock_movements()` llama a `random.seed(seed)` que **resetea el estado global de random**. Si otro código usara random en la misma request, se vería afectado. | `routes.py` línea 70: `random.seed(seed)` |
| **HelperText hardcodeado en inglés** | Los textos de ayuda en `kpi-card.tsx` están en inglés, mientras que el mensaje de error en `App.tsx` está en español. Inconsistencia de i18n. | Comparar `kpi-row.tsx` (inglés) vs `App.tsx` (español) |
| **Sin TypeScript strict mode** | tsconfig no forza `strict: true` — se pierden chequeos de null safety, tipos implícitos, etc. | Verificar `tsconfig*.json` |

---

## 🎯 Siguientes Prioridades (recomendadas)

### Prioridad Alta — Integración Frontend-Backend

1. **Consumir endpoints no utilizados**:
   - `GET /api/metrics/summary` para datos agrupados directamente desde backend.
   - `GET /api/metrics/facets` para poblar dinámicamente filtros en UI.
   - `GET /api/metrics/alerts` para mostrar alertas de anomalías.
   - `GET /api/metrics/comparison` para mostrar variación vs período anterior.

2. **Agregar filtros en la UI**:
   - Selector de período (date range picker).
   - Selector de categoría y tipo de operación.
   - Selector B2B / B2C / Todos.
   - Añadir query params a la llamada `/api/metrics`.

3. **Mejorar experiencia de carga**:
   - Agregar polling periódico (ej. cada 60s) o refetch manual (botón "Actualizar").
   - Mostrar indicador de cuándo fue la última actualización.

### Prioridad Media — Calidad y Testing

4. **Tests de componentes frontend**:
   - Agregar `@testing-library/react` y `vitest-dom`.
   - Tests para `KPICard`, `IncomeOutcomeChart`, manejo de errores y loading.

5. **Tests de integración frontend-backend**:
   - Test que verifique el contrato de tipos entre frontend y backend.
   - Test de snapshot de la API response.

6. **Pipeline CI**:
   - Crear GitHub Actions que ejecute `pytest` y `npm test` en cada PR.
   - Agregar lint check automático.

### Prioridad Baja — Refinamiento

7. **Internacionalización consistente**:
   - Unificar idioma (inglés, dado el público del proyecto) o agregar i18n.

8. **Paginación en API**:
   - Agregar `limit` y `offset` a endpoints que devuelven listas.

9. **Desacoplar `random.seed()`**:
   - Usar instancia local `random.Random(seed)` en lugar de `random.seed()`
     global para evitar side effects.

10. **Strict mode TypeScript**:
    - Habilitar `"strict": true` en `tsconfig.app.json` y corregir errores.

11. **Quitar debugpy del Dockerfile de producción**:
    - Usar `CMD` condicional o multi-stage build para separar dev de prod.
