---
applyTo: '**'
---

# Usar tipos explícitos en modelos Pydantic

**Hecho**: `backend/app/routes.py` declara modelos Pydantic con anotaciones de tipo explícitas, por ejemplo `amount: float`, `create_date: date`. No se usan `Any` ni `dict` genérico.

**Regla**: Todos los modelos Pydantic en el backend deben declarar tipos explícitos para cada campo. Prohibido usar `Any`, `dict` genérico o tipos sin anotación en modelos de datos.
