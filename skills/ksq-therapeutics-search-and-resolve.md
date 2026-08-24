---
name: ksq-therapeutics-search-and-resolve
description: >-
  Find any published KSQ Therapeutics content by keyword using the WordPress cross-type search
  endpoint, then resolve each hit to its full record. Use this as the first call when you do not know
  which collection holds what you want — it spans pages and press releases in one request.
generated: '2026-08-23'
method: generated
source: >-
  Grounded in openapi/ksq-therapeutics-search-api-openapi.yml and
  openapi/ksq-therapeutics-discovery-api-openapi.yml, derived from the live route index at
  https://ksqtx.com/wp-json/ and verified anonymously on 2026-08-23.
api: https://ksqtx.com/wp-json
operations:
- search
- getRouteIndex
- listTypes
- getPage
- getPressRelease
---

# Search KSQ Therapeutics content and resolve the hits

## Step 1 — search across every REST-exposed type

`search` — `GET /wp/v2/search`

```
GET https://ksqtx.com/wp-json/wp/v2/search?search=USP1&per_page=20&_fields=id,title,url,type,subtype
```

48 records were reachable on 2026-08-23. Results are deliberately lightweight — `id`, `title`,
`url`, `type`, `subtype` — not full objects.

Narrow with `subtype` when you already know what you want:

```
GET https://ksqtx.com/wp-json/wp/v2/search?search=eTIL&subtype=press_release
```

## Step 2 — resolve a hit

`subtype` tells you which collection to fetch from:

| `subtype` | follow-up | operation |
|---|---|---|
| `press_release` | `GET /wp/v2/press_release/{id}` | `getPressRelease` |
| `page` | `GET /wp/v2/pages/{id}` | `getPage` |
| `post` | `GET /wp/v2/posts/{id}` | `getPost` — empty on this deployment |

## Step 3 — when a result will not resolve

If a `subtype` has no matching `/wp/v2/{base}` route, it is a content type that is registered but not
REST-exposed. Confirm against the route index:

```
GET https://ksqtx.com/wp-json/
```

`getRouteIndex` returns all 349 routes across 9 namespaces, and `listTypes`
(`GET /wp/v2/types`) returns the registered post types. Anything present in `types` with no route is
HTML-only — fall back to fetching the `url` from the search result directly.

## Rate-limit discipline

The origin's firewall answers **403 with an HTML body** after a burst — no `429`, no `Retry-After`.
Search once and resolve serially rather than fanning out. Details in
`rate-limits/ksq-therapeutics-rate-limits.yml`.
