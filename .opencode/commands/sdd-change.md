---
description: Analiza el impacto de un cambio sobre el SDD - nunca toca el codigo primero
agent: sdd
---

Cambio solicitado por el usuario: `$ARGUMENTS`.

Si falta `$ARGUMENTS`: pide la descripcion del cambio y DETENTE.

REGLA: NUNCA modifiques primero el codigo. El flujo correcto es:
Cambio solicitado -> analizar impacto -> actualizar SDD -> aprobacion -> actualizar
plan -> implementacion -> tests -> validacion.
NUNCA: Cambio -> codigo -> adaptar SDD posteriormente.

Procedimiento:

1. Analiza el cambio y su impacto en:
   - CHANGE (descripcion clara del cambio),
   - REQ afectados (existentes que cambian, nuevos que se necesitan, obsoletos),
   - AC afectados (que criterios cambian, se anaden o se eliminan),
   - ADR afectados (decisiones de arquitectura que se invalidan o actualizan),
   - IMP afectados (del plan actual),
   - `specs/architecture/architecture.md` (si el cambio altera componentes,
     interfaces o flujo de datos),
   - codigo potencialmente afectado,
   - tests afectados.
2. Crea `specs/changes/CHANGE-XXX-nombre.md` con el formato de
   `.opencode/templates/sdd/CHANGE-template.md` (ID incremental `CHANGE-001`,
   `CHANGE-002`, ...; crea la carpeta si no existe) y registra ahi: fecha,
   descripcion del cambio, impacto en tres bloques (CONFIRMADO: consecuencias
   que se derivan con certeza de lo existente, con lista concreta por cada area,
   riesgos, orden de aplicacion previsto y si altera requisitos aprobados;
   PROPUESTA: acciones recomendadas pero no decididas; DESCONOCIDO: efectos que
   no se pueden determinar sin mas informacion), elementos afectados
   (REQ/AC/ADR/IMP/fases), y `Aprobacion: pendiente`.
3. Presenta ese impacto al usuario y DETENTE esperando aprobacion explicita.
   Tras la aprobacion, anade la fecha en `Aprobacion` del fichero CHANGE.

Solo tras aprobacion explicita:

4. Actualiza los SDD afectados (REQ/AC/ADR con su trazabilidad; nunca reescribas un
   requisito aprobado para encajar con codigo existente - el requisito manda).
   Regresa estados: cada REQ alterado que estuviera `Implemented` o `Validated`
   pasa a `Approved` (cambio aprobado, pendiente de re-implementar), y cada IMP
   invalidado por el cambio vuelve a `Pending`. Los REQ nuevos que cree el cambio
   nacen en `Approved` (cubiertos por la aprobacion del paso 3) con origen
   `CHANGE-XXX` (el ID de su ficha en `specs/changes/`); no los dejes en `Draft`.
   Los REQ que el cambio deja fuera del alcance pasan a `Obsolete` (no se borran;
   se excluyen de cobertura). Alterar o eliminar un AC de un REQ cuenta como
   alterar ese REQ.
5. Actualiza `specs/plans/implementation-plan.md` (nuevos o modificados IMP, fases;
   los IMP eliminados se retiran del plan y de la trazabilidad dejando constancia
   en la ficha CHANGE; las fases cuyos IMP o AC cambien vuelven a `Aprobado:
   pendiente`; deja la cabecera en `Aprobado: pendiente` para la re-aprobacion del
   paso 7) y `specs/architecture/architecture.md` si el cambio lo altera.
6. Actualiza `specs/validation/traceability.md`. Si existe
   `specs/validation/final-validation.md`, anade en su cabecera que queda
   invalidado por este cambio (re-auditoria pendiente). En `acceptance.md`, los AC
   alterados pasan a `NOT_TESTED`: su evidencia previa pertenece a la
   especificacion anterior.
7. DETENTE y espera aprobacion del plan actualizado antes de implementar nada.
