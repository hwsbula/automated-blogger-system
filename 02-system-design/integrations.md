# Integrations

## Objective

Define how the phase 1 control plane integrates with WordPress while remaining plugin-agnostic and operationally safe.

## Direct Integration Scope

Phase 1 requires one direct application integration only:

- WordPress REST API

Later integrations such as Search Console, GA4, CRM systems, or email platforms are explicitly deferred.

## Authentication Model

Default approach:
- HTTPS WordPress site URL
- dedicated service account per client site
- WordPress Application Password for REST authentication

Connection requirements:
- validate base site URL
- confirm REST API availability
- test authentication before enabling publish jobs
- store credentials encrypted at rest

Rejected by default:
- HTTP endpoints
- shared human admin credentials
- XML-RPC as a primary publish path

## WordPress Endpoints Used

- `GET /wp-json/wp/v2/categories`
- `GET /wp-json/wp/v2/tags`
- `GET /wp-json/wp/v2/posts`
- `POST /wp-json/wp/v2/posts`
- `POST /wp-json/wp/v2/posts/{id}`
- `POST /wp-json/wp/v2/categories` only when category auto-create is enabled
- `POST /wp-json/wp/v2/tags` only when tag auto-create is enabled
- `GET /wp-json/wp/v2/media` only when featured media resolution is configured

## Publish Payload Contract

Required fields:
- `title`
- `content`
- `status`
- `slug`
- `categories`

Optional fields:
- `excerpt`
- `tags`
- `author`
- `date_gmt`
- `featured_media`

Publishing rules:
- use `status=draft` for `draft_only` and `require_review`
- use `status=future` and `date_gmt` for scheduled publish actions
- do not publish with `status=publish` directly in phase 1 unless a later policy explicitly allows immediate publish
- require at least one resolved non-blocked category before post create or update
- allow zero tags only when the draft records a `no_useful_tags` reason

## Category and Tag Handling

- Maintain tenant-scoped taxonomy cache and alias tables for categories and tags
- Sync the live WordPress term catalog during onboarding, before the first publish job of each worker cycle, and immediately after any term auto-create
- Store WordPress term slug, normalized name, taxonomy type, usage count, last-seen timestamp, and approved aliases
- Resolve WordPress term IDs before publish using this reuse order:
  1. exact slug match
  2. case-insensitive name match
  3. approved alias match
  4. highest-usage reusable term in the same semantic bucket
- Reject blocked or generic fallback terms such as `uncategorized` unless a client policy explicitly allows them
- Require one primary category and allow one secondary category only when it materially improves site navigation
- Limit tags to intent-bearing terms and reuse existing tags before proposing any new term
- If a required taxonomy term does not exist:
  - confirm no reusable live term exists by slug, name, or alias
  - create it only if the client policy allows auto-create
  - refresh the tenant term cache immediately after creation
  - otherwise route to review with a taxonomy-missing incident

## Slug and Duplicate Prevention

Slug rules:
- derive from canonical topic key
- append short disambiguators only when required
- never generate random slugs on retry

Duplicate-prevention flow:
1. Check internal `wordpress_post_links` for an existing mapped post
2. If absent, query WordPress for a matching slug
3. If a slug exists but is not mapped to the same content identity, raise a duplicate incident and route to review
4. If mapped, update in place rather than creating a second post

## Draft and Scheduled Publish Behavior

### Draft Creation

Used when:
- content is useful but not eligible for scheduling
- content requires review
- a human reviewer wants to inspect or edit inside WordPress

Behavior:
- create or update a WordPress draft
- record edit link and post ID
- keep authoritative validation metadata internal

### Scheduled Publish Creation

Used when:
- the system decision is `auto_publish`
- or a reviewer approves `save_scheduled`

Behavior:
- compute publish time from the cadence engine
- set `status=future`
- store internal schedule rationale, window ID, and publish-state history

## Featured Image Placeholder Strategy

Phase 1 does not require dynamic image generation.

Supported options:
- leave `featured_media` unset
- use a tenant-configured placeholder media ID if one is already available in WordPress
- route to review if client policy requires featured images and no placeholder is configured

## Metadata and Audit Storage

Authoritative metadata remains internal:
- source ledger
- validation status
- claim support details
- approval decision
- taxonomy resolution log, including reuse-versus-create rationale
- publish-state history
- reviewer notes

Optional WordPress sync:
- store a lightweight compliance note in registered custom meta only if the site exposes a compatible field
- sync SEO title or meta description only through an optional adapter layer

No core behavior may depend on plugin-specific metadata being available.

## Retry Strategy

Retry on transient failures:
- connection timeout
- `429`
- `500`
- `502`
- `503`
- `504`

Retry policy:
- exponential backoff
- bounded max attempts
- preserve the same content identity and slug
- append each attempt to `publish_state_history`

Do not retry automatically on:
- `401` or `403`
- schema or payload validation failure
- duplicate slug conflict with mismatched identity
- missing required taxonomy with auto-create disabled

These create review tasks or incidents instead.

## Rollback and Unpublish

Rollback cases:
- incorrect content was scheduled
- a factual dispute invalidated the post
- a WordPress update created a malformed post state

Phase 1 actions:
- if still draft or future, revert to `draft` and add an internal incident
- if already published, create an unpublish task for human execution and record the incident, because WordPress site policies may vary

The internal system of record always reflects the current publish status and why it changed.

## Optional Adapter Layer

Optional adapters may be added later for:
- Yoast
- Rank Math
- custom registered meta fields

Adapter rules:
- adapter failure cannot block the core WordPress post create or update flow
- adapter enablement is per client
- adapters must write only to fields already registered and supported by the target site
