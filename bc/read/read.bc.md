# Read Behaviour Contract

Implements **BH-READ** — `blog.design.md` "Click a card: scrollable modal with the full public entry".

Iteration 2. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: fetch one public post for the diary in view.
Out of scope: the feed list (BH-FEED), subscribe, control panel, listing private posts.

## Surface — `{site}/diary/posts/{id}`

Tests call these HTTP routes on the website host. JSON, camelCase. No session.

The diary in view is the owner’s diary.

### GET /diary/posts/{id}

- **[READ-01]** Success (public post on this diary): **200** `{ id, content, topic, postedAt }`
- **[READ-02]** Fail (unknown id, or a **private** post, or not this diary): **404** `{ error: "not_found" }` — never **403**
- **[READ-03]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **404** when the store is down

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }`.

## Rules

- **[READ-R1]** A private post is indistinguishable from a missing post to a viewer (**404** `not_found`)
- **[READ-R2]** Do not require a session to read a public post
