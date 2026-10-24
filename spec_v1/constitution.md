# Constitución del proyecto — Gestión de RH y Proyectos

> Estado: **borrador**, abierto a revisión del cliente.
> inspired by: `github/spec-kit` — `.specify/memory/constitution.md` (adaptado a la topología `specs/ designs/ tasks/`).
> Relación con la spec de requisitos: esta constitución **especifica** los artículos que la spec de requisitos deja
> abiertos (BR-18 ↔ Art. V, BR-19 ↔ Art. VI, BR-20 ↔ Art. IX).

Principios **inmutables**. Toda propuesta, diseño o tarea que los contradiga debe rechazarse en la fase `/analyze`.
Estos artículos son la base de la **trazabilidad**: cada criterio de aceptación puede mapearse a al menos un artículo.

---

## Artículo I — La spec es la fuente de verdad
El comportamiento del sistema se define en `specs/`, no en el código. Si el código y la spec discrepan, la spec gana
y el código es el bug. Los artefactos de `specs/001-*/` no se archivan al terminar una tarea: se versionan y evolucionan
(spec **anchored**, no solo spec-first — ver `specs/0001-vision/research-sdd-y-dominio.md` §1.2).

## Artículo II — Verificabilidad
Cada requisito funcional lleva criterio de aceptación en formato EARS y un identificador estable (`R-XX`).
Ningún requisito se acepta sin al menos una prueba automatizada o una comprobación manual reproducible
en `traceability/matrix.md`. Un requisito no verificable es un requisito inválido.

## Artículo III — El dominio manda
El glosario (`Persona`, `Rol base`, `Rol de proyecto`, `Asignación`, `Préstamo temporal`, `Ausencia`,
`Capacidad`, `Utilización`) es la ley. El código no puede introducir conceptos que no estén en el glosario;
si un concepto nuevo es necesario, primero se añade al glosario (cambio de spec, con revisión).

## Artículo IV — Invariantes del dominio, no "casos raros"
Las reglas que el sistema no puede violar (capacidad, solapamiento, respeto al préstamo temporal, cobertura de
ausencias) se implementan **en el dominio**, no en la UI ni en el agente. Si una regla puede romperse desde la
interfaz, es un bug de diseño, no de implementación.

## Artículo V — El agente propone, la persona decide
El agente coordinador **nunca** activa, modifica ni cancela una asignación, un préstamo o una cobertura por su cuenta.
Produce *propuestas* con estado `EN_REVISION`; solo una persona las aprueba. La excepción será, como máximo, una
notificación. Esto aplica también en modo demo: la demo no debe romper el principio.

## Artículo VI — Determinismo y auditabilidad
Ninguna salida del agente depende de aleatoriedad no sembrada. Toda propuesta guarda el *snapshot* de entradas
(capacidades, ausencias, skills, umbrales) con las que se calculó, de modo que la misma entrada produce la misma
propuesta. Toda mutación deja traza en `BitacoraEvento` con actor, antes/después y justificación.

## Artículo VII — Privacidad por diseño
Datos personales sensibles (salud, fecha de nacimiento) se almacenan, pero **solo se exponen como estado y rango de
fechas**; el detalle clinicalo no existe en el modelo. El dataset por defecto es sintético y determinista.

## Artículo VIII — La demo es parte del entregable
Cada épica se demuestra con el flujo real (no con mocks del agente): seed → operación → resultado visible.
Una épica sin evidencia de ejecución no se cierra.

## Artículo IX — Sesgo y datos sensibles
El motor que puntúa candidatos **solo** puede usar atributos no sensibles: skills, disponibilidad, rol, carga,
historial y ausencias. Queda prohibido usar género, edad, origen, estado civil o motivo clínico.
El *scoring* es público y auditable: cualquier persona puede ver por qué un candidato fue elegido.
Base: BR-20 de la spec de requisitos y la obligación de que el sistema no reproduzca sesgos de contratación.

## Artículo X — Autonomía graduada y apagado de emergencia
El agente opera por defecto en nivel `propose`: crea propuestas y alertas, nunca aplica.
Existe un **interruptor global de pausa** que un humano puede activar en cualquier momento y que debe ser visible
en la UI. El nivel `apply_with_revert` solo puede habilitarse con decisión explícita y registrada.

## Artículo XI — Un único glosario
Existe **un** vocabulario para el dominio (el de la spec de requisitos: rol contractual base, rol de proyecto, asignación,
cobertura, préstamo temporal, ausencia, carga estimada, índice de presión). Ningún artefacto puede introducir un
sinónimo: si dos documentos usan nombres distintos para lo mismo, es un defecto de la spec, no una variación de estilo.
