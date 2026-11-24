---
applyTo: '**'
---

# Cachear datos mock o inyectar dependencia de datos

**Hecho**: Cada uno de los 9 endpoints en `backend/app/routes.py` ejecuta `generate_mock_movements(seed=42)`, regenerando 360 objetos por cada request. No hay caché.

**Regla**: Usar `functools.lru_cache` sobre `generate_mock_movements()` o, si se introduce una base de datos, crear un `Depends()` de FastAPI que inyecte el repositorio de datos en cada endpoint.
