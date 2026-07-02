---
title: Changelog
description: Release history for DocPlatform. All notable changes documented in Keep a Changelog format.
publish: true
---
# Changelog

All notable changes to DocPlatform are documented in this file.

Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.13.0] — 2026-07-02

### Added
- **Versions UI** — Settings → Versions tab (both editions): list, create, set-default, and delete workspace documentation versions, wired to the existing versions API; typed-slug delete confirmation and duplicate-slug guard. (U-02) (#576)
- **Privacy & data self-service UI (cloud)** — customers can export their data (JSON) and delete their account (typed-confirm modal) from Settings, without contacting support. (U-06) (#580)
- **Invite-only mode** — `DOCPLATFORM_DISABLE_SIGNUP=true` blocks public registration (403 `SIGNUP_DISABLED`) and OIDC first-login auto-provisioning, while invitation-accept and share-link joins keep working; `/api/auth/providers` exposes `signup_disabled` so the login screen hides the register form. (community#6) (#577)
- **Tunable login lockout** — `DOCPLATFORM_LOGIN_LOCK_THRESHOLD` / `DOCPLATFORM_LOGIN_LOCK_DURATION_SEC` make the existing per-account lockout policy configurable (defaults unchanged: 5 attempts / 900 s), with fail-fast config validation. (#577)

### Security
- **GDPR right-to-erasure now reaps residual PII on BOTH deletion paths** (admin hard-delete and customer self-service): invitation rows matching the user's email are deleted (empty-email guarded), and terms-acceptance IP/user-agent are blanked while retaining the acceptance itself as Article 7 proof. (ops#343) (#580)

### Internal
- Release pipeline verifies valoryx.org serves the new version post-publish and fails loudly if not (#578); health-sweep gained a version-drift monitor (live site vs latest release, 45-min grace) (#579).
- Test-only `DOCPLATFORM_RATE_LIMIT_DISABLED` escape hatch for the ephemeral e2e instance — construction-time, loud WARN, never set in production (#576); CI actions bump (#575).

## [0.12.0] — 2026-06-23

### Added
- **MCP server is now mounted on the main app at `/mcp`** — the AI-native flagship: the full tool surface over Streamable HTTP, with graceful SIGTERM connection drain. Previously `/mcp` was served as SPA HTML. (ops#33, ops#106) (#559)
- **Admin credential self-management UI** — add a second passkey and regenerate recovery codes while logged in (owner anti-lockout). (ops#324) (#562)
- **Cloud billing boot fail-fast** — a cloud build with billing enabled but an unusable Stripe config now refuses to start, instead of booting "healthy" and 500-ing every checkout. Hardened to trim config at the source, reject bare-prefix keys, and loudly surface partial/disabled billing config. (ops#320, ops#342) (#569, #573)

### Security
- **`/mcp` is host-guarded off customer custom domains and sheds tokenless request floods** before the JSON-RPC parser. (ops#325) (#561)
- **GDPR right-to-erasure scrubs only the target user, not the whole org** — analytics anonymization was org-scoped; it now erases just the deleted user. (audit P1 #6, ops#329) (#565)
- **GDPR right-to-erasure now scrubs `analytics_consent_log` PII** (user_id / workspace_id / ip_hash / user_agent) on both the admin and customer self-service delete paths, retaining the anonymized consent decision as Article 7 proof. (ops#332) (#572)

### Fixed
- **MCP `/mcp` data race under shutdown** — the per-request context is detached from the fasthttp RequestCtx so the notification-forwarder goroutine cannot race fasthttp teardown during drain. (ops#336) (#571)
- **Static-site export is self-contained** — exported pages carry the resolved theme and a static hash-based CSP, instead of an empty nonce that blocked all inline JS. (audit P1 #7) (#570)
- **Editor "keep my changes" conflict-override + crash-durable save** — the force path no longer 400s on an empty hash, and a save records a crash-durable recovery row. (audit P1 #8/#10, ops#331) (#566)
- **Search indexes tags as discrete terms** and removes the dead AI keyword-boost branch. (audit P2 #11, ops#335) (#568)
- **Published `/changelog` publishes from the main tip, not the tag commit** — fixes the ~2-release-stale changelog. (audit P2 #12, ops#333) (#567)

### Internal
- Re-adopt `modernc.org/sqlite` 1.50.1 → 1.52.0. (ops#305) (#560)
- CI: pin heavy `-race` jobs to the full-power runner via a `bigcpu` label (ops#321) (#556); migrate once into a template DB cloned per test (ops#304) (#558); degrade the usage-report runner-roster 403 instead of failing the whole report (ops#322) (#557).

## [0.11.8] — 2026-06-16

### Security
- **Cross-org membership write is now ordered AFTER the identity proof on invitation-accept** — a leaked-token-without-password caller can no longer write membership (or resurrect an archived, role-elevated membership) before proving identity; the write is atomic with workspace AddMember + invitation Accept. (#285) (#550)
- **Custom domains gated to paid plans at the attach path** — Free → 403, Team/Business → 200, via the PLAN_LIMIT feature-lock (#261); the default plan is edition-gated fail-closed (cloud → free, community → community) (#260). (#551)

### Fixed
- **Sync errors are surfaced instead of swallowed** — migration 037 adds an `error` status to `git_sync_state`; the surfaced message redacts `scheme://userinfo@` credentials before truncation. (#5) (#552)
- **Git remote URLs validated pre-DNS** — additive SSRF hardening (rejects doubled-credential prefixes / missing repo path) without weakening existing checks. (#4) (#552)
- **Published-page conditional-GET 304 revived** — `If-None-Match` is now compared against the same `"<hash>-<epoch>"` validator that is emitted (it was compared against the bare `"<hash>"`, so 304 never fired); paired with active render-cache invalidation on workspace mutation. (#3) (#552)

## [0.11.7] — 2026-06-15

### Security
- **Creator's auto-granted seat is now charged against the editor seat cap at workspace-create** — closes the last bypass in the seat-cap series (the create path was previously ungated). Completes the v0.11.5/v0.11.6 promotion/redemption/accept checks. (ops#273 item 4) (#540)

### Fixed
- **Webhooks: deliveries are held (not dropped) when a configured signing secret is unresolvable** — fail-closed rather than sending unsigned, with the hold surfaced in `GitSyncState` (ops#292 activation-gate B1) (#541).
- **Webhooks: PATCH semantics for the signing secret on update** — updating a webhook no longer clobbers an unset secret field (ops#292 activation B2) (#546).

### Internal
- **CI: `internal/api/handlers` `-race` suite given timeout headroom (18m → 26m)** on both `test` and `test-admin` — the suite straddles the old 18m per-package cap under load and flaked the merge queue. Interim mitigation; root cause is the per-test 27-migration fixture cost (ops#304) (#553).
- **CI: failure-alerting restored and Health Sweep red-run noise stopped** (ops#297) (#545).
- **CI: external uptime-monitor matrix added to Health Sweep** (Workstream H) (#519).
- **Dependencies** — `golang.org/x/net` 0.55.0 → 0.56.0 (#549).

## [0.11.6] — 2026-06-13

### Security
- **Editor seat cap enforced on the invitation-accept and invitation/share-link-create paths** — the cap is re-checked when an invitation is accepted (#538) and is charged to the *workspace's owning org* (not the actor's) when an invitation or share-link is created (#537), closing two grant paths that previously bypassed the limit. This extends the v0.11.5 promotion/redemption checks (#529, #536); the creator's auto-granted seat at workspace-create (ops#273 item 4) is tracked separately and is **not** in this release. (ops#273 items 2 & 3)
- **Admin bootstrap (re-)enrollment hardened** — added an audit of the admin bootstrap re-enrollment path, a backup-eligible/backup-state (BE/BS) credential round-trip regression net, and a lockout-remediation runbook (ops#277) (#539).

### Changed
- **Admin console: parity-series tile/number helpers unified** and `AdminBoot.version` reconciled with the build version (admin binary; not yet deployed) (ops#266) (#542).
- **Dependencies** — `github.com/mark3labs/mcp-go` 0.54.0 → 0.54.1 (#514).

### Internal
- **CI: `test-admin` serialized after `test`** (`needs: [test]`) to end the deterministic shared-runner `-race` timeout collision between the two heaviest suites (ops#198) (#544).
- **`modernc.org/sqlite` held at 1.50.1** — the 1.52.0 bump (#513) was reverted from this release pending investigation of a test-suite performance regression that pushes the `internal/api/handlers` `-race` suite over its CI timeout (ops#304, ops#305). Production is unaffected (it already runs 1.50.1).
- **CI: `actions/checkout` 6.0.2 → 6.0.3** (#512).

## [0.11.5] — 2026-06-12

### Security
- **Editor seat caps enforced on role promotion and share-link redemption** — role promotions check the cap (demotions and editor↔admin laterals never consult it, so no false denials) (#529), and share-link redemption re-checks it at use time: an unlimited-use editor link can no longer mint unbounded seats past the plan cap. Denials leave zero partial state (no membership row, no use-count burn) and surface the standard upgrade prompt (#536).
- **Webhook signing secrets encrypted at rest** — outbound-webhook HMAC secrets are now AES-256-GCM-encrypted in the database via the same cipher and key that protect git credentials; decryption is transparent at the store layer; with the encryption key configured, stored rows never contain plaintext (deployments without `GIT_ENCRYPTION_KEY` retain prior behavior) (#532).
- **Admin WebAuthn synced-passkey fix** — backup-eligible/backup-state credential flags are persisted and reconstructed, so synced passkeys (e.g. Windows Hello) no longer fail login with a flag-consistency error (#526).

### Added
- **Outbound webhook delivery engine, wired behind a flag** — the delivery engine (HMAC signing, retry with backoff, dead-lettering) is now constructed and running behind the boot flag `webhook_delivery_enabled`, default OFF. Wiring exposed and fixed two latent engine bugs that would have silently dropped every delivery. Events dispatch from the central content-mutation seam (`page_created`/`page_updated`/`page_deleted`); git-pull reconciliation deliberately does not emit (#527). The subscribable event list is trimmed to exactly the events that fire; legacy subscriptions to never-implemented events are visibly disabled by migration, and a webhook can no longer be re-enabled with an empty event list (#531).
- **Admin console: five new views** — Users (cross-org list, search, detail with sessions) (#525), Billing overview (unit-honest tiles with Stripe deep-link) (#530), Custom Domains (persisted bindings, honest no-DNS-state) (#533), Platform Analytics (windowed traffic, honest disabled-state) (#534), and a sudo-gated break-glass Impersonation flow (one-shot link, credential never displayable; feature remains environment-disabled in production) (#535).
- **Admin console: plan-override dialog** — set/clear plan overrides with WebAuthn sudo step-up and typed confirmation (#511).

### Changed
- **Business plan: pages capped at 500 per workspace** (cloud only; community remains unlimited) — migration 034 (#523).
- **Plan overrides now take effect at runtime** — license enforcement resolves the effective plan as active-override-else-org-plan across all limit and feature checks; previously overrides were stored but never read (#524).

### Fixed
- **Web edits no longer drop unknown frontmatter keys** — Hugo/Jekyll metadata (`date:`, `author:`, `weight:`, arbitrary keys) survives the web-edit round-trip; files without extra keys serialize byte-identically, so no content-hash churn (#528).

## [0.11.4] — 2026-06-10

### Fixed
- **Two-segment published-site slugs resolve** — `owner/repo`-style workspace slugs now work on `/p/{slug}/...` routes and custom domains; previously only single-segment slugs resolved, leaving multi-segment published sites unreachable except via subdomain (#518).

### Changed
- **cosign pinned to v2.6.3** — the cosign installer's default floated to cosign v3, whose new-bundle-format default ignores `--output-signature`/`--output-certificate` and failed the 0.11.3 release run at the signing step. Pinning preserves the documented `.sig` + `.pem` verification contract; adopting the v3 bundle format is tracked as a deliberate future change.

## [0.11.3] — 2026-06-10

### Note
- Source-identical to 0.11.2 plus the pipeline fixes below. Its release run failed at the signing step (cosign v3 CLI drift); the GitHub release exists unsigned and was not published to the public channel. Superseded by 0.11.4.

### Changed
- **Release pipeline hardened** — release test/lint/security jobs moved to the self-hosted runner (the `-race` suite exceeded GitHub-hosted runners' 10-minute per-package limit and killed two release runs); GoReleaser version pinned; binaries are cosign-signed from the published release assets with the Fulcio certificate (`.pem`) shipped alongside each `.sig`, so `cosign verify-blob` works under cosign ≥ 2.0 (#515, #516). No product-code changes — source-identical to 0.11.2.

## [0.11.2] — 2026-06-10

### Note
- Source-identical to 0.11.1 plus CI-only changes (#515). Its release run failed at the signing step (GoReleaser layout drift); the GitHub release exists unsigned and was not published to the public channel. Superseded by 0.11.3.

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
