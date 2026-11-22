# Lifecycle configuration

Read this file when resolving defaults, reading a project overlay, or answering a config question. Do not load it on every phase if config is already known.

## Precedence

```text
global defaults  <  project overlay  <  explicit user instruction in this chat
```

Later wins for **config flags only**. Overlay / chat cannot:

- skip plan approval
- auto-commit
- raise `max_fix_cycles` above 2 without an explicit user ask in this chat
- start lifecycle without `/task-lifecycle`

## Defaults

| Flag | Default |
|------|---------|
| `git_roots` | single root = workspace / discovered git root |
| `build_commands` | from `AGENTS.md` / README; else skip with a note |
| `bugbot.enabled` | `true` |
| `bugbot.diff` | `uncommitted changes` |
| `security.enabled` | `false` |
| `max_fix_cycles` | `2` |
| `state_file` | `.cursor/task-lifecycle-state.json` (workspace root) |
| `response_language` | user's language |
| `auto_commit` | `false` (always; never enable via overlay) |
| `clarify.enabled` | `true` |
| `clarify.max_questions` | `8` (clamp to 5–10) |
| `clarify.batch_size` | `3` (clamp to 1–3) |

If `bugbot.diff` is `uncommitted changes` and the working tree is clean: report once and **do not** fall back to `branch changes` unless the user or overlay requests it.

## Overlay discovery (Phase 1 / Boot)

Check in order; **later wins** when keys overlap:

1. `.cursor/skills/task-lifecycle/SKILL.md` (project skill)
2. `.cursor/rules/task-lifecycle.mdc`
3. Workspace / repo `AGENTS.md` sections about `/task-lifecycle`

Also accept a YAML/JSON block inside those files matching the schema below.

## Overlay schema

```yaml
task-lifecycle:
  git_roots:
    - .
  build_commands:
    - npm test
  bugbot:
    enabled: true
    diff: uncommitted changes   # or: branch changes
  security:
    enabled: false
  max_fix_cycles: 2
  state_file: .cursor/task-lifecycle-state.json
  response_language: ru         # optional; omit to follow user language
  auto_commit: false            # must stay false
  clarify:
    enabled: true
    max_questions: 8            # clamp 5–10
    batch_size: 3               # clamp 1–3
```

Minimal overlay example (only overrides):

```yaml
task-lifecycle:
  build_commands:
    - pnpm --filter api test
  security:
    enabled: true
  clarify:
    max_questions: 10
```

## Multi-root

- If `git_roots` lists several paths, treat each as a separate review target.
- Run Bugbot (and Security if enabled) **per root that has** a matching non-empty diff for the configured Diff mode.
- Roots with no relevant changes → skip (note in Review summary).
- Implementer still scopes edits to the plan's target packages/paths.

## Effective config checklist

Before Phase 1 handoff (or on resume), hold in working memory:

```text
git_roots: ...
build_commands: ...
bugbot.enabled / bugbot.diff: ...
security.enabled: ...
max_fix_cycles: ...
state_file: ...
clarify.enabled / max_questions / batch_size: ...
```

Do not paste the full overlay into every user message.
