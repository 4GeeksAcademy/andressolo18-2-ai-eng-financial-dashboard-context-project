---
applyTo: '**'
---

# No duplicar lógica de negocio entre backend y frontend

**Hecho**: `frontend/src/App.tsx` llama a `GET /api/metrics` y recalcula KPIs localmente con `computeKPIs()` y `computeMonthlyData()`, mientras que `backend/app/routes.py` tiene endpoints como `/api/metrics/summary` y `/api/metrics/comparison` que ya realizan esos cálculos.

**Regla**: Antes de implementar un cálculo financiero en el frontend, verificar si el backend ya expone un endpoint que lo resuelva. Si no existe, crearlo en el backend primero y consumirlo desde el frontend.
