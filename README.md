# Chess Project

Project-level configuration for the Chess application. Contains the Product Engineer agent instructions, permissions, and test configuration.

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

# Install Playwright (for agent testing)
npm install
npx playwright install
```

## Directory Structure

```
chess-project/              ← this repo (project config)
├── CLAUDE.md               ← Product Engineer agent instructions
├── .claude/settings.json   ← pre-defined agent permissions
├── .gitignore              ← ignores sub-repos, worktrees, node_modules
├── chess-client/           ← client repo (bh679/chess-client)
├── chess-api/              ← server repo (bh679/chess-api)
├── Wiki/                   ← Chess wiki (bh679/chess-client.wiki)
├── chess-api-wiki/         ← chess-api wiki (bh679/chess-api.wiki)
└── worktrees/              ← created by agent sessions (gitignored)
```

## Product Engineer Sessions

Open a Claude Code Desktop session from this directory. Each session handles one feature end-to-end. See `CLAUDE.md` for the full workflow.
