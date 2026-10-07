# SDD Toolkit para opencode - Documentacion del sistema

Toolkit de Spec-Driven Development (SDD) para [opencode](https://opencode.ai).
Fuente: carpeta `.opencode/` de este repositorio.

Contenido: que es cada concepto, los 3 agentes, los 8 comandos, las plantillas,
un flujo de ejemplo (solo prompts), el diagrama del ciclo, la matriz de gates,
convenciones, instalacion y FAQ.

---

## Indice

1. [Que es](#1-que-es)
2. [Glosario de conceptos](#2-glosario-de-conceptos)
3. [Estados y transiciones](#3-estados-y-transiciones)
4. [Arquitectura del sistema](#4-arquitectura-del-sistema)
5. [Comandos](#5-comandos)
6. [Plantillas](#6-plantillas)
7. [Flujo de llamadas (ejemplo, solo prompts)](#7-flujo-de-llamadas-ejemplo-solo-prompts)
8. [Diagrama de flujo](#8-diagrama-de-flujo)
9. [Matriz de gates](#9-matriz-de-gates)
10. [Convenciones](#10-convenciones)
11. [Instalacion](#11-instalacion)
12. [FAQ / troubleshooting](#12-faq--troubleshooting)

---

## 1. Que es

SDD es desarrollar definiendo **primero el comportamiento esperado** en una
carpeta `specs/` (la FUENTE DE VERDAD) y obligando a que todo codigo tenga detras
un requisito aprobado, testeado y validado.

Cadena central del sistema:

```
REQ -> AC -> IMP -> CODE -> TEST -> VALIDATION
```

Regla raiz: **cuando existe un requisito aprobado, el codigo NO sustituye al
requisito como fuente de verdad.**

Tres pilares separados (asi lo define el agente):

| Pilar | Vive en | Responde a |
|---|---|---|
| AGENTE | `.opencode/agents/` | Como debe trabajar el modelo (reglas, gates, estados) |
| COMMAND | `.opencode/commands/` | Que operacion ejecuta cada `/sdd-*` ahora |
| SDD | `specs/` del proyecto | Que debe hacer el software (contrato del proyecto) |

El sistema tiene 5 gates de aprobacion explicitos: nada avanza sin que el
usuario lo apruebe, y **toda aprobacion se registra en un fichero**, nunca solo
en la conversacion.

---

## 2. Glosario de conceptos

| Concepto | Definicion |
|---|---|
| **REQ** (Requirement) | Requisito: una capacidad coherente del sistema (no una frase trivial). Fichero `specs/requirements/REQ-XXX-nombre.md`. Campos: ID, nombre, estado, prioridad (P0/P1/P2), necesidad, requisito, motivacion, comportamiento esperado, AC, dependencias, decisiones pendientes, origen. |
| **AC** (Acceptance Criterion) | Criterio de aceptacion de un REQ. Debe ser **verificable objetivamente** (condicion + evidencia esperada); si no lo es, se senala y se pregunta - nunca se inventa otro. ID `AC-XXX-NN`. |
| **IMP** (Implementation Task) | Tarea de implementacion dentro del plan. ID `IMP-001`, ... con REQ/AC relacionados, dependencias, componentes, tests y estado. |
| **ADR** (Architecture Decision Record) | Decision de arquitectura registrada con contexto, decision y consecuencias. Fichero `specs/decisions/ADR-XXX-nombre.md`. Estados: `Propuesta`, `Aprobada`. |
| **FASE** | Agrupacion coherente de IMP con objetivo y condiciones de finalizacion propias. ID `FASE-01`, ... Citable literalmente en `/sdd-implement` y `/sdd-validate`. Cada fase tiene su propio campo `Aprobado`. |
| **CHANGE** | Registro persistente de un cambio de alcance analizado con `/sdd-change`. Fichero `specs/changes/CHANGE-XXX-nombre.md` con impacto CONFIRMADO / PROPUESTA / DESCONOCIDO, elementos afectados y campo `Aprobacion` con fecha. |
| **Gate de aprobacion** | Punto donde el flujo se DETIENE hasta aprobacion explicita del usuario. Cada gate tiene un artefacto donde registrarse; un gate no registrado no existe. |
| **Fuente de verdad** | `specs/`. Todo comportamiento esperado vive ahi versionado en git; el codigo no la reemplaza. |
| **Trazabilidad** | Cadena `REQ -> AC -> IMP -> CODE -> TEST -> VALIDATION` mantenida en `specs/validation/traceability.md`, una fila por AC. Cada eslabon es `ok` o `hueco` (hueco = reportar). |
| **Cobertura** | Porcentaje de REQ vigentes y AC cubiertos por el plan. Debe ser 100% antes de implementar y 100% para declarar `SDD COMPLIANT`. Se calcula solo sobre REQ **vigentes**: estados `Approved`, `Implemented`, `Validated` (excluye `Draft`, `Pending`, `Obsolete`). |
| **Evidencia** | Unico camino a `PASS`: test ejecutado, comando reproducible o inspeccion directa del comportamiento. Encontrar una clase relacionada NO es evidencia. |
| **REQ vigente** | REQ en `Approved`, `Implemented` o `Validated`. Los `Draft`/`Pending` son alcance sin cerrar; los `Obsolete` estan fuera de cobertura. |
| **UNSPECIFIED_IMPLEMENTATION** | Implementacion significativa sin REQ aprobado detras. La detecta la comprobacion inversa `CODE -> REQ` de la auditoria. No se borra: se informa. |
| **SDD COMPLIANT** | Veredicto final de la auditoria: `YES` solo si se cumplen todas sus condiciones (>=1 REQ, 0 REQ sin cerrar, coberturas al 100%, 100% AC en PASS, 0 NOT_TESTED/FAIL/PARTIAL, tests verdes, sin decisiones pendientes, trazabilidad y aprobaciones coherentes). |
| **Validacion independiente** | `/sdd-validate` y `/sdd-audit` delegan en subagentes con contexto limpio que parten de cero leyendo el disco: el que implementa nunca valida ni audita su propio trabajo. |

---

## 3. Estados y transiciones

### REQ (requisitos)

```
Draft -> Pending -> Approved -> Implemented -> Validated
```

Estado especial: `Obsolete` = retirado del alcance (no se borra; se excluye de
cobertura).

| Transicion | Quien la hace | Cuando |
|---|---|---|
| `Draft`/`Pending` -> `Approved` | agente SDD | usuario aprueba el gate de `/sdd-requirements` |
| nacen en `Approved` | agente SDD | REQ nuevo creado por un `/sdd-change` aprobado (origen `CHANGE-XXX`) |
| `Approved` -> `Implemented` | agente SDD | fin de fase: todos los IMP del REQ en `Implemented` |
| `Implemented` -> `Validated` | **sdd-validator** | todos los AC de TODAS las fases del REQ en `PASS` con evidencia |
| `Implemented`/`Validated` -> `Approved` | agente SDD | `/sdd-change` altera el REQ (pendiente de re-implementar) |
| cualquier -> `Obsolete` | agente SDD | `/sdd-change` lo deja fuera de alcance |

Nota: alterar o eliminar un AC de un REQ **cuenta como alterar ese REQ**.

### IMP (tareas del plan)

```
Pending -> InProgress -> Implemented -> Validated
                         Blocked (cuando un AC de los suyos queda BLOCKED)
```

- `InProgress`: al iniciar su implementacion (`/sdd-implement`).
- `Implemented`: fin de fase, con tests ejecutados.
- `Validated`: **solo lo asigna sdd-validator** con evidencia. El implementador
  esta prohibido de marcarlo.
- Regresion a `Pending`: invalidado por un `/sdd-change` aprobado.

### AC (criterios de aceptacion)

`PASS` | `FAIL` | `PARTIAL` | `NOT_IMPLEMENTED` | `NOT_TESTED` | `BLOCKED`

- `PASS` solo con evidencia verificable.
- `/sdd-change` pone los AC alterados en `NOT_TESTED`: su evidencia previa
  pertenece a la especificacion anterior.

---

## 4. Arquitectura del sistema

### 4.1 Carpetas del toolkit

```
.opencode/
├── agents/
│   ├── sdd.md             agente primario (orquestador)
│   ├── sdd-validator.md   subagente: validacion de fases
│   └── sdd-auditor.md     subagente: auditoria final
├── commands/
│   ├── sdd-init.md
│   ├── sdd-requirements.md
│   ├── sdd-plan.md
│   ├── sdd-implement.md
│   ├── sdd-validate.md
│   ├── sdd-change.md
│   ├── sdd-status.md
│   └── sdd-audit.md
└── templates/
    └── sdd/
        ├── specs-README-template.md
        ├── REQ-template.md
        ├── ADR-template.md
        ├── CHANGE-template.md
        ├── implementation-plan-template.md
        ├── traceability-template.md
        ├── acceptance-template.md
        └── final-validation-template.md
```

### 4.2 Los tres agentes

#### `sdd` - agente primario (orquestador)

- Modo: `mode: primary`. Se activa automaticamente al invocar cualquier
  `/sdd-*`; tambien manualmente con `@sdd`.
- Su trabajo: hacer cumplir la metodologia SIEMPRE (sus reglas aplican aunque
  no haya comando), orquestar los subagentes y gestionar los gates.
- Reglas fundamentales (resumen de la lista NUNCA): no inventar requisitos, no
  asumir decisiones no especificadas, no rellenar huecos en silencio, no
  modificar requisitos aprobados para encajar con el codigo, no ignorar AC, no
  marcar PASS por existir codigo, no avanzar ante ambiguedades, no implementar
  comportamiento no aprobado, no saltarse fases, no iniciar fases tras un gate
  sin aprobacion, no declarar terminado el proyecto sin auditoria.
- Ante ambiguedad: DETENER -> explicar -> preguntar -> esperar. No decide el.
- Nunca valida ni audita su propio trabajo: delega (ver subagentes).

#### `sdd-validator` - subagente de validacion

- Modo: `mode: subagent`. Lanzado por `/sdd-validate` via herramienta Task.
- Contexto limpio: no conoce la conversacion, los resumenes del implementador
  ni los estados escritos; todo lo lee del disco (plan, REQ originales, codigo,
  tests).
- Hace: valida CADA AC de la fase con evidencia, ejecuta tests, detecta
  comportamiento no aprobado, escribe `traceability.md` y `acceptance.md`,
  asigna estados `Validated`/`Blocked` donde corresponde.
- Permisos: `edit` denegado salvo `specs/validation/**`; `read`/`grep`/`glob`/
  `bash` permitidos. **No puede tocar codigo, planes ni requisitos.**
- Devuelve un informe estructurado (tabla AC, tests, discrepancias, siguiente
  paso) y termina: no aprueba gates ni propone correcciones automaticas.

#### `sdd-auditor` - subagente de auditoria final

- Modo: `mode: subagent`. Lanzado por `/sdd-audit` via herramienta Task.
- Contexto limpio: la auditoria es DESDE CERO; no se fia de nada escrito por
  implementacion ni validacion.
- Hace: recorre todos los REQ vigentes -> AC -> implementacion real -> tests;
  comprobacion inversa `CODE -> REQ` (busca `UNSPECIFIED_IMPLEMENTATION`);
  comprueba trazabilidad, decisiones pendientes y coherencia de aprobaciones
  (cabecera/fases del plan, estados de REQ, ficheros CHANGE); escribe
  `final-validation.md` y concilia `traceability.md`.
- Permisos: identicos al validador - solo `specs/validation/**` es editable.
- Emite el veredicto `SDD COMPLIANT: <YES | NO> (<motivo>)`. El certificado no
  lo emite quien construyo el proyecto.

### 4.3 Separacion de responsabilidades

| Quien | Puede | No puede |
|---|---|---|
| `sdd` (primario) | todo el workflow, editar specs/ y codigo | validarse ni auditarse a si mismo |
| `sdd-validator` | editar `specs/validation/**` | tocar codigo, planes, requisitos; corregir FAIL |
| `sdd-auditor` | editar `specs/validation/**` | tocar codigo, planes, requisitos; corregir hallazgos |

---

## 5. Comandos

Todos los comandos se ejecutan como `/sdd-*`, corren bajo el agente `sdd`
(frontmatter `agent: sdd`) y terminan en DETENTE (o en un gate).

### `/sdd-init`

| | |
|---|---|
| Que hace | Inicializa `specs/` en el proyecto actual |
| Argumentos | ninguno |
| Lee | proyecto existente (estructura, docs, codigo); `specs/` si ya existe |
| Escribe | carpetas de `specs/` que falten + `specs/README.md` (con su plantilla) |
| Gate | no; termina indicando que el siguiente paso es `/sdd-requirements` |

Reglas clave: nunca borra ni sobrescribre; si `specs/` ya existe la reutiliza y
repara solo lo que falte; prefiere plantillas/estandares ya existentes en el
repo; no inventa requisitos; si hay git, sugiere commitear `specs/`.

### `/sdd-requirements`

| | |
|---|---|
| Que hace | Toma iterativa de requisitos: analiza todo lo existente y pregunta en bloques |
| Argumentos | ninguno (o descripcion inicial del alcance) |
| Lee | `specs/` completo, documentacion del proyecto, codigo relevante |
| Escribe | `specs/requirements/REQ-XXX-*.md`, `specs/decisions/ADR-XXX-*.md` |
| Gate | **si** - fin de toma: presenta resumen/riesgos/alcance y DETIENE |

Reglas clave: pregunta solo lo que falta, en bloques razonables, esperando cada
respuesta; nunca repite preguntas contestadas; distingue REQUISITO / PROPUESTA /
PENDIENTA / DECISION APROBADA; los requisitos se presentan en `Pending` y pasan
a `Approved` solo con aprobacion explicita.

Guard: si ya hay REQ `Approved` y plan generado, redirige a `/sdd-change` (el
alcance esta cerrado); solo continua si el usuario confirma reabrir.

### `/sdd-plan`

| | |
|---|---|
| Que hace | Genera plan de implementacion (IMP por fases) + arquitectura + cobertura 100% |
| Argumentos | ninguno |
| Lee | todos los REQ de `specs/requirements/` directamente del disco |
| Escribe | `specs/plans/implementation-plan.md`, `specs/architecture/architecture.md`, `specs/validation/traceability.md` |
| Gate | **si** - fin de planificacion: exige cobertura 100% y DETIENE |

Reglas clave: NO toca codigo (solo `specs/`); segunda pasada independiente de
cobertura; si a un REQ le faltan AC verificables, DETIENE y pregunta (no
inventa); regenerar un plan ya aprobado lo devuelve a `pendiente`.

### `/sdd-implement <FASE-XX>`

| | |
|---|---|
| Que hace | Implementa UNA fase del plan: codigo + tests + trazabilidad |
| Argumentos | `$1` = ID de la fase (si falta: la pide y DETIENE) |
| Lee | plan, REQ originales, AC de la fase |
| Escribe | codigo, tests, estados de IMP/REQ en el plan, `traceability.md` |
| Gates | **si** (x2) - inicio de fase (antes de tocar codigo) y fin de fase |

Reglas clave: precondiciones antes de modificar NINGUN codigo (plan existe,
cabecera aprobada, fase aprobada - si esta `pendiente` la presenta y DETIENE);
implementa EXCLUSIVAMENTE esa fase; ante ambiguedad DETIENE y pregunta; marca
IMP `InProgress` -> `Implemented` (nunca `Validated`); NO inicia la siguiente
fase.

### `/sdd-validate <FASE-XX>`

| | |
|---|---|
| Que hace | Validacion INDEPENDIENTE de una fase, delegada en `sdd-validator` |
| Argumentos | `$1` = ID de la fase |
| Lee | el subagente lee del disco: plan, REQ, codigo, tests |
| Escribe | `specs/validation/traceability.md`, `specs/validation/acceptance.md`, estados `Validated`/`Blocked` |
| Gate | no formal; termina esperando decisiones sobre el informe |

Reglas clave: el agente SDD solo orquesta (Task -> `sdd-validator`), pasa fase +
ruta y **nada mas** (sin resumenes ni estados: el validador parte de cero);
presenta el informe tal cual, sin suavizar; nunca marca el mismo `Validated`;
ante FAIL propone y espera.

### `/sdd-change <descripcion>`

| | |
|---|---|
| Que hace | Analiza el impacto de un cambio ANTES de tocar nada |
| Argumentos | `$ARGUMENTS` = descripcion del cambio (si falta: la pide) |
| Lee | SDD completo + codigo/potencialmente afectado |
| Escribe | `specs/changes/CHANGE-XXX-*.md`, REQ/AC/ADR afectados, plan, arquitectura, trazabilidad, `acceptance.md`, cabecera de `final-validation.md` si existe |
| Gates | **si** (x2) - impacto (antes de modificar el SDD) y plan actualizado (antes de implementar) |

Reglas clave: NUNCA codigo primero; impacto separado en CONFIRMADO /
PROPUESTA / DESCONOCIDO; registra la ficha CHANGE con `Aprobacion: pendiente` y
anade la fecha al aprobar; regresa estados (REQ alterado -> `Approved`, IMP
invalidado -> `Pending`, AC alterado -> `NOT_TESTED`, fuera de alcance ->
`Obsolete`); fases afectadas y cabecera vuelven a `pendiente`; invalida el
certificado de auditoria si existia.

### `/sdd-status`

| | |
|---|---|
| Que hace | Estado compacto del proyecto |
| Argumentos | ninguno |
| Lee | `specs/` completo + tests del proyecto si es factible |
| Escribe | nada |
| Gate | no; termina al mostrar el estado |

Salida: requisitos, AC, progreso, fase actual/siguiente, IMP por estado, tests,
decisiones pendientes, cambios sin aprobar, blockers, cumplimiento. Marca
`desconocido` lo que no pueda verificar; no infiere PASS de estados obsoletos;
si la ultima auditoria esta invalidada por un cambio, lo dice.

### `/sdd-audit`

| | |
|---|---|
| Que hace | Auditoria final DESDE CERO, delegada en `sdd-auditor` |
| Argumentos | ninguno |
| Lee | el subagente lee todo: specs/ + codigo + tests |
| Escribe | `specs/validation/final-validation.md`, `traceability.md` (conciliacion) |
| Gate | **si** - gate final: `SDD COMPLIANT: YES/NO` |

Reglas clave: el agente SDD solo orquesta (Task -> `sdd-auditor`) sin pasarle
estados ni resumenes; presenta el informe tal cual; NO corrige ningun hallazgo
automaticamente; DETIENE esperando instrucciones.

---

## 6. Plantillas

`templates/sdd/` define el formato oficial de cada artefacto para que no derive
entre fases. Los comandos y subagentes las referencian explicitamente (rutas
`.opencode/templates/sdd/...`).

| Plantilla | Define el formato de |
|---|---|
| `specs-README-template.md` | `specs/README.md` (workflow, estructura, IDs, estados, idioma) |
| `REQ-template.md` | `specs/requirements/REQ-XXX-nombre.md` |
| `ADR-template.md` | `specs/decisions/ADR-XXX-nombre.md` |
| `CHANGE-template.md` | `specs/changes/CHANGE-XXX-nombre.md` |
| `implementation-plan-template.md` | `specs/plans/implementation-plan.md` (cabecera, fases, tabla de IMP) |
| `traceability-template.md` | `specs/validation/traceability.md` (tabla REQ/AC/IMP/CODE/TEST/VALIDATION) |
| `acceptance-template.md` | `specs/validation/acceptance.md` (tabla AC/REQ/Fase/Estado/Evidencia) |
| `final-validation-template.md` | `specs/validation/final-validation.md` (recuentos, coberturas, veredicto) |

---

## 7. Flujo de llamadas (ejemplo, solo prompts)

Secuencia de prompts del USUARIO en un proyecto de principio a fin (las respuestas
entre `<...>` son texto que teclea el usuario; el sistema se omite):

```
/sdd-init

/sdd-requirements
<descripcion inicial del proyecto y su alcance>
<respuestas al primer bloque de preguntas>
<respuestas al segundo bloque de preguntas>
Apruebo los requisitos

/sdd-plan
Apruebo el plan

/sdd-implement FASE-01
Aprobada la fase

/sdd-validate FASE-01
Aprobada, siguiente fase

/sdd-implement FASE-02
Aprobada la fase

/sdd-validate FASE-02

/sdd-status

/sdd-change añadir exportacion de pedidos a PDF
<respuestas a las dudas de impacto>
Apruebo el impacto
Apruebo el plan actualizado

/sdd-implement FASE-02
Aprobada la fase

/sdd-validate FASE-02

/sdd-audit
```

Variante minima (proyecto ya con requisitos conocidos):

```
/sdd-init
/sdd-requirements
<requisitos directos>
Apruebo
/sdd-plan
Apruebo
/sdd-implement FASE-01
Aprobada
/sdd-validate FASE-01
/sdd-audit
```

---

## 8. Diagrama de flujo

```
                        /sdd-init
                            |
                            v
                 /sdd-requirements
                            |
                     (GATE requisitos) -----> REQ en Approved
                            |
                            v
                       /sdd-plan
                            |
                      (GATE plan) ---------> cabecera Aprobado
                            |
                            v
                /sdd-implement FASE-XX
                     |            ^
        (GATE inicio |            | correccion de FAIL)
         de fase)    |            |
                     v            |
              codigo + tests      |
              IMP Implemented     |
                     |            |
        (GATE fin de fase)        |
                     |            |
                     v            |
               /sdd-validate FASE-XX
                     |
                     +--> Task(sdd-validator) --> informe --> usuario decide
                     |
                     +--> PASS: siguiente fase  ---(vuelve a /sdd-implement)
                     +--> FAIL: correccion      ---(vuelve a /sdd-implement)
                            |
                            v (todas las fases validadas)
                       /sdd-audit
                            |
                  Task(sdd-auditor) --> final-validation.md
                            |
                    (GATE auditoria) -----> SDD COMPLIANT: YES / NO

     EN CUALQUIER MOMENTO:

     /sdd-change <desc> --> ficha CHANGE --> (GATE impacto)
                                |
                                v
                     actualiza REQ/AC/ADR/plan/arquitectura
                     (cabecera y fases -> pendiente;
                      AC alterados -> NOT_TESTED;
                      certificado -> invalidado)
                                |
                         (GATE plan) -----> vuelve a /sdd-implement

     /sdd-status  (solo lectura, sin gate)
```

---

## 9. Matriz de gates

| # | Comando | Gate | Que se aprueba | Donde se registra | Desbloquea |
|---|---|---|---|---|---|
| 1 | `/sdd-requirements` | Fin de toma de requisitos | Alcance completo | REQ presentados -> estado `Approved` | `/sdd-plan` |
| 2 | `/sdd-plan` | Fin de planificacion (exige cobertura 100%) | Plan + arquitectura | cabecera `Aprobado (fecha)` en el plan | `/sdd-implement` |
| 3 | `/sdd-implement` | **Inicio** de cada fase | Alcance de la fase antes de tocar codigo | campo de la fase `Aprobado (fecha)` | escribir codigo |
| 4 | `/sdd-implement` | **Fin** de cada fase implementada | Continuar al validador | sin campo: el avance consta en IMP + trazabilidad | `/sdd-validate` |
| 5 | `/sdd-change` | Impacto del cambio | Analisis CONFIRMADO/PROPUESTA/DESCONOCIDO | ficha CHANGE: `Aprobacion: <fecha>` | actualizar el SDD |
| 6 | `/sdd-change` | Plan actualizado | Plan con los cambios aplicados | cabecera del plan -> `Aprobado (fecha)` | re-implementar |
| 7 | `/sdd-audit` | Auditoria final | Veredicto de cumplimiento | `specs/validation/final-validation.md` | declarar terminado |

Reglas transversales:

- Todo gate exige DETENER hasta aprobacion explicita.
- Gate no registrado = gate inexistente ("nunca des por aprobado un gate que no
  este registrado").
- `/sdd-status` y la validacion/auditoria como lectura no tienen gate de salida
  propio.

---

## 10. Convenciones

**Identificadores** (formato unico en todo el sistema):

| Entidad | ID | Fichero |
|---|---|---|
| Requisito | `REQ-001` | `specs/requirements/REQ-001-nombre.md` |
| Criterio de aceptacion | `AC-001-01` (asociado a su REQ) | dentro del REQ |
| Tarea | `IMP-001` | `specs/plans/implementation-plan.md` |
| Decision de arquitectura | `ADR-001` | `specs/decisions/ADR-001-nombre.md` |
| Cambio | `CHANGE-001` | `specs/changes/CHANGE-001-nombre.md` |
| Fase | `FASE-01` | `specs/plans/implementation-plan.md` |

**Otros:**

- Fechas: `YYYY-MM-DD`.
- Origen de un REQ: `necesidad directa` | `CHANGE-XXX` | `heredado de <doc>`.
- Prioridad de REQ: `P0` | `P1` | `P2`.
- Idioma de `specs/`: el acordado en `specs/README.md` (lo fija `/sdd-init`).
- Este toolkit (ficheros de `.opencode/` y este README) usa texto en espanol
  sin acentos, estilo ASCII.
- Prioriza claridad y brevedad en toda salida al usuario; tono directo,
  respondiendo en el idioma del usuario.

---

## 11. Instalacion

1. **Renombra** la carpeta del toolkit a `.opencode/` en la raiz del proyecto
   (opencode solo carga `.opencode/`; ademas los comandos referencian
   `.opencode/templates/sdd/...`).
2. **Reinicia opencode** (la configuracion no se recarga en caliente).
3. Ejecuta `/sdd-init`.
4. Versiona `specs/` en git: es el contrato del proyecto.

El agente `@sdd` se activa automaticamente al invocar cualquier `/sdd-*`.

Estructura resultante en el proyecto:

```
specs/
├── README.md          workflow, IDs, estados, idioma
├── requirements/      REQ-XXX-nombre.md
├── architecture/      architecture.md
├── decisions/         ADR-XXX-nombre.md
├── changes/           CHANGE-XXX-nombre.md (impacto y aprobacion de cambios)
├── plans/             implementation-plan.md (IMPs, FASE-XX, aprobaciones)
└── validation/        traceability.md, acceptance.md, final-validation.md
```

---

## 12. FAQ / troubleshooting

**No veo los comandos `/sdd-*` ni el agente**
La carpeta debe llamarse `.opencode/` exactamente y hay que reiniciar opencode:
la configuracion no se recarga en caliente.

**`/sdd-implement` me pide aprobar la fase antes de empezar**
Es el gate de inicio (fila 3 de la matriz): la fase nace en `Aprobado:
pendiente` y se aprueba justo antes de tocar codigo. Presenta la fase, aprueba
y seguira.

**`/sdd-plan` dice que no hay requisitos `Approved`**
Todavia no se ha aprobado el gate de requisitos. Vuelve a
`/sdd-requirements` y aprueba la presentacion final.

**Quiero cambiar un requisito ya aprobado**
Usa `/sdd-change`. `/sdd-requirements` tiene un guard: con el alcance cerrado
redirige a `/sdd-change` (a menos que confirmes reabrir la toma). La ruta
cambiante garantiza impacto analizado, re-aprobacion e invalidacion del
certificado.

**La auditoria decia YES, pero ha habido cambios despues**
El cambio invalida el certificado y `/sdd-status` lo reporta como invalidado.
Re-audita con `/sdd-audit` cuando el alcance este estable.

**`/sdd-init` dice que `specs/` ya existe**
Reutiliza lo que hay y solo crea lo que falta: nunca borra ni sobrescribe.
Sirve tambien para reparar una estructura parcial.

**El validador/auditor devuelve FAIL - lo corrijo yo?**
No automaticamente. El sistema propone y tu instruyes: normalmente
`/sdd-implement <fase>` para corregir (o `/sdd-change` si el problema es de
alcance) y despues re-validar. Los subagentes estan prohibidos de corregir.

**Validar o auditar tarda mas que el resto**
Es esperado: `/sdd-validate` y `/sdd-audit` lanzan un subagente independiente
con contexto limpio (herramienta Task) que re-verifica todo desde el disco.

**Como veo el estado sin ejecutar el flujo**
`/sdd-status`: requisitos, coberturas, fase actual, cambios sin aprobar,
blockers y cumplimiento. Marca `desconocido` lo que no puede verificar.

**Puedo tocar `specs/` a mano?**
Si, pero el sistema es el administrador de estados: si editas a mano, el
siguiente comando re-leera el disco y puede detectar incoherencias (la
auditoria comprueba justamente eso). Para cambios de alcance, usa
`/sdd-change`.

**Como se versiona?**
Commitea `specs/` en cada hito (tras aprobar requisitos, plan, cada fase
validada, cada CHANGE). git es tu red de seguridad ante regresiones de estado.
