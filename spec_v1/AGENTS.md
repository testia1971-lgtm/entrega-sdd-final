# AGENTS.md — protocolo de trabajo para agentes de código

Este repositorio se construye con **Spec Driven Development**. Cualquier agente (humano o IA)
que trabaje aquí sigue este protocolo. Es la parte del proyecto final que se entrega al curso.

## 1. Reglas inviolables

1. **No se escribe código sin un requisito (`US-*`) y un criterio EARS que lo verifique.**
   Si la tarea no tiene requisito asociado, primero se escribe el requisito.
2. **Un PR = una tarea.** La tarea referencia `US-*` y el criterio EARS que satisface.
3. **Toda cifra mostrada en la UI o devuelta por una tool del agente debe ser verificable.**
   Nada de números "aproximados" inventados por el modelo.
4. **El agente IA de producto es siempre *draft***: propone, un humano aprueba. (BR-18)
5. **Prohibido usar atributos sensibles** (salud, edad, género, estado civil, origen) en el
   scoring de candidatos o en cualquier métrica de evaluación de personas. (BR-20)
6. Si el código revela que una spec es ambigua o falsa, **se corrige la spec en el mismo PR**
   y se anota en el changelog del requisito.

## 2. Bucle de trabajo

```
spec → design → tasks → implement → verify → (refactor: corrige la spec)
```

### 2.1 Spec

- Épicas en `specs/<módulo>/requirements.md`.
- Cada historia: `US-<épita>.<n>` + criterio en estilo EARS
  (*When \<evento\>, the system shall \<respuesta\>*).
- Cada historia incluye escenarios Given/When/Then y una referencia al diseño que la implementa.

### 2.2 Design

- Decisiones con contexto y consecuencias (ADR cuando la decisión es estructural).
- Todo diseño referencia los `US-*` que implementa y los `BR-*` que respeta.

### 2.3 Tasks

- Formato: `- [ ] T-<id> [US-xx.x] descripción — criterio de aceptación — test esperado`.
- Tarea pequeña: cabe en un PR, un dominio, un revisor.

### 2.4 Implement

- El agente ejecuta tareas en orden de la matriz de trazabilidad.
- Cada commit referencia la tarea: `T-041 (US-04.3)`.

### 2.5 Verify

- Test por criterio EARS. Un criterio sin test es un criterio no implementado.
- CI corre: lint + typecheck + unit + integración + e2e de criterios + validación de trazabilidad.

## 3. Definition of Ready (spec)

- [ ] Actor y valor para el usuario explícitos.
- [ ] Criterio EARS presente y verificable.
- [ ] Mock o wireframe de la UI cuando aplica.
- [ ] Depende de ≤ 2 historias.
- [ ] Reglas de negocio aplicables identificadas por ID.

## 4. Definition of Done (tarea)

- [ ] Tests verdes del paquete tocado completo (no solo del módulo).
- [ ] Criterio EARS cubierto por al menos un test.
- [ ] Matriz de trazabilidad actualizada.
- [ ] Spec/design actualizados si hubo ambigüedad descubierta.
- [ ] Segunda opinión (otro agente) sobre el diff.

## 5. Comandos de referencia

```bash
# --- Backend (Python + FastAPI + MongoDB) ---
pytest                                        # suite completa del backend (nunca solo el módulo tocado)
ruff check . && mypy fastapi                  # lint + tipos
python -m scripts.traceability_check          # trazabilidad: US -> diseño -> tarea -> prueba
python -m scripts.verify_seed                 # invariantes I-1..I-8 y cobertura S-01..S-23 sobre el seed
python -m scripts.seed                        # carga idempotente del dataset semilla

# --- Frontend (React + TS + Vite) ---
npm --prefix web test                        # vitest
npm --prefix web run e2e                     # playwright (evidencia de la demo)

# correr los tests de un criterio concreto
pytest -k "us_04_3"
```

> **A-4, aplicado en el ensamblado final** (los nombres de los comandos son **una propuesta**): se ajustan cuando
> se cierren D-07/D-08, y vale cualquier equivalente mientras el gate **pueda rechazar**. Lo que no cambia es el
> *qué*: los invariantes se verifican en el motor determinista en Python, no en el prompt (Constitution Art. IV).
>
> **Stack ya decidido por el cliente** (no está en discusión): React + TS + Vite · FastAPI + Python · MongoDB ·
> agentes con el **OpenAI Agents SDK**. Lo único que sigue abierto es el *cómo* (detalles de D-07/D-08), no el *qué*.

## 6. Responsabilidad actual de las secciones abiertas

| Sección | Rol responsable | Estado |
|---|---|---|
| `designs/0001-arquitectura.md` | Arquitectura y plataforma | **Entregada** — arquitectura escrita sobre el stack decidido; **D-07 (auth)** y **D-08 (despliegue)** siguen abiertas, con default propuesto en `docs/PROPUESTA_FINAL.md` §15 |
| `designs/0003-tableros-ux.md` | UX y frontend | **Entregado** — 13 decisiones X-01..X-13, mockups S-1..S-4, sistema visual y reglas de rendimiento |
| `designs/0005-modelo-carga-y-presion.md` | Dominio y agente | Entregado |
| `designs/0002-modelo-datos.md` | Dominio y agente | Entregado |
| `designs/0004-agente-coordinador.md` | Dominio y agente | Entregado (con §4.1 añadida en el ensamblado: Agents SDK de OpenAI) |
| `specs/**` | Requisitos y trazabilidad | **Entregadas** — 36 historias con criterio EARS, 22 BR sin huérfanas; US-02.6 alineado con la métrica unificada (BR-04) |
| `tasks/0001-tareas.md` | Requisitos y trazabilidad | **Entregado** — 43 tareas en 7 sprints (Sprint 0..6), reescritas en el vocabulario del stack final (A-3) |
| `AGENTS.md` §5 (comandos) | Plataforma | **Propuestos y en uso** (A-4): `pytest`, `ruff`, `mypy`, `playwright` y los `scripts.*` de Python |
| `traceability/matrix.md` | Requisitos y trazabilidad | **Entregado** — 36/36/36/0 + 43/43 tareas + gaps declarados |
| `docs/PROPUESTA_FINAL.md` | Ensamblado final | **Entregado** (2026-10-21) — integra la opinión de todas las áreas; D-07/D-08 pendientes de confirmación del cliente |
