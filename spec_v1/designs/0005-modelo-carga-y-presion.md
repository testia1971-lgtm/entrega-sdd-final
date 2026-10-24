# Design 0005 — Modelo de carga y presión (y qué significa exactamente el "75 %")

> Área responsable: dominio y agente · Estado: **propuesta de unificación** — requiere visto bueno del cliente
> Problema que resuelve: el enunciado del cliente admite **tres lecturas** del 75 % y las áreas de trabajo
> interpretaron tres cosas distintas. Aquí se formaliza una sola métrica con tres entradas, de modo que las tres
> lecturas queden cubiertas sin contradecirse.

---

## 1. Las tres lecturas del enunciado

> *"buscar de algún modo registrar cargas de trabajo de los proyectos y el Agente identifique donde se vuelve crítico
> (casi siempre al 75% de los proyectos empieza la presión)"*

| # | Lectura | Formulación | Origen |
|---|---|---|---|
| **L1** | Presión por **avance** | "cuando el proyecto llega al 75 % de avance, la presión empieza" | Spec de requisitos (BR-04: `avance ≥ 75 %` ∧ `carga_proyectada > capacidad_disponible` ⇒ `EN_RIESGO`) |
| **L2** | Presión por **carga** | "cuando el 75 % de la capacidad del equipo está comprometida" | `specs/0001-vision/research-sdd-y-dominio.md` §2.2 |
| **L3** | Presión de **portafolio** | "cuando el 75 % de los proyectos de la cartera están en problemas" | Alterna al enunciado |

**Las tres son defendibles y ninguna es falsa por sí sola.** El problema aparece si se implementan como reglas
independientes: entonces el agente puede disparar tres alertas contradictorias sobre el mismo proyecto.

**Regla de resolución adoptada:** el 75 % es un **parámetro de configuración**, no una constante del código, y la
presión se calcula con **una sola métrica** que combina avance y déficit. El 75 % aparece en tres lugares
distintos y con tres significados distintos, y eso es intencional.

---

## 2. Las magnitudes (hechos, no umbrales)

Se separan tres conceptos que la intuición mezcla:

```
avance(p, t)        = (t − p.fecha_inicio) / (p.fecha_fin_prevista − p.fecha_inicio)        # 0..1  (L1)
capacidad_restante(p, t) = Σ_a  a.fte × días_laborables(a ∩ [t, p.fecha_fin_prevista])     # en días-persona
carga_restante(p, t)      = max(0, p.demanda_horas − p.horas_consumidas)                   # de la CargaEstimada
deficit(p, t)             = max(0, carga_restante(p, t) − capacidad_restante(p, t))       # el gap real
indice_presion(p, t)      = deficit(p, t) / max(capacidad_restante(p, t), 1)
```

La distinción load-bearing es esta:

> **El 75 % es una heurística de *cuándo* mirar. El déficit es un *hecho*.**
> Si el agente solo usara "pasó el 75 % → buscar a alguien", inventaría demanda en proyectos que van bien:
> es exactamente el modo de fallo "necesidad alucinada". El agente propone **solo cuando el déficit es real**,
> y usa la ventana del 75 % para subir la **urgencia** de la propuesta, no para crearla.

---

## 3. La métrica única: `indice_presion` + `ventana_75`

```
indice_presion(p, t) = deficit(p, t) / max(capacidad_restante(p, t), 1)      # 0..∞
ventana_75(p, t)     = (avance(p, t) ≥ 0.75)                                    # booleano
en_ventana_75(p,t)   = (avance ≥ 0.75) AND (indice_presion ≤ 1.00)           # "la ventana de presión"
```

### Clasificación (tabla única, configurable)

| Condición | Estado | Qué hace el agente |
|---|---|---|
| `avance < 0.75` ∧ `indice_presion ≤ 1.00` | `SALUDABLE` | Nada |
| `avance ≥ 0.75` ∧ `indice_presion ≤ 1.00` | `VIGILADO` | **Reporta, no propone.** Entró en la ventana de presión pero no hay déficit: el problema, si lo hay, es de mezcla de skills, no de falta de gente |
| `indice_presion ∈ (1.00, 1.25]` | `EN_RIESGO` | **Propone refuerzo** (asignación, préstamo o cobertura) con impacto cuantificado |
| `indice_presion > 1.25` | `CRITICO` | Propone refuerzo **y escala a humano** |
| `deficit = 0` ∧ `avance ≥ 0.75` | `VIGILADO` | Sugiere **rebalanceo de skills**, no más personas |

### Cómo caen las tres lecturas

- **L1 (avance)** → es el eje de la **ventana de presión**: define *cuándo* se mira, no *si* hay problema.
- **L2 (carga)** → es el **índice de presión**: define *cuánta* falta. El umbral por defecto de banda
  (`verde < 70 % ≤ ámbar < 85 % ≤ rojo`) se calcula sobre `carga_comprometida / capacidad_equipo`, que es
  un segundo indicador, más simple, para pintar el tablero.
- **L3 (portafolio)** → es un **indicador de cartera**, aparte: `% de proyectos activos en EN_RIESGO o CRITICO`.
  Con 75 % por defecto dispara una alerta organizacional ("la fábrica está al límite"), que **no** genera propuestas
  por proyecto sino un informe de cartera para dirección.

**Consecuencia:** con una sola tabla de clasificación, las tres lecturas conviven. Cada una queda en su nivel:
proyecto (L1+L2) y cartera (L3). Ninguna regla se contradice.

---

## 4. Indicadores del tablero (qué se ve, no qué se calcula)

| Indicador | Definición | Dónde se ve |
|---|---|---|
| Utilización de persona | `Σ fte × (días_laborables(a ∩ W) / días_laborables(W))` | Ficha de persona, panel de disponibles |
| `AT_CAPACITY` | `util ≥ 0.85` → no se asigna sin override humano | Ficha, tablero |
| `OVERLOADED` | `util > 1.00` → bloqueo de nuevas asignaciones | Alerta + ficha |
| Carga del proyecto | `Σ fte × capacidad` / capacidad del equipo | Semáforo del tablero |
| Índice de presión | `deficit / capacidad_restante` | Card de proyecto |
| Gap de roles | roles de la plantilla con cobertura < 100 % | Card de proyecto |
| Cobertura de ausencia | `oferta / demanda` de la ausencia | Panel de ausencias |
| Índice de disponibilidad | % del pool realmente disponible | Pool de desasignados |
| Cumplimiento de devoluciones | préstamos devueltos en su fecha | Panel de préstamos |
| Presión de cartera | % de proyectos activos en EN_RIESGO/CRITICO | Vista de dirección |

---

## 5. Insumos que hay que registrar (si no, no hay presión que medir)

La presión es una comparación entre **demanda** y **capacidad**. Por eso la `CargaEstimada` es el insumo
crítico del sistema:

| Entidad | Campos | Quién la registra | Cadencia |
|---|---|---|---|
| `CargaEstimada` | `proyecto_id, rol, periodo_semana, esfuerzo_horas, origen (estimado/real)` | PM o Scrum Master | Al inicio del proyecto; luego se contrasta con avance real |
| `HorasConsumidas` | `proyecto_id, periodo, horas` | Derivado de `CargaEstimada` con `origen=real` | Semanal |
| `Hito` | `proyecto_id, fecha, titulo, estado` | PM | Al planificar |
| `Capacidad` | por persona y periodo: días laborables, días ausentes, días disponibles | Derivado | Automático (feriados + ausencias) |

> **Riesgo R-01, aceptado con mitigación:** si la carga se registra a mano y mal, el agente recomienda
> mal. Mitigación: el agente **no propone** si la confianza de los datos de demanda es baja; en su lugar pide
> actualizar la estimación. La calidad de la recomendación es inseparable de la calidad del input — y eso también
> es una lección de SDD: *una spec no puede prometer lo que los datos no sostienen*.

---

## 6. Parámetros de configuración (todo aquí, nada hardcodeado)

```yaml
umbrales:
  avanceVentanaPresion: 0.75      # L1
  cargaBanda: { verde: 0.70, ambar: 0.85 }   # L2
  indicePresion: { riesgo: 1.00, critico: 1.25 }
  presionCartera: 0.75            # L3
  pisoReservaPrestamo: 0.40       # BR-08: % mínimo que conserva el proyecto origen
  capacidadMaximaPersona: 1.00
  utilizacionATCapacidad: 0.85
  anticipacionVacacionesMinimaDias: 14   # BR-14
```

---

## 7. Pregunta para el cliente (la que decide todo esto)

> **¿El 75 % es el avance del proyecto, la carga comprometida de su equipo, o el porcentaje de proyectos de la
> cartera en problemas?** ¿O son las tres cosas a la vez, en niveles distintos?

Esta propuesta asume **"las tres, en niveles distintos"** (proyecto / proyecto / cartera) porque es la única
lectura que no deja fuera ningún caso y la que más se parece a lo que dijo el cliente en las tres OC. Si el
cliente confirma, el resto de la propuesta sigue sin cambios. Si no, se cambia **un parámetro**, no el diseño.
