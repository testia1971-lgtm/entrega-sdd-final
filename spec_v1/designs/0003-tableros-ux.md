# DESIGN 0003 — Tableros (UX)

> **Área responsable: UX y frontend** · Estado: **cerrada** (2026-10-21) · Cierra las 13 decisiones X-01..X-13 que estaban abiertas.
>
> Requisitos que cubre: `US-02.2`, `US-02.4`, `US-02.5`, `US-02.6`, `US-02.7`, `US-03.1`…`US-03.6`, `US-01.2`, `US-01.4`, `US-01.7`, `US-03.7`, `US-04.6`, `US-04.7`.
> Reglas que respeta: `BR-01`…`BR-22` · Invariantes `I-1..I-13` · Constitución (Artículos III, IV, VII, XI).
> Sources: [`specs/0003-proyectos/requirements.md`](../specs/0003-proyectos/requirements.md) ·
> [`specs/0004-tablero-personal/requirements.md`](../specs/0004-tablero-personal/requirements.md) ·
> [`designs/0005-modelo-carga-y-presion.md`](0005-modelo-carga-y-presion.md) (definiciones de carga y presión) ·
> [`designs/0001-arquitectura.md`](0001-arquitectura.md) (stack ylibs) ·
> [`specs/0004-tablero-personal/research-frontend-stack.md`](../specs/0004-tablero-personal/research-frontend-stack.md) (licencias verificadas).

---

## 1. Principio rector: **un motor de timeline, dos tableros**

El cliente pidió "un tablero de proyectos" y "otro tablero por recurso humano" con el mismo nivel de riqueza.
La decisión que lo hace barato (y que evita el doble trabajo) es:

> **Un solo componente `Timeline`, parametrizado por `rowSource: 'proyecto' | 'persona'`.**
> El tablero de proyectos y el tablero por recurso son la misma tabla con distinta origen de fila.

Consecuencias que se pagan solas:
- el **zoom es un solo estado global** (`zoomLevel`, `visibleRange`, `filtersByProject`) → `US-03.6` sale gratis;
- el código de las bandas de ausencia, del resaltado de sobre-asignación y del drag de FTE es **uno solo**;
- el timeline de la ficha 360 (`US-03.3`) es **el mismo componente** con `rowSource='persona'` y una sola fila.

**Anti-objetivo explícito:** no se construyen dos Gantt distintos "porque cada tablero tiene su forma".
Eso duplica el 60 % del código y duplica los bugs.

---

## 2. Inventario de superficies (cerrado por el cliente, con componente asignado)

| # | Superficie | Requisitos | Componente / stack |
|---|---|---|---|
| S-1 | Tablero de Proyectos (timeline con zoom) | US-02.1, US-02.2, US-02.4, US-02.6, US-02.7 | `Timeline` propio (`d3-scale` + `d3-zoom`) + `d3-time` |
| S-2 | Card de proyecto | US-02.2 | `Drawer` lateral derecho + ECharts (sparkline carga vs capacidad) |
| S-3 | Card / ficha 360 de persona | US-01.2, US-03.2 | Vista larga con anclas sticky + ECharts (radar, gauge, heatmap) |
| S-4 | Organigrama de proyecto | US-02.5 | `@xyflow/react` + `@dagrejs/dagre` |
| S-5 | Tablero de Personal (listado + filtros) | US-03.1 | Tabla densa (virtualizada) + `Combobox` de filtros |
| S-6 | Timeline individual de persona | US-03.3 | `Timeline` con `rowSource='persona'` |
| S-7 | Pool de desasignados | US-01.4, US-03.4 | `Kanban`/`Table` propio con estado "en propuesta" |
| S-8 | Bandeja de aprobaciones del coordinador | US-04.6, US-05.4 | Bandeja + detalle de propuesta con evidencia |

---

## 3. Las 12 decisiones

### X-01 · Zoom → **4 anclas + arrastre libre con imantado**

**Decisión.** Las 4 anclas que pidió el cliente (semana / quincena / mes / año) son **botones de ancla**;
entre anclas hay zoom continuo por pellizco o `Ctrl+scroll`, y **al soltar se imanta a la ancla más cercana**.
Se puede desplazar (pan) libremente en cualquier nivel.

**Por qué.** Zoom continuo puro hace ilegible cualquier nivel ("el año" con píxeles por día no se lee) y anclas
puras hacen frustrante el "entre semana y mes". El imantado da lo mejor de ambos: el usuario siempre cae en un
nivel con lectura conocida.

**Cómo se verifica.** `US-02.4`: cambiar el nivel reescala timeline, hitos y asignaciones conservando legibilidad
y sin perder información. Test: para cada ancla, la ventana visible tiene el rango esperado y ningún elemento
queda fuera del viewport sin indicador de scroll.

**Riesgo.** El imantado puede pelearse con el gesto de pan → se desactiva el snap si el usuario arrastra en los
primeros 200 ms tras un zoom.

---

### X-02 · Grano en zoom fino → **fila por persona, agrupable por rol**

**Decisión.** En `semana` y `quincena`, la fila es **una persona asignada al proyecto** (con su rol de proyecto en
la etiqueta). Un toggle **"Agrupar por rol"** colapsa las filas por rol de proyecto, mostrando el agregado.

- La barra es **continua** (no una barra por día): el detalle diario aparece al hover.
- **Hover** sobre una barra: popover con *proyecto · rol de proyecto · FTE · fechas · tipo · estado de préstamo*.
- En `semana`, el eje muestra días laborables y **feriados en gris**; en `quincena`, semanas ISO.

**Por qué.** El cliente preguntó "en qué está asignada esta persona", no "en qué días trabaja". Filas por persona
son las que permiten leer sobre-asignación a simple vista. Y agrupar por rol responde "me falta un QA" de un clic.

**Cómo se verifica.** `US-02.4` (detalle diario o semanal de la asignación), `US-02.7` (curva demanda vs capacidad).

---

### X-03 · Codificación visual de la carga → **capas separadas, un canal por dato**

**Decisión.** Cada canal visual codifica **una sola magnitud** (nunca dos):

| Canal | Codifica | Dónde |
|---|---|---|
| **Color de fondo de la fila/banda** | salud del proyecto (semáforo) | fondo de la swimlane y borde de la card |
| **Opacidad/saturación de la barra** | FTE de la persona en esa asignación | barra de asignación |
| **Banda rayada roja sobre la barra** | sobre-asignación (>100 % de su capacidad en esa ventana) | sobre la barra |
| **Trama diagonal azul + borde doble** | préstamo temporal activo | barra |
| **Contorno punteado gris** | rol vacante | organigrama |

**Por qué.** El error clásico es pintar "la barra verde si el proyecto va bien", y entonces el FTE deja de leerse.
Separar canales hace que **cualquier estado sea legible sin color** (patrón + texto), que es el requisito de
accesibilidad X-11.

**Cómo se verifica.** `US-02.6`, `US-02.7` y el test de accesibilidad de la ficha (no solo color).

---

### X-04 · Semáforo de presión → **4 estados de presión, con umbral visible al lado del número**

**Decisión.** Los estados son **exactamente** los de `designs/0005-modelo-carga-y-presion.md`:

| Estado | Condición | Presentación |
|---|---|---|
| `SALUDABLE` | `avance < 75 %` ∧ `indice_presion ≤ 1.00` | verde + icono `●` |
| `VIGILADO` | `avance ≥ 75 %` ∧ `indice_presion ≤ 1.00` | ámbar + icono `◐` + etiqueta "en la ventana del 75 %" |
| `EN_RIESGO` | `1.00 < indice_presion ≤ 1.25` | naranja + icono `▲` |
| `CRITICO` | `indice_presion > 1.25` | rojo + icono `■` + aviso al manager |

Aparte, `PAUSADO` y `CERRADO` son **estados de ciclo de vida**, no de presión: **no llevan semáforo** (evita el
"proyecto cerrado en rojo" que confunde).

Al lado del porcentaje **siempre** se muestra el umbral aplicado (`≥ 75 %`) y un tooltip con la fórmula y las tres
entradas (`avance`, `deficit`, `capacidad restante`) — porque el cliente pidió entender el 75 %, no solo verlo.

**Por qué.** Cuatro estados y no cinco: el quinto nivel habitual ("amarillo preventivo") no aporta información
accionable; `VIGILADO` ya cubre "hay que mirar" sin generar ruido de propuestas.

**Cómo se verifica.** `US-02.6`: al alcanzar el 75 % de avance se evalúa la carga proyectada vs. capacidad
disponible y se marca `EN_RIESGO` si se supera. Test: casos del golden set (`E-01`, `E-02`) sobre el estado del
semáforo.

---

### X-05 · Card de proyecto → **drawer lateral derecho**, no modal

**Decisión.** Panel lateral derecho (ancho ~480 px), **no modal**, porque `US-02.2` exige que al hacer clic en una
persona dentro de la card se abra su ficha 360 **sin perder el contexto del proyecto**: un modal bloquearía el
tablero, y el contexto perdido es exactamente lo que el requisito quiere evitar.

**Orden de contenido (de arriba abajo):**
1. **Semáforo + índice de presión + umbral** (lo primero que hay que ver).
2. **Carga vs capacidad**: sparkline de las 12 semanas + curva de demanda (`US-02.7`) y la desviación estimado/real.
3. **Roles cubiertos / roles faltantes** (chips; los vacantes van marcados).
4. **Equipo**: tabla persona · rol de proyecto · FTE · tipo · fechas. Click → ficha 360.
5. **Hitos** de la ventana visible.
6. **Ausencias que impactan** al equipo en la ventana visible (solo tipo y rango; el motivo no sale del backend).

**Por qué.** El orden responde a la pregunta que se hace un manager al abrir un proyecto: *¿está en riesgo? ¿por
qué? ¿quién me falta?* Si el equipo fuera primero, nadie llegaría al semáforo.

**Cómo se verifica.** `US-02.2` completo; test E2E: abrir proyecto → click en persona → ficha 360 → volver
conservando zoom y rango.

---

### X-06 · Ficha 360 → **una sola vista larga con anclas**, no pestañas

**Decisión.** El cliente pidió *"muy gráfico y rico de información"* y *"toda la historia en la empresa"*.
Las pestañas esconden precisamente lo que se pidió: con 6 pestañas, "toda la historia" se vuelve 6 visitas.

**Estructura:**
- **Header sticky**: avatar, nombre, rol contractual base, seniority, antigüedad, semáforo de carga, y acciones.
- **Reconocimiento de aniversario (condicional)**: si hoy es el aniversario de ingreso, aparece un **banner celebrativo** con años cumplidos, badge del logro (`ANIVERSARIO_1/3/5/10/15/20`), confeti sutil (respeta `prefers-reduced-motion`) y mensaje personalizado. Se registra en `PersonaHistorial` tipo `ANIVERSARIO`.
- **Logros desbloqueados (gamificación)**: chips de badges ganados (`PLANIFICADOR`, `PRIMERAS_VACACIONES`, `SIN_AUSENCIAS_ANO`, etc.) con tooltip de fecha y contexto. Visibles en la ficha y en el panel del equipo.
- **Anclas de scroll** (sticky) con scroll-spy: *Identidad · Carga · Skills · Asignaciones · Historial · Ausencias · Vacaciones · Cumpleaños · Logros*.
- **Layout de 2 columnas** en desktop: izquierda = Carga (gauge 4/8/12 semanas), Skills (radar + heatmap), Vacaciones (barra de saldo + **botón "Solicitar vacaciones"**), Cumpleaños, Logros; derecha = Asignaciones actuales + **Timeline individual** (`rowSource='persona'`) + ausencias.

**Por qué.** Las dos columnas balancean "cómo está" (estado) y "qué pasó" (historial), que es el eje pedido.

**Cómo se verifica.** `US-03.2` (los 8 puntos en una sola vista integrada).

---

### X-07 · Visualizaciones de skills → **radar (equipo vs rol) + heatmap (persona × herramienta)**

**Decisión.**
- **Radar**: destrezas de la persona vs. destrezas requeridas por el rol del proyecto → responde "¿le falta algo?".
- **Heatmap**: persona × herramienta, nivel 1..5 → responde "¿quién de mi equipo sabe X?".
- **No** se usa "una barra por skill": con 20 skills por persona es ilegible.

**Por qué.** Son las dos preguntas reales de una fábrica de software, y las dos alimentan la búsqueda de candidatos
del agente (que usa el mismo `PersonaSkill`).

**Cómo se verifica.** `US-03.1` (skills principales en la ficha compacta), `US-04.7` (evidencia de la propuesta).

---

### X-08 · Organigrama → **DAG, no árbol**

**Decisión.** `@xyflow/react` + `@dagrejs/dagre` (layout por capas).

- **Grafo, no árbol**: "varios roles por proyecto + un rol contractual base" es **multi-arista**. Un árbol te obliga
  a inventar un nodo padre artificial para el rol base, y ese nodo es una mentira de modelo.
- **Roles vacantes**: nodo punteado gris con etiqueta `VACANTE` + rol requerido.
- **Becarios**: nodo con borde de trazo y su **mentor** como arista punteada + periodo del convenio en el tooltip.
- **Multi-rol**: la misma persona aparece en cada nodo de rol con la nota *"rol contractual base: Analista"*.
- Cada nodo es una **tarjeta** (avatar, nombre, rol de proyecto, barra de FTE), no un círculo.

**Por qué.** El organigrama se **deriva de las asignaciones vigentes** (US-02.5), nunca del organigrama de la
empresa: así no puede contradecir los datos.

**Cómo se verifica.** `US-02.5` (derivación desde roles de proyecto, vacantes marcados, multi-rol con nota,
mentor y periodo de becario).

---

### X-09 · Vacaciones y cumpleaños → **ambos, y con roles distintos + autogestión + gamificación**

**Decisión.**
- **Vacaciones**: barra de saldo segmentada (*usado · programado · disponible*) + **heatmap de calendario del año**
  (días por mes) + chip con el próximo periodo. El saldo vive en la ficha; el detalle, en un drawer.
- **Autogestión de vacaciones (nuevo)**: en la ficha 360 y en el portal de la persona, un **botón "Solicitar vacaciones"**
  abre un drawer con selector de rango, validación de saldo en tiempo real, **cobertura sugerida** (según `BR-13`/`BR-14`)
  y envío a aprobación RRHH (estado `SOLICITADA`). Si la antelación es ≥ 30 días, se muestra el badge **"Planificador"**
  y se otorga 1 punto de gamificación (`SolicitudVacaciones.puntosPlanificacion = 1`).
- **Cumpleaños**: **no** en la ficha 360 (es de la persona) sino en un **panel del equipo** (los del mes en curso
  y los próximos 30 días), con aviso a los managers. En la ficha aparece solo la fecha y la edad.
- **Reconocimiento de aniversario (gamificación)**: cuando la persona entra al portal o se abre su ficha 360
  en su día de aniversario de ingreso, se muestra un **banner celebrativo** (años cumplidos, badge del logro
  correspondiente `ANIVERSARIO_1/3/5/10/15/20`, mensaje personalizado). El logro se desbloquea automáticamente
  y queda visible en la ficha y en el panel del equipo (`LogroPersona`). Se registra en `PersonaHistorial` tipo `ANIVERSARIO`.
- **Catálogo de logros**: `ANIVERSARIO_1` (1 año), `ANIVERSARIO_3` (3 años), `ANIVERSARIO_5` (5 años/Quinquenio),
  `ANIVERSARIO_10` (10 años/Década), `ANIVERSARIO_15` (15 años), `ANIVERSARIO_20` (20 años),
  `PLANIFICADOR` (vacaciones con ≥ 30 días antelación), `PRIMERAS_VACACIONES` (primeras vacaciones solicitadas),
  `SIN_AUSENCIAS_ANO` (año completo sin ausencias no planificadas). Cada logro tiene `icono`, `puntos` y `categoria`.

**Por qué.** Mezclar cumpleaños de otros con la ficha individual es ruido; el cliente lo pidió como información de
equipo. La autogestión de vacaciones reduce carga a RRHH y da visibilidad a la persona. La gamificación (aniversarios,
planificación) hace atractivo el portal y fomenta la permanencia y la planificación anticipada — sin competir con
el modelo de asignaciones (BR-20: los puntos **no** afectan scoring del agente).

**Cómo se verifica.** `US-01.7`, `US-03.2` (puntos 5, 6 y nuevos), `US-03.7`: portal muestra saldo + botón solicitar;
aniversario muestra banner + logro; solicitud con ≥ 30 días muestra badge Planificador; logros aparecen en ficha y panel.

---

### X-10 · Préstamo visible → **marca en la barra + sombra en el origen**

**Decisión.** En el timeline:
- La barra del **préstamo**: **trama diagonal azul + borde doble** + tooltip con
  `PRÉSTAMO → <proyecto destino> · hasta <fecha de devolución>`.
- En el **proyecto origen**: la porción cedida se dibuja como una **zona rayada** sobre la barra original, para que
  se vea que esa capacidad ya no está disponible.
- En la ficha 360: un **chip de préstamo** con la fecha de devolución y un contador de días restantes.

**Por qué.** Es el requisito explícito del cliente ("que el Agente cuide que se respete la asignación temporal"):
la UI tiene que hacer **visible** que esa capacidad está comprometida y hasta cuándo.

**Cómo se verifica.** `US-04.3`, `US-04.4`, y `BR-08`/`BR-12` (piso de reserva visible en la ficha del proyecto
origen).

---

### X-11 · Accesibilidad → **el semáforo nunca es solo color**

**Decisión.**
- **Contraste AA** en texto y en los estados del semáforo.
- **Codificación redundante**: cada estado del semáforo lleva **ícono + patrón + texto**, no solo color
  (`●` `◐` `▲` `■`, con trama distinta en el fondo de la swimlane).
- **Modo daltónico**: paleta alternativa con contraste verificado; la trama reemplaza al color como canal principal.
- **Teclado**: `Tab` recorre filas; `←/→` mueve el foco entre barras de la fila; `Enter` abre la card; `Esc` cierra.
  El timeline es navegable **sin ratón**.
- `prefers-reduced-motion` respectedo (las transiciones de pan/zoom se desactivan).
- Foco visible en todo elemento interactivo.

**Por qué.** El semáforo decide sobre la carga de personas; si un usuario daltónico o con baja visión no puede
leerlo, el sistema aplica presión de forma invisible. No es un extra: es corrección.

**Cómo se verifica.** Auditoría de contraste automatizada + test de navegación por teclado sobre el timeline.

---

### X-12 · Tema, densidad y tamaños → **desktop-first, alta densidad, claro/oscuro**

**Decisión.**
- **Escritorio primero (≥ 1280 px)**: es una herramienta de planificación, no una landing.
- **Densidad alta** por defecto (filas de 28–32 px): en una fábrica de software se ven 40–60 personas por pantalla.
- **Claro y oscuro** con el mismo sistema de tokens; el oscuro es el que se usa 8 h seguidas.
- **Responsive**: entre 768 y 1280 px el timeline se degrada a **vista de lista** (no se intenta comprimir el Gantt);
  por debajo de 768 px **no hay timeline** (el cliente pidió web; la app móvil no está en alcance).

**Por qué.** Intentar un Gantt usable en 375 px cuesta más que toda la vista de lista y queda mal en la demo.

**Cómo se verifica.** `NFR-01` (interacción < 100 ms con 100 proyectos y 500 asignaciones) y `NFR-02`
(carga inicial < 1,5 s).

---

## 4. Sistema visual

| Token | Valor | Uso |
|---|---|---|
| `color.saludable` | verde `#16a34a` | semáforo |
| `color.vigilado` | ámbar `#d97706` | semáforo |
| `color.en_riesgo` | naranja `#ea580c` | semáforo |
| `color.critico` | rojo `#dc2626` | semáforo |
| `color.prestamo` | azul `#2563eb` + trama diagonal | préstamo temporal |
| `color.hueco` | gris `#9ca3af` + punteado | rol vacante |
| `color.ausencia` | banda lavanda `#c4b5fd` (35 % opacidad) | vacaciones / incapacidad |
| `color.becario` | turquesa `#0d9488` + borde de trazo | becario |
| `patron.sobreasignado` | rayado rojo 45° | FTE > 100 % |

- **Tipografía**: `Inter` (UI) + `JetBrains Mono` (FTE, fechas y números) — las cifras van en monoespaciada para
  que el ojo las compare en vertical.
- **Un solo color de acento** para lo interactivo (`#2563eb`); el color se reserva para **estado**, no para decoración.
- **Iconografía**: `lucide-react` (MIT, tree-shaking por icono).
- **Tema**: tokens CSS en `:root` + `[data-theme='dark']`; el backend manda el rol del usuario, el tema es local.

---

## 5. Especificación de las superficies

### S-1 · Tablero de Proyectos
- **Arriba**: barra de zoom (`Semana | Quincena | Mes | Año`), selector de rango, filtros (salud, cliente, rol),
  y búsqueda.
- **Izquierda**: swimlanes por proyecto (colapsables), con **franja de calor** de presión por semana (sparkline
  vertical) y semáforo a la izquierda de cada franja.
- **Centro**: las franjas del proyecto; dentro, una fila por persona (o por rol, si el toggle está activo).
- **Sobre las franjas**: banda de **curva de demanda** vs. **capacidad** (línea vs. área), que es la lectura
  "el 75 % se apoya en datos, no en intuición" (US-02.7).
- **Abajo**: **ruler** de hitos (triángulos) para no perderlos en zoom fino.
- **Clic en proyecto** → drawer S-2. **Clic en persona** → ficha 360.

### S-2 · Card de proyecto (drawer)
Ver X-05. Encima del drawer, una **banda de contexto** con el zoom y el rango visibles, para que el usuario sepa
en qué ventana está mirando.

### S-3 · Ficha 360
Ver X-06. Regla dura: **los datos de salud (incapacidad) solo se muestran a quien tiene permiso `RRHH`**; para
el resto, la ausencia aparece como "ausente · del X al Y" sin tipo (BR-20, US-05.5).

### S-4 · Organigrama
Ver X-08. Leyenda explícita: `▢ rol de proyecto` · `◇ rol vacante` · `▨ becario` · `▨⌒ mentor (convenio)`.
Botón "derivar de asignaciones vigentes" que regenera el grafo y **explica qué cambió** (nodo nuevo, nodo vacío).

### S-5 · Tablero de Personal
Tabla densa y virtualizada: avatar · nombre · rol base · seniority · **barra de utilización 4/8/12 semanas** ·
skills principales (3 chips) · asignación actual · estado (libre / asignado / en ausencia / en propuesta).
Filtros combinables en una barra superior; la búsqueda libre cubre nombre, rol y skill (US-03.1).

### S-6 · Timeline individual
`Timeline` con `rowSource='persona'` y una fila. **Comparación** (US-03.5): al seleccionar 2+ personas, el mismo
componente apila las filas alineadas por fecha y marca las bandas de solapamiento.

### S-7 · Pool de desasignados
Tarjetas (no tabla): avatar, rol base, **capacidad libre en %**, skills, y estado. Si hay una propuesta del agente
para esa persona, la tarjeta pasa a **"en propuesta"** con un borde azul y un link a la propuesta — **sigue en el
pool** (US-03.4).

### S-8 · Bandeja de aprobaciones
Lista de propuestas con: proyecto, impacto (presión antes → después), botones **Aprobar / Rechazar (con motivo) /
Editar**. Al abrir una propuesta: los ítems con su **razón**, la **evidencia** (ids de persona/proyecto/asignación)
y **las alternativas descartadas con su motivo** (US-04.7). El rechazo con motivo **se devuelve al agente** para
que re-plantee en la misma corrida (mecanismo A de `designs/0004` §4.1).

---

## 6. Mockups (wireframes en texto)

### S-1 · Tablero de proyectos — zoom "mes"
```
┌───────────────────────────────────────────────────────────────────────────────────────────┐
│ [Semana][Quincena][Mes][Año]  ◀ 2026-05 ──── 2026-12 ▶   filtros:salud▾ cliente▾ rol▾ 🔍 │
├──────────────┬────────────────────────────────────────────────────────────────────────────┤
│ PROYECTOS    │      MAY        JUN        JUL        AGO        SEP        OCT            │
│              │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐       │
│ ● Banco ▓▓▓  │  │▓▓▓▓▓▓▓│ │░░░░░░░░│ │        │ │        │ │        │ │        │       │
│   ALFA   92% │  │ Ana ▓▓ │ │ vacations│ │        │ │        │ │        │ │        │       │
│   ALFA  1.31 │  │ Luis ▓▓│ │ (aus.) │ │        │ │        │ │        │ │        │       │
│              │  ├────────┤ ├────────┤ ├────────┤ ├────────┤ ├────────┤ ├────────┤       │
│ ◐ Tienda ▓▒  │  │ demanda│ │        │ │        │ │        │ │        │ │        │       │
│   BETA   78% │  │ (area) │ │        │ │        │ │        │ │        │ │        │       │
│   BETA  1.02 │  │        │ │        │ │        │ │        │ │        │ │        │       │
│              │  │        │ │        │ │        │ │        │ │        │ │        │       │
│ ◇ Startup    │  │        │ │        │ │        │ │        │ │        │ │        │       │
│   VACANTE    │  │        │ │        │ │        │ │        │ │        │ │        │       │
├──────────────┴────────────────────────────────────────────────────────────────────────────┤
│  ▲hitos:  12/05  30/06  15/09                              ← ruler de hitos                │
└───────────────────────────────────────────────────────────────────────────────────────────┘
```
Leyenda: `▓` asignación · `░` banda de ausencia · `▨` préstamo (trama azul) · `▨▨` sobre-asignación (rayado rojo).

### S-2 · Card de proyecto (drawer derecho)
```
┌──────────────────────────────┬───────────────────────────────┐
│                              │  ALFA · Cliente Banco          │
│   (el tablero sigue          │  ▲ EN_RIESGO · índice 1.18     │
│    visible detrás)           │  ventana 75 % · avance 78 %    │
│                              │  ┌─────────────────────────┐  │
│                              │  │ carga vs capacidad (12s)│  │
│                              │  │ ▁▂▄▆█▇▆▅▃▂            │  │
│                              │  └─────────────────────────┘  │
│                              │  Roles: ✔ Backend ✔ QA ✖ Lead │
│                              │  ┌────┬────────┬────┬────────┐ │
│                              │  │Ana│ Backend│80% │ 12/05→ │ │
│                              │  │Luis│ QA    │60% │ 30/09→ │ │
│                              │  │Eva │ Lead  │40% │ 01/06→ │ │
│                              │  └────┴────────┴────┴────────┘ │
│                              │  Ausencias que impactan:      │
│                              │  · Luis · 05/07 → 19/07        │
│                              └───────────────────────────────┘
```

### S-3 · Ficha 360 (vista larga con anclas)
```
┌──────────────────────────────────────────────────────────────────────────────┐
│ (A) Ana Ruiz  [Backend Developer]  MID   ● LIBRE 42%   Antigüedad 3a 2m    │
│ Identidad · Carga · Skills · Asignaciones · Historial · Ausencias · Vac.    │
├────────────────────────────────┬─────────────────────────────────────────────┤
│ CARGA                          │ ASIGNACIONES ACTUALES                       │
│  ╭──╮ 4s 62%  8s 71% 12s 78% │  ALFA  Backend 80%  12/05→30/09             │
│  │  │                          │  BETA  QA      40%  01/10→31/12  ⧗ préstamo│
│  ╰──╯                          │  ─────────────────────────────────────────── │
│ SKILLS (radar)  REQUISITOS     │ HISTORIAL (Timeline rowSource=persona)      │
│   React  ▰▰▰▰▱                │  2024 ──┬── 2025 ──┬── 2026 ──┬──            │
│   Python ▰▰▱▱▱   Backend  ▰▰▰ │  ALFA ▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░              │
│   Postgres ▰▰▰▱▱             │      ↑ alta   ↑ ascenso  ↑ préstamo         │
│ SKILLS (heatmap persona×tool) │ ─────────────────────────────────────────── │
│ VACACIONES  ▓▓▓▓▓░░░ 14/22   │ AUSENCIAS: (solo RRHH ve el tipo)            │
└────────────────────────────────┴─────────────────────────────────────────────┘
```

### S-4 · Organigrama (DAG)
```
        ┌─────────────┐
        │ Tech Lead   │   (rol vacío → nodo punteado)
        │   VACANTE   │
        └──────┬──────┘
               │
   ┌───────────┴───────────┐
   │                       │
┌──▼────────┐        ┌─────▼──────┐        ┌──────────────┐
│ Ana Ruiz  │        │ Luis Paz   │        │ ▨ Eva (bec.) │
│ Backend   │        │ QA         │        │  mentor: Luis │
│ 80%       │        │ 60%        │        │  hasta 30/11  │
└───────────┘        └────────────┘        └──────┬───────┘
                                                   ┊ (punteada)
                                        ┌──────────▼───────┐
                                        │ Luis Paz (mentor)│
                                        └──────────────────┘
```

---

## 7. Reglas de consistencia (de la spec de requisitos, más las que agrega este diseño)

1. **El zoom es estado global**: los dos tableros comparten escala, rango visible y filtro de proyecto (US-03.6).
2. **Toda vista de persona es navegable desde la vista de proyecto y viceversa** sin perder el contexto.
3. **Lo que el agente propone se ve distinto de lo que está aprobado** (borrador vs. firme), y siempre lleva su
   justificación a un clic (US-04.7).
4. **Nada se oculta sin aviso**: si un filtro oculta datos sensibles, se indica el motivo.
5. **Los datos de salud (incapacidades) no aparecen en superficies de consulta general**; solo en la ficha de la
   persona con permiso y en los tableros de RRHH (BR-20, US-05.5).
6. *(nueva)* **Un solo motor de timeline**: el tablero de proyecto, el de recurso y el de la ficha 360 son el
   mismo componente con distinto `rowSource`.
7. *(nueva)* **Un canal visual por magnitud**: color = salud, opacidad = FTE, trama = préstamo, rayado = sobre-asignación.
   Si hay que elegir entre "bonito" y "legible sin color", gana legible.
8. *(nueva)* **Ninguna cifra sin su definición a un clic**: todo porcentaje mostrado lleva el umbral y la fórmula
   en el tooltip (índice de presión, utilización, saldo de vacaciones).

---

## 8. Rendimiento (para cumplir NFR-01 / NFR-02)

| Problema | Solución |
|---|---|
| 500 asignaciones en pantalla | **virtualización por filas** (solo se pintan las visibles) |
| Zoom "año" con 365 días | **agregación server-side** por `proyecto × mes`; el cliente pide `from/to/zoom` en la query |
| Zoom "mes" con 30 días | agregado por `proyecto × semana` |
| Curva demanda vs capacidad | se calcula en el backend (motor), no en el navegador |
| Recarga por cambios del agente | **SSE**: un stream multiplexado por pestaña con `Last-Event-ID` |
| Overlays de ausencia | se pintan como bandas, no como filas |

Payload del timeline (ejemplo):
```json
{ "zoom": "mes", "from": "2026-05-01", "to": "2026-12-31",
  "rows": [ { "id": "p-01", "projectId": "pr-01", "segments": [
      { "personId": "pe-12", "roleId": "r-03", "fte": 0.8,
        "from": "2026-05-12", "to": "2026-09-30", "type": "PERMANENTE" } ] } ],
  "bands": [ { "kind": "ABSENCE", "personId": "pe-13", "from": "2026-07-05", "to": "2026-07-19" } ],
  "demand": [ { "week": "2026-W18", "roleId": "r-03", "hours": 96, "capacity": 80 } ],
  "meta": { "pressure": { "pr-01": { "state": "EN_RIESGO", "index": 1.18, "threshold": 0.75 } } } }
```
> Nota: `pressure.state` e `index` los produce el **motor determinista** (`designs/0005`), nunca el LLM
> (constitution Art. III y IV; AGENTS.md §1.3).

---

## 9. Trazabilidad de este diseño

| Decisión | Requisitos que habilita | Verificación |
|---|---|---|
| X-01 zoom con anclas | US-02.4 | test de rescale por nivel |
| X-02 fila por persona | US-02.4, US-03.3 | snapshot de filas por nivel de zoom |
| X-03 canales separados | US-02.6, US-02.7 | test de color + patrón por estado |
| X-04 semáforo de 4 | US-02.6 | golden set E-01/E-02 |
| X-05 drawer de proyecto | US-02.2 | E2E drill-down proyecto → persona → volver |
| X-06 ficha 360 larga | US-03.2 | E2E de los 8 puntos en una vista |
| X-07 radar + heatmap | US-03.1, US-04.7 | screenshot + test de datos del chart |
| X-08 DAG | US-02.5 | test de nodos vacantes y multi-rol |
| X-09 vacaciones + cumpleaños | US-03.2, US-03.1 | test de saldo y de panel de equipo |
| X-10 préstamo visible | US-04.3, US-04.4 | E2E de préstamo → marca en origen y destino |
| X-11 accesibilidad | transversal | auditoría de contraste + test de teclado |
| X-12 tema y densidad | transversal | NFR-01, NFR-02 |

---

## 10. Handoff — estado y pendientes

**Cerrado por este documento:** las 13 decisiones X-01..X-13, los mockups de S-1..S-4, el sistema visual, las reglas
de rendimiento y la matriz de trazabilidad.

**Pendientes (no bloquean este design):**
1. **Mockups en imagen**: lo entregado son wireframes en texto (los más útiles para discutir estructura). Si el
   cliente quiere wireframes visuales, se pueden derivar de estas mismas cajas — o usar `archify` para generar un
   diagrama navegable de las superficies.
2. **`docs/PROPUESTA_FINAL.md` §9** (área de ensamblado final): incorporar la descripción de los tableros que queda
   en §5-§6 de este documento.
3. **Coherencia con `designs/0005`**: las definiciones de carga, índice de presión y estados del semáforo ya están
   alineadas (§3, X-04 y §8 de este documento). Si 0005 cambia, hay que revalidar X-03 y X-04.
4. **D-07 (autenticación) y D-08 (despliegue)** siguen abiertas, pendientes de confirmación del cliente: la ficha 360
   depende del rol (`RRHH` ve el tipo de ausencia; el resto no), así que el diseño está listo pero necesita el modelo
   de roles para cerrarse.
