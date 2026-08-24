---
name: ksq-therapeutics-harvest-media-library
description: >-
  Enumerate the KSQ Therapeutics media library over the WordPress REST API — pipeline diagrams,
  leadership and board headshots, conference poster PDFs and site video — and resolve each to a
  direct source URL and size variant. Use this to collect KSQ's own imagery and documents rather
  than scraping rendered pages.
generated: '2026-08-23'
method: generated
source: >-
  Grounded in openapi/ksq-therapeutics-media-api-openapi.yml, derived from the live route index at
  https://ksqtx.com/wp-json/ and verified anonymously on 2026-08-23 (235 attachments, X-WP-Total).
api: https://ksqtx.com/wp-json
operations:
- listMedia
- getMediaItem
---

# Harvest the KSQ Therapeutics media library

235 attachments were anonymously readable on 2026-08-23.

## Step 1 — list, filtered by kind

`listMedia` — `GET /wp/v2/media`

```
GET https://ksqtx.com/wp-json/wp/v2/media?per_page=100&media_type=image&_fields=id,date,slug,alt_text,mime_type,source_url
```

`media_type` accepts `image`, `video`, `application` (this is where the poster PDFs live), `audio`,
`text` and `file`. `mime_type` filters more precisely — `application/pdf`.

Page with `page`, sized from the `X-WP-Total` header. `per_page` maxes at 100.

## Step 2 — pick the right size

`getMediaItem` — `GET /wp/v2/media/{id}`

`media_details.sizes` carries the generated variants (thumbnail, medium, large, full), each with its
own `source_url`, `width` and `height`. Take the smallest that meets your need — the full-size
originals on this site include multi-megabyte scaled JPEGs and an MP4 background video.

## Step 3 — find only what is new

```
GET https://ksqtx.com/wp-json/wp/v2/media?after=2026-01-01T00:00:00&orderby=date&order=desc
```

The newest attachment on 2026-08-23 was `260821_Updated_Pipeline.png`, uploaded 2026-08-21 — the
current pipeline chart. Polling `after` with your last-seen timestamp is how you catch a pipeline
update without re-downloading the library.

## Attribution

These are KSQ Therapeutics assets. `alt_text` and `title.rendered` come from the company. Nothing in
this repo grants a licence to redistribute them; check with KSQ before republishing.

## Rate-limit discipline

Serial requests only. A burst trips the site firewall, which returns **403 with an HTML body** and no
retry guidance. See `rate-limits/ksq-therapeutics-rate-limits.yml`.
