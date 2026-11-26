# Claude setup

A reusable Claude Code setup for a full-stack project: agents, rules, skills, a ticket system, docs and a worktree workflow. It has no application code. `backend/` and `frontend/` are empty folders.

## What is here

| Path | What it is |
| ---- | ---------- |
| `CLAUDE.md` | Entry point Claude reads first: stack, workflow, non-negotiables, definition of done |
| `SETUP.md` | Step-by-step setup for a new project, with the prompts to paste |
| `.claude/agents/` | 6 agents: architect, db-architect, developer, integration-owner, qa-reviewer, ui-implementer |
| `.claude/rules/` | 11 rule files: backend, database, documentation, frontend, general, git-workflow, parallel-work, planning, security, testing, validation |
| `.claude/skills/` | Project skills (`/new-module`, `/db-design`, ...) and Matt Pocock skills (`/grill-me`, `/tdd`, `/to-tickets`, ...) |
| `.claude/settings.json` | Permissions (allow, ask, deny) and enabled plugins |
| `docs/agents/` | How skills use the tracker, labels and domain docs |
| `docs/execution/` | Tickets, questions, decisions and specs: templates, worktree table, wiki |
| `docs/modules/` | Module spec, database and Figma templates |
| `docs/*.md`, `GLOSSARY.md` | Empty project docs to fill in |
| `script/keep-claude-working.ps1` | Keeps Claude working on `TASK.md` through limits and lost internet |
| `skills-lock.json` | Versions of the Matt Pocock skills |
| `.gitignore`, `.gitattributes` | Keep `.env`, `.claude/worktrees/` and local settings out of git; keep line endings LF |

## Start a project

Follow [SETUP.md](SETUP.md). In short:

1. In your project folder (git already initialized), clone this repo, then open the project folder in VS Code and tell Claude: `Help me set up my project. Follow claude_setup/SETUP.md, step 1.` Claude moves everything into the root and removes the clone's `.git`. Restart the chat.
2. Claude asks the developer how they want the backend and frontend set up (stack and libraries), records it and updates the rules (step 2a). Then it builds the foundation on `main`: workspaces, backend, frontend, shared, e2e, tooling and tests. Then it creates the `testing` branch (step 2b).
3. First module: `auth` (authentication + authorization), in its own worktree on `feature/auth` from `testing`.
4. Design and review the database first (`db-architect` agent, `/db-design`, `/db-design-review`, `qa-reviewer`).
5. Build auth one ticket at a time, never in parallel.
6. Merge auth into `testing`. After that, other modules can run in parallel.

## Parallel modules

Each module gets its own branch, folder, ports and databases. See [docs/execution/README.md](docs/execution/README.md) and [.claude/rules/parallel-work.md](.claude/rules/parallel-work.md).
