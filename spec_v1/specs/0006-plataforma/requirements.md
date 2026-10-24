# REQUIREMENTS — EP-05 Plataforma, trazabilidad y aprobaciones

Área responsable: requisitos y trazabilidad · Spec: [`../0001-vision/spec.md`](../0001-vision/spec.md)

---

## US-05.1 — Artefactos de la especificación versionados

*When* una épica se cierra, *the system shall* contar con `spec`, `design`, `tasks` y la
matriz de trazabilidad actualizada en el repositorio.

*When* un requisito cambia, *the system shall* registrar el cambio con fecha, autor y motivo.

**Valor para el usuario:** la especificación es un entregable auditable, no un documento
informal.

---

## US-05.2 — Trazabilidad requisito → diseño → tarea → prueba

*When* se implementa una tarea, *the system shall* referenciar el `US-*` que satisface y la
tarea que la implementa.

*When* se ejecuta la validación de trazabilidad, *the system shall* fallar si existe un
requisito sin criterio EARS, un criterio sin prueba, o una tarea sin requisito.

**Valor para el usuario:** se sabe exactamente qué está construido y qué está verificado.

---

## US-05.3 — Criterios de aceptación ejecutables

*When* existe un criterio EARS, *the system shall* existir al menos un test automatizado que
falle si el criterio se incumple.

*When* se ejecuta la suite, *the system shall* correr los tests del paquete completo, no solo
los del módulo tocado.

**Valor para el usuario:** "funciona" es una afirmación verificable.

---

## US-05.4 — Bandeja de aprobaciones del coordinador

*When* el agente genera una propuesta, *the system shall* aparecer en una bandeja con estado,
impacto, justificación y acciones de aprobar / editar / rechazar.

*When* hay propuestas pendientes de un proyecto, *the system shall* mostrarlas también en la
card del proyecto y en la ficha de la persona afectada.

**Valor para el usuario:** las decisiones del agente están donde se toman las decisiones.

---

## US-05.5 — Roles y control de acceso

*When* un usuario tiene rol `COORDINACION`, *the system shall* poder aprobar propuestas.
*When* un usuario tiene rol `RRHH`, *the system shall* poder ver y editar datos sensibles
(incapacidades, vacaciones) y aprobar los préstamos que involucran becarios.
*When* un usuario tiene rol `CONSULTA`, *the system shall* ver los tableros sin datos sensibles
y sin poder aprobar.

**Valor para el usuario:** el dato de salud no se filtra por accidente.

---

## US-05.6 — Auditoría inmutable

*When* ocurre un cambio de estado relevante (asignación aprobada, préstamo vigente, ausencia
registrada, proyecto en riesgo), *the system shall* escribir un registro de auditoría con
actor, timestamp y estado anterior/nuevo.

*When* se intenta modificar o borrar un registro de auditoría, *the system shall* rechazarlo.

**Valor para el usuario:** se puede reconstruir la historia del sistema.

---

## US-05.7 — Modo demostración

*When* se carga el dataset semilla, *the system shall* poblar el sistema con proyectos en
diversos estados (incluyendo uno al 78 % de avance con presión y uno con una incapacidad
sin cubrir) para poder demostrar el comportamiento del agente de forma reproducible.

**Valor para el usuario:** la demo del proyecto final es repetible y defendible.

---

## Reglas aplicables

`BR-18` (aprobación humana) · `BR-19` (auditoría) · `BR-20` (sin atributos sensibles)
