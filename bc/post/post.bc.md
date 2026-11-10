# Post Behaviour Contract

Implements **BH-POST** — `blog.design.md` "Publisher types a work, topic, private or not. Public goes on this catalog".

Iteration 4. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: type one work on the signed-in publisher’s catalog.
Out of scope: dictate (BH-DICTATE), scan (BH-SCAN), listing (BH-MANAGE), notification detail (BH-NOTIFY).

## Surface — `{site}/admin/works`

Tests call these HTTP routes on the website host. JSON, camelCase.

Header: `Authorization: Bearer {publisherToken}` from BH-AUTH.

### POST /admin/works

- Body: `{ title, content, topic, private }`
- **[POST-01]** Success, `private: false`: **201** `{ id, title, content, topic, postedAt, private: false }`. The work is on this publisher’s catalog for covering grants. Matching subscribers are notified (BH-NOTIFY)
- **[POST-02]** Success, `private: true`: **201** `{ id, title, content, topic, postedAt, private: true }`. Not on the catalog. No notifications
- **[POST-03]** Fail (missing/blank `title`, `content`, or `topic`, or `private` not a boolean): **400** `{ error: "validation_error" }`
- **[POST-04]** Fail (missing, unknown, or revoked publisher token): **404** `{ error: "not_found" }` — never **403**
- **[POST-05]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **201**

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after auth and validation.

## Rules

- **[POST-R1]** A work has exactly one `topic`
- **[POST-R2]** The publisher is the session, not a publisher id in the body
- **[POST-R3]** A public write that cannot issue required notifications is not **201** (BH-NOTIFY)
- **[POST-R4]** Typed capture has no analog source
