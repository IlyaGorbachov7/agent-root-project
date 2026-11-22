# Workspace

Open this repository root in Cursor so the agent can see every project.

- Specs: `openspec/`, an optional git repository for work driven by an agent. Not every project needs it. In real use it is a submodule of the root: the root records the path and commit. Connect it at that path before work when it already exists; otherwise create it with the first spec and then link it. This template does not contain that submodule yet. Current behavior is `openspec/specs/`; proposed changes are `openspec/changes/`. With many specs, nest by project: `openspec/specs/<project>/<domain>/spec.md`. When to use it, and the tradeoffs, are in `README.md`.
- Human documentation: `docs/`, one git repository. In real use it is a submodule of the root: the root records the path and commit. Connect it at that path before work. This template does not contain that submodule yet. Layout is in `README.md`.
- Agent instructions: `.cursor/docs/`, `.cursor/rules/`, `.cursor/skills/`.
- Shared platform, CI templates, and local compose: `devops/platform/`, `devops/ci-templates/`, `devops/local-compose/`.
- Application code: `projects/<name>/`. In real use each project is a submodule of the root (the root records the path and commit). Copy `projects/_template/` when adding a project. The same link is used for repositories under `devops/`, and for `docs/` and `openspec/`. See `README.md`.

Implement tasks in the project repository. When OpenSpec is used for the change, keep `openspec/` as specifications only.

Push and merge requests use the GitLab token from the environment. Do not put the token in the chat or the repository. See `README.md`, section Советы.
