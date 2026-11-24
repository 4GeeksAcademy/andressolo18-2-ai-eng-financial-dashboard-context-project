---
applyTo: '**'
---

# Usar `@apply` solo en `@layer base` (Tailwind 4)

**Hecho**: Los componentes del dashboard usan clases utilitarias directamente en JSX (ej. `kpi-card.tsx`, `income-outcome-chart.tsx`). Esto es correcto y debe mantenerse como práctica principal.

**Regla**: Para estilos de componentes, preferir clases utilitarias en JSX. Usar `@apply` únicamente dentro de `@layer base` para estilos globales o tokens de diseño, no para componentes individuales.
