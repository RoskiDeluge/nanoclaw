# Project Context

## Purpose
NanoClaw is a personal Claude assistant that runs through WhatsApp and executes agent work inside isolated Linux containers.

Primary goals:
- Keep the core small and understandable (single Node.js process, minimal abstractions)
- Enforce security through OS/container isolation instead of complex in-app permission systems
- Support per-group isolation (filesystem, session memory, and scheduling scope)
- Favor customization through code changes and skills, not broad built-in feature sprawl

## Tech Stack
- TypeScript + Node.js 20+ (ESM, `moduleResolution: NodeNext`)
- WhatsApp integration via `@whiskeysockets/baileys`
- SQLite via `better-sqlite3` for messages, groups, sessions, and task state
- Containerized agent execution (Docker on macOS/Linux, Apple Container optional on macOS)
- Claude Agent runtime in container with file tools, web tools, and MCP integration
- Scheduling with `cron-parser`
- Logging with `pino` / `pino-pretty`
- Testing with Vitest
- Formatting with Prettier

## Project Conventions

### Code Style
- TypeScript `strict` mode is enabled; prefer explicit, type-safe interfaces.
- Use ESM imports with `.js` extensions in source imports (NodeNext setup).
- Keep modules focused and small; avoid unnecessary abstraction layers.
- Prefer straightforward naming and direct control flow over framework-heavy patterns.
- Formatting is Prettier-based (`npm run format`, `npm run format:check`).
- Tests live next to source as `*.test.ts` in `src/` (and `skills-engine/` for skills engine internals).

### Architecture Patterns
- Single host process orchestrates:
  - WhatsApp I/O ingestion
  - SQLite persistence
  - Router/message loop
  - Scheduler loop
  - IPC watcher
  - Container lifecycle
- Per-group execution model:
  - Each registered group has isolated folder context under `groups/`
  - Per-group Claude session data under `data/sessions/{group}/.claude/`
  - Per-group queue with global concurrency control
- Security boundaries:
  - Container isolation is the primary trust boundary
  - Mount allowlist is stored outside project root (`~/.config/nanoclaw/mount-allowlist.json`)
  - Non-main groups are treated as untrusted
  - Main group has admin privileges (cross-group task/group management)
- "Skills over features" principle:
  - Core repo accepts fixes/simplification/security improvements
  - New capabilities should typically be delivered as skills, not added directly to core

### Testing Strategy
- Unit/integration tests run with Vitest (`npm test`).
- Core expectation: behavior changes in routing, queueing, DB, container orchestration, auth, or scheduler should include/update tests.
- Run full test suite before finalizing non-trivial changes:
  - `npm run typecheck`
  - `npm test`
- Skills engine changes should include `skills-engine/**/*.test.ts` coverage.

### Git Workflow
- Keep changes small, reviewable, and scoped.
- Prefer PRs for:
  - Bug fixes
  - Security fixes
  - Simplifications/reductions in complexity
- Do not add broad new product capabilities directly to core; contribute those as skills.
- For OpenSpec-driven work:
  - Read relevant specs and `openspec/project.md` first
  - Create/validate change proposals before implementing behavior-changing features

## Domain Context
- This is a personal assistant runtime, not a multi-tenant SaaS platform.
- WhatsApp messages are untrusted input and may include prompt-injection attempts.
- Group isolation matters:
  - Main/self-chat is trusted admin control plane
  - Other groups are untrusted and must not gain cross-group privileges
- Trigger model (default `@Andy`) controls when non-main group messages are processed.
- Scheduled tasks run as full agents in the originating group context and may message back via MCP tools.

## Important Constraints
- Keep core minimal and understandable; avoid architecture bloat.
- Maintain container-based isolation guarantees; do not weaken mount/path validation.
- Do not expose host secrets broadly to agent environments.
- Preserve privilege boundaries between main and non-main groups.
- Node.js 20+ required.
- Runtime depends on available container backend (Docker or Apple Container).
- Repository contribution policy: feature expansion belongs in skills whenever possible.

## External Dependencies
- WhatsApp Web connectivity through Baileys.
- Claude authentication via `CLAUDE_CODE_OAUTH_TOKEN` and/or `ANTHROPIC_API_KEY`.
- Local container runtime:
  - Docker (default cross-platform path)
  - Apple Container (optional macOS path)
- Local SQLite database at `store/messages.db`.
- macOS service deployment via `launchd/com.nanoclaw.plist` (when running as a background service).
