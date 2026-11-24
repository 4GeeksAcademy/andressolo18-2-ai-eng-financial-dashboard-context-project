---
applyTo: '**'
---

# Incluir herramientas de linting y formateo en el backend

**Hecho**: `backend/requirements.txt` tiene solo dependencias funcionales (fastapi, uvicorn, pytest). No incluye `ruff`, `mypy` ni `black`. El frontend sí tiene ESLint configurado.

**Regla**: Agregar `ruff` (linting + formateo) y `mypy` (type checking) a `requirements.txt` y crear un `pyproject.toml` con la configuración correspondiente.
