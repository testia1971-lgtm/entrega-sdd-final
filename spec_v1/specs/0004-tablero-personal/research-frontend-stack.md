> **Área responsable de este artefacto: UX y frontend.** Este research es la *base*, no la entrega final:
> queda en el repo para que el área de UX lo use, lo corrija y lo convierta en `designs/0003-tableros-ux.md`.
> Puntos que ya estan decididos y hay que respetar: timeline propio sobre `d3-scale` + `d3-zoom`,
> `@xyflow/react` + `@dagrejs/dagre` para el organigrama, Tailwind + shadcn/ui, ECharts para los paneles,
> SSE con un solo stream multiplexado, y el timeline parametrizado por *row source* (proyecto | persona).
---
title: "Frontend stack recommendation for Gestión de RH y Proyectos (React + TS + Vite) — v2, verified"
tags: [frontend, gantt, orgchart, charts, licensing, stack]
status: active
created: 2026-10-20
---

> **v2**: rewritten after the full library verification (npm registry, GitHub API, vendor licence files).
> **Correction to v1**: v1 recommended `vis-timeline` as primary for the Gantt. The deeper verification shows that
> no off-the-shelf Gantt library provides all three of (a) resource lanes with avatars, (b) absence/availability
> overlay bands, (c) pairwise over-allocation highlighting — which are exactly the three things that make this
> app "visually rich". The recommendation is therefore a **thin custom timeline layer**, with Gantt libraries
> demoted to optional accelerants. `vis-timeline` remains a good *substrate* if we do build on a library.

Stack fixed by the human: **web only, React + TS + Vite**, backend **FastAPI + Python + MongoDB**, agents inside the
product using the OpenAI standard.

# 1. The core finding (read this first)

> No off-the-shelf Gantt library gives you all three of: resource-lane rows with avatars, vacation/incapacity
> overlay bands, and pairwise over-allocation conflict highlighting. DHTMLX has them — in the **paid** tier.

**Recommendation: build the timeline as a thin custom layer** on `d3-scale` + `d3-zoom`:
- `d3-scale` → the time axis; week / bi-week / month / quarter / year is one `scaleTime` + `scaleBand` per zoom.
- `d3-zoom` → pan/zoom (the same primitive React Flow uses).
- Bars as **absolutely-positioned divs** (avatar + name + role) → no SVG text-metrics pain, CSS gives free
  hover/animation.
- Background bands (vacation, incapacity, intern window) = rects behind the bars.
- Over-allocation = a red hatched band computed server-side.
- Cost: **~1,500–2,500 LOC**. Budget it explicitly in the plan.

**The biggest architectural win: parameterise the timeline by "row source" (`project | person`).** One component,
two boards — the project board and the by-resource board are the same code with a different row source. That
single decision is what keeps the scope inside a final project.

## 1.1 Gantt library landscape (verified 2026-10-20)

| Library | Licence | Version / date | dl/wk | gzip | Verdict |
|---|---|---|---|---|---|
| **frappe-gantt** | MIT | 1.2.2 (2026-02-25) | 142,693 | **14.1 KB** | Lightest, zero deps, `view_modes` customisable (week/bi-week/quarter are just `{step, column_width}`), `popup` API → perfect for the "click project → rich card". ❌ left column is a single fixed label → no avatar columns |
| **vis-timeline** | **Apache-2.0 OR MIT** | 8.5.4 (2026-08-12) | 125,247 | 119.8 KB | Continuous zoom (`zoomMin`/`zoomMax`), `groups` for resource lanes, `items` + range items, `editable.updateTime`. ✅ Best substrate if we use a library. ❌ no dependency arrows; vanilla JS → needs a `useTimeline` wrapper |
| **@svar-ui/react-gantt** | MIT | 2.8.1 (2026-10-12) | 70,952 | 98.6 KB | **React 19 native**, task grid with **customisable columns** (avatar, utilisation %, cost), **weekend/holiday highlighting** built in, zoom, virtualisation (10k tasks), real-time updates. ⚠️ repo created 2025-10-13, 214★ → 1-year-old dep; PRO holds auto-scheduling |
| **@event-calendar/core** | MIT | 5.17.3 (2026-10-20) | 48,469 | ~35 KB br | Native `ResourceTimeline` with Day/Week/Month/Year, by FullCalendar's author. ⚠️ **no React package** (vanilla API + a `svelte` dep); plugin packages stuck at v3.12.0 → version skew |
| **dhtmlx-gantt v10 Community** | **MIT** (changed from its old GPL reputation) | 10.0.3 (2026-09-03) | 23,726 | ~180 KB | Richest features, MIT, readable source. 🚩 **resource management (assignments, histogram, grouping by resource), auto-scheduling, critical path and working-time calendars are PRO-only.** Other DHTMLX components (`grid`, `calendar`, `chart`, `tree`, `scheduler`) are still **GPL-2.0** → don't mix |
| **fullcalendar** | MIT core | 7.1.3 (2026-10-16) | 149,358 | 39.8 KB | 🚩 `resource-timeline` is **Premium**: tri-licensed Commercial / **CC BY-NC-ND 4.0** / **AGPLv3** → hard blocker for a commercial HR app |
| `gantt-task-react` | MIT | 0.3.9 (2022-07-10) | 31,103 | — | 🚩 **repo ARCHIVED** — and still the most-downloaded dead Gantt on npm. The trap |
| `react-gantt-chart` | MIT | 0.3.0-rc.11 (2022-03-28) | 232 | — | 🚩 dead RC, repo renamed → 404 |
| `vue-ganttastic` | MIT | 0.9.34 (2022-01-08) | 204 | — | 🚩 abandoned (repo last push 2024-06) |
| `gantt-schedule-timeline-calendar` | **"NEURONET Free/Trial"** | 3.43.14 | 2,154 | 259.3 KB | 🚩 **NOT open source** — forbids modification and distribution; must be registered in the vendor's system |
| `@bryntum/gantt` | Commercial | 7.3.7 | 253 | ~500 KB | 🚩 paid, trial-only npm builds |
| `dhtmlx-scheduler` | GPL-2.0 | 7.2.15 | — | — | 🚩 GPL lock, and not a Gantt |
| `mantine-gantt` | MIT | 0.6.1 | 1,464 | 11.2 KB | ⛔ 3★ — reference implementation only |
| `@mui/x-scheduler` | MIT | 9.0.0-beta.14 | 14,006 | — | ⛔ beta |
| `mermaid` | MIT | 12.2.0 | 10.9M | 169.5 KB | ⛔ non-interactive → documentation of the org-chart model only, never the timeline engine |
| `frappe-charts` | MIT | 1.6.2 | 60,805 | — | 🚩 npm last published **2021-06-16** → release pipeline stalled ~5 yrs. Use ECharts |

**Rule:** check `npm view <pkg> time.modified` + GitHub `archived`/`pushed_at` before adopting. Download counts are
**not** a health signal.

# 2. Org chart — Vista C

| Library | Licence | Version | dl/wk | gzip | Fit |
|---|---|---|---|---|---|
| **@xyflow/react** (React Flow 12) | MIT | 12.12.0 | **8,349,970** | 58.6 KB | ⭐ **Best.** Node canvas, custom node types (avatar + name + role card), zoom/pan/minimap/background, edge routing. Deps: `classcat`, `zustand`, `@xyflow/system` only |
| **@dagrejs/dagre** | MIT | 3.1.1 (2026-08-08) | 2,999,871 | **15.4 KB** | Layout only. Layered/DAG → "quién reporta a quién". ⚠️ many projects still install the stale `dagre@0.8.5` (2019, 1.6M dl/wk) |
| `d3-org-chart` | MIT | 3.1.1 (2023-09-18) | 60,867 | — | Best "card" look, expand/collapse, export. ⚠️ npm 17 months stale |
| `orgchart` (dabeng) | MIT | 7.0.0 | 2,987 | **18.3 KB** | Lightest pure-DOM tree, actively maintained (push 2026-10-19) |
| `react-d3-tree` | MIT | 3.6.7 (2026-09-22) | 137,518 | 27.5 KB | Tree-only; nodes are SVG circles → needs `foreignObject` for avatars. Read-only hierarchy |
| `elkjs` | **EPL-2.0 OR GPL-3.0** | 0.12.0 | 7,499,856 | **423.2 KB** | Best-in-class layout, but ⚠️ **weak copyleft** and huge → lazy-load in a Web Worker. Prefer dagre |
| `@balkangraph/orgchart.js` | **"FREE Trial"** | 9.3.28 | 1,606 | — | 🚩 proprietary, no LICENSE file, npm license field literally "FREE Trial" |

**Recommendation: `@xyflow/react` + `@dagrejs/dagre`**, custom `PersonNode` component (avatar, name, base role,
project-role chips, load bar).
**Why a graph and not a tree:** "one person, several roles per project, one base/contract role" is a **multi-edge,
non-tree** structure. A strict tree library (react-d3-tree, d3-org-chart) forces a fake "roles" parent node; a
node-based editor handles it natively.
**Fallback:** `d3-org-chart` if we later want a printable/server-rendered org chart (PDF export).

# 3. UI kit

| Kit | Licence | Version | gzip | Notes |
|---|---|---|---|---|
| **shadcn/ui** | MIT | CLI (code-copied) | per-component | 126k★, push 2026-10-20. You own the code. Radix + Tailwind v4. Best for a bespoke rich dashboard |
| **Radix Primitives** | MIT | radix-ui@1.8.1 | — | Unstyled, accessible |
| Radix Themes | MIT | 3.3.0 (2026-01-31) | — | Fastest polished "dashboard" look. ⚠️ 7-month release cadence |
| **Mantine** | MIT | 9.7.3 (2026-10-17) | 170.6 KB | 31.8k★, very active, `@mantine/charts` + `@mantine/dates`, 120+ hooks. Best batteries-included kit |
| Ant Design | MIT | 6.6.7 | **445.2 KB** | 48 deps, enterprise tables — but the visual language is "admin back-office" and fights a bespoke brief |
| PrimeReact | 🚩 **PrimeUI License** | 11.2.1 | — | **No longer MIT**: free only if <$1M revenue, <5 devs, <10 employees, <$3M VC, **annual re-confirmation**, **license key required**; else per-developer paid. The v10 GitHub repo is still MIT — check the *pinned npm version*, not the repo badge |

**Recommendation: Tailwind + shadcn/ui.** Only option where the visual language is 100% ours — critical for
"visualmente rico". **Fallback: Mantine v9** for maximum velocity with one coherent MIT tree (Mantine + ECharts look
coherent; Mantine + shadcn look like two kits).

# 4. Charts

**Primary: Apache ECharts 6.1 (Apache-2.0) + `echarts-for-react` 3.0.6.** One engine for every panel:
Radar (skills vs. required skills) · Gauge (utilisation %, 0–160% with the over-allocation zone) · **calendar
heatmap** (tenure, absences, birthdays, vacation days) · stacked bar + `dataZoom` (monthly workload history) ·
Graph (mini overlap view). One `theme` object → visual coherence with the rest of the app.
**Bundle:** full build 359.3 KB gzip → tree-shake via `echarts/core` + only the 6 chart types we use
(≈120–150 KB) and lazy-load the board chunk.

| Alternative | Licence | gzip | Note |
|---|---|---|---|
| Recharts 3.10.1 | MIT | 148.0 KB | Most popular; Area/Bar/Line/Pie/Radar/RadialBar/Sankey/Sunburst/Treemap. v3 = rewritten internals |
| visx 4.0.0 | MIT | 34.8 KB (`@visx/shape`) | Low-level d3 primitives, no chart types — assemble yourself |
| Nivo | MIT | 87.6 KB | 🚩 **stale**: every `@nivo/*` last published 2025-05-23, version stuck at 0.99.0 since 2018 |

**Do not use a chart library for a 3-segment progress bar** — a plain `<div>` or Mantine `RingProgress` is enough.

# 5. Realtime: SSE

The agent panel must stream proposals and push flags without a refresh, and accept/reject is a plain POST.
→ **95% server→client**, so **SSE wins on every axis**: auto-reconnect with `Last-Event-ID`, no sticky sessions,
native support in FastAPI (`sse-starlette`) — WebSocket would need an extra server.

**Design — one multiplexed stream per tab** (never one per panel):
```
GET  /api/events            -> text/event-stream
      event: agent.delta        (streaming del análisis)
      event: agent.proposal     (propuesta concreta: persona, proyecto, rol, periodo, razón)
      event: agent.flag         (sobre-asignación: persona, proyectos[], solape, total_pct)
      event: allocation.updated (cambio aplicado -> la board refresca)
      id: <seq>                 (Last-Event-ID -> reanudar/replay)
POST /api/agent/proposals/:id/accept
```
⚠️ **MDN**: over HTTP/1.1 a browser allows only **6** SSE connections per origin → one stream, multiplexed; ensure
HTTP/2 at the edge. ⚠️ serverless kills long-lived SSE → a long-lived container (Fly/Railway/VPS) for the agent
channel.

# 6. Build order (pragmatic)

1. **Data + queries first** — get utilisation and conflict detection right before any UI. The difficulty lives here.
2. **Seed data** with deliberate over-allocations so the board is interesting on day one.
3. **Custom timeline layer** (d3-scale + d3-zoom), parameterised by row source → both boards from one component.
4. **ECharts info panels.**
5. **Org chart** (`@xyflow/react` + `@dagrejs/dagre`).
6. **SSE + agent panel last**, once the data model can answer "qué está sobre-asignado" de forma determinista.

# 7. Backend adaptations (MongoDB, not Postgres)

The Postgres-first advice (`daterange`, `EXCLUDE USING GIST`, `range_agg`, `WITHOUT OVERLAPS` in PG 18) does
**not** apply: the human chose MongoDB, which has no range types and no exclusion constraints. The equivalent
design, which is already in `designs/0004-agente-coordinador.md`:
- Overlap detection and utilisation are computed in the **deterministic engine** (Python), over indexed queries:
  `assignments {person_id: 1, start_date: 1, end_date: 1}`, `{project_id: 1, start_date: 1, end_date: 1}`.
- Invariants I1..I9 are enforced in the service layer + the tool guardrail, **plus** a nightly reconciler that
  scans for violations and raises an alert (defence in depth).
- Business-day math needs an explicit holiday table → `holidays` collection, seeded from
  **`date-holidays` v3.37.0** (ISC **+ CC-BY-3.0 → attribution required in the UI**).
- Same principle survives: **do not let the LLM compute over-allocation.** Compute it deterministically; let the
  LLM explain and propose.

# 8. Risk register (frontend)

| Risk | Mitigation |
|---|---|
| Licence lock (FullCalendar Premium, PrimeReact v11, DHTMLX PRO/grid, elkjs copyleft, `gantt-schedule-timeline-calendar`, `orgchart.js`) | stay on MIT/Apache/ISC only; verify `npm view <pkg> license` for the *pinned version*; state every licence in the MD |
| Dead-but-popular deps (`gantt-task-react` 31k dl/wk, `nivo`, `frappe-charts`) | check `time.modified` + `archived` before adopting |
| Bundle bloat (antd 445 KB, echarts 359 KB, elkjs 423 KB, mermaid 169 KB) | lazy-load per board; target **< 450 KB gzip** initial JS; import only the ECharts modules used |
| Custom timeline is 1,500–2,500 LOC | budget it; the row-source parameterisation is what makes it affordable |
| SSE connection limit (6/origin over HTTP/1.1) | one multiplexed stream per tab; HTTP/2 at the edge; long-lived container |
| AGPL contamination from reference apps (Worklenz, Plane, Kimai, ERPNext, OpenProject, OrangeHRM, Odoo) | read them for patterns, never `npm install` their application code into the product |
| 5-year zoom performance | server-side aggregation by project-month at year zoom; windowed payloads (`from`/`to`/`zoom`) |

# 9. Reference apps worth reading (not linking)

| Project | Licence | What to take |
|---|---|---|
| **Worklenz** | AGPL-3.0 | ⭐ **Closest match to the brief** — "plan team capacity, avoid overallocation, visual scheduler", cross-project planner with utilisation at a glance. Read the capacity/utilisation maths and the planner UX |
| **ERPNext** | GPL-3.0 | The leave / leave-type / leave-policy / holiday-list model = our availability overlay source of truth |
| **Kimai** | AGPL-3.0 | Timesheet → activity → allocation chain, absence handling, JSON API |
| **OpenProject** | GPL-3.0 | Work-package/assignment model, cost & time reporting. ❌ no capacity/resource Gantt |
| **Plane** | AGPL-3.0 | Modern Gantt *view* implementation. ❌ no resource-capacity concept |
| OrangeHRM | GPL-3.0 | Leave + skill + job-title taxonomy, "base/contract role" modelling |

# 10. Sources

[Gantt](https://github.com/frappe/gantt) · [svar react-gantt](https://github.com/svar-widgets/react-gantt) ·
[vis-timeline](https://github.com/visjs/vis-timeline) · [EventCalendar](https://github.com/vkurko/calendar) ·
[DHTMLX editions](https://docs.dhtmlx.com/gantt/guides/editions-comparison/) ·
[FullCalendar premium LICENCE](https://github.com/fullcalendar/fullcalendar-workspace/blob/main/premium/LICENSE.md) ·
[PrimeReact v11 LICENCE](https://unpkg.com/primereact@11.2.1/LICENSE.md) ·
[xyflow](https://github.com/xyflow/xyflow) · [@dagrejs/dagre](https://github.com/dagrejs/dagre) ·
[shadcn/ui](https://github.com/shadcn-ui/ui) · [Mantine](https://github.com/mantinedev/mantine) ·
[Apache ECharts](https://github.com/apache/echarts) · [Recharts](https://github.com/recharts/recharts) ·
[MDN SSE](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events) ·
[date-holidays](https://github.com/commenthol/date-holidays) ·
[Worklenz](https://github.com/Worklenz/worklenz) · [ERPNext](https://github.com/frappe/erpnext) ·
[Kimai](https://github.com/kimai/kimai) · [OpenProject](https://github.com/opf/openproject) ·
[Plane](https://github.com/makeplane/plane)
