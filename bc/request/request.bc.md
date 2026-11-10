# Request Behaviour Contract

Implements **BH-REQUEST** — `blog.design.md` "Visitor requests this catalog or topic(s); publisher accept/reject. Fails if requester is this publisher".

Iteration 3. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: subscription request and publisher decide.
Out of scope: invite (BH-INVITE), unlock (BH-UNLOCK).

## Surface — `{site}/publishers/{username}/requests` and `{site}/admin/requests`

Tests call these HTTP routes on the website host. JSON, camelCase.

### POST /publishers/{username}/requests

- Body: `{ email?, phone?, catalog?: true, topics?: [string] }` — exactly one contact; either whole catalog or 1..N topics, not both
- **[REQUEST-01]** Success: **201** `{ id, status: "pending" }`. No grant yet. Catalog remains blurred for this contact
- **[REQUEST-02]** Fail (unknown `{username}`): **404** `{ error: "not_found" }`
- **[REQUEST-03]** Fail (requester is this publisher — self-subscribe): **409** `{ error: "self_subscribe" }`
- **[REQUEST-04]** Fail (invalid body): **400** `{ error: "validation_error" }`
- **[REQUEST-05]** Fail (topic not used on this publisher’s works): **400** `{ error: "unknown_topic" }`
- **[REQUEST-06]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **201**

### GET /admin/requests

Header: `Authorization: Bearer {publisherToken}`.

- **[REQUEST-07]** Success: **200** `{ requests: [ { id, to, catalog?, topics?, status } ] }` pending and decided requests for **this** publisher
- **[REQUEST-08]** Fail (no publisher token): **404** `{ error: "not_found" }`
- **[REQUEST-09]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **200** with `[]` when the store is down

### POST /admin/requests/{id}/decision

Header: `Authorization: Bearer {publisherToken}`.

- Body: `{ accept: true }` or `{ accept: false }`
- **[REQUEST-10]** Success accept: **200** `{ id, status: "accepted" }`. A grant exists (BH-SUBSCRIBE) for the requested catalog or topics
- **[REQUEST-11]** Success reject: **200** `{ id, status: "rejected" }`. No grant
- **[REQUEST-12]** Fail (unknown id, or not this publisher’s request): **404** `{ error: "not_found" }`
- **[REQUEST-13]** Fail (`accept` missing or not boolean): **400** `{ error: "validation_error" }`
- **[REQUEST-14]** Fail (no publisher token): **404** `{ error: "not_found" }`
- **[REQUEST-15]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **200**

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after validation/auth.

## Rules

- **[REQUEST-R1]** Accept is what creates the grant, not the incoming request
- **[REQUEST-R2]** Reject leaves the catalog blurred for that contact
- **[REQUEST-R3]** A publisher session for `{username}` cannot create a successful REQUEST-01 on that same `{username}`
