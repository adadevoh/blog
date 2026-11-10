# Feed Behaviour Contract

Implements **BH-FEED** — `blog.design.md` "Per-publisher catalog: blurred without a covering subscriber grant; otherwise public works in the grant, 5 then more, newest first. Private never listed".

Iteration 2. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: list a publisher’s catalog page.
Out of scope: full body (BH-READ), unlock (BH-UNLOCK), grants (BH-SUBSCRIBE), compose.

## Surface — `{site}/publishers/{username}/works`

Tests call these HTTP routes on the website host. JSON, camelCase.

Page size is **5**. `offset` skips that many works that would appear in this response (newest first).

Optional header: `Authorization: Bearer {subscriberToken}` from BH-UNLOCK.

### GET /publishers/{username}/works

- Query: `offset` (optional, default `0`)
- **[FEED-01]** Success, no subscriber session or no grant on this publisher: **200** `{ blurred: true, works: [ { id, title, topic, preview } ] }`. At most **5**. Public works only. No full `content`. Newest first
- **[FEED-02]** Success, subscriber session with **whole-catalog** grant: **200** `{ blurred: false, works: [ { id, title, content, topic, postedAt } ] }`. At most **5** public works of **this** publisher
- **[FEED-03]** Success, subscriber session with **topic** grant(s): same shape as FEED-02 but only public works whose topic is in the grant
- **[FEED-04]** Success (none left): **200** with `works: []` when there are no further works that this caller is allowed to list
- **[FEED-05]** Fail (unknown `{username}`): **404** `{ error: "not_found" }`
- **[FEED-06]** Fail (invalid `offset`): **400** `{ error: "validation_error" }`
- **[FEED-07]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **200** with `[]` when the store is down

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after validation.

## Rules

- **[FEED-R1]** Private works never appear, even with a whole-catalog grant
- **[FEED-R2]** Do not mix another publisher’s works into this list
- **[FEED-R3]** A publisher session is not a covering grant for this catalog
