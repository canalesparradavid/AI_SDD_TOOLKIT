# Implementation Plan

- Aprobado: pendiente (fecha: -)
  <pasa a `Aprobado (<YYYY-MM-DD>)` cuando el usuario apruebe el plan en el GATE;
  regenerar un plan aprobado lo devuelve a `pendiente` y obliga a re-aprobarlo>

## Fases

### FASE-01

- Aprobado: pendiente (fecha: -)
  <lo aprueba el usuario al inicio de la fase, en `/sdd-implement`, antes de
  tocar codigo>
- Objetivo: <que se consigue al terminar la fase>
- IMP incluidos: IMP-001, ...
- REQ afectados: REQ-XXX, ...
- AC afectados: AC-XXX-NN, ...
- Dependencias: <otras fases o "ninguna">
- Condiciones de finalizacion: <criterios de cierre de la fase>

### FASE-02

<repetir estructura>

## Tareas (IMP)

| ID | Descripcion | REQ | AC | Dependencias | Componentes | Tests | Estado |
|---|---|---|---|---|---|---|---|
| IMP-001 | ... | REQ-XXX | AC-XXX-01 | ... | ... | ... | Pending |

Estados de IMP: `Pending`, `InProgress`, `Implemented`, `Validated`, `Blocked`.
`Validated` solo lo asigna `/sdd-validate` con evidencia.
