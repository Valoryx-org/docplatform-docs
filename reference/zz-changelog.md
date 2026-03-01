---
title: Changelog
description: Release history for DocPlatform. All notable changes documented in Keep a Changelog format.
publish: true
---

# Changelog

All notable changes to DocPlatform are documented in this file.

Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [0.5.0] — 2025-02-27 (Founders Edition)

Initial public beta. Single binary, zero external dependencies.

### Core

- Content Ledger with per-path mutex locking and ULID-based page IDs
- SQLite storage via modernc.org/sqlite (pure Go, no CGO)
- Markdown rendering with goldmark (syntax highlighting, frontmatter parsing)
- Full-text search ready schema with FTS5

### Git Integration

- Bidirectional git sync (push + pull) with conflict detection
- Hybrid git engine (go-git in-process + shell fallback for clone/push)
- Worker pool (4 concurrent) with async job queue
- SSH deploy key and HTTPS authentication support

### Authentication & Authorization

- JWT access + refresh tokens (HMAC-SHA256)
- Role-based access control: owner, admin, editor, viewer
- Local password auth with bcrypt hashing
- Password reset flow (SMTP or stdout token)

### Publishing

- Public docs at `/p/:workspace-slug/:page-path`
- SEO metadata from frontmatter, ETag caching
- Auto-generated sitemap.xml and robots.txt per workspace

### Operations

- `docplatform doctor` — 9-point diagnostic health check
- `docplatform doctor --bundle` — ZIP diagnostics for support
- Automatic daily backups with configurable retention
- Opt-in anonymous telemetry (weekly, SHA-256 install ID)
- Cross-platform release pipeline (goreleaser + GitHub Actions)

### CLI

- `docplatform serve` — Start HTTP server with graceful shutdown
- `docplatform init` — Initialize workspace (local or git-backed)
- `docplatform rebuild` — Rebuild database from filesystem
- `docplatform version` — Print version, commit, build date

### Infrastructure

- Multi-stage Docker image (Alpine 3.20, non-root, healthcheck)
- CI pipeline: lint + test + build (Go 1.24)
- Tag-triggered release to GitHub Releases + GHCR
- 233 tests across 13 packages
