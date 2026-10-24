---
title: "SDD Tooling Landscape 2026: what to use for the final project"
tags: [sdd, spec-driven-development, tooling, research]
status: active
created: 2026-10-20
---

Research date: 2026-10-20. Sources fetched live.

## The 3-level taxonomy (the single most useful concept)

| Level | Spec role | Code role | Enforcement | Examples |
|---|---|---|---|---|
| Spec-first | guides initial build, may be discarded | primary deliverable | social/manual | Claude Code plan mode |
| Spec-anchored | maintained alongside code | validated deliverable | CI gates | OpenSpec, Tessl, Kiro, spec-kit |
| Spec-as-source | only artifact humans edit | generated | regeneration | Tessl Framework |

Source: [Fowler / Böckeler, Oct 2025](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) and
[Piskala, arXiv:2602.00180](https://arxiv.org/html/2602.00180v1).

**Verdict for us:** aim for **spec-anchored** — spec survives implementation and is enforced by CI.

## Tool comparison (short version)

| Tool | License | Artifacts | Sync mechanism | Notes |
|---|---|---|---|---|
| **Spec Kit** (GitHub) | MIT | `.specify/memory/constitution.md`, `specs/<feature>/{spec,plan,tasks}.md` | `/speckit-analyze`, `/speckit-converge` (appends tasks from code gaps) | 142k stars. Heavy; Böckeler calls it "spec-first only, not spec-anchored over time" |
| **Kiro** (AWS) | commercial | `.kiro/specs/<slug>/{requirements,design,tasks}.md` | task→requirement IDs (`_Requirements: 1.1, 1.5_`), Markdown Review | Best requirement formalism (**EARS**). Vendor-locked to Kiro IDE/CLI |
| **OpenSpec** (Fission-AI) | MIT | `openspec/specs/<capability>/spec.md` (canonical) + `openspec/changes/<id>/{proposal,design,tasks}.md` + deltas | **delta merge on archive**; `openspec validate --strict` in CI | 68k stars. Only real spec *lifecycle* in OSS. Agent-agnostic |
| **BMAD** | MIT | persona-driven docs (`docs/`), 21+ agents | file handoff between personas | v6 stable; v7 preview alpha (Oct 2026). Overhead warning |
| **Taskmaster AI** | MIT + **Commons Clause** | `tasks.json` graph with complexity scores | dependency graph | Task graph, not spec layer. `TASK_MASTER_TOOLS=core` saves ~16k tokens |
| **Tessl** | commercial (SDD tile MIT) | `.spec.md` with `[@generate]`, `[@test]` links | `[@test]` links + `check-spec-links.sh` | True spec-as-source. Framework still beta |
| **GSD** | MIT | `PLAN.md`, `STATE.md`, ephemeral subagents | goal-backward verification | Context-engineering angle; context rot as first-class failure mode |
| **cc-sdd** | MIT | `requirements.md` (EARS) + `design.md` (Mermaid) + `tasks.md` | fresh implementer/reviewer subagents | Kiro model on OpenCode/Claude Code/Codex/Cursor. Best fit for our harness |

Adoption (2026-10-20): spec-kit 142.2k★, OpenSpec 68.0k★, GSD 64.4k★, BMAD 48.4k★ (Jun 2026, third-party).

## Requirement syntaxes

- **EARS** (Kiro, cc-sdd, spec-workflow-mcp): `WHEN <trigger> THE SYSTEM SHALL <response>`.
  Origin: Rolls-Royce, Mavin et al., IEEE 2009. Removes ambiguity, does NOT ensure completeness.
- **Gherkin/BDD**: Given/When/Then — "executable specifications"; inherited by OpenSpec (`#### Scenario:`) and Tessl.
- **Delta specs** (OpenSpec): `## ADDED / MODIFIED / REMOVED / RENAMED Requirements`. Only approach with real lifecycle.
- **`[NEEDS CLARIFICATION: …]`** (spec-kit): forces the agent to ask instead of guessing.

## Keeping specs honest — the CI gates that actually work

| Gate | Blocks | Source |
|---|---|---|
| `openspec validate --strict` | malformed/missing deltas, wrong heading depth, zero-delta changes | [OpenSpec schema](https://openspec.dev/docs/schemas/spec-driven) |
| ADR Guard | PR touching watched paths without an ADR | [ADR repo](https://github.com/architecture-decision-record/architecture-decision-record) |
| Specmatic / Pact | implementation vs contract drift | [Piskala](https://arxiv.org/html/2602.00180v1) |
| ArchUnit / ArchUnitTS | architectural rules as fitness functions | [ADR repo](https://github.com/architecture-decision-record/architecture-decision-record) |
| LLM spec-compliance reviewer | spec non-conformance + scope creep, **before** human review | [AWS Security Blog](https://aws.amazon.com/blogs/security/balancing-speed-and-safety-a-control-framework-for-ai-coding-agents) |

**Key operational rule:** *a gate is a control only if it can refuse* — hard failure on main, advisory on feature
branches ([Consort, arXiv:2609.09671](https://arxiv.org/html/2609.09671v1)).

## Human-in-the-loop pattern

- **Spec PR first (docs-only), code PR after.** Cheap to review, lives in git history.
- **Implementor ≠ Verifier** (Augment): "Implementors and Verifiers have opposing goals. One optimizes for
  completing the task, the other for finding failures." Forcing explicit verification criteria improves the spec itself.
- **Agent reviewer must not be in the approval path.** Keep branch protection requiring a human approval.
  ([AIDev via secondary source](https://www.aibuilderclub.com/blog/reviewing-ai-generated-pull-requests) — verify before citing as fact.)
- **Risk-proportional autonomy**: autonomy falls as risk rises ([AWS](https://aws.amazon.com/blogs/security/balancing-speed-and-safety-a-control-framework-for-ai-coding-agents)).

## AI agents doing resource allocation (directly relevant here)

**The pattern that works: LLM-as-formulator → deterministic solver → LLM-as-explainer.**

- MIT: "As long as the inputs to the solver are correct, it will give the right answer. Any mistakes in the
  solution come from errors in the formulation process."
  ([MIT News, Apr 2025](https://news.mit.edu/2025/researchers-teach-llms-to-solve-complex-planning-challenges-0402))
- Timefold: assigning shifts to 10 employees for a week is 10^63 combinations; LLMs hit both problem-space and
  context limits. Hybrid is the recommendation. Constraint taxonomy: **hard** (infeasible if violated),
  **medium** (handle shortage, most important first), **soft** (weighted business goals).
  ([Timefold](https://timefold.ai/blog/llms-cant-optimize-schedules-but-ai-can))
- **Tool-grounded availability is the mitigation for hallucinated availability** ([Planet survey](https://arxiv.org/html/2504.14773v1)).

### Failure modes to guard against (with source)

| Failure | Evidence | Guardrail in this project |
|---|---|---|
| Hallucinated availability | Planet; [Atlan](https://atlan.com/know/llm-hallucinations) (vendor claim: 52% fabrication) | Agent reads availability via tools at decision time; never from memory |
| Over-assignment / conflicting ownership | MAST [arXiv:2503.13657](https://arxiv.org/abs/2503.13657) — 14 failure modes, 1,600+ traces, κ=0.88 | Hard constraints in the deterministic engine, never in the prompt |
| Inventing plans / ignoring new constraints | [LLM-as-Planner and Formalizer, ACL 2026](https://aclanthology.org/2026.acl-long.624.pdf) | Engine re-validates every agent proposal before it can be submitted |
| Not asking for clarification | MAST: "failing to ask for clarification" = 11.65% of failures | `NEEDS CLARIFICATION` + one-question-at-a-time rule |
| Context rot | GSD; [Augment](https://www.augmentcode.com/guides/what-is-spec-driven-development) | Fresh-context subagents per capability; engine returns minimal facts |
| Gate that can be waived | [Consort](https://arxiv.org/html/2609.09671v1) | Gate must be able to refuse; hard-fail on main |

## Pitfalls to avoid (consolidated)

1. **Right-sizing.** Kiro on a 1-line bug = 4 user stories, 16 acceptance criteria. Don't SDD the trivial.
2. **Review burden is the real cost** — "I'd rather review code than all these markdown files."
3. **False confidence** — "A passing spec test only guarantees the software matches the spec. If the spec is wrong,
   the code faithfully implements the wrong thing."
4. **Over-specification = pseudo-code** — "If your spec reads like code, you've gone too far."
5. **Spec-as-bureaucracy** — overhead without improved understanding → teams game or abandon the system.
6. **Model tiering** — write the spec with the strongest model, implement mid-tier, verify cheap.
7. **Human domain expertise is the bottleneck**, not the tool.
8. **Don't build "waterfall with an AI typing."** (Adam Wiggins, cited in Tessl's pattern library.)
9. **Don't repeat unverified claims** like "3–10× first-pass success rate" (low-credibility blogs). The defensible
   claim is narrower: controlled studies showing error reductions up to ~50% with human-refined specs
   (Red Hat research, per Piskala's summary).
10. **CI gaming by agents** — "Any change that weakens CI is a blocker. Full stop." ([GitHub Blog](https://github.blog/ai-and-ml/generative-ai/agent-pull-requests-are-everywhere-heres-how-to-review-them))

## Recommendation for this project

**OpenSpec as the spine** (delta lifecycle + real CI gate + agent-agnostic + MIT)
**+ EARS requirement syntax** (from Kiro/cc-sdd)
**+ a project `constitution.md`** (spec-kit's governance idea, adapted)
**+ `NEEDS CLARIFICATION` markers** (spec-kit's ask-don't-guess convention)
**+ spec→task→test traceability IDs** (Kiro's `_Requirements: 1.1, 1.5_`)

Fallback if we want zero tool dependencies: same folder shape by hand + our own `python -m scripts.spec_validate`.

## Open uncertainties

- Kiro's "Analyze Requirements uses SMT solvers" is a vendor claim, not confirmed in Kiro docs.
- BMAD 48.4k★ is a Jun 2026 third-party number, not a live count.
- AIDev PR-review percentages are relayed via a vendor blog — consult the dataset directly before citing.
- Taskmaster's Commons Clause matters if hosting; we won't host it, so it's acceptable as a task-graph layer.
