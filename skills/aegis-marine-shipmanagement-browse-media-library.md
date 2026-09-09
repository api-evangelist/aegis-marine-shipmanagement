---
name: browse-aegis-media-library
description: >-
  Enumerate and filter the 121 media attachments behind aegisships.com and resolve each to its
  original file URL.
api: aegis-marine-shipmanagement:aegis-marine-shipmanagement-media-api
operations:
  - getMediaItems
  - getMediaItem
generated: '2026-09-09'
method: generated
source: openapi/aegis-marine-shipmanagement-media-api-openapi.yml
---

# Browse the Aegis Marine Shipmanagement media library

The media library behind aegisships.com holds 121 attachments — vessel and corporate photography,
logos and page imagery — readable anonymously through the WordPress REST API.

## Base

```
https://aegisships.com/wp-json
```

No credential required.

## Steps

**1. List attachments (`getMediaItems`).**

```
GET /wp/v2/media?per_page=100&_fields=id,slug,title,media_type,mime_type,source_url,date&page=1
```

`X-WP-Total` read 121 and `X-WP-TotalPages` read 2 at that page size. Walk pages until you have
them all; do not assume the count.

**2. Filter by type when you want images only.**

```
GET /wp/v2/media?media_type=image&per_page=100
GET /wp/v2/media?mime_type=image/jpeg&per_page=100
```

**3. Resolve to the file.** `source_url` is the direct URL to the original upload, for example
`https://aegisships.com/wp-content/uploads/2019/05/Preeminent-Shipping.jpg`. Use it rather than
constructing a path.

**4. Fetch one attachment for its variants (`getMediaItem`).**

```
GET /wp/v2/media/767
```

`media_details.sizes` carries the generated thumbnail and intermediate renditions with their own
`source_url` values — use the smallest size that meets your need instead of the original.

## Rules

- **`per_page` maximum is 100.** Above it, HTTP 400 `rest_invalid_param`.
- **`alt_text` is usually empty** on this site. Do not rely on it for accessibility text or for
  describing an image; caption and title are frequently just the filename.
- **Check licensing before reuse.** These are a company's own corporate images. Nothing on this API
  grants a licence to republish them; see `https://aegisships.com/disclaimer/`.
- **The library is static.** The newest upload dates from 2019-05-20. Polling for new media will
  find nothing.
- **`post` and `author` may not resolve.** `author` is 401-gated; `post` may be null for unattached
  items.
