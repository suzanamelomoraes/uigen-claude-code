# UIGen

AI-powered React component generator with live preview.

## Prerequisites

- Node.js 18+
- npm

## Setup

1. **Optional** Edit `.env` and replace `your-api-key-here` with your Anthropic API key from [console.anthropic.com](https://console.anthropic.com/settings/keys):

```
ANTHROPIC_API_KEY=sk-ant-...
```

The project runs without an API key — it falls back to a mock provider that returns canned components instead of calling Claude. If you leave the placeholder unchanged, you'll get the mock.

2. Install dependencies and initialize the database:

```bash
npm run setup
```

> **Don't run `npm audit fix`.** Dependencies are pinned to specific versions that work together. The vulnerability warnings are cosmetic for a local-only project, and `audit fix` can bump packages past compatible versions and break the app.

This command will:

- Install all dependencies
- Generate Prisma client
- Run database migrations

## Running the Application

### Development

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

## Usage

1. Sign up or continue as anonymous user
2. Describe the React component you want to create in the chat
3. View generated components in real-time preview
4. Switch to Code view to see and edit the generated files
5. Continue iterating with the AI to refine your components

## Features

- AI-powered component generation using Claude
- Live preview with hot reload
- Virtual file system (no files written to disk)
- Syntax highlighting and code editor
- Component persistence for registered users
- Export generated code

## Tech Stack

- Next.js 15 with App Router
- React 19
- TypeScript
- Tailwind CSS v4
- Prisma with SQLite
- Anthropic Claude AI
- Vercel AI SDK

---

## Learning section:

### Custom Claude Code slash commands

This repo also serves as a sandbox for learning [Claude Code](https://claude.com/claude-code) features. The `.claude/commands/` directory contains two project-scoped slash commands created purely as learning exercises — not as recommended workflows for this project.

**`/audit`**

File: `.claude/commands/audit.md`

A no-argument command that walks Claude through three steps: `npm audit`, `npm audit fix`, and `npm test`. Created to explore how project-scoped commands are defined and discovered.

> **Do not actually run this.** As noted in the Setup section, `npm audit fix` will break dependency compatibility in this repo. The command exists only to demonstrate the slash-command file format.

**`/write_tests <target>`**

File: `.claude/commands/write_tests.md`

An argument-accepting command demonstrating Claude Code's `$ARGUMENTS` placeholder. Invoke it like `/write_tests src/lib/file-system.ts` and Claude will write Vitest + React Testing Library tests for that target, following the repo's conventions (`__tests__/` directories, `@/` import alias, happy paths + edge cases + error states).

> This one is safe to run and reflects the actual testing conventions used in the codebase.

**Notes on the `.claude/` directory**

- `.claude/commands/*.md` is committed so commands are shared with anyone who clones the repo.
- `.claude/settings.local.json` is gitignored — it holds machine-local Claude Code settings (e.g., approved permissions) that shouldn't be shared.

### Managing MCP Permissions

When you first use MCP server tools, Claude will ask for permission each time. You can pre-approve the server by editing your settings.

Open the .claude/settings.local.json file and add the server to the allow array:

```
{
  "permissions": {
    "allow": ["mcp__playwright"],
    "deny": []
  }
}
```

> Note the double underscores in **_mcp\_\_playwright_**. This allows Claude to use the Playwright tools without asking for permission every time.

### GitHub App integration (`/install-github-app`)

Claude Code ships with a built-in `/install-github-app` slash command that wires up the [Claude Code GitHub Action](https://github.com/anthropics/claude-code-action) so Claude can respond to issues and review pull requests directly on GitHub.

**What was set up**

Running `/install-github-app` performed the following:

1. Installed the **Claude GitHub App** on this repository, granting it the permissions it needs to read issues/PRs and post comments.
2. Added `ANTHROPIC_API_KEY` as a repository secret so the workflows can authenticate with the Anthropic API.
3. Created two workflow files under `.github/workflows/`:
   - **`claude.yml`** — Triggers on issue comments, PR review comments, PR reviews, and new/assigned issues. The job only runs when the body or title contains `@claude`, at which point Claude reads the context and follows the instructions in the comment.
   - **`claude-code-review.yml`** — Runs automatically on every pull request (`opened`, `synchronize`, `ready_for_review`, `reopened`) and invokes the `code-review` plugin from the official Claude Code plugin marketplace to leave an automated review.

**Purpose**

- **`claude.yml`** turns `@claude` into an on-demand assistant inside GitHub: tag it in an issue to draft an implementation, or in a PR comment to ask for changes. It uses the same toolset as local Claude Code, just executed in a GitHub Actions runner.
- **`claude-code-review.yml`** provides a "second pair of eyes" on every PR without anyone having to ask — useful for catching issues early in a learning project where there isn't always a human reviewer available.

**Notes**

- Both workflows request the minimum permissions needed (`contents: read`, `pull-requests: read`, `issues: read`, `id-token: write`). `claude.yml` additionally requests `actions: read` so Claude can inspect CI results when asked about a failing build.
- The workflow files include commented-out examples showing how to scope reviews to specific paths, filter by PR author, or pass extra `claude_args` (e.g. restricting tool access). They are kept as-is here for learning visibility.
- The API key lives in repo secrets — it is never echoed into logs, and the workflows reference it via `${{ secrets.ANTHROPIC_API_KEY }}`.
