# Scan Behaviour Contract

Implements **BH-SCAN** — `blog.design.md` "Publisher document becomes the corresponding work; analog document kept as reference".

Iteration 4. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: create one work from a document on this publisher’s catalog.
Out of scope: typed create (BH-POST), dictate (BH-DICTATE), naming OCR or file brands, summarization.

## Surface — `{site}/admin/works/scan`

Tests call these HTTP routes on the website host. JSON, camelCase.

Header: `Authorization: Bearer {publisherToken}`.

### POST /admin/works/scan

- Body: `{ document, title, topic, private }` — `document` is the analog payload
- **[SCAN-01]** Success, `private: false`: **201** `{ id, title, content, topic, postedAt, private: false, analog: "document" }`. `content` is the corresponding article. Analog document is kept as reference. Work is on the catalog for covering grants. Notify (BH-NOTIFY)
- **[SCAN-02]** Success, `private: true`: **201** `{ id, title, content, topic, postedAt, private: true, analog: "document" }`. Analog kept. Not on the catalog. No notify
- **[SCAN-03]** Fail (missing `document`, `title`, or `topic`, or `private` not a boolean): **400** `{ error: "validation_error" }`
- **[SCAN-04]** Fail (document present but cannot become the corresponding article): **422** `{ error: "transform_failed" }` — no work created
- **[SCAN-05]** Fail (missing, unknown, or revoked publisher token): **404** `{ error: "not_found" }` — never **403**
- **[SCAN-06]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **201**

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after auth and validation.

## Rules

- **[SCAN-R1]** The analog document remains stored as reference for that work
- **[SCAN-R2]** `content` corresponds to the document; do not invent a summary
- **[SCAN-R3]** Do not name an OCR, LLM, or file brand in this contract
