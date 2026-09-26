# Feed Behaviour Contract

Implements **BH-FEED** — `blog.design.md` "Shareable diary: public posts as a card feed, 5 then more on scroll, newest first".

Iteration 1. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: read the diary in view as a public card feed.
Out of scope: opening a full entry (BH-READ), subscribe, control panel, other diaries, public signup.

## Surface — `{site}/diary/posts`

Tests call these HTTP routes on the website host. JSON, camelCase. No session.

The diary in view is the owner’s diary. A later amendment may add a diary name on this path; do not invent one now.

Page size is **5**. `offset` is a non-negative count of public posts to skip (newest first).

### GET /diary/posts

- Query: `offset` (optional, default `0`)
- **[FEED-01]** Success: **200** `{ posts: [ { id, content, topic, postedAt } ] }`. At most **5** posts. Only **public** posts for this diary. Newest `postedAt` first. `offset` skips that many public posts.
- **[FEED-02]** Success (none left): **200** `{ posts: [] }` when the diary has no further public posts
- **[FEED-03]** Fail (invalid `offset`): **400** `{ error: "validation_error" }`
- **[FEED-04]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **200** with `[]` when the store is down

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after validation.

## Rules

- **[FEED-R1]** Private posts never appear in `posts`
- **[FEED-R2]** The list is this diary only. Do not mix in another poster’s posts
- **[FEED-R3]** Do not require a session to read the feed
