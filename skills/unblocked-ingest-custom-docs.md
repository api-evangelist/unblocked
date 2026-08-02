---
name: Ingest custom documents into Unblocked
description: Create a collection and upsert documents so Unblocked can use them as a data source for answers, respecting size and count limits.
api: openapi/unblocked-public-api-openapi-original.json
operations: [createCollection, listCollections, getCollection, saveCollection, deleteCollection, putDocument, listDocuments, deleteDocument]
generated: '2026-07-21'
method: generated
---

# Ingest custom documents into Unblocked

Base URL `https://getunblocked.com/api/v1`, `Authorization: Bearer <token>`.

## Steps

1. `listCollections` (`GET /collections`) to check whether a suitable
   collection already exists — teams are capped at 25 collections.
2. `createCollection` — `POST /collections` with `{name, description,
   iconUrl}`. Name 1-32 chars; description 1-4096 chars (write it for the
   model: describe what the documents are). `201` returns the collection with
   its `id`.
3. `putDocument` — `PUT /documents` with `{collectionId, title, body, uri}`.
   Documents are unique by `uri` across the organization: the same `uri`
   updates in place, so re-running ingestion is safe. Body is plain text or
   Markdown (preferred); max request size 10MB. Documents take up to a minute
   to become available.
4. Verify with `listDocuments` (`GET /documents`, cursor pagination via
   `limit`/`after`/`before` + `link` header).
5. Maintain: `saveCollection` (`PATCH /collections/{collectionId}`) to update
   metadata, `deleteDocument` (`DELETE /documents/{documentId}`) to remove one
   document, `deleteCollection` (`DELETE /collections/{collectionId}`) to
   remove a collection AND all its documents.

## Rules

- `uri` must be a real link back to the source; it is what answers cite.
- Errors return `{"status": <code>}`; `400` usually means a field-constraint
  violation.
