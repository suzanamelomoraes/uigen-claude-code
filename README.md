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
