# Subscriber grants Behaviour Contract

Implements **BH-SUBS** — `blog.design.md` "Signed-in subscriber lists their grants and deletes their own".

Iteration 3. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: subscriber’s own grant list and delete.
Out of scope: compose, invite, accept, publisher work list (BH-MANAGE).

## Surface — `{site}/me/subscriptions`

Tests call these HTTP routes on the website host. JSON, camelCase.

Header: `Authorization: Bearer {subscriberToken}` from BH-UNLOCK.

### GET /me/subscriptions

- **[SUBS-01]** Success: **200** `{ subscriptions: [ { id, publisher, catalog?, topics? } ] }` — every grant this subscriber holds
- **[SUBS-02]** Success (none): **200** `{ subscriptions: [] }`
- **[SUBS-03]** Fail (missing, unknown, or revoked subscriber token): **404** `{ error: "not_found" }` — never **403**
- **[SUBS-04]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **200** with `[]` when the store is down

### DELETE /me/subscriptions/{id}

- **[SUBS-05]** Success: **204**. That grant is gone. Catalog of that publisher is blurred for this subscriber unless another covering grant remains
- **[SUBS-06]** Fail (unknown id, or not this subscriber’s grant): **404** `{ error: "not_found" }`
- **[SUBS-07]** Fail (missing, unknown, or revoked subscriber token): **404** `{ error: "not_found" }`
- **[SUBS-08]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **204**

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after auth.

## Rules

- **[SUBS-R1]** Subscriber has no compose and cannot list another subscriber’s grants
- **[SUBS-R2]** Delete does not delete works, publishers, or analog sources
- **[SUBS-R3]** A publisher token is not accepted on this surface (**404** `not_found`)
