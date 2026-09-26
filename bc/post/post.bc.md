# Post Behaviour Contract

Implements **BH-POST** — `blog.design.md` "Signed-in poster, in their control panel: write a post, tag it, mark private or not. Public posts go on that diary".

Iteration 4. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: create one post on the signed-in poster’s diary.
Out of scope: listing and changing existing posts (BH-MANAGE), notification fan-out detail (BH-NOTIFY), compose on the shareable diary, other posters’ diaries.

## Surface — `{site}/admin/posts`

Tests call these HTTP routes on the website host. JSON, camelCase.

Header: `Authorization: Bearer {token}` from BH-AUTH.

The diary written is **this poster’s** diary. Today that is the owner.

### POST /admin/posts

- Body: `{ content, topic, private }`
- **[POST-01]** Success, `private: false`: **201** `{ id, content, topic, postedAt, private: false }`. The post is on the shareable diary (BH-FEED, BH-READ). Matching subscribers are notified (BH-NOTIFY)
- **[POST-02]** Success, `private: true`: **201** `{ id, content, topic, postedAt, private: true }`. The post is **not** on the shareable diary. No notifications
- **[POST-03]** Fail (missing/blank `content` or `topic`, or `private` not a boolean): **400** `{ error: "validation_error" }`
- **[POST-04]** Fail (missing, unknown, or revoked token): **404** `{ error: "not_found" }` — never **403**
- **[POST-05]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **201**

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after auth and validation.

## Rules

- **[POST-R1]** A post has exactly one `topic`
- **[POST-R2]** The poster is the session, not a poster id in the body or path
- **[POST-R3]** A public write that cannot issue required notifications is not **201** (BH-NOTIFY)
