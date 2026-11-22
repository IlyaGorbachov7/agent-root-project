---
name: task-lifecycle
description: >-
  Explicit task workflow: Analyze → Clarify (Plan UI) → Plan → Implement →
  Review (plan compliance + Bugbot) → Fix (max 2), as one continuous pipeline
  after plan approval. Use only when the user invokes /task-lifecycle. Do not
  auto-start on ordinary tasks. Project overlays may override defaults.
disable-model-invocation: true
---

# Task lifecycle

Orchestration: **Analyze → Clarify (UI) → Plan → Implement → Review → Fix (max 2)**.

Works in any workspace. Project overlays override defaults (see [configuration.md](references/configuration.md)).

## Contract

| Item | Value |
|------|-------|
| Trigger | only `/task-lifecycle` |
| Escape | стоп, отмена, без цикла, просто сделай → normal mode immediately |
| Not triggers | `/ship`, «полный цикл», «жизненный цикл» |
| Approve gate | never implement before user approves the plan |
| Clarify | after analysis, Plan UI grilling; max **5–10** questions (default 8), batches of 1–3; click options, not chat a/b/c |
| Fix limit | max **2** cycles |
| Auto-commit | never |
| Response language | user's language (unless overlay sets otherwise); code/paths/commands in English |
| Session continuity | After plan approval, **one lifecycle** runs Phase 2 → 3 → (4 if needed) before the final handoff; never stop at «todos done» |

## Boot (every run)

1. If escape words present → exit lifecycle, work normally.
2. Load effective config: defaults ⊕ project overlay. Read [configuration.md](references/configuration.md) only if overlay exists or flags are unclear.
3. If state file exists → resume per [resume.md](references/resume.md). Do not re-plan an already approved plan.
4. Otherwise start Phase 1.

## Phase 1 — Planner

Order is fixed: **Analyze → Clarify (UI) → CreatePlan → Stop**.

1. `SwitchMode` → `plan` if not already in Plan (needed for clickable question UI).
2. **Analyze (readonly):** explore code, `AGENTS.md`, related modules. Resolve facts yourself. Do not ask what the repo can answer.
3. **Clarify (UI grilling):** if sharp decisions remain, read [clarify.md](references/clarify.md) and run bounded grilling:
   - Use `AskQuestion` so the user **clicks** options (never require typing `a`/`b`/`c` in chat when UI is available).
   - Batches of **1–3**; total cap **5–10** (default max **8**); stop early when clear.
   - Ask only decisions; each question includes your **recommended** option.
   - If nothing sharp remains → skip clarify.
4. `CreatePlan` with todos, file paths, target repo/package (encode clarify answers + any remaining assumptions). Fallback: markdown plan under `.cursor/plans/` if Plan tools unavailable.
5. **Stop.** Wait for user approval. Do not implement yet.

Handoff: short plan summary + «подтверди план, чтобы начать реализацию».

### Plan approval → start implementation (same lifecycle)

Treat as **approved plan** (enter Phase 2 immediately, no re-planning):

- Explicit confirm: «подтверждаю», «ок», «go», «implement the plan», «реализуй план», «делай по плану»
- Cursor **Build** / execution of an attached approved plan with todos
- Resume when state has `approved: true` and `phase: "implement"`

On first entry to Phase 2 after approval: **create** state (`approved: true`, `phase: "implement"`, `fix_cycle: 0`, `plan_uri`, `repos`, `task_summary`). Schema: [resume.md](references/resume.md).

## Phase 2 — Implementer

1. `SwitchMode` → `agent` after approval.
2. Execute approved todos only; minimal diff; follow project rules / `AGENTS.md`.
3. Run build/test for affected parts (from overlay or `AGENTS.md` / README). If none found, skip and note it.
4. Update state: `phase: "review"`, keep `fix_cycle: 0`, refresh `repos` if needed.
5. **Immediately continue to Phase 3 in the same turn** — do not send a final reply that ends with only «implementation complete» or «all todos done». Marking plan todos complete is **not** the end of the lifecycle.

If the turn must end early (context/tool limits): state **must** already be `phase: "review"`; next `/task-lifecycle` or «продолжи lifecycle» resumes at Phase 3 per [resume.md](references/resume.md).

## Phase 3 — Review

Read [review.md](references/review.md) for templates and verdict rules.

1. **Plan compliance** (parent): table Todo | Status | Notes (`done` / `partial` / `missing`). Gaps = partial/missing or out-of-scope edits.
2. **Bugbot** (if `bugbot.enabled`): one `bugbot` subagent per affected git root with uncommitted changes. Diff from config (default `uncommitted changes`). Do not compute the diff yourself. Skip roots with empty uncommitted diff; do not fall back to `branch changes` unless overlay/user says so.
3. **Security** (only if `security.enabled` or user asked): same Diff policy via `security-review` subagent.
4. **Verdict:** FAIL if gaps or any Critical/High (Bugbot; Security if enabled). Else PASS. Medium/Low → summary only.
5. **On PASS:** go to Phase 4 step 3 (close lifecycle). **On FAIL:** go to Phase 4 step 1 in the **same turn** when possible.

## Phase 4 — Fix (max 2)

1. FAIL and `fix_cycle < max_fix_cycles`: fix only gaps + Critical/High; set `phase: "fix"` then apply fixes; `fix_cycle++`; set `phase: "review"`; **return to Phase 3** in the same turn when possible.
2. FAIL and limit reached: stop, report, no commit. Keep state for resume or delete only if user cancels.
3. PASS: delete state; **final** user-visible summary only here: verdict PASS/FAIL, fix cycles used, plan-compliance table, remaining Medium/Low, optional «runtime not verified» note.

## Continuous pipeline (mandatory)

```text
/task-lifecycle → Phase 1 (stop for approval)
     ↓ user approves / Implement plan
Phase 2 Implement → state review → Phase 3 Review → PASS? → close
                                      ↓ FAIL
                              Phase 4 Fix (≤2) → Phase 3 again
```

Anti-patterns (never):

- Stopping after Phase 2 because plan todos are checked off
- Treating «Implement the plan» as a normal task outside Review
- Skipping Bugbot when `bugbot.enabled` and there is a non-empty configured diff in affected roots
- Deleting state before PASS verdict

## Do not

- Skip plan approval
- **Skip Phase 3 Review** after implementation (or end the lifecycle without PASS/FAIL-at-limit)
- Skip analysis before clarify, or clarify via chat a/b/c when AskQuestion UI is available
- Exceed clarify question cap or run endless `/grilling` inside lifecycle
- Exceed `max_fix_cycles` autofixes
- Auto-commit / commit without an explicit ask
- Start lifecycle without `/task-lifecycle`
- Ignore an existing project overlay
- Load all reference files up front — read only what the current phase needs

## Progressive disclosure

- Config flags / overlay YAML → [configuration.md](references/configuration.md)
- Clarify / Plan UI grilling → [clarify.md](references/clarify.md)
- Review templates & verdict → [review.md](references/review.md)
- State schema & resume → [resume.md](references/resume.md)
