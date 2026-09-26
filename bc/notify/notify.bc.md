# Notify Behaviour Contract

Implements **BH-NOTIFY** — `blog.design.md` "When a public post is added to a diary, matching subscribers of that diary receive a “new post” notification on the channel they gave".

Iteration 2. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: who is notified, on which channel, and that the payload is **“new post”**.
Out of scope: creating the post’s fields (BH-POST), creating subscriptions (BH-SUBSCRIBE), unsubscribe, full post body in the message, other diaries’ subscribers.

## Surface — `{site}/admin/posts`

This BC is the notification outcome of making a post **public** on this diary: BH-POST `POST /admin/posts` with `private: false`, and BH-MANAGE `PATCH /admin/posts/{id}` with `private: false` when it was private. Tests create subscriptions (BH-SUBSCRIBE), then write or un-private a post, and observe issued notifications. No extra consumer route.

An issued notification is `{ kind: "new_post", channel, to }`. `kind` is always `"new_post"`. It does not include post content.

Until the write/manage BC and this BC are done, those operations may return **501** `{ error: "not_implemented" }`.

### When a post becomes public on this diary

Session required (BH-AUTH).

- **[NOTIFY-01]** Success public create or un-private: every whole-diary subscriber of **this diary** is issued `{ kind: "new_post", channel, to }` on the channel they subscribed with
- **[NOTIFY-02]** Success public create or un-private, matching tag: each subscriber of this diary whose `topic` equals the post’s tag is also issued `new_post` on their channel
- **[NOTIFY-03]** A subscriber of this diary whose `topic` is set and **does not** equal the post’s tag is not issued a notification for this post
- **[NOTIFY-04]** Create with `private: true`, or PATCH to `private: true`: **no** notifications are issued
- **[NOTIFY-05]** Fail (cannot issue a required notification, or store unreachable while issuing): **503** `{ error: "storage_unavailable" }` — never a success status
- **[NOTIFY-06]** Fail (no session): **404** `{ error: "not_found" }` — never **403**

## Rules

- **[NOTIFY-R1]** Notifications are for this diary’s subscribers only. A later second diary does not receive these
- **[NOTIFY-R2]** Never put post content or private posts on the notification
- **[NOTIFY-R3]** Whole-diary and matching-topic subscriptions both fire for the same public post when both exist for a contact; that is two issues if they are two subscriptions
