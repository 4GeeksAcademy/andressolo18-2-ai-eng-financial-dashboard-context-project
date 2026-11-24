---
applyTo: '**'
---

# No usar hostname de Docker en configuraciones fuera de Docker

**Hecho**: `frontend/vite.config.ts` define `proxy: { "/api": { target: "http://backend:8000" } }`. El hostname `backend` solo resuelve dentro de Docker.

**Regla**: En configuraciones de desarrollo local fuera de Docker, usar `localhost` o permitir sobrescribir el target vía variable de entorno. La variable `VITE_API_BASE_URL` de `.env.example` debe respetarse en el código.
