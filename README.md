# Zen MCP Bridge

**Connect your WordPress site to Claude.ai via the Model Context Protocol — giving Claude read-only access to your plugins, themes, post types, custom fields, database, source files, logs, hooks, and more.**

Zen MCP Bridge turns your WordPress site into an MCP server that Claude.ai can connect to. Once
connected, Claude can inspect your site's architecture in real time — plugins, database schema,
source code, ACF fields, REST routes, cron jobs — so it can write plugins, debug issues, and answer
questions that are tailored to your exact setup instead of to WordPress in general.

**Claude gets read-only access by default. It cannot write, delete, or modify anything unless you
explicitly turn write mode on.**

- **Website:** https://guramzhgamadze.github.io/zen-mcp-bridge/
- **WordPress.org:** https://wordpress.org/plugins/zen-mcp-bridge/
- **Requires:** WordPress 6.9+ · PHP 8.0+
- **Licence:** GPL-2.0-or-later
- **Current version:** 3.2.3

---

## Features

- **Full OAuth 2.1 authorization server** with PKCE (S256), dynamic client registration (RFC 7591),
  and auto-discovery endpoints (RFC 8414 + RFC 9728).
- **Bearer Token auth** as a simpler alternative to OAuth — never expires.
- **Configurable OAuth token lifetime** — 1 hour, 24 hours, 7 days or 30 days, switchable from the
  admin settings page, with an emergency "Revoke All" button for instant invalidation.
- **25 read-only MCP tools** covering every major aspect of a WordPress site — including code
  search, transients, Action Scheduler, WooCommerce and Elementor inspectors.
- **Opt-in write mode** (off by default) — six scoped, audited write tools: draft-only posts,
  allowlisted options, plugin toggling, cache clearing, term creation. OAuth needs an explicit
  consent-screen write grant (`claudeai:write`); deleting, publishing, user management and write
  SQL are never possible.
- **MCP resources & prompts** — site snapshots at `wordpress://` URIs and canned prompts for site
  analysis, debugging and plugin writing.
- **WordPress Abilities API bridge** — tools discoverable by the WP AI ecosystem (admin-only);
  stands down automatically when the official AI Provider for Anthropic plugin is active.
- **MCP spec 2025-11-25 compliant**, including `client_id_metadata_document_supported`, scope in
  `WWW-Authenticate`, and `description` in `serverInfo`.
- **Rate limiting** — 120 tool calls per minute per IP.
- **Security hardened** — path traversal guards, credential redaction, single-use auth codes,
  clickjacking protection, and more.
- **Apache + FastCGI compatible** — Authorization header normalization built in.
- **Zero dependencies** — pure WordPress, no Composer required.

---

## Requirements

| Requirement | Version |
|---|---|
| WordPress | 6.9 or higher (for the Abilities API) |
| PHP | 8.0 or higher |
| MySQL / MariaDB | Any version supported by your WordPress install |

---

## Installation

1. Install through **Plugins → Add New** (search for "Zen MCP Bridge"), or from
   [WordPress.org](https://wordpress.org/plugins/zen-mcp-bridge/). With WP-CLI:
   ```bash
   wp plugin install zen-mcp-bridge --activate
   ```
2. Go to **Settings → MCP Bridge** to find your endpoint URL and Bearer Token.
3. Flush rewrite rules: **Settings → Permalinks → Save Changes**.

---

## Connecting to Claude.ai

There are two connection methods. Option A is the fastest.

### Option A — Bearer Token (simplest)

1. In Claude.ai, go to **Profile → Settings → Connectors → Add custom connector**.
2. Paste your MCP endpoint URL (shown in Settings → MCP Bridge):
   ```
   https://your-site.com/wp-json/zen-mcp/v1/bridge
   ```
3. Set auth type to **Bearer Token** (or **API Key**).
4. Paste the token from the settings page.
5. Save — Claude.ai connects immediately.

### Option B — OAuth 2.1 (browser login flow)

Claude.ai discovers the OAuth server automatically via the `.well-known/` endpoints. If asked for
details manually:

| Field | Value |
|---|---|
| Authorization URL | `https://your-site.com/wp-json/zen-mcp/v1/oauth/authorize` |
| Token URL | `https://your-site.com/wp-json/zen-mcp/v1/oauth/token` |
| Client ID | `claude.ai` (any value except literally "Bearer Token") |
| Scopes | `claudeai` |

Claude.ai opens a browser window to your WordPress site, where you log in as an administrator and
click **Allow Access**.

---

## Endpoints

| Endpoint | Purpose |
|---|---|
| `POST /wp-json/zen-mcp/v1/bridge` | Main MCP JSON-RPC endpoint |
| `GET/POST /wp-json/zen-mcp/v1/oauth/authorize` | OAuth 2.1 authorization |
| `POST /wp-json/zen-mcp/v1/oauth/token` | OAuth 2.1 token exchange |
| `POST /wp-json/zen-mcp/v1/oauth/register` | Dynamic client registration (RFC 7591) |
| `GET /.well-known/oauth-authorization-server` | OAuth discovery (RFC 8414) |
| `GET /.well-known/oauth-protected-resource` | Protected resource metadata (RFC 9728) |
| `GET /authorize` · `POST /token` | Old-spec fallbacks (auto-proxied) |

---

## Available MCP tools

### Read-only

| Tool | Description |
|---|---|
| `wp_get_site_info` | WP/PHP/MySQL versions, active theme, plugins, debug settings, upload dirs |
| `wp_get_plugins` | All installed plugins — name, version, author, status |
| `wp_get_themes` | All themes — version, author, parent theme, active state |
| `wp_get_post_types` | All post types — labels, supports, taxonomies, REST base, post count |
| `wp_get_taxonomies` | All taxonomies — object types, REST settings, term counts |
| `wp_get_options` | WordPress options (credentials always redacted) |
| `wp_query_posts` | Query any post type via `WP_Query` — post data, meta and terms |
| `wp_get_db_schema` | All database tables — columns, types, row counts |
| `wp_db_query` | Read rows from one table with filters/order/limit — identifiers whitelisted against the live schema, values parameterized (no raw SQL) |
| `wp_list_files` | List files in any `wp-content` subdirectory, filtered by extension |
| `wp_read_file` | Read any source file in `wp-content` (max 512 KB) |
| `wp_get_logs` | Read `debug.log` or the PHP error log (last N lines) |
| `wp_get_hooks` | Inspect `$wp_filter` — all registered actions and filters with priorities |
| `wp_get_acf_fields` | All ACF field groups and fields (requires ACF) |
| `wp_get_users` | List users with roles (passwords never returned) |
| `wp_get_menus` | Navigation menus, assigned locations, and menu items with hierarchy |
| `wp_get_cron_jobs` | All scheduled WP cron events — next run times, schedules and args |
| `wp_get_active_widgets` | Active widget areas and widgets with their settings |
| `wp_get_rest_routes` | All registered REST API routes with methods and namespaces |
| `wp_search_code` | Search file contents across `wp-content` — file, line number, matching line |
| `wp_get_post` | Single post by ID or slug — full content, meta, terms, featured image |
| `wp_get_transients` | Transients with expiry and value preview (auth/secret values redacted) |
| `wp_get_scheduled_actions` | Action Scheduler counts by status and next pending actions |
| `wp_get_woocommerce` | WooCommerce overview — settings, counts, gateways, zones (no PII or keys) |
| `wp_get_elementor` | Elementor overview — versions, kit settings, templates, experiments |

### Write (opt-in, off by default)

| Tool | Description |
|---|---|
| `wp_create_draft_post` | Create a post as a **draft** — publishing always needs a human |
| `wp_update_draft_post` | Edit an existing draft or pending post — published posts refused |
| `wp_update_option` | Change an allowlisted option — identity, code and secret options hard-denied |
| `wp_toggle_plugin` | Activate or deactivate an installed plugin — never this plugin itself |
| `wp_clear_cache` | Flush object cache, rewrite rules, expired transients |
| `wp_create_term` | Create a taxonomy term |

---

## Security

### Authentication

- Bearer tokens are stored with `autoload = false` — never loaded on regular page requests.
- Tokens are validated with `hash_equals()` to prevent timing attacks.
- OAuth token lifetime is configurable (1 hour default, up to 30 days). Bearer Token auth never
  expires; only OAuth access tokens are time-limited.
- Authorization codes expire after 10 minutes and are single-use — deleted immediately on
  redemption.
- PKCE S256 is mandatory; plain code challenges are rejected.
- The plugin's own token and config options are permanently blocked from `wp_get_options`.
- **Revoke All OAuth Tokens** on the admin page for immediate emergency invalidation. Revocation
  uses an options-backed generation counter and registry, so it works correctly even behind a
  persistent object cache.

### Authorization

- Only WordPress administrators (`manage_options`) can grant OAuth consent.
- The consent page is protected against clickjacking via `X-Frame-Options: DENY` and
  `Content-Security-Policy: frame-ancestors 'none'`.
- Write mode requires both the site-level toggle **and**, for OAuth clients, an explicit
  `claudeai:write` grant on the consent screen.

### File & database access

- All file operations are contained within `wp-content` using `realpath()` plus separator-aware
  path checks (OWASP path traversal prevention).
- Log file paths are validated against an allowlist of roots.
- **There is no raw SQL.** `wp_db_query` builds a structured query itself: table and column
  identifiers are validated against the live schema and a strict pattern before being interpolated
  in backticks, while all values are bound through `$wpdb->prepare()`. The cost is no JOINs or
  aggregate expressions — single-table filtered reads only — and the benefit is no injection
  surface at all.
- Secret redaction is applied in a shared helper used by every tool, not per-tool, so a secret
  cannot be read through a different exit.
- `user_pass`, `user_activation_key`, and `SELECT *` on the users table are blocked outright.

### CORS & network

- CORS headers on the sensitive `/bridge` endpoint are scoped to `claude.ai`, `app.claude.ai` and
  `www.claude.ai` only. The public `.well-known` discovery documents are open, as they are for
  every OAuth provider — they contain no secrets.
- DNS rebinding protection via a per-request origin check.
- `X-Content-Type-Options: nosniff` on every plugin response.
- `WWW-Authenticate` on 401 responses points Claude.ai to the OAuth discovery endpoint, with the
  required scope parameter.
- Apache + FastCGI Authorization header normalization — no `.htaccess` workaround needed.

### External service

Claude.ai (Anthropic) connects **inbound** to your site; the plugin never makes an outbound request
to it. What Claude can read is exactly what the tools above expose, gated by your token or OAuth
grant. See Anthropic's [terms](https://www.anthropic.com/legal/consumer-terms) and
[privacy policy](https://www.anthropic.com/legal/privacy).

---

## OAuth flow

```
Claude.ai                     Your WordPress Site
    │                               │
    │──GET /.well-known/oauth-protected-resource──▶│
    │◀─── { authorization_servers: [...] } ────────│
    │                               │
    │──GET /.well-known/oauth-authorization-server─▶│
    │◀─── { authorization_endpoint, token_endpoint, registration_endpoint } ──│
    │                               │
    │──POST /zen-mcp/v1/oauth/register──▶│  (RFC 7591 Dynamic Client Registration)
    │◀─── { client_id } ────────────│
    │                               │
    │──GET /zen-mcp/v1/oauth/authorize──▶│  (PKCE code_challenge S256)
    │◀─── redirect to WP login ─────│
    │                               │
    [User logs in as WP admin, clicks "Allow Access"]
    │                               │
    │◀─── redirect with ?code= ─────│
    │                               │
    │──POST /zen-mcp/v1/oauth/token─────▶│  (code + code_verifier)
    │◀─── { access_token, expires_in } ───────────│
    │                               │
    │──POST /zen-mcp/v1/bridge──────────▶│  (Authorization: Bearer <token>)
    │◀─── MCP JSON-RPC response ────│
```

---

## Troubleshooting

### 404 / `rest_no_route`

1. Go to **Settings → Permalinks** and click **Save Changes** to flush rewrite rules.
2. Confirm the plugin is active.
3. Test the endpoint manually:
   ```bash
   curl -s -X POST https://your-site.com/wp-json/zen-mcp/v1/bridge -H "Authorization: Bearer YOUR_TOKEN" -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-11-25","capabilities":{}}}'
   ```
   A working response starts with `{"jsonrpc":"2.0","id":1,"result":{...`

### The connector worked before 3.0.0 and now fails

The 3.0.0 rename changed the REST namespace to `zen-mcp/v1`, so the endpoint URL is now
`…/wp-json/zen-mcp/v1/bridge`. Reconnect and re-authenticate your Claude.ai connector. Your Bearer
token, redirect URIs and token lifetime are preserved by a one-time migration.

### Authorization header always empty (Apache + FastCGI)

Handled automatically since 2.4.0. If you still see issues, add to `.htaccess`:

```apache
RewriteRule ^ - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

### Enable debug logging

Add to `wp-config.php`:

```php
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', false);
```

Then check `wp-content/debug.log`, or read it with the `wp_get_logs` tool.

---

## Changelog

### 3.2.3
Fixed a `BIGINT UNSIGNED` underflow in audit-log pruning, and corrected the reported Abilities
count.

### 3.2.2
**Replaced free-form SQL with a structured query.** The `wp_db_query` tool previously executed a
caller-supplied `SELECT`. Free-form SQL cannot be `prepare()`d — the whole string is the input —
and exposing arbitrary database reads over a remote, token-authenticated endpoint is not
acceptable regardless of the guards layered on top. The tool now assembles the query itself:
identifiers are whitelisted against the live schema, values are bound. Single-table filtered reads
only, and no injection surface.

### 3.2.1
Added `X-Content-Type-Options: nosniff` to all plugin responses. Replaced the use of WordPress auth
salts for client identifiers with a dedicated plugin secret. The OAuth consent page's styles are
now registered and printed by handle rather than emitted inline. Documented the external service in
the readme.

### 3.0.0
**Renamed** — "WordPress MCP Bridge" → **Zen MCP Bridge** (slug `zen-mcp-bridge`, text domain, and
REST namespace `zen-mcp/v1`). Existing installs keep their Bearer token, redirect URIs and token
lifetime via a one-time migration on activation. **Breaking:** the MCP endpoint URL changed —
reconnect your Claude.ai connector. Also: the settings page was rebuilt to the Zen family theme
with setup and troubleshooting moved into a Help tab, and the plugin gained `readme.txt`,
`uninstall.php`, internationalization, and input-sanitization fixes for WordPress.org.

### 2.8.0
Configurable OAuth token lifetime (1 hour, 24 hours, 7 days, 30 days) and a "Revoke All OAuth
Tokens" button. MCP protocol version `2025-11-25` is now the highest supported — Claude.ai's
version was previously always downgraded. Added `client_id_metadata_document_supported` to the
authorization server metadata, `scope` to the `WWW-Authenticate` 401 header, and `description` to
`serverInfo`. Consent-page expiry now reads "24 hours" rather than "1440 minutes".

### 2.7.0
Fixed SQL injection in the schema tool (`esc_like()` was used without `prepare()`); the OAuth
consent nonce is sanitized with `sanitize_key()`.

### 2.6.0
Fixed `rest_cookie_invalid_nonce` 403 on "Allow Access" with a two-nonce form. Fixed
`wp_get_options` being able to expose the plugin's own Bearer token. Fixed a cron-tool crash on
PHP 8.2+ and cleaned up transients on deactivation.

### 2.5.0
Added the missing `registration_endpoint` (Claude Code could not connect). Corrected the declared
PHP requirement to 8.0. Fixed a path containment guard that could be bypassed when `realpath()`
returned `false`, and stopped valid empty-string OAuth parameters being stripped.

### 2.4.0
Fixed a double-firing `rest_api_init` inside a tool callback, a path prefix check bypassable by
sibling directories, N+1 queries in the users tool, an unescaped output in the consent page,
missing clickjacking protection on the consent page, and the Authorization header being stripped on
Apache/FastCGI.

### 2.3.0
Fixed `wp_safe_redirect()` blocking OAuth code delivery to claude.ai, a PHP 8 `TypeError` on null
JSON params, a log-path traversal via `ini_get('error_log')`, and token-endpoint responses always
returning HTTP 200 even on errors.

### 2.2.0
Full OAuth 2.1 PKCE authorization server, with RFC 8414 and RFC 9728 discovery documents and
old-spec `/authorize` + `/token` fallbacks.

### 2.1.0
Fixed the missing GET handler, OPTIONS preflight being blocked by the auth callback, missing CORS
headers, an always-`1` post count, a missing `wp_reset_postdata()`, the API key autoloading, a
spoofable rate-limit key, and inline `onclick` JS.

---

## Reporting a bug

Open an issue on this repository. Please include:

- WordPress version
- PHP version
- The exact error message or HTTP response
- Output of `curl` against the MCP endpoint, with the token redacted

---

## About this repository

This repository hosts the **documentation and project page** for Zen MCP Bridge. The plugin source
is developed privately; the released, installable plugin is distributed through the
[WordPress plugin directory](https://wordpress.org/plugins/zen-mcp-bridge/) under the
GPL-2.0-or-later licence, which includes its complete source.

Other plugins in the Zen family:

- [Zen Site Security](https://github.com/guramzhgamadze/zen-site-security) — HTTPS, headers and hardening
- [Zen Login & Authentication](https://github.com/guramzhgamadze/zen-login-authentication) — identity and two-factor auth
- [Zen GEO](https://github.com/guramzhgamadze/zen-geo) — generative engine optimization
- [Zen Blogger](https://github.com/guramzhgamadze/zen-blogger) — accessible Elementor blog widgets

---

Built by [Guram Zhgamadze](https://github.com/guramzhgamadze).
