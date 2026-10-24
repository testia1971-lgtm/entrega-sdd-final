# REQUIREMENTS — EP-02 Proyectos, roles y timeline

Área responsable: requisitos y trazabilidad · Spec: [`../0001-vision/spec.md`](../0001-vision/spec.md) · Diseño: [`../../designs/0003-tableros-ux.md`](../../designs/0003-tableros-ux.md)

---

## US-02.1 — Crear un proyecto

*When* un usuario crea un proyecto, *the system shall* capturar: nombre, cliente, descripción,
fecha de inicio, fecha fin prevista, plantilla de roles con el esfuerzo semanal estimado por
rol, y un presupuesto opcional.

*When* se confirma la creación de un proyecto, *the system shall* generar una **propuesta de
asignación del agente** (US-04.1) y dejarla en estado `EN_REVISION`, **sin** aplicar ninguna
asignación.

**Valor para el usuario:** crear el proyecto y arrancar con equipo en un solo paso, con
recomendación incluida.

---

## US-02.2 — Card de detalle de proyecto

*When* se hace clic en un proyecto, *the system shall* abrir una card con: fechas y avance,
estado, salud y semáforo de presión, carga vs. capacidad, roles cubiertos y roles faltantes,
equipo con rol de proyecto y FTE de cada persona, hitos, y ausencias que impactan al equipo
en la ventana visible.

*When* se hace clic en una persona dentro de la card, *the system shall* abrir su ficha 360
(US-01.2) sin perder el contexto del proyecto.

**Valor para el usuario:** drill-down de proyecto → persona sin navegación lateral.

---

## US-02.3 — Asignar personas y roles

*When* se asigna una persona a un proyecto, *the system shall* registrar rol de proyecto, FTE,
fechas de inicio/fin y tipo de asignación.

*When* ya existe ese mismo rol para esa persona en ese proyecto, *the system shall* rechazar
el duplicado y explicar que una persona puede tener **varios roles distintos** en un mismo
proyecto.

*When* la suma de FTE de las asignaciones activas de una persona supera su capacidad nominal,
*the system shall* bloquear la operación, salvo que se registre una justificación explícita que
quede auditada.

*When* una persona tiene rol contractual base "Desarrollador" y se le asigna como "Analista"
en un proyecto, *the system shall* permitirlo y reflejarlo en su historial y en su ficha.

**Valor para el usuario:** la realidad de los equipos (multi-rol) cabe en el modelo.

---

## US-02.4 — Timeline con zoom

*When* el usuario cambia el nivel de zoom a **semana**, **quincena**, **mes** o **año**,
*the system shall* reescalar la línea de tiempo de proyectos, hitos y asignaciones, conservando
la legibilidad y sin perder información.

*When* el zoom es semana o quincena, *the system shall* mostrar el detalle diario o por
semana de la asignación (FTE por día) y los hitos con su fecha.

*When* el zoom es mes o año, *the system shall* agregar (no ocultar) hitos y periodos de
ausencia de forma agregada.

**Valor para el usuario:** la misma vista sirve para planificar la semana y para revisar el año.

---

## US-02.5 — Organigrama del proyecto

*When* se abre el organigrama de un proyecto, *the system shall* derivar la jerarquía **a
partir de los roles de proyecto** (no del organigrama de la empresa), mostrar personas y
reportes, y marcar los roles vacantes.

*When* una persona tiene varios roles en el proyecto, *the system shall* mostrarla en cada
uno, con nota de que su rol contractual base es otro.

*When* un becario está asignado, *the system shall* mostrar su mentor y su periodo.

**Valor para el usuario:** ver la estructura del equipo sin cruzarlo con la estructura de la
empresa.

---

## US-02.6 — Semáforo de presión

*When* el avance de un proyecto alcanza el **75 %** (umbral configurable por proyecto) **y** la
carga proyectada supera la capacidad disponible del equipo, *the system shall* marcar el
proyecto como `EN_RIESGO`.

*When* el avance alcanza el 75 % pero **no** hay déficit de capacidad, *the system shall* marcar el
proyecto como `VIGILADO`: informa, pero **no** propone refuerzo, porque el problema no es de falta
de gente sino de mezcla de skills.

*When* la capacidad disponible baja por ausencias o por préstamos temporales vigentes,
*the system shall* recalcular la presión y notificar al manager del proyecto.

*When* un proyecto queda en `EN_RIESGO`, *the system shall* generar una **propuesta de
alivio** (US-04.2, US-04.3) para revisión humana.

**Valor para el usuario:** enterarse de la presión en el 75 %, no en el 95 %.

---

## US-02.7 — Carga de trabajo registrada

*When* se define la plantilla de roles de un proyecto, *the system shall* registrar la
**carga estimada** por rol y periodo, y mostrar la curva demanda vs. capacidad en el
timeline.

*When* se reportan horas o tareas consumidas, *the system shall* contrastarlas con la carga
estimada y mostrar la desviación.

**Valor para el usuario:** el "75 %" se apoya en datos, no en intuición.

---

## Reglas aplicables

`BR-01` (capacidad) · `BR-02` (rol duplicado) · `BR-03` (ausencias) · `BR-04`/`BR-05`
(presión) · `BR-06` (el agente propone, no decide) · `BR-08`/`BR-09`/`BR-10` (préstamo) ·
`BR-11` (becarios no son candidatos a préstamo sin RRHH) · `BR-12` (piso del origen)
