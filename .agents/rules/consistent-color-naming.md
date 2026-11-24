---
applyTo: '**'
---

# Mantener un solo sistema de naming para colores de gráficos

**Hecho**: `frontend/src/index.css` define `--chart-income`, `--chart-outcome` (semántico) y `--chart-1`, `--chart-2` (genérico), con valores duplicados. Si se actualiza `--chart-income` pero no `--chart-1`, los colores se desfasan.

**Regla**: Elegir un solo sistema de naming. Si se usan nombres semánticos (`--chart-income`, `--chart-outcome`), eliminar los nombres genéricos (`--chart-1`), o viceversa. No mantener ambos.
