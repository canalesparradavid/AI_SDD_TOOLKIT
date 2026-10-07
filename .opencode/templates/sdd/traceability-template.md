# Trazabilidad

Cadena: `REQ -> AC -> IMP -> CODE -> TEST -> VALIDATION`. Una fila por AC.
Marcas: `ok` si el eslabon existe, `hueco` si falta (reportar).

| REQ | AC | IMP | CODE | TEST | VALIDATION |
|---|---|---|---|---|---|
| REQ-XXX | AC-XXX-NN | IMP-001 | <ruta o hueco> | <ruta o hueco> | <PASS/FAIL/... o vacio> |
