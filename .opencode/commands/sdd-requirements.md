---
description: Toma iterativa de requisitos SDD - detecta, pregunta en bloques y espera aprobacion
agent: sdd
---

Ejecuta la fase de toma de requisitos SDD.

Si ya existen requisitos `Approved` y hay plan generado: informa de que el alcance
esta cerrado y que toda alta o modificacion de requisitos se gestiona con
`/sdd-change` (impacto, re-aprobacion, invalidacion de certificado). Solo continua
aqui si el usuario confirma expresamente reabrir la toma de requisitos.

Procedimiento obligatorio:

1. Analiza TODA la informacion existente antes de preguntar nada: `specs/` completo
   (requirements, decisions, architecture, changes, planes, validacion),
   documentacion del proyecto y codigo relevante si lo hay.
2. Detecta y clasifica:
   - requisitos ya conocidos o deducibles de lo existente,
   - decisiones ya tomadas (ADRs u otras fuentes),
   - limitaciones, seguridad, comportamiento operativo y manejo de errores no
     especificados,
   - ambiguedades,
   - contradicciones.
3. Pregunta unicamente lo que realmente falte. En bloques razonables de preguntas
   relacionadas (unas pocas por bloque). Despues ESPERA la respuesta del usuario
   antes de continuar con el siguiente bloque. No repitas preguntas ya respondidas.
4. Avanza iterativamente: por cada bloque contestado, actualiza los REQ afectados.
   Registra cada decision de arquitectura aprobada (o existente sin documentar) en
   `specs/decisions/ADR-XXX-nombre.md` con el formato de
   `.opencode/templates/sdd/ADR-template.md` (contexto, decision y consecuencias).
5. Almacena cada requisito importante en `specs/requirements/REQ-XXX-nombre.md`
   con el formato de `.opencode/templates/sdd/REQ-template.md` y como minimo estos
   campos:
   - ID, nombre, estado, prioridad (P0/P1/P2), necesidad, requisito, motivacion,
     comportamiento esperado, Acceptance Criteria, dependencias, decisiones
     pendientes, origen.
   Estados: `Draft`, `Pending`, `Approved`, `Implemented`, `Validated`, `Obsolete`.
6. Cada AC lleva ID unico (`AC-XXX-NN`) y debe ser verificable objetivamente. Si un
   criterio no es verificable: NO inventes otro - indica el problema y pregunta.
7. No crees un REQ distinto por cada frase trivial: un REQ = una capacidad coherente.
8. Nunca presentes tu recomendacion como una decision aprobada: mientras el usuario
   no lo confirme explicitamente, es PROPUESTA, no DECISION APROBADA.

GATE DE REQUISITOS - cuando la toma este completa:

9. Presenta: requisitos (resumen, con cada REQ presentado en estado `Pending`),
   decisiones tomadas, decisiones pendientes, riesgos, posibles contradicciones,
   alcance del proyecto/MVP.
10. DETENTE y espera aprobacion explicita. Tras la aprobacion, marca los REQ
    presentados como `Approved`. No pases a planificacion ni a `/sdd-plan` sin esa
    aprobacion.
