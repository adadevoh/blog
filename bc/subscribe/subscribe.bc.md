# Subscribe Behaviour Contract

Implements **BH-SUBSCRIBE** — `blog.design.md` "On a diary: subscribe with email or phone to all public posts, or filter by one tag used on that diary’s public posts".

Iteration 2. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: create a subscription to the diary in view.
Out of scope: sending the notification (BH-NOTIFY), unsubscribe, accounts, other diaries.

## Surface — `{site}/diary/subscriptions`

Tests call these HTTP routes on the website host. JSON, camelCase. No session.

The diary in view is the owner’s diary. The subscription is to **that diary**, so a later second diary does not inherit these subscribers.

Exactly one of `email` or `phone`. `topic` omitted means all public posts on this diary.

### POST /diary/subscriptions

- Body: `{ email?, phone?, topic? }`
- **[SUBSCRIBE-01]** Success (all posts): **201** `{ id, channel, to, topic: null }` when `topic` is omitted. `channel` is `"email"` or `"phone"`; `to` is the contact given
- **[SUBSCRIBE-02]** Success (filtered): **201** `{ id, channel, to, topic }` when `topic` matches a tag already used on a **public** post on this diary
- **[SUBSCRIBE-03]** Fail (both email and phone, or neither, or invalid contact): **400** `{ error: "validation_error" }`
- **[SUBSCRIBE-04]** Fail (`topic` present but not used on any public post on this diary): **400** `{ error: "unknown_topic" }`
- **[SUBSCRIBE-05]** Fail (same contact, same diary, same topic already subscribed — `topic` null counts as all): **409** `{ error: "already_subscribed" }`
- **[SUBSCRIBE-06]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **201**

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after validation.

## Rules

- **[SUBSCRIBE-R1]** A subscription is not a session and does not open the control panel
- **[SUBSCRIBE-R2]** Whole-diary (`topic` null) and a tagged subscription for the same contact are two subscriptions
- **[SUBSCRIBE-R3]** Uniqueness of (diary, contact, topic) is in the store, not only in application checks
