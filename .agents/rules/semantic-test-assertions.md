---
applyTo: '**'
---

# Los tests no deben depender de valores numéricos exactos sin semántica

**Hecho**: `backend/tests/test_routes.py` (línea 14) verifica `len(movements) == 360`, que asume 30 días × 12 meses. Si alguien cambia el rango de días en `_build_movement`, el test falla sin relación semántica.

**Regla**: Usar verificaciones semánticas: validar que existan movimientos para cada mes del año en lugar de contar un número exacto.
