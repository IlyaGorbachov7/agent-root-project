# Clarify (Plan UI grilling)

Read this file in Phase 1 after readonly analysis and before `CreatePlan`.

## Goal

Resolve only **sharp decisions** the user must make. Prefer clickable Plan UI questions over chat text. Cap total questions so clarify never becomes an endless interview.

## Hard rules

| Rule | Value |
|------|-------|
| When | After analysis, still in **Plan mode** |
| Tool | `AskQuestion` (UI cards). Never ask the user to type `a`/`b`/`c` in chat when the tool is available |
| Batch size | **1–3** questions per `AskQuestion` round |
| Total cap | **5–10** questions for the whole clarify stage (default max **8**; overlay may set `clarify.max_questions` within 5–10) |
| Stop early | If decisions are clear, ask **0** more and go to `CreatePlan` |
| Facts vs decisions | Look up facts in the repo yourself. Ask only for decisions |
| Recommendation | Every question includes your recommended option (label it clearly, e.g. “(recommended)”) |
| Chat fallback | Only if `AskQuestion` is unavailable: one decision per message with A/B/C labels, still no infinite grill |

## What to ask / not ask

**Ask (examples):** API shape, migrate now vs later, scope in/out, breaking change ok?, which package owns the change, test depth.

**Do not ask:** file paths you can find, existing naming, how a function already works, anything answered by `AGENTS.md` / code.

## Flow

1. Finish readonly analysis; list open **decisions** (not facts) mentally.
2. If list empty → skip clarify → `CreatePlan`.
3. Sort by dependency: unblockers first.
4. Round: call `AskQuestion` with 1–3 items; wait for UI answers.
5. Apply answers; derive next decisions if needed.
6. Repeat until decisions settled **or** `questions_asked >= max_questions`.
7. If hit cap with leftovers: state remaining assumptions explicitly in the plan (do not keep grilling).
8. `CreatePlan` → stop for approval.

## Question card shape

For each item in `AskQuestion`:

- Clear prompt (one decision)
- 2–4 concrete options
- Mark the recommended option in the option label
- Optional extra option only when needed: “Other / I’ll type a note” — user may then add a short chat note

Prefer options that are complete choices (no follow-up typing required).

## Counting

- Each discrete decision presented counts as **1** toward the cap (even if several are in one round).
- Re-asking the same decision after a vague answer still counts.
- Informational status lines to the user do **not** count.

## Overlay flags

```yaml
task-lifecycle:
  clarify:
    enabled: true          # default true
    max_questions: 8       # clamp to 5–10
    batch_size: 3          # max per round, clamp to 1–3
```

If `clarify.enabled: false` → after analysis go straight to `CreatePlan` (still allow 1 blocking UI question only if truly impossible to plan).

## Do not

- Start clarify before analysis
- Ask in Agent mode when Plan mode is required for UI cards — switch to Plan first
- Run full `/grilling` / endless interview inside lifecycle
- Exceed `max_questions`
- Implement code during clarify
