---
name: inspect-aegis-api-surface
description: >-
  Discover what this undocumented API actually exposes by reading its self-describing route index,
  content types, taxonomies and statuses.
api: aegis-marine-shipmanagement:aegis-marine-shipmanagement-discovery-api
operations:
  - getApiIndex
  - getTypes
  - getTaxonomies
  - getStatuses
  - getCategories
  - getTags
generated: '2026-09-09'
method: generated
source: openapi/aegis-marine-shipmanagement-discovery-api-openapi.yml, openapi/aegis-marine-shipmanagement-taxonomy-api-openapi.yml
---

# Inspect the Aegis Marine Shipmanagement API surface

Aegis Marine Shipmanagement publishes no developer documentation at all. The API is nonetheless
self-describing, so you can enumerate the whole surface from the wire before calling anything.

## Base

```
https://aegisships.com/wp-json
```

## Steps

**1. Read the root index (`getApiIndex`).**

```
GET /wp-json/
```

Returns `name`, `url`, `namespaces` and the complete `routes` map. At capture: 90 routes across 6
namespaces — `wp/v2`, `oembed/1.0`, `yoast/v1`, `contact-form-7/v1`, `wordfence/v1` and
`wp-site-health/v1`. Each route entry lists its `methods` and its `args`, which is where the real
parameter contract lives.

Note `"authentication": []` — nothing is advertised, which is the signal that this is an
unauthenticated read surface with no way in.

**2. Enumerate content types (`getTypes`).**

```
GET /wp/v2/types
```

Only WordPress core types are registered. **There is no custom post type**, which is the finding
that matters: no `vessel`, `fleet`, `service` or any other company-specific resource exists. Compare
against `rest_base` values to know where each collection lives.

**3. Enumerate taxonomies and terms (`getTaxonomies`, `getCategories`, `getTags`).**

```
GET /wp/v2/taxonomies
GET /wp/v2/categories?per_page=100
GET /wp/v2/tags?per_page=100
```

One category (`News`) and 25 tags. **Every term reports `count: 0`** because the site publishes no
posts. Treat the taxonomy as an empty scaffold — do not infer topic coverage from tag names. Several
of them (`Copenhagen`, `Metro`, `Motorway`, `Warsaw`, `Spiecag`) are unremoved theme demo data and
describe nothing on this site.

**4. Check what is visible (`getStatuses`).**

```
GET /wp/v2/statuses
```

Only `publish` is returned. Anything in another status is invisible to you.

**5. Confirm the empty collections yourself.**

```
GET /wp/v2/posts?per_page=1     # X-WP-Total: 0
GET /wp/v2/comments?per_page=1  # X-WP-Total: 0
```

Both return HTTP 200 with `[]`. An empty array is a valid answer here, not an error — do not retry.

## Rules

- **Read `X-WP-Total` before paging.** It is the only count the API gives you.
- **Routes listing a write method are not open to you.** `/wp/v2/settings` returns HTTP 401
  `rest_forbidden` and `/wp/v2/users` returns HTTP 401 `rest_user_cannot_view`. Registration in the
  route index is not permission.
- **`/contact-form-7/v1/contact-forms` returns HTTP 403** `wpcf7_forbidden`. Do not attempt to
  submit the contact form through the API; use `https://aegisships.com/contact/`.
- **Expect no rate-limit headers.** None are returned and no limit is published. A Wordfence
  installation is present and may throttle at the edge without warning, so keep request rates
  modest and back off on any non-200.
- **`rest_no_route` (404)** means the route or the method is wrong — re-read the root index rather
  than guessing paths.
