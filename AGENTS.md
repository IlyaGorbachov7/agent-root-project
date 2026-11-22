# Workspace map

This directory is the Cursor workspace root. Read across it when the task spans more than one project.

## Where things live

- `openspec/` is an optional separate git repository for OpenSpec specifications. It is not required for every project. Use it when an agent drives the change; a project people edit directly can rely on its README and `docs/`. In real use the root links it as a submodule and records the path and commit. Connect it at `openspec/` before starting work when the repository already exists; otherwise create it with the first spec or change and then link it. This template does not contain that submodule yet. The layout, and the comparison with working without it, is in `README.md`. When it is in use, `openspec/specs/` is the current behavior. A domain is one area of behavior with its own `spec.md`; it is not the same as a project. The capability id is the path from `specs/` to that directory. With many specs across projects, nest them as `openspec/specs/<project>/<domain>/spec.md` (capability id `<project>/<domain>`). A shared domain stays flat: `openspec/specs/<domain>/spec.md`. `openspec/changes/<name>/` is one proposed change, with `proposal.md`, `design.md`, `tasks.md`, and delta specs on the same relative paths. Commit specifications in the `openspec` repository, then record the new commit in the root.
- `docs/` is one separate git repository for people: architecture, ADRs, and runbooks. In real use the root links it as a submodule and records the path and commit. Connect it at `docs/` before starting work. This template does not contain that submodule yet. The layout is in `README.md`.
- `.cursor/docs/` is instructions for the agent. `.cursor/rules/` and `.cursor/skills/` apply across the workspace.
- `devops/` is a container, not a repository. `platform/`, `ci-templates/`, and `local-compose/` are separate DevOps repositories in real use.
- `projects/<name>/` is one project and, in real use, its own git repository linked from the root as a submodule: the root records the path and commit, and the project keeps its own history. `projects/_template/` shows what a new project contains. Repositories under `devops/` use the same link. `docs/` and `openspec/` use it too. The steps are in `README.md`.

## Push and merge requests

When the user asks to push or open a merge request, use the GitLab token already present in the environment (`GITLAB_TOKEN`, host `GITLAB_HOST`) via `git push` and `glab`. Do not ask for the token in chat and do not write it into the repository, remote URLs, or command output. Push a branch and open a merge request into the protected branch. The steps are in `README.md` under Советы.

## Where to implement

Implement behavior in `projects/<name>/`. When the workspace uses OpenSpec for this change, keep `openspec/` as specifications only and implement `tasks.md` in the project repository. A project without specs is implemented from the request, its README, and `docs/`. Keep a project's Dockerfile and CI in that project and point them at the shared pieces under `devops/`.
