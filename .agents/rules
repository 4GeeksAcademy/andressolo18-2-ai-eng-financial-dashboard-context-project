---
applyTo: '**'
---

# Reglas Propuestas para el Proyecto

Cada regla cita al menos un hecho concreto del repositorio.

---

## 1. No duplicar lógica de negocio entre backend y frontend

**Hecho**: `frontend/src/App.tsx` llama a `GET /api/metrics` y recalcula KPIs localmente con `computeKPIs()` y `computeMonthlyData()`, mientras que `backend/app/routes.py` tiene endpoints como `/api/metrics/summary` y `/api/metrics/comparison` que ya realizan esos cálculos.

**Regla**: Antes de implementar un cálculo financiero en el frontend, verificar si el backend ya expone un endpoint que lo resuelva. Si no existe, crearlo en el backend primero y consumirlo desde el frontend.

---

## 2. Extraer lógica de negocio a módulos separados de las rutas

**Hecho**: `backend/app/routes.py` contiene 7 modelos Pydantic, 10 funciones de negocio (`summarize_movements`, `detect_outcome_alerts`, `build_top_categories`, etc.) y 9 endpoints en un solo archivo de ~400 líneas.

**Regla**: Las funciones de negocio deben residir en `backend/app/services/` o `backend/app/metrics_service.py`. `routes.py` solo debe importar y llamar esas funciones.

---

## 3. Cachear datos mock o inyectar dependencia de datos

**Hecho**: Cada uno de los 9 endpoints en `backend/app/routes.py` ejecuta `generate_mock_movements(seed=42)`, regenerando 360 objetos por cada request. No hay caché.

**Regla**: Usar `functools.lru_cache` sobre `generate_mock_movements()` o, si se introduce una base de datos, crear un `Depends()` de FastAPI que inyecte el repositorio de datos en cada endpoint.

---

## 4. El período del dashboard debe ser dinámico, no hardcodeado

**Hecho**: `frontend/src/App.tsx` pasa `period="2024 - Full Year"` a `<DashboardHeader>`, pero `backend/app/routes.py` genera datos con `date.today()`, por lo que los datos serán del año en curso mientras el título siempre dice "2024".

**Regla**: El período debe obtenerse del endpoint `GET /api/metrics/facets` y su respuesta `{ min_date, max_date }`, o calcularse a partir de los datos recibidos. No hardcodear años ni períodos.

---

## 5. Usar fixtures de pytest en lugar de regenerar datos en cada test

**Hecho**: `backend/tests/conftest.py` solo modifica `sys.path`. Cada test en `test_routes.py` importa y llama `generate_mock_movements(seed=42)` manualmente (ej: línea 14, 20, 43).

**Regla**: Crear un fixture `@pytest.fixture` que retorne `generate_mock_movements(seed=42)` y usarlo en todos los tests que necesiten datos mock.

---

## 6. Los tests no deben depender de valores numéricos exactos sin semántica

**Hecho**: `backend/tests/test_routes.py` (línea 14) verifica `len(movements) == 360`, que asume 30 días × 12 meses. Si alguien cambia el rango de días en `_build_movement`, el test falla sin relación semántica.

**Regla**: Usar verificaciones semánticas: validar que existan movimientos para cada mes del año en lugar de contar un número exacto.

---

## 7. No usar hostname de Docker en configuraciones fuera de Docker

**Hecho**: `frontend/vite.config.ts` define `proxy: { "/api": { target: "http://backend:8000" } }`. El hostname `backend` solo resuelve dentro de Docker.

**Regla**: En configuraciones de desarrollo local fuera de Docker, usar `localhost` o permitir sobrescribir el target vía variable de entorno. La variable `VITE_API_BASE_URL` de `.env.example` debe respetarse en el código.

---

## 8. Mantener un solo sistema de naming para colores de gráficos

**Hecho**: `frontend/src/index.css` define `--chart-income`, `--chart-outcome` (semántico) y `--chart-1`, `--chart-2` (genérico), con valores duplicados. Si se actualiza `--chart-income` pero no `--chart-1`, los colores se desfasan.

**Regla**: Elegir un solo sistema de naming. Si se usan nombres semánticos (`--chart-income`, `--chart-outcome`), eliminar los nombres genéricos (`--chart-1`), o viceversa. No mantener ambos.

---

## 9. Incluir herramientas de linting y formateo en el backend

**Hecho**: `backend/requirements.txt` tiene solo dependencias funcionales (fastapi, uvicorn, pytest). No incluye `ruff`, `mypy` ni `black`. El frontend sí tiene ESLint configurado.

**Regla**: Agregar `ruff` (linting + formateo) y `mypy` (type checking) a `requirements.txt` y crear un `pyproject.toml` con la configuración correspondiente.

---

## 10. Los componentes interactivos deben tener atributos de accesibilidad

**Hecho**: En `frontend/src/components/dashboard/`, las secciones tienen `aria-label` pero los gráficos Recharts, tooltips y leyendas carecen de atributos ARIA.

**Regla**: Todo componente interactivo o informativo debe incluir `aria-label` o `aria-describedby`. Los gráficos deben definir un `title` accesible o usar `role="img"` con descripción.

---

## Reglas de estilo y formato propuestas

- **Regla A**: Usar `@apply` solo en `@layer base` (Tailwind 4). Para estilos de componentes, preferir clases utilitarias en JSX.
- **Regla B**: Los nombres de archivo en `frontend/src/lib/` usan kebab-case (`financial-utils.ts`). Los componentes en `frontend/src/components/` usan kebab-case también (`kpi-card.tsx`). Mantener esta convención.
- **Regla C**: Los modelos Pydantic en el backend se declaran con tipos explícitos (`amount: float`, `create_date: date`). No usar `Any` ni `dict` genérico.