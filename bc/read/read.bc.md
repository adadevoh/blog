# Read Behaviour Contract

Implements **BH-READ** — `blog.design.md` "Full public work only with a covering grant; otherwise not the full body. Private is not found".

Iteration 2. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: fetch one work on a publisher catalog.
Out of scope: the list (BH-FEED), unlock, compose.

## Surface — `{site}/publishers/{username}/works/{id}`

Tests call these HTTP routes on the website host. JSON, camelCase.

Optional header: `Authorization: Bearer {subscriberToken}`.

### GET /publishers/{username}/works/{id}

- **[READ-01]** Success (public work, subscriber session, grant covers this work): **200** `{ id, title, content, topic, postedAt }`
- **[READ-02]** Fail (no session, no grant, or grant topics do not include this work’s topic): **404** `{ error: "not_found" }` — do not return full `content`
- **[READ-03]** Fail (unknown id, unknown publisher, or **private** work): **404** `{ error: "not_found" }` — never **403**
- **[READ-04]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **404** when the store is down

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }`.

## Rules

- **[READ-R1]** Private is indistinguishable from missing for a subscriber (**404**)
- **[READ-R2]** A publisher token for **this** username is not a substitute for a subscriber grant on this read surface
