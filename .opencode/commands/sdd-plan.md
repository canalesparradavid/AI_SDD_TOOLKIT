---
description: Genera el plan de implementacion SDD con IMP por fases y cobertura REQ/AC al 100%
agent: sdd
---

Ejecuta la fase de planificacion SDD.

NO modifiques ni crees codigo en esta fase: los unicos ficheros que se tocan son
los de `specs/`.

Precondicion: requisitos con estado `Approved`. Si no los hay, indicalo y DETENTE
(vuelve a `/sdd-requirements`).

Procedimiento obligatorio:

1. Lee TODOS los ficheros de `specs/requirements/` directamente. No te fies de los
   resumenes anteriores ni de la conversacion.
2. Genera `specs/plans/implementation-plan.md` con el formato de
   `.opencode/templates/sdd/implementation-plan-template.md`:
   - Cabecera: `Aprobado: pendiente` (pasa a `Aprobado` con fecha cuando el usuario
     apruebe el plan en el GATE). Si regeneras un plan ya aprobado, informa de que
     pierde la aprobacion y vuelve a pasar por el GATE.
   - Tareas `IMP-001`, `IMP-002`, ... cada una con: ID, descripcion, REQ relacionados,
      AC relacionados, dependencias, componentes previstos, tests previstos, estado.
      Estados (inicial `Pending`): `Pending`, `InProgress`, `Implemented`, `Validated`, `Blocked`.
   - Fases coherentes (no un bloque gigante), cada una con ID estable (`FASE-01`,
     `FASE-02`, ...) citable tal cual en `/sdd-implement` y `/sdd-validate`, y campo
     `Aprobado: pendiente` (lo aprueba el usuario al inicio de la fase, en
     `/sdd-implement`, antes de tocar codigo). Por cada fase: objetivo, IMP
     incluidos, REQ afectados, AC afectados,
     dependencias, condiciones de finalizacion.
3. Genera o actualiza `specs/architecture/architecture.md` con la estructura que el
   plan necesita: componentes, interfaces entre ellos, flujo de datos y tecnologia,
   coherente con los IMP del plan y con los ADR de `specs/decisions/`. Si no hay
   nada estructural que documentar, DILO en vez de crear el fichero vacio.
4. SEGUNDA PASADA INDEPENDIENTE de cobertura, partiendo otra vez de
   `specs/requirements/`:
   - cada REQ esta incluido en el plan,
   - cada AC de cada REQ esta asociado como minimo a un IMP,
   - existe estrategia de validacion para cada AC tecnicamente verificable,
   - las dependencias estan contempladas.
   Si algun REQ carece de AC suficientemente verificables: DETENTE, senala cual y
   pregunta al usuario. No inventes AC.
5. Calcula Requirement Coverage y Acceptance Criteria Coverage sobre los REQ
   vigentes (estados `Approved`, `Implemented` o `Validated`; excluye `Draft`,
   `Pending` y `Obsolete`).
6. Crea o actualiza `specs/validation/traceability.md` con el formato de
   `.opencode/templates/sdd/traceability-template.md` y la cadena
   `REQ -> AC -> IMP -> CODE -> TEST -> VALIDATION` para todos los REQ y AC del
   plan. Formato: una fila por AC con columnas `REQ | AC | IMP | CODE | TEST |
   VALIDATION`. Si el fichero no existe, crealo ahora (la fase de validacion lo
   rellenara con evidencia).

GATE DE PLAN:

7. Presenta: plan resumido por fases, arquitectura resumida de
   `specs/architecture/architecture.md`, cobertura REQ, cobertura AC, huecos si los
   hay, riesgos.
8. Si la cobertura NO es 100% en REQ y AC: declaralo y NO permitas avanzar a
   implementacion.
9. DETENTE y espera aprobacion explicita antes de cualquier `/sdd-implement`.
