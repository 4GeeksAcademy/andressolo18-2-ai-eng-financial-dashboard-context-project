---
applyTo: '**'
---

# Usar fixtures de pytest en lugar de regenerar datos en cada test

**Hecho**: `backend/tests/conftest.py` solo modifica `sys.path`. Cada test en `test_routes.py` importa y llama `generate_mock_movements(seed=42)` manualmente (ej: línea 14, 20, 43).

**Regla**: Crear un fixture `@pytest.fixture` que retorne `generate_mock_movements(seed=42)` y usarlo en todos los tests que necesiten datos mock.
