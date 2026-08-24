---
name: ksq-therapeutics-track-press-release-archive
description: >-
  Read and monitor the KSQ Therapeutics press-release archive over the WordPress REST API at
  ksqtx.com — list, filter by date, page through, and fetch a single release. Use this when you need
  KSQ's own announcements about its eTIL programs (KSQ-001EX, KSQ-004EX), KSQ-4279, or its Takeda,
  Ono, CRISPR Therapeutics and CTMC partnerships, rather than secondhand coverage.
generated: '2026-08-23'
method: generated
source: >-
  Grounded in openapi/ksq-therapeutics-press-releases-api-openapi.yml, derived from the live route
  index at https://ksqtx.com/wp-json/ and verified against anonymous responses on 2026-08-23.
api: https://ksqtx.com/wp-json
operations:
- listPressReleases
- getPressRelease
---

# Track the KSQ Therapeutics press-release archive

KSQ Therapeutics publishes no developer documentation. It does run WordPress, and the
`press_release` custom post type is anonymously readable. 39 items on 2026-08-23.

## Before you start

- **No credentials.** Send no `Authorization` header. Asking for `context=edit` returns
  `401 rest_forbidden`.
- **Back off.** The origin runs a firewall plugin that starts returning **HTTP 403 with an HTML
  body** ("WP Remote Firewall / Blocked because of Malicious Activities") after a burst of requests.
  There is no `Retry-After` and no `429`. Keep concurrency at 1, pause between calls, and treat a
  403 with a `text/html` content type as throttling, not authorization failure. See
  `rate-limits/ksq-therapeutics-rate-limits.yml`.

## Step 1 — list the archive, newest first

`listPressReleases` — `GET /wp/v2/press_release`

```
GET https://ksqtx.com/wp-json/wp/v2/press_release?per_page=20&orderby=date&order=desc&_fields=id,date,slug,link,title
```

- `per_page` maxes at **100** (default 10). `page` starts at 1.
- Read **`X-WP-Total`** and **`X-WP-TotalPages`** from the response headers to size the walk — do not
  page until you get an empty array.
- `_fields` trims the payload. Without it every item carries the full rendered HTML body.

## Step 2 — narrow by date instead of paging everything

```
GET https://ksqtx.com/wp-json/wp/v2/press_release?after=2024-01-01T00:00:00&orderby=date&order=asc
```

`after`, `before`, `modified_after` and `modified_before` all take ISO 8601 datetimes. For a polling
loop, store the highest `modified` you have seen and filter on `modified_after` next run — that
catches edits as well as new posts.

## Step 3 — fetch one release

`getPressRelease` — `GET /wp/v2/press_release/{id}`

```
GET https://ksqtx.com/wp-json/wp/v2/press_release/3459
```

`content.rendered` is HTML. This site is built with the Divi page builder, so bodies frequently
contain `et_pb_*` shortcode markup around the prose — strip it before summarising.

## Step 4 — resolve the hero image in the same request

```
GET https://ksqtx.com/wp-json/wp/v2/press_release/3459?_embed
```

`_embed` inlines `featured_media` and terms under `_embedded`, which saves a second round trip
against a rate limit you cannot see.

## Errors you will actually hit

| code | status | what to do |
|---|---|---|
| `rest_post_invalid_id` | 404 | The id is not published. List first, then fetch. |
| `rest_forbidden` | 401 | You asked for `context=edit`. Use `context=view`. |
| `rest_invalid_param` | 400 | Usually `per_page` over 100 or a malformed date. |
| *(HTML body)* | 403 | Firewall block. Stop, wait, resume slowly. |

Full catalog: `errors/ksq-therapeutics-problem-types.yml`.

## What is NOT here

The pipeline, platform, leadership, board and investor pages are **not** REST-exposed — no
`/wp/v2/pipelines`, no `/wp/v2/platforms`. They exist only as HTML at `https://ksqtx.com/pipelines/…`
and `https://ksqtx.com/platforms/…`. See `data-model/ksq-therapeutics-data-model.yml`.
