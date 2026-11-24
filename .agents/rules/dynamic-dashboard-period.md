---
applyTo: '**'
---

# El período del dashboard debe ser dinámico, no hardcodeado

**Hecho**: `frontend/src/App.tsx` pasa `period="2024 - Full Year"` a `<DashboardHeader>`, pero `backend/app/routes.py` genera datos con `date.today()`, por lo que los datos serán del año en curso mientras el título siempre dice "2024".

**Regla**: El período debe obtenerse del endpoint `GET /api/metrics/facets` y su respuesta `{ min_date, max_date }`, o calcularse a partir de los datos recibidos. No hardcodear años ni períodos.
