# Git Workflow Rules

- Check `git status`, current branch, and recent history before editing.
- Do not overwrite, discard, reset, or clean up user changes unless explicitly authorized.
- Use descriptive feature branches and focused commits.
- Never commit secrets, credentials, local environment files, generated artifacts, or personal data.
- Do not amend or rewrite shared/published history unless explicitly requested.
- Review the diff before committing; keep commits scoped to the task.
- Merge only after review and required checks pass.
- If a conflict appears, inspect both sides and preserve intended behavior.
- Do not push, open pull requests, merge, release, or deploy unless the task/request and permissions authorize it.

## Branch flow

`agent/<module>/<task>` (optional, inside a module) → `feature/<module>` → `testing` → `main`

| Branch | Who works on it | Lives in |
| ------ | --------------- | -------- |
| `main` | Nobody directly. Receives `testing` by merge or pull request. | the main checkout (project root) |
| `testing` | Receives finished modules. The user tests here manually. | the main checkout |
| `feature/<module>` | The module's own Claude session. | its module worktree `..\<project>-<module>` |
| `agent/<module>/<task>` | One sub-agent, one task. | `.claude/worktrees/<task>` inside the module worktree |

## Project foundation (once, before any module)
The only work committed straight to `main` is the one-time foundation from `SETUP.md` steps 2a and 2b (stack choice, then workspaces, backend, frontend, shared, e2e, tooling, tests), done in the main checkout. After it passes `npm run check`, push `main` and create `testing` from it: `git checkout -b testing && git push -u origin testing`. From then on every change reaches `main` only through `testing`.

## Module lifecycle

### 1. Start: one worktree per module
The first module is always `auth` (authentication + authorization), built alone and one ticket at a time. Other modules start only after `feature/auth` is merged into `testing`; from then on they can run in parallel, each in its own worktree.

When the user asks to start a module, create its worktree from the main checkout, branched from the latest `testing` (or `main` if `testing` does not exist yet):

```
git fetch origin
git worktree add -b feature/<module> ..\<project>-<module> origin/testing
```

- Add the module's row to the tables in `docs/execution/README.md` (prefix, branch, folder, ports, databases) and create its `backend/.env` and `.env.worktree`.
- The user then opens that folder in its own Claude Code session. All work for the module happens there, fully independent of the main checkout and of other module folders.
- Never `git checkout` another branch inside a module folder, and never edit files in another module's folder.

### 2. While working: stay aware of other modules through git
Each module session learns what the others are doing by reading git, not by editing their folders. At the start of every session, and before touching anything shared (`shared/`, routes index, models index, migrations, auth):

```
git fetch origin
git log --oneline -15 origin/testing
git branch -r --list "origin/feature/*"
git log --oneline -10 origin/feature/<other-module>
git show origin/feature/<other-module>:docs/execution/<other-module>/STATUS.md
```

- If this module depends on another one (for example `orders` needs the `auth` user and token), read that module's shared schemas, API contract and models from its branch with `git show origin/feature/<other>:<path>`. Build against that contract.
- Get other modules' **code** into this branch only from `testing`: `git merge origin/testing`. Do not merge another `feature/*` branch directly. If you need code that is not in `testing` yet, build against its documented contract and record the dependency in this module's `STATUS.md`. Ask the user only if the contract is missing or unclear.
- Keep edits to shared foundations small and additive, so modules merge into `testing` cleanly.
- Push `feature/<module>` regularly (after `npm run check` passes) so other module sessions can read the progress.

### 3. Inside the module: task worktrees for sub-agents
Split a module into independent tasks and run each in a sub-agent with its own worktree. Claude creates these worktrees and deletes them itself. See `.claude/rules/parallel-work.md`, "Task worktrees inside a module".

### 4. Finish the module
When every ticket is done and tested in the module folder:

1. All task worktrees are already removed and their work is on `feature/<module>`.
2. `npm run check` passes, then push: `git push -u origin feature/<module>`.
3. Tell the user the module is ready. The **user** removes the module worktree (or asks Claude to: `git worktree remove ..\<project>-<module>`, never `--force`). The branch `feature/<module>` stays, locally and on GitHub.

### 5. Integrate into `testing` (only when the user asks)
From the main checkout:

```
git checkout testing
git pull
git merge --no-ff feature/<module>
npm run check
git push origin testing
```

The user then tests manually on `testing`. Bugs are fixed on `feature/<module>` (recreate its worktree if it was removed: `git worktree add ..\<project>-<module> feature/<module>`) and merged into `testing` again.

### 6. Release to `main` (only when the user asks)
After the user's manual testing on `testing`, either:
- open a pull request: `gh pr create --base main --head testing`, which the user reviews and merges; or
- merge directly when the user says so: `git checkout main && git pull && git merge --no-ff testing && git push origin main`.

## Never
- Never force-push `testing` or `main`. Fix bugs on the feature branch and merge again.
- Never delete a branch (`feature/*`, `agent/*`, `testing`, `main`) unless the user asks for that branch by name. Removing a worktree never deletes its branch.
