# Manage Behaviour Contract

Implements **BH-MANAGE** — `blog.design.md` "Publisher sees all of their works and manages them (including private or not)".

Iteration 4. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: list all of this publisher’s works; set a work private or not.
Out of scope: creating a work, subscriber grant list (BH-SUBS), other publishers’ works, delete work.

## Surface — `{site}/admin/works`

Tests call these HTTP routes on the website host. JSON, camelCase.

Header: `Authorization: Bearer {publisherToken}`.

### GET /admin/works

- **[MANAGE-01]** Success: **200** `{ works: [ { id, title, content, topic, postedAt, private } ] }`. Every work of this publisher, public and private. Newest first
- **[MANAGE-02]** Success (none): **200** `{ works: [] }` when this publisher has no works
- **[MANAGE-03]** Fail (missing, unknown, or revoked token): **404** `{ error: "not_found" }` — never **403**
- **[MANAGE-04]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **200** with `[]` when the store is down

### PATCH /admin/works/{id}

- Body: `{ private }`
- **[MANAGE-05]** Success, set `private: true`: **200** `{ id, title, content, topic, postedAt, private: true }`. Leaves the catalog. No “new post” notification
- **[MANAGE-06]** Success, set `private: false`: **200** `{ id, title, content, topic, postedAt, private: false }`. On the catalog for covering grants. If it was private before, matching subscribers are issued `new_post` (BH-NOTIFY)
- **[MANAGE-07]** Fail (unknown id, or not this publisher’s work): **404** `{ error: "not_found" }` — never **403**
- **[MANAGE-08]** Fail (`private` missing or not a boolean): **400** `{ error: "validation_error" }`
- **[MANAGE-09]** Fail (missing, unknown, or revoked token): **404** `{ error: "not_found" }`
- **[MANAGE-10]** Fail (store unreachable, or cannot issue required notifications when making public): **503** `{ error: "storage_unavailable" }` — never **200**

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after auth and validation.

## Rules

- **[MANAGE-R1]** The panel never lists another publisher’s works
- **[MANAGE-R2]** Setting private does not delete the work
- **[MANAGE-R3]** Subscribers still get **404** for a work after it is made private (BH-READ)
