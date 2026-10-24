# REQUIREMENTS — EP-04 Agente coordinador (IA)

Área responsable: requisitos y trazabilidad · Spec: [`../0001-vision/spec.md`](../0001-vision/spec.md) · Diseño: [`../../designs/0004-agente-coordinador.md`](../../designs/0004-agente-coordinador.md)

> Postura del producto: el agente es un **coordinador**, no un jefe. Propone con evidencia,
> nunca aplica cambios por su cuenta y jamás evalúa a las personas con atributos sensibles.

---

## US-04.1 — Propuesta de asignaciones al crear un proyecto

*When* se crea un proyecto con plantilla de roles, *the system shall* generar una **propuesta
de asignación** con candidatos por rol, cada uno con: rol, persona, FTE, fechas, **score de
ajuste**, desglose del score y justificación legible.

*When* se calcula el score de un candidato, *the system shall* usar **solo** estos factores:
coincidencia de skills y nivel, disponibilidad en la ventana, encaje de rol (incluido rol
contractual base y roles previos), carga actual, historial de proyectos similares y
compatibilidad con restricciones (vacaciones, incapacidad, no ser becario en préstamo).

*When* un candidato no puede cubrir una ventana por ausencia, *the system shall* excluirlo de
esa ventana o marcarlo explícitamente como "requiere cobertura".

*When* la propuesta se genera, *the system shall* incluir **alternativas** (al menos una por
rol crítico) y el **impacto proyectado** (utilización resultante, presión del proyecto,
efecto sobre proyectos origen).

*When* la propuesta se genera, *the system shall* quedar en estado `EN_REVISION`: **no**
aplica cambios al plan.

**Valor para el usuario:** arrancar un proyecto con un equipo propuesto en minutos.

---

## US-04.2 — Monitoreo de presión

*When* el agente se ejecuta de forma programada (o ante un evento de cambio), *the system
shall* recalcular la presión de todos los proyectos activos.

*When* un proyecto supera el umbral de presión, *the system shall* emitir una alerta con:
diagnóstico (qué rol y en qué periodo), causa probable (esfuerzo, ausencia, préstamo
vigente), opciones de alivio y nivel de confianza (que baja si falta información de carga).

*When* la alerta se emite, *the system shall* notificar a PMO y al manager del proyecto, y
registrar el caso en la bandeja de aprobaciones.

**Valor para el usuario:** enterarse de la presión a tiempo y con diagnóstico.

---

## US-04.3 — Propuesta de préstamo temporal

*When* un proyecto está en riesgo y su capacidad interna libre no cubre la necesidad,
*the system shall* buscar personas disponibles en **otros** proyectos que cumplan todas las
condiciones: skill suficiente, disponibilidad real en la ventana, ausencia del piso de reserva
del proyecto origen, no ser becario (o tener aprobación de RRHH), y no tener una asignación
fija que entra en conflicto.

*When* propone un préstamo, *the system shall* marcar la asignación resultante como
`TEMPORARIA`, con `proyecto_origen`, `fecha_fin` **obligatoria** y el detalle de la capacidad
reservada en el origen.

*When* propone un préstamo entre proyectos de clientes distintos, *the system shall* marcar la
propuesta como `REQUIERE_REVISIÓN_DE_CONFIDENCIALIDAD`.

**Valor para el usuario:** tapar el hueco sin romper lo ya prometido.

---

## US-04.4 — Respeto de la asignación temporal

*When* existe un préstamo temporal vigente, *the system shall* bloquear cualquier
asignación nueva que consuma la capacidad reservada por ese préstamo en el proyecto origen.

*When* se acerca la fecha de fin de un préstamo, *the system shall* avisar al coordinator, al
manager del proyecto origen y al del proyecto destino.

*When* llega la fecha de fin, *the system shall* **reembolsar automáticamente** la capacidad
al proyecto origen, cerrar el préstamo y registrar el evento en el historial de la persona.

*When* la persona prestada sigue ausente o con carga completa al vencer el préstamo,
*the system shall* generar una nueva propuesta de renovación o de reasignación, sin dejar el
proyecto origen por debajo de su piso.

*When* se consulta la vista de una persona prestada, *the system shall* mostrar de forma
visible que está en préstamo temporal, hacia dónde y hasta cuándo.

**Valor para el usuario:** el préstamo es un compromiso con fecha, no un "loan" informal.

---

## US-04.5 — Cobertura de ausencias

*When* se registra una incapacidad sobre una persona con asignaciones activas, *the system
shall* generar una propuesta de sustitutos calificados por rol y proyecto, priorizando
personas con capacidad libre y sin conflicto de ausencias.

*When* se registran vacaciones con impacto en un proyecto en riesgo, *the system shall*
generar la propuesta de cobertura **recomendada** (no bloqueante).

*When* una ausencia termina, *the system shall* liberar la cobertura y devolver la capacidad.

**Valor para el usuario:** la incapacidad no se convierte en un agujero silencioso.

---

## US-04.6 — Propuestas y aprobaciones humanas

*When* el agente genera una propuesta, *the system shall* registrarla con su estado, las
herramientas invocadas, los datos usados y la justificación.

*When* un usuario con rol de Coordinación **aprueba** una propuesta, *the system shall* aplicar
las asignaciones resultantes de forma atómica y registrar quién aprobó y cuándo.

*When* un usuario **rechaza** una propuesta, *the system shall* registrar el motivo, cerrar la
propuesta y **no** reintentarla automáticamente.

*When* un usuario **edita** una propuesta antes de aprobarla, *the system shall* registrar
las diferencias entre la versión del agente y la aprobada.

*When* el agente propone asignar a una persona no disponible, *the system shall* bloquear la
aprobación automática y exigir corrección manual.

**Valor para el usuario:** control real sobre las decisiones de personal.

---

## US-04.7 — Explicabilidad y auditoría

*When* un usuario pregunta por qué una persona fue propuesta, *the system shall* mostrar: los
factores con su peso, los datos comparados con los candidatos rechazados, las reglas
aplicadas y el estado de la propuesta.

*When* el agente produce una afirmación con datos, *the system shall* haberlos obtenido de
una tool verificable en esa misma ejecución; no se admiten cifras inventadas.

*When* se consulta la auditoría, *the system shall* devolver: prompt, tools invocadas, entradas
y salidas, propuesta resultante, decisión humana y actor.

*When* se evalúa la distribución de propuestas, *the system shall* poder mostrar métricas de
equidad de recomendación **sin** exponer atributos sensibles (usando agregados y proxy
no sensibles).

**Valor para el usuario:** confianza y trazabilidad, no una caja negra.

---

## US-04.8 — Alertas y avisos

*When* se acerca el fin de una vacaciones, de una incapacidad o de un préstamo, *the system
shall* emitir el aviso correspondiente con antelación configurable (por defecto 3 días).

*When* se acerca un cumpleaños, *the system shall* avisar al equipo del proyecto.

*When* un proyecto entra en `EN_RIESGO` y no hay ninguna acción tras 48 h, *the system shall*
escalar el aviso.

**Valor para el usuario:** nada se queda en un tablero que nadie mira.

---

## Reglas aplicables

`BR-04`–`BR-06` (presión y decisión humana) · `BR-07`–`BR-12` (préstamo temporal) ·
`BR-13`–`BR-15` (ausencias) · `BR-18` (human-in-the-loop) · `BR-19` (auditoría) ·
`BR-20` (sin atributos sensibles)
