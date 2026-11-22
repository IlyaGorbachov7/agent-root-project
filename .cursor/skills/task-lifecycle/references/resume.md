# State file and resume

Read this file when creating/updating state, resuming after a break, or finishing PASS/FAIL-at-limit.

## State path

Default: `.cursor/task-lifecycle-state.json` at workspace root.  
Override with overlay `state_file` if set.

## Schema

```json
{
  "phase": "implement | review | fix",
  "fix_cycle": 0,
  "max_fix_cycles": 2,
  "plan_uri": "<absolute or workspace-relative path to plan>",
  "repos": ["<git root absolute or relative>"],
  "task_summary": "<one-line user task>",
  "approved": true
}
```

Field notes:

- `phase` — next work to do (not “last finished”)
- `fix_cycle` — completed Fix rounds so far (0 before first Fix)
- `approved` — plan was confirmed; if `true`, never re-run Planner from scratch
- `task_summary` — short label for handoff messages

Optional (allowed):

```json
{
  "last_verdict": "FAIL",
  "open_gaps": ["…"],
  "open_critical_high": ["file:line — finding"]
}
```

## When to write

| Moment | Action |
|--------|--------|
| After plan approval, entering Implementer | create state: `phase: "implement"`, `fix_cycle: 0`, `approved: true` |
| After Implementer build step | set `phase: "review"`; agent **must** run Phase 3 next (same turn if possible) |
| User says «продолжи lifecycle» / `/task-lifecycle` with `phase: "review"` | run Phase 3 only; do not re-implement unless compliance found gaps needing code |
| Entering Fix | set `phase: "fix"`, then after edits set `phase: "review"` and bump `fix_cycle` |
| Each Fix start | `fix_cycle` already incremented before leaving Fix toward Review |
| PASS | **delete** state file |
| FAIL at max cycles | keep state (for manual resume/report) unless user cancels lifecycle |
| User escape / cancel | delete state if user wants a clean slate; otherwise leave and stop |

Recommended Fix counter sequence:

1. Review → FAIL, `fix_cycle` is `N`, `N < max`
2. Set `phase: "fix"`, apply fixes
3. Set `fix_cycle: N+1`, `phase: "review"`
4. Re-run Review

## Resume algorithm

On Boot, if state file exists:

1. Read state; show a one-line status: phase, fix_cycle/max, plan_uri, task_summary.
2. Confirm with user only if unclear whether to resume or start fresh. Prefer resume when `/task-lifecycle` continues the same task.
3. Branch:

| `phase` | Action |
|---------|--------|
| `implement` | Skip Planner; continue Implementer from remaining plan todos |
| `review` | Skip Planner/Implementer; run Phase 3 |
| `fix` | Continue Fix for open Critical/High + Gaps, then Review |

4. Never recreate a plan when `approved: true` unless the user explicitly asks to re-plan.
5. If `plan_uri` is missing/broken → ask user for the plan path or re-plan (exception).

## Fresh start

If the user says the previous run is abandoned (or escape words):

1. Delete state file (after stating you will).
2. Start Phase 1 if they still invoked `/task-lifecycle` with a new task; otherwise normal mode.
