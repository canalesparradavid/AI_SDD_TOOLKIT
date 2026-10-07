# <nombre del proyecto> - Specs

Fuente de verdad del comportamiento esperado del software. Versionalo en git.

## Workflow

`REQ -> AC -> IMP -> CODE -> TEST -> VALIDATION` con gates de aprobacion
explicitos gestionados por el toolkit SDD (`/sdd-*`).

## Estructura

```
specs/
├── README.md          (este fichero: workflow, IDs, estados, idioma)
├── requirements/      REQ-XXX-nombre.md
├── architecture/      architecture.md
├── decisions/         ADR-XXX-nombre.md
├── changes/           CHANGE-XXX-nombre.md
├── plans/             implementation-plan.md (IMPs, FASE-XX, aprobaciones)
└── validation/        traceability.md, acceptance.md, final-validation.md
```

## Identificadores

- Requisitos: `REQ-001`, `REQ-002`, ...
- Criterios: `AC-001-01`, ... (asociados a su REQ)
- Tareas: `IMP-001`, ... (en `plans/implementation-plan.md`)
- Decisiones: `ADR-001`, ...
- Cambios: `CHANGE-001`, ...
- Fases: `FASE-01`, ... (en `plans/implementation-plan.md`)

## Estados

- REQ: `Draft` -> `Pending` -> `Approved` -> `Implemented` -> `Validated`;
  `Obsolete` = fuera de alcance (no se borra, se excluye de cobertura).
- IMP: `Pending`, `InProgress`, `Implemented`, `Validated`, `Blocked`.
- AC: `PASS`, `FAIL`, `PARTIAL`, `NOT_IMPLEMENTED`, `NOT_TESTED`, `BLOCKED`.
  `PASS` exige evidencia verificable.

## Idioma de los ficheros de specs/

<idioma acordado, p.ej. espanol; indicar si los IDs y estados van en ingles>
