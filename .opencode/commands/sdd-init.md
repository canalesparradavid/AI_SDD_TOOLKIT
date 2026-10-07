---
description: Inicializa el SDD en el proyecto - crea specs/ minimo sin destruir nada existente
agent: sdd
---

Ejecuta la fase de inicializacion SDD del proyecto actual.

Procedimiento obligatorio:

1. Analiza el proyecto: estructura, documentacion existente, contexto.
2. Detecta si ya existe `specs/`.
   - Si existe: inspeccionala completa (requirements, architecture, decisions,
     changes, planes, validacion),
     reutilizala, informa de que encontraste y que se reutiliza. NO la sobrescribas.
   - Si no existe: continua con el punto 3.
   En cualquier caso: crea las subcarpetas del punto 3 que falten y
   `specs/README.md` si no existe. Nunca borres ni sobrescribas contenido existente.
3. Crea UNICAMENTE lo minimo necesario; si el repositorio ya tiene plantillas o
   estandares documentales equivalentes, prefierelos y adaptate a ellos:

```
specs/
├── README.md
├── requirements/
├── architecture/
├── decisions/
├── changes/
├── plans/
└── validation/
```

4. `specs/README.md`: usa `.opencode/templates/sdd/specs-README-template.md` como
   base con la informacion del proyecto. Debe explicar brevemente:
   el workflow `REQ -> AC -> IMP -> CODE -> TEST -> VALIDATION`, la estructura de
   carpetas, los identificadores (REQ-XXX, AC-XXX-NN, IMP-XXX, ADR-XXX,
   CHANGE-XXX, FASE-XX) y los estados permitidos, y fija el idioma acordado para
   los ficheros de `specs/`.
5. NO crees ficheros vacios (ni architecture.md, ni diagrams.md, ni traceability.md,
   ni acceptance.md, ni final-validation.md): se crearan cuando su contenido exista.
6. NO inventes requisitos. Si el proyecto no tiene requisitos conocidos, DILO
   explicitamente.
7. Informa de exactamente que has creado, reutilizado, omitido (por ya existir) y
   los conflictos encontrados. Si el proyecto usa git, sugiere commitear `specs/`:
   es la fuente de verdad y debe ir versionada.
8. DETENTE. Indica que el siguiente paso es `/sdd-requirements`.
