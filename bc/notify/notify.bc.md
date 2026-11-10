# Notify Behaviour Contract

Implements **BH-NOTIFY** — `blog.design.md` "When this publisher pushes a **public** work, covering grants get a “new post” notification. Never private".

Iteration 3. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: who is notified when a work becomes public on **this** publisher.
Out of scope: creating the work fields (BH-POST / BH-DICTATE / BH-SCAN), creating grants, full work body in the message.

## Surface

Notification outcome of making a work **public** on this publisher: create (BH-POST, BH-DICTATE, BH-SCAN) with `private: false`, or BH-MANAGE PATCH to `private: false`. Tests create grants, then publish, and observe issued notifications.

An issued notification is `{ kind: "new_post", channel, to }`. It does not include work content.

Until the write/manage BC and this BC are done, those operations may return **501** `{ error: "not_implemented" }`.

### When a work becomes public on this publisher

Publisher session required.

- **[NOTIFY-01]** Success public: every subscriber with a **whole-catalog** grant on this publisher is issued `{ kind: "new_post", channel, to }`
- **[NOTIFY-02]** Success public: each subscriber with a **topic** grant that includes this work’s topic is issued `new_post`
- **[NOTIFY-03]** A topic grant that does **not** include this work’s topic is not issued a notification for this work
- **[NOTIFY-04]** Create or PATCH with `private: true`: **no** notifications
- **[NOTIFY-05]** Fail (cannot issue a required notification, or store unreachable while issuing): **503** `{ error: "storage_unavailable" }` — never a success status
- **[NOTIFY-06]** Fail (no publisher session): **404** `{ error: "not_found" }` — never **403**

## Rules

- **[NOTIFY-R1]** Only this publisher’s covering grants. Another publisher’s subscribers are not notified
- **[NOTIFY-R2]** Never put work content or private works on the notification
- **[NOTIFY-R3]** The publisher is never notified via a self-grant (there is none)
