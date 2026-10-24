# REQUIREMENTS — EP-03 Tablero de personal

Área responsable: requisitos y trazabilidad · Spec: [`../0001-vision/spec.md`](../0001-vision/spec.md) · Diseño: [`../../designs/0003-tableros-ux.md`](../../designs/0003-tableros-ux.md)

---

## US-03.1 — Listado de personal con filtros

*When* se abre el Tablero de Personal, *the system shall* listar el personal con una ficha
compacta por persona (avatar, nombre, rol contractual base, seniority, skills principales,
asignación actual y estado de disponibilidad).

*When* el usuario filtra, *the system shall* permitir filtrar por: rol, seniority, herramienta,
proyecto, disponibilidad (libre / asignado / en ausencia), estado de asignación
(nómina / temporal / cobertura) y antigüedad mínima.

*When* el usuario busca texto libre, *the system shall* buscar por nombre, rol y skill.

*When* se ordena por antigüedad, disponibilidad o carga, *the system shall* reordenar sin
perder los filtros activos.

**Valor para el usuario:** encontrar a la persona adecuada en segundos.

---

## US-03.2 — Ficha 360 (panel de detalle)

*When* el usuario hace clic en una persona, *the system shall* abrir un panel rico que
muestre, de forma integrada y no en pestañas dispersas:

1. **Identidad y antigüedad** en la empresa (fecha de ingreso, años, seniority, rol contractual base).
2. **Skills y herramientas** con nivel, años de uso y último uso.
3. **Asignaciones actuales**: proyecto, rol de proyecto, FTE, fechas, tipo.
4. **Historial completo** (proyectos atendidos, cambios de rol, ascensos, asignaciones
   temporales, devoluciones, ausencias) en una línea de tiempo.
5. **Vacaciones**: saldo del año, taken, próximo periodo, días consumidos y programados.
6. **Cumpleaños**: próximos cumpleaños de las personas del equipo, con aviso previo.
7. **Ausencias activas e incapacidades** (visibles solo a quien tenga permiso).
8. **Utilización** (4/8/12 semanas) y carga comparada con el equipo.

**Valor para el usuario:** una sola pantalla con "toda la historia en la empresa".

---

## US-03.3 — Línea de tiempo individual

*When* se abre la línea de tiempo de una persona, *the system shall* mostrar sus proyectos,
roles, periodos de alta, ausencias, préstamos temporales y devoluciones, en la **misma escala
de zoom** que el tablero de proyectos.

*When* el zoom cambia, *the system shall* mantener la alineación temporal con el resto de
la aplicación.

**Valor para el usuario:** comparar la historia de una persona con la del proyecto.

---

## US-03.4 — Pool de desasignados

*When* el usuario filtra por "sin proyecto", *the system shall* mostrar un tablero propio de
personas disponibles para asignación inmediata, con su capacidad libre, rol base, skills y
disponibilidad.

*When* el agente propone una asignación para una persona del pool, *the system shall*
reflejarlo en el pool como "en propuesta", sin quitarla del pool hasta la aprobación.

**Valor para el usuario:** "reserva de RH desasignados" como espacio de primera clase.

---

## US-03.5 — Comparación entre personas

*When* el usuario selecciona dos o más personas, *the system shall* alinear sus líneas de
tiempo y mostrar solapamientos de asignaciones, capacidades y solapamiento de skills.

**Valor para el usuario:** decidir a quién reasignar sin hoja de cálculo.

---

## US-03.6 — Timeline compartido y consistente

*When* el usuario navega entre el Tablero de Proyectos y el Tablero de Personal, *the system
shall* conservar el nivel de zoom, el rango de fechas visible y el filtro de proyecto.

**Valor para el usuario:** la aplicación se siente como un solo tablero, no como dos.

---

## US-03.7 — Portal de vacaciones y reconocimiento de aniversario

*When* una persona abre su portal, *the system shall* mostrar su saldo de vacaciones, días
pendientes, próximas vacaciones programadas y un botón para solicitar nuevas vacaciones.

*When* la persona solicita vacaciones, *the system shall* validar saldo, recomendar cobertura
según `BR-13`/`BR-14` y enviar a aprobación de RRHH (estado `SOLICITADA`).

*When* hoy es el aniversario de ingreso de la persona, *the system shall* mostrar en el portal
y en la ficha 360 un **reconocimiento de aniversario** con: años cumplidos, badge del
logro correspondiente (1/3/5/10/15/20 años), y mensaje personalizado.

*When* la persona alcanza hitos de antigüedad (1, 3, 5, 10, 15, 20 años), *the system shall*
mostrar el **logro desbloqueado** en su ficha 360 y en el panel del equipo.

*When* la persona solicita vacaciones con ≥ 30 días de antelación, *the system shall* mostrar
el badge **"Planificador"** y sumar puntos en su perfil de gamificación.

**Valor para el usuario:** autogestión de vacaciones, celebración visible de la permanencia y
reconocimiento por planificar con antelación.

---

## Reglas aplicables

## Reglas aplicables

`BR-01` · `BR-03` · `BR-13` · `BR-18` (las propuestas del agente se muestran como borrador) ·
`BR-19` (auditoría) · `BR-20` (sin atributos sensibles en ordenaciones o scores visibles)
