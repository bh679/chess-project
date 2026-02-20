# Chess Project

An online chess application built with a **Product Engineer** agent workflow — where an AI agent owns each feature end-to-end: plan, build, test, ship.

The project is a full-stack chess game with a static HTML/CSS/JS client and an Express.js API backed by SQLite. Every feature is delivered through a structured agent workflow that uses plan mode, git worktrees, automated testing, and wiki documentation.

**Live:** [brennan.games/chess](https://brennan.games/chess/)

## Architecture

```
chess-project/                  ← this repo (orchestration layer)
├── CLAUDE.md                   ← Product Engineer agent instructions
├── .claude/settings.json       ← agent permissions (allow/deny rules)
├── playwright.config.js        ← Playwright test configuration
├── ports/                      ← dev server port claims (per session)
├── worktrees/                  ← git worktrees for concurrent development
│
├── chess-client/               ← client repo (bh679/chess-client)
│   ├── index.html              ← single-page app, loads all JS/CSS via script tags
│   ├── js/                     ← one module per file, vanilla JavaScript
│   ├── css/                    ← stylesheets
│   └── img/pieces-*/           ← art style directories (SVGs per piece)
│
├── chess-api/                  ← server repo (bh679/chess-api)
│   ├── index.js                ← Express app, health endpoint, static file serving
│   ├── db.js                   ← SQLite schema, migrations, query helpers
│   └── routes/games.js         ← game CRUD endpoints
│
├── Wiki/                       ← Chess wiki (bh679/chess-client.wiki)
├── chess-api-wiki/             ← chess-api wiki (bh679/chess-api.wiki)
└── chess-project-wiki/         ← chess-project wiki (bh679/chess-project.wiki)
```

### chess-client

Static site — HTML, CSS, vanilla JavaScript. No framework, no build step, no package manager for client code. Game state is stored in `localStorage` first (local-first), then synced to the server API in the background every 10 seconds.

### chess-api

Express.js REST API with SQLite (`better-sqlite3`). Serves both the API endpoints (`/api/*`) and the client static files from a single port. Idempotent move recording via `INSERT OR IGNORE` with UNIQUE constraints.

### chess-project

This repo. Contains the Product Engineer agent instructions (`CLAUDE.md`), permission configuration (`.claude/settings.json`), Playwright test setup, and orchestrates the sub-repos via git worktrees.

## Agent Workflow Overview

Each feature is delivered by a **Product Engineer** agent in a dedicated Claude Code session. The workflow follows a strict lifecycle:

```
┌─────────┐    ┌──────────┐    ┌────────────────┐    ┌──────────────────┐
│  Idea   │───▶│ Planned  │───▶│ In Development │───▶│ Ready for Testing│
└─────────┘    └──────────┘    └────────────────┘    └──────────────────┘
  Agent          User              Agent                  User tests
  explores       approves          implements             locally
  codebase       plan              in worktree
                                                              │
                                                              ▼
                                                         ┌─────────┐
                                                         │  Done   │
                                                         └─────────┘
                                                          PR merged,
                                                          wiki documented,
                                                          worktree cleaned
```

1. **Intake** — User describes a feature. Agent creates a project board item, estimates effort, and triggers score recalculation.
2. **Planning** — Agent enters plan mode (read-only exploration), designs the implementation, writes a plan file. User approves via the Desktop UI.
3. **Implementation** — Agent creates a git worktree for isolated development, starts a dev server on a unique port, and implements the feature following repo-specific coding standards.
4. **Testing** — Agent runs API tests via curl and Playwright browser tests with screenshot analysis. Results are posted in the session and on the project board.
5. **Shipping** — Agent creates a PR, user confirms merge, agent merges via squash. Feature is documented in the appropriate wiki. Worktree and port are cleaned up.

See the [chess-project wiki](https://github.com/bh679/chess-project/wiki) for full documentation:
- [Product Engineer](https://github.com/bh679/chess-project/wiki/Product-Engineer) — agent role, responsibilities, and configuration
- [Workflow Design](https://github.com/bh679/chess-project/wiki/Workflow-Design) — detailed workflow stages and mechanisms
- [Permissions](https://github.com/bh679/chess-project/wiki/Permissions) — how agent permissions work

## Permissions

Agent permissions are defined in `.claude/settings.json` and control what the Product Engineer can do autonomously.

**Allowed (no user prompt):**
- `Edit` — edit existing files
- `Write` — create new files
- `Bash` — run shell commands
- `WebFetch(domain:github.com)` — fetch from GitHub
- `WebFetch(domain:raw.githubusercontent.com)` — fetch raw GitHub content

**Denied (blocked entirely):**
- `Bash(git push --force *)` — no force pushing
- `Bash(git reset --hard *)` — no hard resets
- `Bash(rm -rf *)` — no recursive force deletion

This gives the agent broad autonomy for normal development while preventing destructive operations that could lose work.

## Project Board

Features are tracked on a [GitHub Project V2 board](https://github.com/users/bh679/projects/1) with automated scoring. The Score field is auto-calculated by a GitHub Action and ranks features by priority, effort, complexity, pipeline progress, and dependency impact.

See [Scoring](https://github.com/bh679/chess-client/wiki/Scoring) for the full formula.

## Setup

Clone this repo, then clone the sub-repositories inside it:

```bash
git clone git@github.com:bh679/chess-project.git chess-project
cd chess-project

# Clone the sub-repositories
git clone git@github.com:bh679/chess-client.git chess-client
git clone git@github.com:bh679/chess-api.git chess-api
git clone git@github.com:bh679/chess-client.wiki.git Wiki
git clone git@github.com:bh679/chess-api.wiki.git chess-api-wiki
git clone git@github.com:bh679/chess-project.wiki.git chess-project-wiki

# Install Playwright (for agent testing)
npm install
npx playwright install
```

## Running Locally

The chess-api serves both the API and the client static files from a single port:

```bash
CLIENT_DIR=./chess-client PORT=3001 node ./chess-api/index.js
```

- `http://localhost:3001/` — chess client UI
- `http://localhost:3001/api/*` — API endpoints

## Product Engineer Sessions

Open a Claude Code Desktop session from this directory. Each session handles one feature end-to-end. The agent will:

1. Discover its session ID and rename the session title
2. Create a project board item with the feature description
3. Enter plan mode and present a plan for approval
4. Implement, test, and ship the feature
5. Document the feature in the wiki and clean up

See `CLAUDE.md` for the full agent instructions.
