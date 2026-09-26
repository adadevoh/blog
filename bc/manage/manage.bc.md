# Manage Behaviour Contract

Implements **BH-MANAGE** — `blog.design.md` "Signed-in poster, in their control panel: see all of their posts and manage them (including private or not)".

Iteration 4. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: list all of this poster’s posts; set a post private or not.
Out of scope: creating a post (BH-POST), the public feed, other posters’ posts, delete, signup.

## Surface — `{site}/admin/posts`

Tests call these HTTP routes on the website host. JSON, camelCase.

Header: `Authorization: Bearer {token}` from BH-AUTH.

All operations are this poster’s diary only.

### GET /admin/posts

- **[MANAGE-01]** Success: **200** `{ posts: [ { id, content, topic, postedAt, private } ] }`. Every post on this diary, public and private. Newest first
- **[MANAGE-02]** Success (none): **200** `{ posts: [] }` when this poster has no posts
- **[MANAGE-03]** Fail (missing, unknown, or revoked token): **404** `{ error: "not_found" }` — never **403**
- **[MANAGE-04]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **200** with `[]` when the store is down

### PATCH /admin/posts/{id}

- Body: `{ private }`
- **[MANAGE-05]** Success, set `private: true`: **200** `{ id, content, topic, postedAt, private: true }`. The post leaves the shareable diary. No “new post” notification
- **[MANAGE-06]** Success, set `private: false`: **200** `{ id, content, topic, postedAt, private: false }`. The post is on the shareable diary. If it was private before, matching subscribers are issued `new_post` (BH-NOTIFY) — it is a new public post from their point of view
- **[MANAGE-07]** Fail (unknown id, or not this poster’s post): **404** `{ error: "not_found" }` — never **403**
- **[MANAGE-08]** Fail (`private` missing or not a boolean): **400** `{ error: "validation_error" }`
- **[MANAGE-09]** Fail (missing, unknown, or revoked token): **404** `{ error: "not_found" }`
- **[MANAGE-10]** Fail (store unreachable, or cannot issue required notifications when making public): **503** `{ error: "storage_unavailable" }` — never **200**

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after auth and validation.

## Rules

- **[MANAGE-R1]** The panel never lists another poster’s posts
- **[MANAGE-R2]** Setting private does not delete the post
- **[MANAGE-R3]** Viewers still get **404** `not_found` for a post after it is made private (BH-READ)
