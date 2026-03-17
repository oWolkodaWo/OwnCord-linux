# CLAUDE.md

This file provides guidance to Claude Code when working with
code in this repository.

OwnCord is a self-hosted Windows chat platform with two
components: a Go server (`chatserver.exe`) and a Tauri v2
desktop client (Rust + TypeScript).

## Codex CLI - Code REVIEW

After builds, run Codex for a second opinion:

codex exec --sandbox read-only \
"Review for bugs and logic errors"

## Reference Files (read before implementing)

- **CHATSERVER.md** -- Master spec: phases, tasks, security
  priorities, Windows-specific details.
- **PROTOCOL.md** -- WebSocket message format. Every message
  type, payload shape, and rate limit. Server and client
  must agree on this exactly.
- **SCHEMA.md** -- SQLite table definitions, indexes, FTS5
  setup, permission bitfield definitions.
- **API.md** -- REST endpoints, request/response shapes,
  error codes. All paths start with `/api/v1/`.
- **SETUP.md** -- Tooling requirements for both server and
  client development.
- **CLIENT-ARCHITECTURE.md** -- Tauri v2 client project
  structure, component map, store design, and conventions.
- **TESTING-STRATEGY.md** -- Test infrastructure, coverage
  targets, and patterns for every test type.

## Project Structure

```text
OwnCord/
├── Server/                  # Go server (implemented)
│   ├── config/
│   ├── db/
│   ├── auth/
│   ├── api/
│   ├── ws/
│   ├── admin/static/
│   └── migrations/
├── Client/
│   ├── tauri-client/        # Tauri v2 client
│   │   ├── src-tauri/       #   Rust backend
│   │   │   └── src/
│   │   ├── src/             #   TypeScript frontend
│   │   │   ├── lib/         #     Core services
│   │   │   ├── stores/      #     Reactive state
│   │   │   ├── components/  #     UI components
│   │   │   ├── pages/       #     Page layouts
│   │   │   └── styles/      #     CSS (from mockups)
│   │   └── tests/
│   │       ├── unit/
│   │       ├── integration/
│   │       └── e2e/
│   └── ui-mockup.html      # Design source of truth
└── docs/
```

## Build Commands

### Server (Go)

```bash
cd Server
go build -o chatserver.exe -ldflags "-s -w -X main.version=1.0.0" .
go test ./...                        # all tests
go test ./... -cover                 # with coverage
```

### Client (Tauri v2)

```bash
cd Client/tauri-client

# Development (hot reload)
npm run tauri dev

# Build release
npm run tauri build

# Run tests
npm test                             # all tests (vitest)
npm run test:unit                    # unit tests only
npm run test:integration             # integration tests
npm run test:e2e                     # Playwright E2E tests
npm run test:coverage                # with coverage report
```

### Dev Tools

```bash
# Server
go install github.com/air-verse/air@latest
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

# Client (installed via npm)
# vitest, playwright, typescript, vite — all in package.json
```

## Branch Strategy

- `main` -- stable releases
- `tauri-migration` -- active development

## Critical Rules (always apply)

- **API paths**: Always `/api/v1/*` (matches server router)
- **WS field names**: `threshold_mode` NOT `mode` in
  VoiceConfig and VoiceSpeakers payloads
- **Roles**: Always use role NAME strings ("admin",
  "member"), never numeric role\_id in UI-facing code
- **Rate limiting**: Client must respect PROTOCOL.md
  limits (typing 1/3s, presence 1/10s, voice 20/s)
- **Status values**: Only `online`, `idle`, `dnd`,
  `offline`. Never `invisible`.

## Conventions & Details (see canonical files)

- **Client architecture & conventions**:
  CLIENT-ARCHITECTURE.md
- **Server spec & conventions**: CHATSERVER.md
- **Security rules**: CHATSERVER.md (Security section)
- **Testing requirements**: TESTING-STRATEGY.md
- **Coverage target**: 80%+ (TDD: RED → GREEN → IMPROVE)

## gstack Skills

gstack is installed at `~/.claude/skills/gstack`.

- **Web browsing**: Always use `/browse` from gstack for
  all web browsing. Never use `mcp__claude-in-chrome__*`
  tools.

Available skills:

- `/plan-ceo-review` — CEO-level plan review
- `/plan-eng-review` — Engineering plan review
- `/review` — Code review
- `/ship` — Ship checklist
- `/browse` — Headless browser for QA and browsing
- `/qa` — QA testing
- `/qa-only` — QA testing (no fixes)
- `/setup-browser-cookies` — Configure browser cookies
- `/retro` — Retrospective
- `/document-release` — Document a release

## Zettelkasten Knowledge Base (Obsidian)

## Vault Location

`D:\Local-Lab\Coding\Repos\OwnCord\Obsidian-Brain\BIGBRAIN`

## When to Write Notes

After completing any meaningful task, create or update
a Zettelkasten note capturing the insight.

## Folder Structure

```text
BIGBRAIN/
├── 0-inbox/          # Fleeting notes, quick captures
├── 1-zettel/         # Permanent atomic notes (the core)
├── 2-projects/       # Project-specific MOCs (Maps of Content)
├── 3-resources/      # Reference material, snippets, configs
└── templates/        # Note templates
```

## Note Format

Every note in `1-zettel/` uses this template:

```markdown
---
id: {{YYYYMMDDHHMMSS}}
title: "Short descriptive title"
tags: [tag1, tag2]
created: {{YYYY-MM-DD}}
---

# {{title}}

One atomic idea expressed clearly in a few paragraphs.

## Context
Why this matters or when it applies.

## Related
- [[link-to-related-note]]
- [[another-related-note]]
```

## Rules

1. **Atomic**: One idea per note. Split if covering
   two concepts.
2. **Linked**: Add `[[wikilinks]]` to related notes.
   Search the vault first.
3. **Own words**: Write in plain language, not
   copy-paste from docs.
4. **ID as filename**: Use
   `{{YYYYMMDDHHMMSS}}-short-slug.md`.
5. **Inbox first**: If unsure, drop in `0-inbox/`.
6. **Project MOCs**: Each project gets a MOC in
   `2-projects/` linking relevant zettels.
7. **Search before creating**: Search existing notes
   to avoid duplicates.
