---
description: Auditoria final SDD desde cero - gate para declarar SDD COMPLIANT
agent: sdd
---

Ejecuta la auditoria final del proyecto. Es el gate final.

Principio: la auditoria la realiza un SUBAGENTE INDEPENDIENTE (`sdd-auditor`)
con contexto limpio, partiendo DESDE CERO: no se fia de estados anteriores,
resumenes ni de lo escrito por `/sdd-implement` o `/sdd-validate`. Tu (agente
SDD) no audites: solo orquestas, presentas el informe y esperas decisiones.
Asi el certificado no lo emite quien construyo el proyecto.

Procedimiento obligatorio:

1. Lanza la auditoria con la herramienta Task:
   - subagent_type: `sdd-auditor`
   - prompt: la ruta del proyecto y la instruccion de auditar todo el SDD. NO le
     pases estados, coberturas, resumenes de fases ni resultados anteriores: el
     auditor los re-verifica por su cuenta contra el disco (specs/ + codigo +
     tests).
2. Espera el informe del auditor. No lo edites ni lo suavizes.

Resultado:

3. Presenta al usuario el informe tal cual: recuentos REQ/AC, coberturas,
   `UNSPECIFIED_IMPLEMENTATION`, hallazgos, `specs/validation/final-validation.md`
   actualizado y el veredicto `SDD COMPLIANT: <YES | NO> (<motivo>)`.
4. NO corrijas automaticamente ningun hallazgo de la auditoria (ni FAIL, ni
   `UNSPECIFIED_IMPLEMENTATION`, ni huecos de trazabilidad): presenta el informe y
   DETENTE. Espera instrucciones.
