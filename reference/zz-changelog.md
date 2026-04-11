---
title: Changelog
description: Release history for DocPlatform. All notable changes documented in Keep a Changelog format.
publish: true
---
# Changelog

All notable changes to DocPlatform are documented in this file.

Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.5.2] — 2026-03-07

Phase 1 complete — major feature expansion across billing, analytics, authentication, AI, publishing, and platform operations.

### Added

#### Billing & Licensing (Stripe)
- Stripe Checkout + Customer Portal integration
- 4 plan tiers: Community (free, unlimited), Free (1 workspace, 3 editors, 50 pages), Team ($29/mo), Business ($79/mo, Coming Soon)
- Annual pricing with 2 months free ($290/yr, $790/yr)
- 14-day free trial for paid plans (configurable via `TRIAL_DURATION_DAYS`)
- Feature gating: analytics, custom domains, advanced AI locked to paid plans
- Editor limits enforced per plan (Community: unlimited, Free: 3, Team: 15, Business: 50)
- Workspace limits enforced per plan (Community: unlimited, Free: 1, Team: 3, Business: 10)
- Page limits enforced per plan (Community: unlimited, Free: 50, Team: 150, Business: unlimited)
- Stripe webhook handler with idempotent event processing
- Subscription lifecycle: trial → active → grace → restricted → canceled
- Plan definitions seeded in database on first run

#### Analytics (GDPR-Compliant)
- Pageview tracking (path, referrer, truncated UA — no IP storage)
- Search analytics (query frequency, top searches)
- Top pages aggregation with configurable time period
- Dashboard overview: total views, searches, unique pages
- Cookie-based opt-in consent with `POST /api/analytics/consent`
- Separate `analytics.db` SQLite database (isolates from main DB)
- 90-day retention with automatic pruning
- Platform-wide analytics for super admins

#### WebAuthn / Passkeys
- Passwordless authentication with hardware security keys and biometrics
- Registration and login flows: `POST /api/auth/webauthn/register/begin|finish`, `POST /api/auth/webauthn/login/begin|finish`
- Credential management: list stored credentials, delete individual credentials
- Clone detection via sign count tracking
- Configurable via `WEBAUTHN_RP_ID`, `WEBAUTHN_RP_DISPLAY_NAME`, `WEBAUTHN_RP_ORIGINS`

#### AI Writing Assist
- Writing operations: rewrite, improve, shorten, expand
- Doc Chat: multi-turn conversation about workspace documentation
- Provider support: Claude (Anthropic) and OpenAI
- Configurable model via `AI_PROVIDER`, `AI_API_KEY`, `AI_MODEL`
- Status endpoint: `GET /api/v1/ai/status`

#### Doc Quality Scanner
- Readability scoring (Flesch-Kincaid grade level)
- Dead link detection (broken internal markdown links)
- Completeness check (missing frontmatter, orphan pages)
- Workspace-level aggregate quality report: `GET /api/v1/workspaces/:id/quality`

#### Publishing Enhancements
- 7 built-in themes: Default, Dark, Forest, Rose, Amber, Minimal, Corporate
- Auto-generated RSS feed at `/p/{slug}/rss.xml`
- Static HTML ZIP export via CLI (`docplatform export`) and API (`GET /api/v1/workspaces/:id/export`)
- Local preview server: `docplatform preview --workspace <slug> --port 4000`
- Custom domains with auto-TLS via Caddy integration
- Domain management API for workspace admins and super admins

#### Doc Versioning
- Named documentation versions (e.g., v1, v2) within a workspace
- Default version selection
- CRUD endpoints: `GET|POST|PUT|DELETE /api/v1/workspaces/:id/versions`

#### Super Admin Dashboard
- Organization management: list, view details, change plans, override subscriptions
- User management: list, search, view details, impersonate, delete (GDPR)
- Append-only audit log with filterable query endpoint
- System health: disk usage, memory, uptime, DB stats
- Billing dashboard: platform-wide subscriptions, webhook event log
- Rate limit overrides per organization
- Domain management: list, verify DNS, provision TLS, delete
- GDPR compliance: export org/user data, anonymize records

#### API Keys
- Org-level programmatic access with `dp_live_` prefix
- HMAC hashing with configurable pepper (`API_KEY_PEPPER`)
- Create, list (prefix only), delete, rotate endpoints
- Scoped authentication for CI/CD and MCP integrations

#### MCP Server
- `docplatform mcp` CLI command for Model Context Protocol integration
- Stdio-based server compatible with Claude Code, Claude Desktop
- Scoped to single workspace, authenticated via API key

#### Durable Job Queue
- DB-backed async job worker with 500ms polling interval
- Handler registry: `search_reindex`, `manifest_rebuild`, `asset_cleanup`
- Max 3 retry attempts with exponential backoff
- Dead-letter on exhaustion, `ListPending` for retry orchestration
- Graceful shutdown drains in-flight jobs

#### Per-Path Mutex with TTL/LRU
- Background GC goroutine every 60s evicts entries idle >5 minutes
- LRU eviction of oldest 25% when hard cap (10,000 entries) exceeded
- `mutexEntry` struct with atomic last-access timestamp

#### Testing
- Property-based tests using `pgregory.net/rapid` (6 tests: create/read consistency, no duplicate IDs, update preserves ID, delete removes FS, frontmatter roundtrip, reconcile consistency)
- Chaos test harness (6 scenarios: FS write before DB commit, DB commit before search index, partial reconciliation, concurrent writes, FS-only delete, job queue recovery)

### Changed
- Resend email provider support alongside SMTP (`RESEND_API_KEY`, `RESEND_FROM`)
- Git SSH known_hosts configuration via `GIT_SSH_KNOWN_HOSTS` (built-in pinned keys for GitHub/GitLab/Bitbucket when unset)
- Storage info visibility controls: `HIDE_STORAGE_PATHS`, `SHOW_DISK_PATHS_TO_WS_ADMIN`
- Prometheus metrics endpoint (`/metrics`) gated behind `FF_METRICS=true`
- Development proxy via `DEV_FRONTEND_URL` for frontend HMR

### Infrastructure
- 95+ REST API endpoints (up from ~40 in v0.5.1)
- 8 CLI commands (added: `export`, `preview`, `mcp`)
- Comprehensive OpenAPI 3.1.0 specification

## [0.5.1] — 2026-03-02

### Security
- Complete security audit remediation (Phases 1-5): input validation, rate limiting, CSRF protection, secure headers
- Update vulnerable dependencies (go-git, golang.org/x/crypto, bluemonday, and others)
- Switch Windows binary signing from SignPath to Azure Trusted Signing

### Fixed
- OIDC login flow: redirect callback, stable provider button order, cookie path, error handling
- OIDC callback serves HTML page with postMessage instead of 302 redirect (fixes SPA flow)
- Broken documentation links: OIDC flow, conflicts API, reset password, navigation
- All download URLs corrected for public release repository
- Workspace role display and editor permission checks
- All errcheck lint violations resolved across 47+ files (proper error handling for Close, Rollback, Write, rows.Close, etc.)
- Dead code removed: stub repositories (108 lines), unused handler function
- Race condition fixes in tests; auth test timeouts increased under `-race`
- staticcheck, ineffassign, and unused lint findings resolved

### Added
- SPA frontend with embedded assets, published docs UX improvements
- Editor undo/redo support and improved new-page form UX
- Premium UI polish (typography, spacing, color refinements)
- Link validation CI rule — catches broken internal links on every push
- Handler test coverage for auth, content, workspace, search, upload, sync, conflicts, health, invitations, admin

### Changed
- Bump Go from 1.24 to 1.25; add .dockerignore for smaller build context
- Migrate CI lint action to golangci-lint v2 (`@v7` action, `version: "2"` config)
- Update all remaining dependencies to latest stable versions

### Infrastructure
- Docker image publishing to GHCR on tagged releases
- Resilient `publish-public` job with retry logic + auto-update website version
- Auto-update docs changelog on release via CI
- Fly.io deployment pipeline for managed hosting option

## [0.5.0] — 2025-02-27 (Founders Edition)

Initial public beta. Single binary, zero external dependencies.

### Core
- Content Ledger with per-path mutex locking and ULID-based page IDs
- SQLite storage via modernc.org/sqlite (pure Go, no CGO)
- Markdown rendering with goldmark (syntax highlighting, frontmatter parsing)
- Full-text search with embedded Bleve engine (pure Go, on-disk index)

### Git Integration
- Bidirectional git sync (push + pull) with conflict detection
- Hybrid git engine (go-git in-process + shell fallback for clone/push)
- Worker pool (4 concurrent) with async job queue
- SSH deploy key and HTTPS authentication support

### Authentication & Authorization
- JWT access + refresh tokens (RS256 RSA-SHA256)
- Role-based access control: 5 roles (Super Admin, Admin, Editor, Commenter, Viewer)
- Local password auth with argon2id hashing (OWASP 2024)
- Password reset flow (SMTP or stdout token)
- Google and GitHub OIDC sign-in

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
- CI pipeline: lint + test + build (Go 1.25)
- Tag-triggered release to GitHub Releases + GHCR
- 300+ tests across 25 packages
