---
applyTo: '**'
---

# Los componentes interactivos deben tener atributos de accesibilidad

**Hecho**: En `frontend/src/components/dashboard/`, las secciones tienen `aria-label` pero los gráficos Recharts, tooltips y leyendas carecen de atributos ARIA.

**Regla**: Todo componente interactivo o informativo debe incluir `aria-label` o `aria-describedby`. Los gráficos deben definir un `title` accesible o usar `role="img"` con descripción.
