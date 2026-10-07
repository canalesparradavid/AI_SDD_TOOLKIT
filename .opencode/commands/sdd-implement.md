---
description: Implementa una fase del plan SDD - REQ -> AC -> IMP -> CODE -> TEST -> VALIDATION, solo esa fase
agent: sdd
---

Ejecuta la implementacion de la fase indicada: `$1`.

Si falta `$1`: pide la fase y DETENTE. Si `$1` no coincide con ninguna fase del
plan: lista las fases existentes y DETENTE.

Precondiciones - antes de modificar NINGUN codigo:

1. Localiza la fase en `specs/plans/implementation-plan.md`. Si el plan no existe:
   informa de que falta `/sdd-plan` y DETENTE.
2. Identifica sus IMP, sus REQ y sus AC.
3. Relee los REQ originales completos de `specs/requirements/`.
4. Relee los AC asociados.
5. Comprueba en `specs/plans/implementation-plan.md` que la cabecera figure
   `Aprobado`; si esta `pendiente` y no conste aprobacion explicita del usuario:
   DETENTE y recuerda que falta aprobar el plan. Si la fase figura `pendiente`:
   presenta la fase (objetivo, IMP, REQ, AC) y DETENTE esperando aprobacion de esa
   fase; solo tras ella, marca `Aprobado` con fecha en la fase y continua.

Durante la implementacion:

6. Implementa EXCLUSIVAMENTE la fase indicada. No implementes fases posteriores
   planificadas, salvo dependencias tecnicas explicitamente contempladas en el plan.
7. Aplica la cadena: REQ -> AC -> IMP -> CODE -> TEST -> VALIDATION.
8. Marca los IMP en curso como `InProgress` en el plan.
9. AMBIGUEDAD: si aparece comportamiento no especificado, REQ incompleto, AC ambiguo,
   contradiccion o decision funcional necesaria: DETENTE. No modifiques el REQ. No
   decidas tu mismo. Pregunta al usuario y espera.

FIN DE FASE:

10. Ejecuta los tests relevantes y registra el resultado.
11. Comprueba cada AC de la fase con evidencia (no basta con existir codigo).
12. Marca los IMP de la fase como `Implemented`. NO marques `Validated`: ese estado
    lo asigna `/sdd-validate` con evidencia independiente. Cuando todos los IMP de un
    REQ esten `Implemented`, marca ese REQ como `Implemented`.
13. Actualiza `specs/validation/traceability.md`.
14. Presenta el resumen: IMP completados, REQ afectados, AC validados, tests
    creados/modificados, resultado de tests, archivos modificados, pendientes,
    bloqueos.
15. DETENTE. NO inicies la siguiente fase. Espera aprobacion explicita.
    El flujo correcto es: `/sdd-validate <fase>` y tras aprobar, la siguiente fase.
