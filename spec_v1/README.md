# SDD · Gestión de RH y Proyectos — Propuesta de proyecto final

Repositorio de la propuesta (**Spec Driven Development**) para el sistema de **gestión de recursos humanos de una
fábrica de software**: asignación de personas a proyectos, carga de trabajo, ausencias, préstamos temporales,
becarios, **autogestión de vacaciones con reconocimiento de aniversario (gamificación)** y un **agente IA coordinador**.

> Este repo es el material compartido y versionado de la propuesta.
>
> **Entrega final**: [`docs/PROPUESTA_FINAL.md`](docs/PROPUESTA_FINAL.md) — 18 secciones que integran la opinión
> de todas las áreas de trabajo.

## Topología (`specs/ → designs/ → tasks/`)

```
constitution.md                       # Principios inmutables (11 artículos)        — Dominio y agente
PROPUESTA.md                          # Base consolidada (15 secciones)             — Dominio y agente
docs/
  PROPUESTA_FINAL.md                  # ENTREGA FINAL (18 secciones)                — Ensamblado final
  DATASET_SEED.md                     # Dataset determinista + 23 escenarios        — Dominio y agente
  RIESGOS_Y_VALIDACION.md             # Riesgos, supuestos, validación, demo       — Dominio y agente
specs/
  0001-vision/
    spec.md                           # Visión, glosario, objetivos                 — Requisitos y trazabilidad
    research-sdd-y-dominio.md         # Evidencia SDD + modelo de asignación        — Dominio y agente
    research-tooling-2026.md          # Panorama de herramientas SDD (comparativa)  — Dominio y agente
  0002-personas/                      # requirements.md (historias + EARS)          — Requisitos y trazabilidad
  0003-proyectos/                     # requirements.md (historias + EARS)          — Requisitos y trazabilidad
  0004-tablero-personal/              # requirements.md (historias + EARS)          — Requisitos y trazabilidad
    research-frontend-stack.md        # Licencias y libs del timeline (verificado)   — UX y frontend
  0005-agente-coordinador/            # requirements.md (historias + EARS)          — Requisitos y trazabilidad
    spec-exemplar-c11.md              # Plantilla de formato de spec (EARS+Gherkin) — Dominio y agente
  0006-plataforma/                    # requirements.md (historias + EARS)          — Requisitos y trazabilidad
designs/
  0001-arquitectura.md                # Arquitectura sobre el stack decidido (D-07/D-08 abiertas) — Arquitectura y plataforma
  0002-modelo-datos.md                # Entidades, temporalidad, invariantes I-1..I-13  — Dominio y agente
  0003-tableros-ux.md                 # UX de los tableros (X-01..X-13)             — UX y frontend
  0003-tableros-ux.borrador-alternativo.md  # Anexo preservado, no pisado              — (anexo)
  0004-agente-coordinador.md          # Motor + tools + guardarraíles                — Dominio y agente
  0005-modelo-carga-y-presion.md      # Qué significa el 75 % (3 lecturas, 1 métrica) — Dominio y agente
tasks/
  0001-tareas.md                      # 43 tareas con US-* y test (Sprint 0..6)       — Requisitos y trazabilidad
traceability/
  matrix.md                           # US-* ↔ diseño ↔ tarea ↔ test ↔ estado       — Requisitos y trazabilidad
```

## Decisiones de tooling SDD

- **Topología**: `specs/ designs/ tasks/ traceability/` — equivalentemente el árbol de
  `github/spec-kit` reordenado para que la separación **qué / cómo / quién lo hace** sea evidente.
- **Sintaxis de requisitos**: **EARS** (`WHEN <trigger> THE SYSTEM SHALL <respuesta>`), de Kiro — la mejor
  formalización disponible y la que ya usa la spec de requisitos.
- **Convención de aclaración**: `[NEEDS CLARIFICATION: …]` de spec-kit — obliga al agente a preguntar en vez de suponer.
- **Trazabilidad**: ids `US-*` → `BR-*` → `I-*` → tarea → test, en `traceability/matrix.md`.
- **Puertas de CI que de verdad bloquean** (un gate solo es control si puede rechazar):
  `scripts.spec_validate` (formato + referencias rotas), `scripts.verify_seed` (invariantes I-1..I-13 y
  cobertura S-01..S-23), el **golden set** del agente en `pytest` (E-01..E-12) y el determinismo del seed.
- **Alternativa sin dependencias**: la misma forma de carpetas a mano + `python -m scripts.spec_validate` propio.
  Decisión completa y fuentes en `specs/0001-vision/research-tooling-2026.md`.

## Reglas de trabajo

1. Todo requisito del `requirements.md` es verificable y trazable a una prueba (`tasks/0001-tareas.md` + `traceability/matrix.md`).
2. La spec es la fuente de verdad del dominio: el glosario manda sobre la implementación.
3. Nada se declara "implementado" sin evidencia (test, captura o salida del agente).
4. Se atribuye cada resultado al commit exacto que lo produjo (`git rev-parse HEAD` en el mismo shell).
5. El agente propone, la persona decide (constitution, Artículos V y X).

## Comandos de referencia

```bash
# Backend (Python + FastAPI + MongoDB)
python -m scripts.spec_validate      # formato + referencias rotas + trazabilidad incompleta
python -m scripts.verify_seed        # I-1..I-13 y cobertura S-01..S-23 sobre el dataset
pytest                                # golden set del agente (E-01..E-12) y suite completa
ruff check . && mypy fastapi

# Frontend (React + TS + Vite)
npm --prefix web test                 # vitest
npm --prefix web run e2e              # flujos de la demo (Playwright)

git rev-parse HEAD
```
