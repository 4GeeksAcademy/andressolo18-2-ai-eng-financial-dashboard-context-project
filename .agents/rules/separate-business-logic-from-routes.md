---
applyTo: '**'
---

# Extraer lógica de negocio a módulos separados de las rutas

**Hecho**: `backend/app/routes.py` contiene 7 modelos Pydantic, 10 funciones de negocio (`summarize_movements`, `detect_outcome_alerts`, `build_top_categories`, etc.) y 9 endpoints en un solo archivo de ~400 líneas.

**Regla**: Las funciones de negocio deben residir en `backend/app/services/` o `backend/app/metrics_service.py`. `routes.py` solo debe importar y llamar esas funciones.
