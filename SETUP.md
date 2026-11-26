# Project setup steps

This file is for the developer and for Claude. Do the steps in order. Each step ends with a report: send it to your lead before you start the next step.

When the developer says **"help me set up my project"**, Claude reads this file and first explains the **whole workflow step by step**: every step below, where it is done, which agents and skills it uses, and what to send back after it. Then it works out which step is next (from the checklist at the bottom and the state of the folder) and starts that step, stopping after it with a report.

## Before you start (developer)

- Your project folder already exists and **git is already initialized** in it, with a GitHub remote (`git remote -v` shows `origin`).
- Installed: Node.js (current LTS), npm, Git, MySQL **8.4 LTS** (8.0.16 or newer, not 5.7 or MariaDB), VS Code with Claude Code.
- You have a MySQL user that can create databases. You will put its credentials in `backend/.env` yourself. Never paste passwords into the chat.

---

## Step 1: Bring the Claude setup into your project

1. Open a terminal **in your project folder** and run:
   ```bash
   git clone https://github.com/SyedMehandiRaza/claude_setup.git
   ```
   `git clone` creates a `claude_setup/` subfolder with its own `.git`. Your project now has two git folders. The one inside `claude_setup/` must go, and everything else must move up into your project root. Claude Code only loads `CLAUDE.md` and `.claude/` (skills, agents, rules) from the folder it is opened in.
2. Open **your project folder** (not `claude_setup`) in VS Code and start a new Claude Code chat.
3. Send: `Help me set up my project. Follow claude_setup/SETUP.md, step 1.` Or paste this prompt:

```text
In this folder there is a subfolder called "claude_setup" that I just cloned with git. This folder already has its own git repository (.git in the root): keep it untouched. Claude Code only loads CLAUDE.md and .claude/ (skills, agents, rules, settings.json) from the project root, so everything must be moved up into the root. Do the following:

1. Delete ONLY the ".git" folder inside "claude_setup/" (claude_setup/.git). Never touch the root .git.
2. Move EVERYTHING from "claude_setup/" into the current folder (the project root), including hidden files and folders such as .claude, .gitignore and .gitattributes. Keep the same folder structure. If a file with the same name already exists in the root (for example README.md or .gitignore), do not overwrite it: stop, show me both versions and ask me how to merge them.
3. Delete the now-empty "claude_setup" folder. If it fails with "busy" or "in use", this session's working directory is inside it: change the working directory to the project root and try again.
4. Check that:
   - the root contains CLAUDE.md, SETUP.md, README.md, GLOSSARY.md, skills-lock.json, .claude/, docs/, backend/, frontend/ and script/
   - .claude/ directly contains skills/, agents/, rules/ and settings.json
   - docs/execution/ exists with _templates/ and wiki/
   - every folder in .claude/skills/ has a SKILL.md file
   - the root .git is still there and `git status` works; there is no claude_setup folder and no other .git folder
5. Read CLAUDE.md, SETUP.md, README.md, every file in .claude/rules/ and docs/agents/, and list the skills in .claude/skills/ and the agents in .claude/agents/.
6. Tick step 1 in the checklist at the bottom of SETUP.md.

Do not commit yet. When done, give me a short report with:
- what you moved and deleted
- the final folder structure of the root (top level only)
- the result of each check in step 4
- a short summary of the setup steps and rules you read
- anything missing, failed, or that needs my decision
```

4. **Restart the chat (required).** Skills, agents and rules load only when a chat starts. Close the chat, keep your project folder open, start a new chat and type `/`: you should see skills such as `/new-module`, `/grill-me`, `/db-design`, `/tdd`.
5. Send the report, and whether the skills show, to your lead.

---

## Step 2a: Tell Claude your backend and frontend stack (main folder)

Before anything is installed, Claude asks you how you want the backend and frontend set up: languages, frameworks, database, every library, testing tools. Answer each question, or say "default" to keep the default shown (the stack in `CLAUDE.md`). Claude records your answers and updates the rules and skills to match, so every later chat uses your stack.

Paste this prompt in a new chat in your project root:

```text
Before setting up the project, ask me how I want the backend and frontend set up. Follow CLAUDE.md ("Stack") and .claude/rules/.

1. Ask me in small batches (backend first, then frontend, then testing and tooling). For every question show the current default from CLAUDE.md so I can just say "default". Cover at least:
   - Backend: language and runtime (and version), JavaScript or TypeScript, framework, database and version, ORM / query builder and migrations tool, authentication (JWT access + refresh, sessions, OAuth / social login), password hashing, validation library, file uploads and storage (local, S3, ...), email sending, logging, security middleware (helmet, CORS, rate limiting), caching / queues if any, API style (REST, GraphQL), API docs (Swagger / OpenAPI), env config, and any other library I want.
   - Frontend: framework and build tool, JavaScript or TypeScript, routing, state management, data fetching, forms, validation (shared with the backend), styling / UI library (CSS, Tailwind, MUI, ...), icons, charts, date handling, i18n, and any other library I want.
   - Testing and tooling: unit and integration test tools for backend and frontend, API test tool, E2E tool, linting and formatting, package manager and workspaces / monorepo layout, Node version, git hooks.
   - Anything else: deployment target, Docker, CI, Figma for the UI (yes / no).
2. Do not ask whether to validate every field with the same message in the API, the form and the UI, or whether to write tests and test cases: those are fixed rules for any stack. If I choose another validation or test library, apply the same rules with it.
3. When I have answered, show me the final stack as one table (area, choice, default or changed) and wait for my OK.
4. After my OK, record it:
   - the Stack section of CLAUDE.md (mark it "confirmed for this project"), docs/ARCHITECTURE.md, docs/DEPENDENCIES.md (each library and why), and a DEC-001 entry in docs/DECISIONS.md;
   - if anything differs from the default, update the matching rules (.claude/rules/backend.md, frontend.md, database.md, validation.md, testing.md) and skills (express-module, db-design, db-design-review, shared-validation, figma-to-module, jest-testing, playwright-e2e) so they use my stack, and step 2b of SETUP.md so its prompt installs my choices. List every file you changed.
5. Do not install anything or commit yet. Tick step 2a in the SETUP.md checklist.

Report: the final stack table, the files you changed, and anything that needs my decision.
```

Send the stack table to your lead.

---

## Step 2b: Set up the backend, frontend, shared and tests (in the main folder, on `main`)

This is the foundation every module builds on. It is done once, in your project root (the main checkout), on the `main` branch. No feature code (no login, no pages) yet: only the skeleton, tooling and tests. Then the `testing` branch is created from it.

Paste this prompt in a new chat in your project root:

```text
Set up the project foundation, following CLAUDE.md, SETUP.md and every file in .claude/rules/. Use the stack confirmed in step 2a (the Stack section of CLAUDE.md). The list below is the default stack: where step 2a chose something else, use the chosen equivalent and keep the same structure (shared validation, error shape, auth, tests). Do not ask me about the stack again. Validation (one shared schema, the same message in the API response, the form and the UI) and testing (tests plus written test cases) are rules, not questions: just apply them.

Work in this folder (the main checkout) on the main branch. Check `git status` first.

1. Root: npm workspaces `shared/`, `backend/`, `frontend/`, `e2e/`, JavaScript ESM, no TypeScript.
   - Scripts: dev (backend + frontend together with concurrently), test, test:e2e, lint, format, build, check (tests + lint + prettier --check + build).
   - Tooling: ESLint (flat config, @eslint/js, eslint-plugin-react, eslint-plugin-react-hooks, globals, eslint-config-prettier), Prettier, concurrently.
   - Make sure .gitignore covers node_modules, .env, .env.worktree, build output, test reports, .claude/worktrees/.

2. shared/ (package name @<project>/shared, matching the project name):
   - zod
   - src/messages/common.messages.js, src/errors/codes.js (VALIDATION_ERROR, NOT_FOUND, CONFLICT, UNAUTHORIZED, FORBIDDEN, INTERNAL_ERROR with status and message), src/schemas/common.schema.js (pagination page/pageSize with a max), index.js exports.
   - Jest tests for the common schema (valid case, each rule failing, exact message asserted).

3. backend/ (Node.js + Express 5), layout exactly as in .claude/rules/backend.md:
   - Libraries: express, sequelize, mysql2, sequelize-cli, zod (via shared), jsonwebtoken, bcryptjs, cookie-parser, cors, helmet, express-rate-limit, multer (file uploads), pino + pino-http (logging), dotenv.
   - Dev/test: jest, supertest, cross-env (and node --watch for dev).
   - config/env.js (all env vars validated with Zod, fail fast on startup), config/database.cjs (shared by app and sequelize-cli), .sequelizerc, MySQL 8.0.16+ version check at startup.
   - middlewares: validate({ body, params, query }), errorHandler (the exact 422 shape from validation.md), notFound, authenticate + authorize(...roles), rateLimiter, upload (multer: size limit, allowed MIME types, safe file names, files outside the web root).
   - utils: AppError, jwt (sign/verify access and refresh, hashToken), password (bcryptjs), cookies (refreshCookieOptions), response, logger.
   - app.js (createApp, no listen), server.js (connect DB, listen, graceful shutdown), routes/index.js mounting /api, GET /api/health.
   - backend/env.example with every variable name and no real values. Test DB name must end with _test and test setup must refuse otherwise.
   - Tests: supertest for /api/health, the 404 shape and the 422 validation shape (using a test-only route or schema).

4. frontend/ (React + Vite), layout exactly as in .claude/rules/frontend.md:
   - Libraries: react, react-dom, vite, @vitejs/plugin-react, @reduxjs/toolkit, react-redux, react-router, react-hook-form, @hookform/resolvers, zod (via shared).
   - Tests: jest, jest-environment-jsdom, babel-jest with @babel/preset-env and @babel/preset-react, @testing-library/react, @testing-library/user-event, @testing-library/jest-dom, msw, identity-obj-proxy.
   - app/store.js, router, services/api/baseApi.js (Bearer header, one shared refresh on 401), auth slice (token in memory only, never localStorage), ProtectedRoute, lib/applyServerErrors.js, styles/tokens.css (placeholder until Figma), components/ui/ (empty, ready), pages Home and NotFound, test/ with the MSW server and renderWithProviders.
   - Vite proxies /api to the backend; ports and URLs come from .env.worktree when it exists (see docs/execution/README.md).
   - Tests: a render test for Home/NotFound and a test that applyServerErrors puts an API fields error on the right form field with the exact message.

5. e2e/: @playwright/test, config reading .env.worktree when it exists, a smoke test that opens the app and calls /api/health.

6. Docs: fill docs/ARCHITECTURE.md (layout, env var names, ports), docs/DEPENDENCIES.md (every library above and why it is used), update the Commands section of CLAUDE.md if anything differs, and create docs/execution/setup/TEST-CASES.md from docs/execution/_templates/TEST-CASES.md listing every test you wrote, how you tested it and the result.

7. Ask me to fill backend/.env myself (DB host, user, password, names). Do not ask me to paste secrets into the chat. Then run the migrations (none yet is fine), `npm run check` and `npm run test:e2e`, and fix anything failing.

8. When everything passes: show me the diff summary, then commit on main ("chore: project foundation"), push main, create the testing branch from main and push it:
   git checkout -b testing && git push -u origin testing && git checkout main
   Ask me before pushing if the remote or branch looks wrong.

9. Tick step 2b in the checklist at the bottom of SETUP.md (in the same commit as the foundation, together with the step 2a stack changes).

Report: what you installed (with versions), the folder structure, commands and their results, test cases written (link to TEST-CASES.md), what you could not verify, and anything that needs my decision.
```

Send the report to your lead.

---

## Auth first, one task at a time

Authentication and authorization are built **first**, as one module called `auth`, before any other module. Every other module needs its users, roles, permissions, tokens and the `authenticate`/`authorize` middleware. Once `auth` is in `testing`, modules can be built in parallel, each one on top of the same login and permissions.

- `auth` is built **one stage and one ticket at a time** in the auth chat. No parallel sub-agents and no task worktrees for `auth`.
- No other module starts until `auth` is merged into `testing`.

---

## Step 3: Create the auth worktree (from `testing`)

Paste this prompt in a chat in your project root (the main checkout):

```text
Create the worktree for the auth module (authentication + authorization), following .claude/rules/git-workflow.md ("Module lifecycle") and docs/execution/README.md:

1. git fetch origin, then create the module worktree from testing:
   git worktree add -b feature/auth ..\<project>-auth origin/testing
   (use the real project folder name for <project>).
2. Add the auth row to the tables in docs/execution/README.md (prefix AUTH, branch, folder, ports, databases), and create backend/.env (from env.example, I will fill the secrets) and .env.worktree in the new folder with the next free ports and databases.
3. Run npm install in the new folder and check that npm run check still passes there.
4. Tick step 3 in the SETUP.md checklist (in the new folder, committed on feature/auth).
5. Tell me the absolute path of the new folder.

Do not start building auth here. Report what you created.
```

Open the new folder `..\<project>-auth` in VS Code and start a new chat there. Fill its `backend/.env` yourself. Do steps 4 to 6 in this folder.

---

## Step 4: Design and review the database (auth folder)

The database is designed and reviewed **before** any auth code. This step uses the `db-architect` agent for the design, the `/db-design` and `/db-design-review` skills, and the `qa-reviewer` agent for a second, independent review.

Paste this in the auth chat (it starts the module workflow and runs stages 1 to 3 only):

```text
/new-module auth

This module covers authentication AND authorization. Build it one stage at a time, no parallel sub-agents, no task worktrees (see SETUP.md "Auth first"). In this run do stages 1 to 3 only, then stop.

Stage 1 (Align): run /grill-with-docs on the auth scope and fill docs/modules/auth/README.md. Cover at least: sign up, login, logout, access + refresh token (rotation, reuse detection), email verification, onboarding, forgot / reset password, change password, roles and permissions (who can do what), account status (active, blocked), rate limits, and the admin side of user and role management. Ask me the business questions; do not invent rules.

Stage 2 (Pages): ask me for the Figma links for every auth page and fill docs/modules/auth/figma.md.

Stage 3 (Database):
- Use the db-architect agent with /db-design auth.
- First draw a high-level ERD of the whole project in docs/database/SCHEMA.md (every module listed in docs/PROJECT.md, main entities and how they link to users), marked "planned", so later modules connect to auth cleanly. Ask me for the module list if PROJECT.md does not have it.
- Then design the auth tables in detail (for example users, roles, permissions, role_permissions, user_roles, refresh_tokens, password_reset_tokens, email_verification_tokens: confirm each with me). Show the ERD and table specs and wait for my approval.
- After approval: write the migrations (with working down), Sequelize models, a seeder for the default roles and permissions, and update docs/modules/auth/db.md and docs/database/SCHEMA.md. Check that every VARCHAR length and ENUM matches the Zod limits planned for stage 4.
- Review: use the db-architect agent with /db-design-review auth, then the qa-reviewer agent for an independent review of the same migrations, models and docs. Fix every finding (or tell me why not) and review again until clean.
- Test: run the migrations up, down and up again on the _test database, and run the seeder. Add these to docs/execution/auth/TEST-CASES.md.
- Commit on feature/auth and push.

Tick step 4 in the SETUP.md checklist. Report: the approved ERD, tables and constraints, review findings and how they were fixed, migration test results, and open questions.
```

Send the report to your lead.

---

## Step 5: Build authentication and authorization (auth folder, one ticket at a time)

Paste this in the auth chat after step 4 is approved:

```text
Continue /new-module auth from stage 4 to stage 9, one stage and one ticket at a time, in this folder only. No parallel sub-agents and no task worktrees for auth.

Stage 4 (Contract): use the integration-owner agent with /shared-validation auth for every auth form and request (schemas and messages in shared/), and write the API contract table in docs/modules/auth/README.md. Include the authorization contract other modules will use: authenticate, authorize(...roles) and permission checks, the roles and permission names, and the 401/403 error shapes.

Stage 5 (Tickets): /to-tickets in dependency order, for example: migration/seed (done), sign up, email verification, login, refresh + logout, forgot / reset password, change password, onboarding, roles and permissions middleware, admin user and role management. Show me the list for approval.

Stages 6 to 8, for each ticket in order, one at a time:
- Backend: the developer agent with /express-module, test-first with /tdd and /jest-testing.
- Frontend: the ui-implementer agent with /figma-to-module for that ticket's pages.
- E2E: /playwright-e2e for that ticket's journey and validation-message parity.
- Every validation message must appear the same in the API response, the form and on screen.
- Fill the ticket's Verification and docs/execution/auth/TEST-CASES.md (what, how, expected, result).
- npm run check, commit on feature/auth, push, then the next ticket.

Stage 9 (Review): the qa-reviewer agent, then /code-review, /db-design-review diff if migrations changed, and /security-review (tokens, password hashing, rate limits, cookies, permission bypass). Fix the findings and re-run all tests.

Tick step 5 in the SETUP.md checklist. Report: tickets done, test counts and results, review findings and fixes, what is not verified, and open questions.
```

Send the report to your lead.

---

## Step 6: Finish auth and open the way for parallel modules

1. In the auth chat: `Check that every auth ticket is done, npm run check and npm run test:e2e pass, and feature/auth is pushed. Tick step 6a in the SETUP.md checklist, then tell me it is ready to merge.`
2. Your lead asks for the merge. In the **main checkout** chat: `Merge feature/auth into testing following .claude/rules/git-workflow.md step 5, run npm run check, push testing, and tick step 6b in the SETUP.md checklist once I confirm testing passed.`
3. Your lead tests on `testing`. Bugs are fixed in the auth folder on `feature/auth` and merged again.
4. When the lead says so, remove the auth worktree (`git worktree remove ..\<project>-auth`). The branch stays.

From now on, other modules can run **in parallel**: each one gets its own worktree from `testing` (as in step 3, with its own name) and its own chat, and inside each module Claude splits the work into task worktrees run by sub-agents and deletes them when done.

---

## Every new chat

Paste this first in every new Claude Code chat and wait for the reply before giving any task:

```text
Before doing anything, read CLAUDE.md, SETUP.md, and all files in .claude/rules/ and docs/agents/. Follow these setup steps and rules for all work in this session. Reply with a short summary of the rules you will follow and which setup step we are on, then wait for my task.
```

---


## Setup checklist

Claude ticks these as each step is done.

- [ ] Step 1: Claude setup moved into the project root, chat restarted, skills visible
- [ ] Step 2a: backend and frontend stack chosen by the developer, recorded, rules and skills updated
- [ ] Step 2b: foundation on `main` (shared, backend, frontend, e2e, tests), `testing` branch created
- [ ] Step 3: `feature/auth` worktree created from `testing`
- [ ] Step 4: whole-project ERD and auth database designed, reviewed (db-architect + qa-reviewer) and migrations tested
- [ ] Step 5: authentication and authorization built and reviewed, one ticket at a time
- [ ] Step 6a: auth ready to merge (all checks pass, `feature/auth` pushed)
- [ ] Step 6b: auth merged into `testing` and tested; parallel modules can start
