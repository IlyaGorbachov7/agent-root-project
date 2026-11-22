# Review phase — templates and verdict

Read this file at the start of Phase 3 (and again after each Fix before re-review).

## 3a. Plan compliance (parent agent)

Compare the approved plan todos to the actual changes.

| Todo | Status | Notes |
|------|--------|-------|
| … | done / partial / missing | … |

Rules:

- `done` — intent fully implemented within scope
- `partial` — started but incomplete or incorrect
- `missing` — not done
- Out-of-scope file/behavior changes count as **Gaps** even if todos look done

**Gaps** = any `partial` / `missing` **or** out-of-scope edits.

## 3b. Bugbot

Follow the `review-bugbot` skill shape. Defaults for lifecycle:

- `subagent_type: "bugbot"`
- `run_in_background: false`
- `description: "Bugbot"`
- Do **not** compute the diff yourself before launch
- One launch per affected git root with a non-empty configured diff
- Use `bugbot.diff` from effective config (default: `uncommitted changes`)

Prompt per root:

```text
Full Repository Path: <absolute path>
Diff: uncommitted changes
```

If overlay sets `branch changes`, use that Diff value instead.

Empty uncommitted diff on a root → skip that root; say so once. Do not silently switch to `branch changes`.

After the subagent finishes, summarize findings as:

| Severity | Location (file:line) | Finding |
|----------|----------------------|---------|
| Critical / High / Medium / Low | path:line | short text |

Sort by severity (highest first).

## 3c. Security (optional)

Only if `security.enabled: true` or the user explicitly asked in this run.

- `subagent_type: "security-review"`
- `run_in_background: false`
- `description: "Security Review"`
- Same Diff policy as Bugbot

```text
Full Repository Path: <absolute path>
Diff: uncommitted changes
```

## 3d. Verdict

| Result | Condition |
|--------|-----------|
| **FAIL** | Gaps **or** any Bugbot Critical/High **or** (if Security on) any Security Critical/High |
| **PASS** | No gaps and no Critical/High. Medium/Low listed in summary only |

Do not treat Medium/Low as FAIL.

## Output templates

### Review block

```markdown
## Plan compliance
| Todo | Status | Notes |
|------|--------|-------|
| … | … | … |

Gaps: none | <list>

## Bugbot
| Severity | Location (file:line) | Finding |
|----------|----------------------|---------|
| … | … | … |

(or: skipped — no uncommitted changes / disabled)

## Security
skipped | table same as Bugbot

## Verdict
PASS | FAIL — <one-line reason>
```

### Final summary (end of lifecycle)

```markdown
## Lifecycle result
- Verdict: PASS | FAIL
- Fix cycles used: N / max
- Remaining issues: none | Medium/Low list | unresolved Critical/High if FAIL at limit
```

## Fix scope reminder

On FAIL with cycles left, fix **only**:

- plan Gaps (`partial` / `missing` / out-of-scope rollback or completion)
- Critical/High findings

Then re-run the full Review (3a → 3d). Do not expand into unrelated refactors.
