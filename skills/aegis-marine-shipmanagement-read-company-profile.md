---
name: read-aegis-company-profile
description: >-
  Read Aegis Marine Shipmanagement's published corporate profile — services, fleet posture,
  operations and leadership — from the site's own content API instead of scraping HTML.
api: aegis-marine-shipmanagement:aegis-marine-shipmanagement-pages-api
operations:
  - getPages
  - getPage
  - search
generated: '2026-09-09'
method: generated
source: openapi/aegis-marine-shipmanagement-pages-api-openapi.yml, openapi/aegis-marine-shipmanagement-search-api-openapi.yml
---

# Read the Aegis Marine Shipmanagement company profile

Aegis Marine Shipmanagement is a ship management company in Georgetown, Guyana. Its website runs on
WordPress, and the WordPress REST API is open, so you can read the company's published positioning
as structured JSON rather than parsing pages.

**What this cannot do.** There is no vessel, voyage, charter, port or crew data on this API. The
company publishes no operational data of any kind. If you need fleet particulars, contact the
company at `info@aegisships.com`.

## Base

```
https://aegisships.com/wp-json
```

No credential. No signup. No key. The root index advertises an empty `authentication` array.

## Steps

**1. List the pages (`getPages`).**

Ask for only the fields you need — the full record carries a large `yoast_head` HTML blob you almost
never want.

```
GET /wp/v2/pages?per_page=100&_fields=id,slug,title,link,parent,modified
```

There are 15 pages. Read `X-WP-Total` to confirm the count rather than assuming it.

**2. Identify the profile pages by slug.** The corporate substance lives in these:

| Slug | What it holds |
|---|---|
| `company-profile` | Who the company is and what it manages |
| `ceo-note` | Founder positioning |
| `board-of-directors` | Leadership |
| `operations` | How the company operates |
| `services` | The service lines offered |
| `fleet` / `fleet-selection` / `fleet-management` | Fleet posture and management approach |
| `contact` | Addresses, phone, email |

The four About children (`company-profile`, `ceo-note`, `board-of-directors`, `operations`) all
carry `parent: 429`, so you can also gather them with `?parent=429`.

**3. Fetch each page body (`getPage`).**

```
GET /wp/v2/pages/433
```

Read `content.rendered` for the body HTML and `excerpt.rendered` for a summary. Both are HTML, not
plain text — strip tags before feeding them to a summarizer.

**4. Search when you do not know the slug (`search`).**

```
GET /wp/v2/search?search=crew&per_page=20
```

Returns lightweight `id` / `title` / `url` / `type` / `subtype` records. Follow `_links.self` to the
concrete page. All 15 searchable objects are pages.

## Rules

- **Cap `per_page` at 100.** Above that the API returns HTTP 400 `rest_invalid_param`. Page with
  `page=2,3,…` and stop when you have `X-WP-TotalPages`.
- **Do not try to resolve `author`.** Every page reports `author: 1`, but `/wp/v2/users` returns
  HTTP 401 `rest_user_cannot_view`. There is no way to resolve it anonymously, and no credential to
  obtain.
- **Treat the content as historical.** Every page was last modified between 2018 and 2019. Do not
  present anything read here as the company's current position without saying when it was written —
  check the `modified` field and quote it.
- **Do not attempt writes.** POST/PUT/PATCH/DELETE are registered but closed; they return HTTP 401.
- **Errors** come back as `{code, message, data.status}` — not RFC 9457 problem+json. See
  `errors/aegis-marine-shipmanagement-problem-types.yml`.
