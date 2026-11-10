# Dictate Behaviour Contract

Implements **BH-DICTATE** — `blog.design.md` "Publisher dictation becomes work text; analog audio kept as reference".

Iteration 4. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: create one work from speech on this publisher’s catalog.
Out of scope: typed create (BH-POST), scan (BH-SCAN), naming a speech vendor.

## Surface — `{site}/admin/works/dictate`

Tests call these HTTP routes on the website host. JSON, camelCase.

Header: `Authorization: Bearer {publisherToken}`.

### POST /admin/works/dictate

- Body: `{ audio, title, topic, private }` — `audio` is the analog payload
- **[DICTATE-01]** Success, `private: false`: **201** `{ id, title, content, topic, postedAt, private: false, analog: "audio" }`. `content` is the article text from the audio. Analog audio is kept as reference. Work is on the catalog for covering grants. Notify (BH-NOTIFY)
- **[DICTATE-02]** Success, `private: true`: **201** `{ id, title, content, topic, postedAt, private: true, analog: "audio" }`. Analog kept. Not on the catalog. No notify
- **[DICTATE-03]** Fail (missing `audio`, `title`, or `topic`, or `private` not a boolean): **400** `{ error: "validation_error" }`
- **[DICTATE-04]** Fail (audio present but cannot become article text): **422** `{ error: "transform_failed" }` — no work created
- **[DICTATE-05]** Fail (missing, unknown, or revoked publisher token): **404** `{ error: "not_found" }` — never **403**
- **[DICTATE-06]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **201**

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after auth and validation.

## Rules

- **[DICTATE-R1]** The analog audio remains stored as reference for that work
- **[DICTATE-R2]** `content` is the corresponding article text, not an invented summary
- **[DICTATE-R3]** Do not name a speech vendor in this contract
