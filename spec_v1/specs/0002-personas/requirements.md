# REQUIREMENTS — EP-01 Personas (RRHH)

Área responsable: requisitos y trazabilidad · Spec: [`../0001-vision/spec.md`](../0001-vision/spec.md) · Diseño: [`../../designs/0002-modelo-datos.md`](../../designs/0002-modelo-datos.md)

Criterios en estilo **EARS**: *When \<evento\>, the system shall \<respuesta\>*.

---

## US-01.1 — Registrar una persona

*When* RRHH da de alta una persona, *the system shall* registrar: nombre, tipo de contrato,
**rol contractual base**, fecha de ingreso, seniority, correo, cumpleaños, capacidad nominal
horaria y habilidades (herramienta + nivel + años de uso).

*When* la persona es becaria, *the system shall* exigir mentor y fecha de fin del periodo de
asignación, y clasificar su tipo de contrato como `BECARIO`.

*When* una persona se registra, *the system shall* crear su línea de tiempo de historial
con el evento inicial `ALTA`.

**Valor para el usuario:** RRHH mantiene un único Maestro de Personas sin duplicar planillas.

---

## US-01.2 — Ficha 360 de una persona

*When* un usuario abre una persona en el Tablero de Personal, *the system shall* mostrar en una
sola vista: antigüedad en la empresa, rol contractual base, seniority, herramientas y nivel,
asignaciones activas, historial completo de proyectos y roles, ausencias, saldos de vacaciones,
préstamos temporales, y próximos cumpleaños.

*When* la persona participa en un proyecto, *the system shall* mostrar su rol **dentro de ese
proyecto** junto con el organigrama correspondiente.

**Valor para el usuario:** responder "¿quién es esta persona y qué puede hacer?" sin pedirle
archivos a nadie.

---

## US-01.3 — Registrar y cubrir ausencias

*When* se registra una **incapacidad**, *the system shall*: (a) marcarla como no asignable en
el rango de fechas; (b) **no** consumir saldo de vacaciones; (c) **exigir** un plan de
cobertura; (d) poner en `SUSPENDIDA` sus asignaciones activas y activar la asignación de
cobertura asociada.

*When* se registran **vacaciones**, *the system shall* descontar el saldo correspondiente y
**recomendar** una cobertura (sin bloquear el registro).

*When* una ausencia se acerca a su fin (≤ 3 días), *the system shall* avisar para la
reincorporación y devolver la capacidad al pool.

*When* se registra un permiso, *the system shall* tratarlo como ausencia no vacationable.

**Valor para el usuario:** nadie se va de vacaciones sin que su proyecto sepa quién lo cubre.

---

## US-01.4 — Pool de desasignados

*When* se filtra por "sin proyecto", *the system shall* listar las personas sin asignación
activa mostrando: rol contractual base, seniority, skills, capacidad libre, fechas de
reincorporación si hubo ausencia reciente, y Antigüedad.

*When* una persona sale de una asignación temporal y no tiene proyecto destino, *the system
shall* reincorporarla automáticamente al pool con su capacidad liberada.

**Valor para el usuario:** el talento ocioso deja de ser invisible.

---

## US-01.5 — Gestión de becarios

*When* se asigna un becario a un proyecto, *the system shall* registrar el periodo de
asignación, el mentor y el tipo `BECARIO`, y mostrarlo en el organigrama del proyecto.

*When* se acerca el fin del periodo de un becario, *the system shall* avisar y **proponer**
(como propuesta del agente) su cierre o conversión.

**Valor para el usuario:** se sabe dónde está cada becario y hasta cuándo.

---

## US-01.6 — Historial append-only

*When* cambia el rol de una persona en un proyecto, su seniority, su tipo de contrato o su
asignación, *the system shall* registrar un evento en su historial inmutable, con fecha,
valor anterior y valor nuevo.

**Valor para el usuario:** "toda la historia en la empresa" no se pierde ni se reescribe.

---

## US-01.7 — Solicitar vacaciones con reconocimiento de aniversario

*When* una persona accede a su portal, *the system shall* permitirle solicitar vacaciones
seleccionando rango de fechas, validando saldo disponible y mostrando cobertura sugerida
antes de enviar a aprobación.

*When* el día actual coincide con el aniversario de ingreso de la persona, *the system shall*
mostrar un **reconocimiento de aniversario** (badge, años cumplidos, mensaje personalizado)
en el portal y en la ficha 360, y registrar el evento en `PersonaHistorial` tipo `ANIVERSARIO`.

*When* la persona completa N años (1, 3, 5, 10, 15, 20), *the system shall* desbloquear un
**logro de antigüedad** visible en su ficha y en el panel del equipo (gamificación).

*When* la persona solicita vacaciones con ≥ 30 días de antelación, *the system shall* otorgar
un **punto de planificación** (badge "Planificador") que suma en su perfil de gamificación.

**Valor para el usuario:** la persona gestiona sus vacaciones sola, ve su aniversario celebrado
y recibe reconocimientos que fomentan la planificación y la permanencia.

---

## Reglas aplicables

## Reglas aplicables

`BR-01` (capacidad ≤ 100 %) · `BR-03` (no asignar en ausencia sin cobertura) ·
`BR-13` (vacaciones consumen saldo, incapacidad no) · `BR-16` (periodo acotado del becario) ·
`BR-17` (mentor obligatorio)
