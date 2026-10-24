> **BORRADOR ALTERNATIVO - no es la version canonica.**
> Este archivo es un borrador alternativo escrito en paralelo en el mismo workspace, preservado tal cual
> para no perder trabajo. La version canonica de `designs/0003-tableros-ux.md` es la que
> **responde explicitamente las 13 decisiones X-01..X-13** que la seccion tiene abiertas, e incluye mockups
> S-1..S-4, sistema de tokens y la matriz X -> US -> test.
> Si algo de este borrador es mejor (o no esta en la canonica), corresponde fusionarlo a mano: **no** sobreescribir
> el canonico sin avisar en el canal.

---
# Design 0003 — Tableros y UX

> Área responsable: UX y frontend · Estado: **borrador para revisión** · Revisa la §4.1 de `designs/0004-agente-coordinador.md` (runtime del
> agente) y `designs/0002-modelo-datos.md` (entidades) · Cierra el requisito de "visualmente rico" del enunciado.
> Depende del stack ya decidido: React + TS + Vite · FastAPI + Python · MongoDB.
> Research de librerías y licencias: `specs/0004-tablero-personal/research-frontend-stack.md`.

---

## 0. La decisión que define esta UX

**Un solo componente de timeline, parametrizado por "origen de fila" (`proyecto | persona`).**

```
<Timeline rowSource={proyecto | persona} />
```

De ahí salen los dos tableros que pide el cliente: el **tablero de proyectos** (fila = proyecto, barras =
asignaciones de las personas) y el **tablero de recurso** (fila = persona, barras = sus asignaciones). No son dos
implementaciones parecidas: son **la misma con otro parámetro**. Es lo que hace que el alcance entre en un proyecto
final, y por eso el timeline es **propio** (capa fina sobre `d3-scale` + `d3-zoom`) en vez de una librería que no
sabe dibujar lo que necesitamos.

**Por qué no una librería Gantt:** ninguna de código abierto da las tres cosas que hacen "visualmente rico" este
tablero — filas por recurso con avatar, bandas de ausencia de fondo, y resaltado de sobre-asignación por
solapamiento. La que las tiene (DHTMLX Gantt) las tiene **en la versión PRO**. Ver research.

---

## 1. Superficies

| # | Superficie | Qué resuelve para el cliente | MVP |
|---|---|---|---|
| **A** | **Tablero de proyectos** | Proyectos con su gente, duración, zoom semana/quincena/mes/año, cards de detalle | Sí |
| **B** | **Panel de recurso** | Listado de personal y, al elegir a alguien, su tablero: antigüedad, skills, proyectos, vacaciones, cumpleaños, historial | Sí (lite) |
| **C** | **Organigrama de proyecto** | Quién reporta a quién y con qué rol dentro de cada proyecto | Fase 2 |
| **D** | **Banco de disponibles** | RH desasignados, listos para asignar de inmediato | Fase 2 |
| **E** | **Bandeja del agente** | Propuestas y alertas pendientes de decisión humana | Sí |

---

## 2. Tablero de proyectos (A)

### 2.1 Estructura de filas

Dos modos de agrupación, conmutables:

- **Por proyecto** (default): una fila por proyecto, barra = periodo del proyecto, chips de las personas asignadas.
- **Por persona dentro de un proyecto**: al expandir un proyecto, una subfila por persona con la barra de su
  asignación (rol + % + rango). Esta es la vista que hace visible el conflicto de carga.

### 2.2 Zoom

Un solo `scaleTime` + `scaleBand` por nivel. La API del tablero recibe `from`/`to`/`zoom` y **devuelve solo lo
visible**.

| Nivel | `scaleBand` | Se ve | Uso principal |
|---|---|---|---|
| **Semana** | 1 día | 7 columnas, horas dentro del día | planning de esta semana |
| **Quincena** | 1–2 días | 14–15 columnas | planificación de sprint |
| **Mes** | 1 semana | 4–5 columnas | vista de equipo |
| **Trimestre** | 1 mes | 3 columnas | capacidad por proyecto |
| **Año** | 1 mes (agregado) | 12 columnas | cartera; **agregación server-side** por proyecto-mes |

> En zoom año el backend **agrega** (proyecto-mes: carga consumida, capacidad, presión) en vez de devolver cientos de
> asignaciones. El front nunca dibuja el histórico completo.

### 2.3 Codificación visual

Un solo sistema de color para toda la app:

| Elemento | Señal | Por qué |
|---|---|---|
| Asignación permanente | Barra sólida, color del proyecto | Estado normal |
| **Asignación temporal (préstamo)** | **Borde punteado + icono de ↔ + etiqueta "préstamo"** + color del proyecto destino | Requisito explícito: "marcarlos de algún modo" |
| Fin del préstamo | Marca vertical (⌁) con la fecha de devolución | "el agente cuide que se respete la asignación temporal" |
| **Vacaciones / incapacidad** | **Banda de fondo rayada** en la fila de la persona (gris = vacaciones, ámbar rayado = incapacidad) | Overlay de disponibilidad |
| Periodo de becario | Banda con borde punteado + etiqueta del convenio | Requisito: "saber de becarios, dónde están, periodo" |
| **Sobre-asignación** | **Banda roja sobre el tramo solapado** + chip 🔴 en el proyecto | Requisito: ver dónde se vuelve crítico |
| Presión del proyecto | Semáforo en la barra: 🟢 SALUDABLE · 🟡 VIGILADO · 🟠 EN_RIESGO · 🔴 CRITICO | Requisito: "que el agente identifique dónde se vuelve crítico" |
| Rol en el proyecto | Chip de texto sobre la barra (Dev, QA, Scrum Master…) | "una persona puede tener varios roles por proyecto" |
| % de asignación | Ancho de la barra + número (100 %, 50 %, 40 %) | Carga real |

> **Regla dura de color:** ningún color se asigna a una persona (evita estigmatizar); el color identifica
> **proyecto**, y el estado de la persona se marca con forma/borde/rayado. Es la misma regla que BR-20
> (no usar atributos sensibles para evaluar personas) aplicada al diseño visual.

### 2.4 Interacción

| Gesto | Resultado | Regla |
|---|---|---|
| Click en la barra de un proyecto | Card de detalle: cliente, fechas, estado, roles requeridos vs. cubiertos, carga, presión, ausencias activas, propuestas del agente | La card es un **modal lateral**, no una vista aparte |
| Click en una persona (barra o chip) | Panel de recurso (§3) | El panel se abre sobre el tablero, sin perder el contexto |
| Click en la banda de ausencia | Tooltip: tipo, rango, días, cobertura | Nunca se muestra el motivo médico (§4) |
| Arrastrar una barra | Propuesta de cambio de fecha | **No escribe**: crea una propuesta en la bandeja del agente, con el motivo en el `AuditLog` |
| Zoom con rueda / pinza | Cambia `zoom` y recarga la ventana | La URL guarda el estado (compartible) |
| Hover sobre un tramo rojo | Por qué: "120 % de asignación entre X y Y" | La explicación la da el motor, no el modelo |

### 2.5 Presión: qué se ve sin abrir nada

Un **mapa de calor bajo el timeline**: una fila por proyecto, una celda por mes, color por `indice_presion`. Es la
respuesta de un vistazo a "¿dónde está la presión de la fábrica?" y funciona incluso en zoom año.

---

## 3. Panel de recurso (B)

### 3.1 Listado

Tabla densa con filtros combinables: rol, seniority, estado (disponible / en proyecto / en préstamo / ausente),
skill, rango de capacidad libre, antigüedad. Orden por defecto: **capacidad libre ascendente** (los disponibles
primero), que es lo que el coordinator necesita.

### 3.2 Tablero de la persona (al hacer click)

Rejilla de tarjetas, no un scroll infinito:

| Tarjeta | Contenido |
|---|---|
| **Identidad** | Avatar, nombre, rol contractual base, seniority, fecha de ingreso, **antigüedad calculada**, años en la empresa |
| **Carga** | Gauge 0–160 % con zona roja de sobre-asignación; % comprometido por proyecto |
| **Capas de skill** | Radar: proficiency por categoría (lenguaje, framework, metodología, herramienta) contra el máximo del equipo — responde "qué herramientas conoce" |
| **Historial** | Timeline cronológico de asignaciones (proyecto, rol, rango, %) — "toda la historia en la empresa" |
| **Ausencias** | Vacaciones (tomadas y por tomar, con saldo contra la asignación), incapacidades (tipo + rango, **sin motivo**), Scatter de días por mes |
| **Préstamos** | Prestamos vigentes con % cedido, proyecto origen y **fecha de devolución** |
| **Cumpleaños** | Próximo cumpleaños, edad, si cae en día hábil |
| **Proyectos** | Tarjetas de los proyectos actuales, con su rol y su semáforo |
| **Vacaciones (autogestión)** | **Botón "Solicitar vacaciones"** → drawer con selector de rango, validación de saldo, cobertura sugerida; badge **"Planificador"** si antelación ≥ 30 días (punto gamificación) |
| **Logros (gamificación)** | Badges desbloqueados: `ANIVERSARIO_1/3/5/10/15/20`, `PLANIFICADOR`, `PRIMERAS_VACACIONES`, `SIN_AUSENCIAS_ANO` — con icono, fecha y contexto |
| **Aniversario (reconocimiento)** | Banner celebrativo condicional en el día del aniversario de ingreso: años, badge, mensaje, confeti (respeta `prefers-reduced-motion`) |

> **Privacidad:** el motivo de una incapacidad es dato de salud. En la UI solo se muestra tipo y rango; el texto
> libre vive en el backend y **nunca sale hacia el modelo ni hacia el cliente** sin permiso (C11-NFR4).

### 3.3 Acciones

- **"Ver en el tablero"** → abre el tab timeline filtrado por esa persona (mismo componente, `rowSource=persona`).
- **"Proponer asignación"** → genera una solicitud para el agente coordinator, no una asignación directa.

---

## 4. Reglas de privacidad en la UI

| Dato | Visible para | Nunca visible |
|---|---|---|
| Motivo/diagnóstico de incapacidad | RRHH (backend, con permiso) | En tableros, en el prompt del agente, en las logs del modelo |
| Fecha de nacimiento exacta | Solo día/mes en la UI | El día exacto si el cliente decide así (pregunta abierta PII) |
| Antigüedad, seniority, skills | Todos | Nunca usados como criterio de scoring de candidatos (BR-20) |

---

## 5. Estados de la interfaz

| Estado | Treatment |
|---|---|
| Carga | Skeleton con la geometría real (filas y barras fantasma) → sin salto de layout al llegar los datos |
| Sin datos | Empty state con acción: "crear proyecto", "cargar seed", "importar CSV" |
| Error de API | Banner con reintento y el id de error; nunca pantalla en blanco |
| Proposal pendiente | Chip en la barra affected + badge en la bandeja del agente |
| Agente pausado | Banner global "agente en pausa" (kill switch, C11-R11) |

---

## 6. Accesibilidad

- Contraste AA en todos los chips y estados; **el color nunca es el único portador de información** (el semáforo
  lleva además etiqueta de texto).
- Foco visible y navegable por teclado en el timeline; las barras son `role="button"` con nombre accesible
  ("Ana Ruiz — Tester — Proyecto X — 60 % — del 3 al 28 de marzo").
- Alternativa tabular: la misma información en una tabla, para lectores de pantalla y para exportar.
- `prefers-reduced-motion` respetado en las animaciones de drag y de los estados del agente.

---

## 7. Rendimiento

| Métrica | Objetivo |
|---|---|
| Payload del tablero | Solo la ventana visible (`from`/`to`/`zoom`); la API devuelve lo agregado en zoom año |
| Interaction | Pan/zoom a 60 fps con 12 proyectos × 56–120 personas × 2 años |
| Time to first paint del tablero | < 1,5 s en la red de la demo |
| Bundle | ECharts tree-shakeado (`echarts/core`, ~120–150 KB gzip), timeline y `@xyflow/react` en chunks lazy por tablero; objetivo **< 450 KB gzip** inicial |
| Updates del agente | SSE con 1 stream multiplexado por pestaña (límite de 6 conexiones SSE por origen en HTTP/1.1 → una sola, y HTTP/2 en el borde) |

**Renderizado:** las barras son `div`s posicionados en absoluto (no SVG) → texto, avatares y hover sin pelearse con
las métricas de texto del SVG. Solo las bandas de fondo y las marcas de tiempo van en SVG.

---

## 8. Datos que consume cada superficie

| Superficie | Endpoint | Modo |
|---|---|---|
| A · Tablero | `GET /api/board?from&to&zoom&group=proyecto\|persona&project_id?` | Windowed (obligatorio) |
| B · Listado | `GET /api/people?filters&sort&page` | Paginado |
| B · Panel | `GET /api/people/{id}/dossier` | Payload agregado de una persona |
| C · Organigrama | `GET /api/projects/{id}/org` | Grafo |
| D · Disponibles | `GET /api/pool?from&to&skills` | Windowed |
| E · Bandeja | `GET /api/proposals?status` + `GET /api/alerts?status` | Paginado + SSE |

> Cada respuesta del agente y cada cifra de la UI **debe ser verificable**: el backend nunca devuelve un número
> "aproximado" calculado en el front (Art. III de la constitution y AGENTS.md §1.3).

---

## 9. Wireframes (esquema, no medida exacta)

```
A · TABLERO DE PROYECTOS                       [Semana|Quincena|Mes|Trimestre|Año]  [proyecto ▾]
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │
│ ████████████████████████████  🟠 EN_RIESGO   API Banking · 3 devs, 1 QA, 1 SM           │
│   ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │
│   Ana Ruiz · Tester ····························· 60 %        (punteado = préstamo)      │
│   ╠═══════╗  (banda rayada = vacaciones)                                                  │
│   ███████████████████  🟢 SALUDABLE  Portal Clientes · 4 devs                              │
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│  ← mapa de calor por proyecto-mes (presión)                                               │
└────────────────────────────────────────────────────────────────────────────────────────┘

B · PANEL DE RECURSO                                                                    [x]
┌───────────────────────────────┐
│ (avatar) Ana Ruiz             │
│ Tester · Senior · desde 2023-04│   antigüedad 3 años 2 meses
│ Carga  ▓▓▓▓▓▓▓░░░  120 %  🔴   │
│ Skills (radar)  ▟▙▟▙▟▙▟        │
│ Historial (timeline) ▁▃▅▇▅▃    │
│ Vacaciones  12/20 días · 8 usados│
│ Cumpleaños  14 mar (día hábil)   │
│ Préstamos  → Proyecto Y hasta 30-06│
└───────────────────────────────┘
```

*(El wireframe es esquemático; las medidas exactas salen de `designs/0001-arquitectura.md` y del prototipo.)*

---

## 10. Preguntas abiertas para esta sección

1. ¿La vista default del tablero es **por proyecto** o **por persona**? (Propuesta: por proyecto, con subfilas por
   persona al expandir.)
2. ¿El mapa de calor de presión va siempre visible debajo del timeline o es una vista aparte?
3. ¿Se permite **arrastrar** para proponer cambios, o el cambio de fechas solo se hace desde la ficha? (Arrastrar es
   más lindo pero multiplica los casos de prueba del guardarraíl.)
4. ¿El panel de recurso se abre como modal lateral o como ruta propia (`/people/{id}`)? (Propuesta: ruta propia con
   el tablero embebido arriba, para poder compartir el link.)
5. ¿Idioma de la UI: español, inglés o bilingüe? (La capacitación es en español; el cliente es una fábrica de software
   que puede tener clientes en inglés.)

---

## 11. Trazabilidad

| Requisito del cliente | Donde se resuelve | Verificable por |
|---|---|---|
| "tablero visualmente rico con proyectos y recursos asignados" | §2 | E2E: el tablero carga 19 proyectos con zoom funcional |
| "duración del proyecto con distintos zoom (semana/quincena/mes/año)" | §2.2 | E2E: cambiar de zoom recarga la ventana correcta |
| "click en el proyecto → card con detalles" | §2.4 | E2E: click abre la card con los datos del proyecto |
| "click en un Analista → desde cuándo ingresó, herramientas, experiencia" | §3.2 | E2E: el panel muestra ingreso, radar de skills y seniority |
| "otro tablero por recurso, con antigüedad, proyectos, herramientas, historia, vacaciones, cumpleaños" | §3 | E2E + test de agregados |
| "asignación temporal marcada de algún modo" | §2.3 (borde punteado, ↔, fecha de devolución) | E2E: el préstamo se distingue visualmente y trae su fecha |
| "el agente identifique dónde se vuelve crítico" | §2.3 (semáforo) + §2.5 (mapa de calor) | Golden set: los proyectos EN_RIESGO salen marcados |
| "becarios: dónde están asignados, periodo de asignación" | §2.3 (banda de convenio) | E2E: el becario aparece con su periodo |
| "rol por proyecto + rol base" | §2.3 (chip de rol), §3.2 (rol contractual base) | Test: no se puede ver el rol de otro proyecto |
| "incapacidad: cómo se cubre y en qué periodo se va y regresa" | §2.3 (banda ámbar), §3.2 | E2E: la banda muestra el rango y el tooltip la cobertura |
| "espacio para RH desasignados" | Superficie D | E2E: el pool lista a quien no tiene proyecto |
