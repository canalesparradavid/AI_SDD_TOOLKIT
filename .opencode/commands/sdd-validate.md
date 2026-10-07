---
description: Valida independientemente una fase SDD - parte de los REQ originales, no del estado escrito
agent: sdd
---

Ejecuta la validacion independiente de la fase indicada: `$1`.

Si falta `$1`: pide la fase y DETENTE. Si `$1` no coincide con ninguna fase del
plan: lista las fases existentes y DETENTE.

Principio: la validacion la realiza un SUBAGENTE INDEPENDIENTE (`sdd-validator`)
con contexto limpio. Tu (agente SDD) no validas: solo orquestas, presentas el
informe y esperas decisiones. Asi el validador no hereda ni los sesgos ni los
resumenes de la implementacion.

Procedimiento obligatorio:

1. Comprueba que existe `specs/plans/implementation-plan.md` y que la fase `$1`
   esta en el plan. Si el plan no existe: informa de que falta `/sdd-plan` y
   DETENTE.
2. Lanza la validacion con la herramienta Task:
   - subagent_type: `sdd-validator`
   - prompt: la fase (`$1`), la ruta del proyecto y la instruccion de que todo lo
     demas lo lea del disco (plan, REQ originales, codigo, tests). NO le pases
     estados, resumenes, resultados de tests ni comentarios de la fase: el
     validador parte de cero por diseño.
3. Espera el informe del validador. No lo edites ni lo suavizes.

Resultado:

4. Presenta al usuario el informe tal cual: tabla AC -> estado -> evidencia, tests
   ejecutados, discrepancias con la implementacion, comportamiento no aprobado si
   lo hay, IMP actualizados y ficheros de `specs/validation/` que el validador ha
   actualizado.
5. DETENTE. No corrijas nada por tu cuenta: si hay `FAIL`/`PARTIAL`/
   `NOT_IMPLEMENTED`, propone el siguiente paso y espera instrucciones.
   Nunca marques tu mismo `Validated` en REQ o IMP: ese estado solo lo asigna el
   validador con evidencia.
