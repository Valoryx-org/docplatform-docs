---
title: Changelog
description: Release history for DocPlatform. All notable changes documented in Keep a Changelog format.
publish: true
---
# Changelog

All notable changes to DocPlatform are documented in this file.

Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.11.1] — 2026-06-06

### Security
- **Sudo required on plan-override writes** — admin plan-override endpoints now demand a fresh WebAuthn step-up (#510), with hardened sudo/modal handling in the admin SPA (#509).
- **Admin logs scrubbed of PII** — the admin binary's root logger is wrapped in a redaction handler (#507); observability mailers redact recipient PII and the metrics listener fails loud on bind errors (#495).
- **Impersonation minting fenced out of the cloud binary** — regression test guarantees the cloud build cannot mint impersonation tokens (#508).
- **Webhook event validation on update** — outbound-webhook event lists are validated on update as well as create, with a CI grep audit for SSRF regressions (#500).

### Fixed
- **Org switcher persisted** — `primary_org_id` is now saved when updating users, so the active organization survives re-login (#506).
- **Cursor pagination corrected** in cross-org list endpoints — resumes after the cursor row with a stable `id` tiebreaker (#499, #502).

### Changed
- **Published-site themes single-sourced** — the 7-theme list now lives in one place and every consumer (API, MCP, publishing) validates against it (#503).

## [0.11.0] — 2026-06-01

### Security
- **Outbound-webhook SSRF closed end-to-end** — create-time URL validation routes through the shared IP blocklist (#485) and delivery-time requests pin IPs and reject redirects (#476).
- **API-key authorization bypass fixed** alongside a handler test-coverage parity sweep (#474).
- **Account lockout applied to invitation acceptance** — the per-account lockout now also covers the invitation password path (#455).
- **AI writing-assist actions whitelisted** at the handler boundary (#478).

### Added
- **Admin UI (early)** — WebAuthn login fixed end-to-end (#489), organizations list/detail (#492), audit-log viewer with CSV export (#493), sudo step-up modal and session-expiry banner (#491). The admin binary is not yet generally available.
- **Operational metrics** — internal loopback `/metrics` listener (`METRICS_PORT`) with business gauges (#482) and git-sync worker counters/queue-depth gauge (#487).
- **PII-scrubbing log handler** for cloud logs (#488).
- **Playwright smoke suite** wired into CI as a runtime gate (#484).

### Fixed
- **Importer zip-bomb guard** for archive imports (#465).
- **Reserved slugs rejected** when renaming an organization (#466).
- **Release signing fails loud** with a cosign verify round-trip (#468).

### Changed
- **Frontend CSS built at compile time** — Tailwind JIT replaces the runtime engine, cutting 367 KB of eager vendor JS (#479); the admin SPA carries a no-external-CDN/web-font fence (#481).

## [0.10.0] — 2026-05-16

### Security
- **Login timing oracle closed** — user enumeration via response timing is no longer possible (#442).
- **Stored XSS fixed in the markdown render path** (#444) plus a defense-in-depth hardening bundle (#443).
- **Upload-bomb protection** per SPEC-25 (#422).

### Added
- **Cloud edition shell** — plan chip, usage display, billing banner (#437), plan-limit upgrade modal (#438), `/api/v1/me/context` bootstrap endpoint (#435), and a clean two-SPA community/cloud split enforced by audit (#434).
- **GDPR self-service** — personal data export (#420), org export foundation (#419), and password-gated account deletion (#421).
- **MCP `publish_workspace` tool** with REST publish-toggle parity (#424).
- **Production observability** — Prometheus alert rules and Grafana panels (#446).

### Fixed
- **Editor-cap errors unified** to `PLAN_LIMIT_REACHED` across all creation paths (#439).
- **Dunning/billing email routed to the org creator** (#440).
- **WebSocket hub goroutine leak on shutdown** (#441); **search engine close made idempotent** (#450).

### Changed
- **CI hardening** — enforced coverage gate, blocking govulncheck and Trivy scans, `-trimpath` builds (#445).

## [0.9.0] — 2026-04-26

### Added
- **Publishing overlay (`.docplatform/publish.yaml`)** — git-reviewable publish decisions with read-fallback, write-side sync, and frontmatter-wins-on-write semantics (#387, #388, #389); `nav_order` frontmatter support.
- **Workspace language** — `language` column wired through to the published site's `<html lang>` (#385).
- **Reserved-handle validation** for workspace and org slugs (#386).
- **Pre-deploy smoke test** (`make smoke`) wired into CI (#369, #390).
- **Admin recovery foundations** — passkey re-enrollment tokens, recovery owner notification, and a 2-of-3 cosign recovery flow (#392–#394); Diceware recovery codes (#383). (Admin binary — not yet generally available.)

### Fixed
- **Billing webhook idempotency race closed** — atomic gate removes a time-of-check/time-of-use window (#382).
- **Custom-domain TLS fails loud** when `CADDY_ASK_SECRET` is missing on cloud builds (#399).
- **"Powered by Valoryx" badge truly hidden on paid plans** — the old relabel branch was removed; free-tier badge links to a plan-aware upgrade nudge (#400).

## [0.8.0] — 2026-04-12

### Added
- **Workspace rename** from the settings UI (#291).
- **Clearer member roles** — consistent role labels and an Owner badge (#258); invitation emails state the offered role (#259).

### Fixed
- **Billing plan validation** — `plan_id` checked against the database and unknown billing intervals rejected (#294).
- **Git provider handler rejects empty provider names** (#293).

## [0.7.1] — 2026-04-12

### Security
- **Git credentials encrypted at rest** — workspace `git_remote` credentials are now AES-256-GCM encrypted (#290).

### Fixed
- **API key creation includes `workspace_id`** (#289).

## [0.7.0] — 2026-04-12

### Added
- **Drag-and-drop page moves** in the workspace tree (#271).
- **Editable Welcome page and embedded Help tutorial** (#275).
- **Published-page indicator** in the workspace tree (#256).
- **Empty-remote bootstrap** — connecting an empty git repository initializes and pushes automatically, with an `ls-remote` preflight (#253).
- **Pending invitations list** in workspace settings (#277).

### Fixed
- **Published sidebar rebuilt on every publish** so new pages appear immediately (#265).
- **API key scope payload validation** with backfill of legacy rows (#272).
- **Cookie-consent banner buttons** wired correctly; consent decisions logged (#252).
- **Upstream tracking set after init-and-push** so subsequent pulls work (#255).

### Changed
- **Share Links panel removed from the UI** — page access is controlled by site visibility plus invitations (#273).

## [0.6.4] — 2026-04-08

### Security
- **Published docs auth bypass fixed** — `c.Redirect()` in Fiber returns nil, causing handler to render page after redirect. Pages with `visibility=authenticated` were served without auth.
- **Path validation hardened** — reject dangerous file extensions (.php, .exe), block system directory names (etc/, var/, bin/), enforce 200-char path limit. Paths without extension auto-normalized to .md.
- **Registration email enumeration prevented** — generic error message replaces "email already registered" (OWASP compliance).
- **DeleteMembersByUserID scoped to org** — was deleting workspace memberships across ALL orgs when archiving from one.
- **OIDC state comparison** uses `crypto/subtle.ConstantTimeCompare` instead of `!=`.
- **DeleteUser confirmation** requires exact `{"confirm":"DELETE"}` — empty body no longer bypasses.
- **Max password length** capped at 1024 characters to prevent Argon2id resource exhaustion.
- **DEV_FRONTEND_URL blocked in production** — fails startup if set when `DOCPLATFORM_ENV=production`.
- **HSTS only emitted when TLS active** on MCP HTTP endpoint.

### Added
- **`reset-password` CLI command** — generate password reset links for community deployments without SMTP: `docplatform reset-password --email user@example.com`.
- **Update notification system** — admin dashboard shows banner when new version is available. `GET /api/v1/admin/system/update-check` endpoint.
- **Move/Rename via POST** — `POST /content/:ws/move` with `{source, destination}` matches frontend API contract.

### Fixed
- **Duplicate workspace slug** returns 409 Conflict instead of 500 Internal Server Error.
- **Plan limits enforced on direct member assignment** — `CanAddEditor` now checked in `AssignMemberToWorkspace`, not just invitation flow.
- **Git sync popup dismissable** — close button and "Later" now work (animation-based dismiss replaced with direct removal).
- **Workspace creation transactional** — workspace + member + published site wrapped in `WithTx` to prevent orphaned records.
- **Email Sender thread-safe** — added `sync.RWMutex` for concurrent `Configure`/`SetResend` vs send operations.
- **Publish render cache bounded** — sync.Map capped at 10k entries to prevent unbounded memory growth.
- **LLMs-full.txt capped** at 200 pages to prevent resource exhaustion on public endpoints.
- **Missing `rows.Err()` check** in `ListRecentlyDeleted` — partial results from I/O errors now detected.
- **Filename overflow** in editor header — long titles/paths truncated with ellipsis.
- **Billing error messages sanitized** — Stripe internal details no longer leaked to client.

### Changed
- **Version wiring** — `main.version` (set by GoReleaser) now flows to health endpoint via `handlers.SetVersion()`.
- **GoReleaser** — removed `-s -w` strip ldflags that caused SEGV on some platforms.
- **valoryx.org** — "Join Waitlist" → "Get in Touch" (6 languages), pricing cards show "Best for" personas.

## Phase 1 milestone — 2026-03-04

> Historical note: this entry was originally mislabeled "1.0.0". DocPlatform has not shipped a 1.0 release; this milestone landed between 0.5.1 and 0.6.x.

Commercial feature milestone with billing, MCP, custom domains, and the workspace admin panel.

### Security
- RS256 JWT (auto-generated RSA key pair, access + refresh tokens, JWKS-ready)
- Argon2id password hashing (configurable memory/time/threads, OWASP defaults)
- 3-tier Content Security Policy (SPA with per-request nonce, Published with nonce, API with default-src 'none')
- OWASP security headers (X-Content-Type-Options, X-Frame-Options, HSTS, Referrer-Policy, Permissions-Policy, COOP)
- HttpOnly cookie-based OIDC token passing (eliminates localStorage token injection)
- API key authentication with dp_live_ prefix, HMAC pepper, method+action scope enforcement
- Per-org rate limiting (7 categories, token bucket algorithm, per-plan limits with admin overrides)
- Security build pipeline: govulncheck, gosec, SBOM (CycloneDX), garble obfuscation
- 16 security findings identified and fixed in post-sprint audit

### Billing & Licensing
- Stripe integration: checkout sessions, customer portal, webhook processing
- Plan management: community (free), team, business with automatic enforcement
- Grace period handling: active → grace → restricted state machine
- Seat limits (editors per org) and workspace caps per plan
- Feature gates: published_docs, custom_domains, analytics per plan
- Admin subscription overrides with audit trail

### Custom Domains
- CNAME-based custom domain verification per workspace
- Caddy reverse proxy integration with automatic TLS provisioning
- Ask endpoint for on-demand certificate validation
- Host-based routing for published documentation sites

### MCP Server
- 13 tools accessible via stdio transport for Claude Desktop
- API key authentication with scope enforcement
- Context bundle assembly (get_context tool)

### Analytics
- Separate analytics.db (SQLite, no FK constraints, independent lifecycle)
- Buffered event collector with async batch inserts
- Pageview and search event tracking with GDPR consent gating
- 90-day retention policy with automatic cleanup
- Dashboard queries: top pages, top searches, platform overview, growth metrics
- Race-condition-free collector with sync.WaitGroup synchronization

### Admin Panel
- Super admin guard middleware with JWT-based role check
- Organization management: list, view, plan updates, rate limit overrides
- User management: list, search, view details
- Audit log viewer with pagination
- Billing dashboard: MRR overview, subscription list with status filters, webhook events
- Domain management: list, verify, provision, delete
- Platform analytics: overview stats, growth metrics
- System health: database, search, analytics status indicators
- GDPR compliance: org data export, user data export, user deletion with audit anonymization
- Impersonation support for debugging

### Observability
- Prometheus metrics endpoint (super admin protected)
- Telemetry collection with opt-in controls
- Doctor bundle with system diagnostics

### Frontend
- Onboarding checklist with progress tracking (5 steps, dismissible, auto-dismiss on completion)
- Settings tabs: Custom Domains, API Keys, Billing, Analytics
- Admin panel: 7 tabs (Organizations, Users, Audit, Billing, Domains, Analytics, System Health)
- Conflict resolver modal: keep local / use server version
- Move/rename dialog with path validation, auto-redirect, and wikilink update
- Valoryx branding: Plus Jakarta Sans headings, DM Sans body, JetBrains Mono code
- Brand colors: #124265 (heading), #1d7fc2 (accent), #0c1e2e (navy)

### Published Docs
- 5 theme presets: default, dark, forest, rose, amber
- 6 editor components: code blocks, callouts, tabs, diagrams, badges, copy button
- SEO: sitemap.xml, robots.txt, RSS feed, JSON-LD structured data, Open Graph, Twitter Cards
- Community Edition badge

### Content Management
- Wikilink scanner with async repair
- Page rename/move with redirect creation, path mutex, wikilink updates
- Conflict resolver backend with timestamp-based versioning
- Onboarding state machine with auto-org creation and starter workspace
- OpenAPI 3.0 specification with Scalar UI at /api/docs

### Infrastructure
- Analytics database separation (no FK constraints, independent backup/retention)
- Email improvements: Resend integration, HTML templates, password reset
- Pre-commit hooks: gofumpt + go vet

### Metrics
- 106 source files, 20,904 LOC
- 61 test files, 13,301 test LOC
- 709 tests across 21 packages, 0 failures
- go vet clean, go test -race clean
- govulncheck: 0 vulnerabilities

---

## [0.5.1] — 2026-03-02

### Security
- Complete security audit remediation (Phases 1–5): input validation, rate limiting, CSRF protection, secure headers
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
- Race condition fixes in tests; auth test timeouts increased for bcrypt under `-race`
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
