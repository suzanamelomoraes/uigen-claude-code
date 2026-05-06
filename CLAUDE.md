# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Start dev server with Turbopack on localhost:3000
npm run build        # Production build
npm run lint         # ESLint
npm test             # Vitest test suite
npm run setup        # Install deps + Prisma generate + migrate (first-time setup)
npm run db:reset     # Reset database (destructive — wipes all data)
```

Run a single test file:
```bash
npx vitest src/lib/__tests__/file-system.test.ts
```

**Important**: Do NOT run `npm audit fix` — it breaks version compatibility between dependencies.

## Environment

Copy `.env.example` → `.env` and set `ANTHROPIC_API_KEY`. Without it, the app falls back to a mock provider that returns canned responses without hitting Claude.

## Architecture

UIGen is an AI-powered React component generator. Users describe a component in chat; Claude generates it by calling file system tools; the result renders live in the browser.

### Request flow

1. User sends a chat message from `ChatInterface` → POST `/api/chat`
2. `src/app/api/chat/route.ts` runs `streamText` (Vercel AI SDK) with:
   - System prompt from `src/lib/prompts/generation.tsx` (cached with Anthropic ephemeral cache)
   - Two tools Claude can call: `str_replace_editor` (create/view/edit files) and `file_manager` (rename/delete)
   - Up to 40 tool-call steps (4 for mock provider)
3. Tool calls mutate an in-memory **virtual file system** (`src/lib/file-system.ts`) — nothing is written to disk
4. On stream finish, if the user is authenticated, the serialized VFS + message history are saved to SQLite via Prisma
5. The client receives the updated VFS state and re-renders the live preview

### Virtual File System (`src/lib/file-system.ts`)

All generated components live in an in-memory tree. It is serializable (stored as JSON in the Prisma `Project.data` column) and transmitted between client and server in every chat request. `/App.jsx` is always the root entrypoint.

### Live Preview (`src/lib/transform/jsx-transformer.ts`)

Uses `@babel/standalone` in the browser to compile JSX → JS. Missing imports are stubbed with placeholder modules, and CSS imports are stripped. The compiled output executes inside an iframe.

### State management

Two React Contexts, both provided in `src/app/main-content.tsx`:
- `FileSystemContext` (`src/lib/contexts/file-system-context.tsx`) — VFS state + helpers
- `ChatContext` (`src/lib/contexts/chat-context.tsx`) — wraps the Vercel AI SDK `useChat` hook; passes VFS state in each request body

### Authentication

JWT tokens signed/verified in `src/lib/auth.ts`, stored as HttpOnly cookies (7-day expiry). Passwords hashed with bcrypt. `middleware.ts` protects API routes. Server actions in `src/actions/` handle sign-up, sign-in, sign-out, and project CRUD.

### Database (Prisma + SQLite)

Two models: `User` and `Project`. `Project.messages` stores serialized chat history; `Project.data` stores the serialized VFS. The Prisma client is a singleton (`src/lib/prisma.ts`).

### AI provider (`src/lib/provider.ts`)

Returns either the real Anthropic Claude model or a mock fallback based on whether `ANTHROPIC_API_KEY` is set. The mock is useful for local development without API costs.

### Path alias

`@/*` resolves to `src/*` throughout the project.
