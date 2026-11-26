# Project Instructions for Claude Code

## Purpose

[Project name and one-line purpose: fill in from `docs/PROJECT.md`.] This file is the entry point for project-wide instructions. Read the relevant files under `.claude/rules/` and `docs/` before making substantial changes.

## Project setup

- If the user says "help me set up my project" (or similar), or the project has no `package.json` yet, follow `SETUP.md`:
  1. First show the **whole workflow step by step**: every step in `SETUP.md` (before you start, steps 1 to 6, the "Auth first" rule, the every-new-chat prompt), where each step is done (main folder or module folder), which agents and skills it uses, and what the developer sends back after it. Mark which steps are already done (from the checklist and the folder state).
  2. Then start the next unticked step: give its prompt and instructions, do the work, and tick it when done. Stop after each step with its report.
- Projects start with the `auth` module (authentication + authorization), in its own worktree from `testing` (`SETUP.md` steps 3 to 6). Its database is designed and reviewed first (`db-architect` agent, `/db-design`, `/db-design-review`, `qa-reviewer`), then it is built one ticket at a time, never in parallel. Other modules start only after `auth` is merged into `testing`; then they can run in parallel.

## Stack (default: the developer confirms or changes it in SETUP.md step 2a)

This is the default stack the rules and skills assume. In `SETUP.md` step 2a, Claude asks the developer how they want the backend and frontend set up (every area and library, showing this default), then records the answers here (marked "confirmed for this project"), in `docs/ARCHITECTURE.md`, `docs/DEPENDENCIES.md` and `docs/DECISIONS.md`, and updates the matching rules and skills. Once confirmed, do not ask about the stack again; change it only if the user explicitly asks. Whatever the stack, the validation-parity and test-case rules below always apply.

- Backend libraries: express (5), sequelize, mysql2, sequelize-cli, jsonwebtoken, bcryptjs, cookie-parser, cors, helmet, express-rate-limit, multer (file uploads), pino + pino-http, dotenv. Tests: jest, supertest, cross-env.
- Frontend libraries: react, react-dom, vite, @vitejs/plugin-react, @reduxjs/toolkit, react-redux, react-router, react-hook-form, @hookform/resolvers. Tests: jest, jest-environment-jsdom, babel-jest, @testing-library/react, @testing-library/user-event, @testing-library/jest-dom, msw.
- Shared: zod. E2E: @playwright/test. Root tooling: eslint, prettier, concurrently.

- Language: JavaScript (ESM) everywhere, no TypeScript. npm workspaces: `shared/`, `backend/`, `frontend/`, `e2e/`.
- Backend: Node.js + Express 5, Sequelize + MySQL, JWT access token (Bearer header) + refresh token (httpOnly cookie, rotated, stored hashed)
- Frontend: React + Vite, Redux Toolkit + RTK Query (auto-refresh on 401), React Router, react-hook-form
- Validation: Zod. Schemas and error messages live once in `shared/` (`@app/shared`; rename to match the project) and are imported by both apps.
- Database: MySQL **8.0.16+ required (target 8.4 LTS; no 5.7 or MariaDB)**, checked at startup, via Sequelize; migrations and seeders are `.cjs` files in `backend/src/migrations` / `backend/src/seeders`
- Tests: Jest (shared, backend with supertest, frontend with React Testing Library + MSW), Playwright (`e2e/`)
- UI source of truth: Figma, read through the Figma MCP, one module at a time

## Commands (from the repo root, once `package.json` exists)

- `npm run dev`: backend (http://localhost:5000) + frontend (http://localhost:5173, proxies `/api`)
- `npm test`: all Jest suites. One workspace: `npm test -w backend`
- `npm run test:e2e`: Playwright
- `npm run lint`, `npm run format`, `npm run build`
- `npm run check`: tests, lint, `prettier --check` and build in one go (run before every commit)
- DB: `npm run db:migrate -w backend`, `npm run db:seed -w backend`, `npm run migration:new -w backend -- <name>` (rename the generated file to `.cjs`)

## First actions

1. Check `git status`, then read `docs/PROJECT.md`, `docs/ARCHITECTURE.md`, `docs/REQUIREMENTS.md`, `docs/DECISIONS.md`, the relevant `docs/modules/<module>/` folder, and `docs/execution/<module>/` (board, STATUS, open questions).
2. If important requirements are missing or ambiguous, ask focused questions in small batches (or run `/grill-with-docs`). Record answers in the docs.
3. For a new module or substantial change, propose a plan before broad implementation.

## Module workflow

Run `/new-module <name>`. It drives each step in order and stops for approval between steps:

1. Align: `/grill-with-docs`, then write the module spec in `docs/modules/<module>/README.md`
2. Database: `/db-design`, then `/db-design-review`
3. Contract: `/shared-validation` (Zod schemas + messages) and the API contract
4. Tickets: `/to-tickets` (and `/to-spec` if it needs a published spec)
5. Backend: `/express-module`, test-first through `/tdd` and `/jest-testing`
6. Frontend: `/figma-to-module` for each page listed in `docs/modules/<module>/figma.md`
7. E2E: `/playwright-e2e`
8. Review: `/code-review`, `/db-design-review` if the schema changed, then `/security-review`

## Non-negotiables

- One source of truth for validation: never write a Zod schema or user-facing validation message inside `backend/` or `frontend/`; import it from `shared/`. See `.claude/rules/validation.md`.
- Every field gets explicit validation (type, required/optional, length, format, range), and every rule has a message from `shared/`.
- Database changes go through migrations only. Never use `sequelize.sync({ force/alter })` outside throwaway tests.
- UI must match Figma exactly: use Figma variables/tokens and downloaded assets, and verify with a screenshot comparison.
- Every validation error message is shown with the same text in the API response, the frontend form and on screen in the UI. Never ask whether to do this.
- Every change has tests, and every module lists its test cases (what, how, expected, result) in `docs/execution/<module>/TEST-CASES.md`. Never ask whether to do this.

## General rules

- Follow the existing architecture and conventions unless an approved change is needed.
- Prefer small, testable, maintainable changes over broad rewrites.
- Do not invent business rules, API contracts, permissions, or data relationships. Ask, or document a clearly labeled assumption.
- Reuse existing components and utilities; avoid unnecessary dependencies and duplicated logic.
- Keep secrets, credentials, personal data, and environment files out of source control and logs.
- Never run destructive commands, reset shared data, deploy to production, or perform irreversible actions without explicit approval.
- Do not claim a task is complete until relevant tests/checks have been run or the unverified parts are clearly disclosed.
- Update documentation when behavior, interfaces, setup, or architectural decisions change.

## Branches

`agent/<module>/<task>` → `feature/<module>` → `testing` → `main`.

- Each module has its own worktree folder `..\<project>-<module>` on `feature/<module>` and its own Claude session. It learns about other modules by reading git (`git fetch`, `git log`, `git show origin/feature/<other>:<path>`), never by editing their folders, and takes their code only from `testing`.
- Inside a module, Claude splits the work into tasks (for example login, signup, onboarding, forgot password), runs each in a sub-agent with its own task worktree, merges it into `feature/<module>`, and removes that worktree itself when the task is done.
- Claude pushes only feature branches. The user removes the module worktree when the module is done. Merging into `testing`, and from `testing` into `main` (merge or pull request), happens only when the user asks. Branches are never deleted.
- Full steps: `.claude/rules/git-workflow.md` and `.claude/rules/parallel-work.md`.

## Parallel work

- Before parallelizing, define task boundaries, dependencies, file ownership, interfaces, and acceptance criteria.
- `shared/` (schemas, messages, error codes), migrations, and app bootstrap/routing are shared foundations with a single owner (the `integration-owner` agent).
- Avoid assigning multiple agents to edit the same files at the same time. Use separate branches/worktrees for independent tasks. See `.claude/rules/parallel-work.md` and `docs/execution/README.md`.
- Do not merge work merely because it compiles; review changes and run relevant tests.

## Definition of done

1. Acceptance criteria are met.
2. Jest and Playwright suites relevant to the change pass, plus lint/build where available.
3. Frontend and backend reject the same invalid input with the same message (parity test exists).
4. Security, error handling, and edge cases have been considered.
5. Documentation is updated where needed.
6. Remaining risks, assumptions, failures, and follow-up work are reported.

## Agents, skills and tools available

- Agents (`.claude/agents/`): `architect`, `db-architect`, `developer`, `integration-owner`, `qa-reviewer`, `ui-implementer`
- Project skills (`.claude/skills/`): `new-module`, `db-design`, `db-design-review`, `shared-validation`, `express-module`, `figma-to-module`, `jest-testing`, `playwright-e2e`
- Matt Pocock skills (copied into `.claude/skills/`, tracked in `skills-lock.json`, update with `npx skills@latest update -p`): `/grill-me`, `/grill-with-docs`, `/tdd`, `/to-spec`, `/to-tickets`, `/implement`, `/implement-spec`, `/triage`, `/wayfinder`, `/chief-of-staff` (long-running goal), `/diagnosing-bugs`, `/improve-codebase-architecture`, `/handoff`, `/ask-matt`, and more. Configured by `/setup-matt-pocock-skills` (see Agent skills below).
- Official plugins: `figma`, `playwright` (browser MCP), `context7` (current library docs), `pr-review-toolkit`, `security-guidance`
- Built in: `/code-review`, `/security-review`, `/simplify`

## Agent skills

### Issue tracker

Local markdown, one folder per module: `docs/execution/<module>/` holds the board (README.md), STATUS.md, and one file per ticket (`tickets/<PREFIX>-NNN-<slug>.md`), question, decision and spec, with a prefix per module. See `docs/agents/issue-tracker.md` and `docs/execution/README.md`.

### Triage labels

Default vocabulary (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`), written as text in specs and tickets. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: `GLOSSARY.md`, `docs/DECISIONS.md` (the ADRs) and `docs/modules/<module>/`. See `docs/agents/domain.md`.
