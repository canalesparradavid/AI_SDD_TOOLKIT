---
description: Estado compacto del SDD - requisitos, AC, implementacion, tests, decisiones pendientes
agent: sdd
---

Muestra el estado actual del SDD del proyecto.

Procedimiento:

1. Si `specs/` no existe o esta vacia: informa de que falta `/sdd-init` y DETENTE.
   En otro caso lee `specs/requirements/` (todos los REQ y sus estados).
2. Lee `specs/plans/implementation-plan.md` (IMPs, fases, estados).
3. Lee `specs/validation/traceability.md`, `specs/validation/acceptance.md` y
   `specs/validation/final-validation.md` si existen (cumplimiento y fecha de la
   ultima auditoria). Si la ultima auditoria esta marcada como invalidada por un
   cambio, reportala como invalidada: no la des por valida.
4. Cuenta los AC totales y validados (excluye los REQ `Obsolete` de los
   recuentos).
5. Detecta decisiones pendientes (REQ con `decisiones pendientes` no vacias) y
   gates sin aprobar leyendo la cabecera y las fases de
   `specs/plans/implementation-plan.md`, los estados de los REQ y los ficheros
   `specs/changes/CHANGE-*.md` cuyo `Aprobacion` este `pendiente`.
6. Ejecuta o recoge el resultado de los tests del proyecto si es factible; si no,
   indicalo.
7. NO infieras PASS de estado obsoleto del plan ni de resumenes anteriores: sin
   evidencia verificable el dato no vale. Marca explicitamente como `desconocido`
   cualquier informacion que no puedas verificar.

Presenta una salida compacta, por ejemplo:

```
SDD STATUS

Requirements:           10/12 approved
Acceptance Criteria:    47 total / 31 validated
Implementation:         66%  (current phase: FASE-04)
IMP:                    21 Validated, 4 Implemented, 13 Pending, 0 Blocked
Tests:                  82 passed, 0 failed
Pending decisions:      0
Pending changes:        0
Blockers:               <ninguno o lista>
Next phase:             FASE-XX
SDD compliant:          NO (aun no hay /sdd-audit final)
```

Prioriza claridad, brevedad y utilidad. DETENTE al mostrar el estado.
