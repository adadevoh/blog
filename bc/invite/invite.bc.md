# Invite Behaviour Contract

Implements **BH-INVITE** — `blog.design.md` "Publisher invites a contact to this catalog or topic(s); invite link sent. Fails if that would be self-subscribe".

Iteration 3. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: publisher-created invite to a subscriber contact.
Out of scope: request/accept (BH-REQUEST), open self-serve subscribe, email vendor.

## Surface — `{site}/admin/invites`

Tests call these HTTP routes on the website host. JSON, camelCase.

Header: `Authorization: Bearer {publisherToken}`.

### POST /admin/invites

- Body: `{ email?, phone?, catalog?: true, topics?: [string] }` — exactly one of `email` or `phone`. Either `catalog: true` or a non-empty `topics` array (1..N), not both
- **[INVITE-01]** Success (whole catalog): **201** `{ id, to, catalog: true }`. An invite link is sent to `to`. A whole-catalog grant exists for that subscriber on this publisher (BH-SUBSCRIBE)
- **[INVITE-02]** Success (topics): **201** `{ id, to, topics }`. Invite link sent. A topic grant exists for those topics
- **[INVITE-03]** Fail (would subscribe this publisher to **their** catalog): **409** `{ error: "self_subscribe" }` — no invite, no grant
- **[INVITE-04]** Fail (both or neither contact, both catalog and topics, empty topics, or invalid contact): **400** `{ error: "validation_error" }`
- **[INVITE-05]** Fail (topic named that this publisher has not used on any work): **400** `{ error: "unknown_topic" }`
- **[INVITE-06]** Fail (already invited/granted this contact this catalog-or-topic set): **409** `{ error: "already_subscribed" }`
- **[INVITE-07]** Fail (missing, unknown, or revoked publisher token): **404** `{ error: "not_found" }` — never **403**
- **[INVITE-08]** Fail (store unreachable, or cannot send the invite link): **503** `{ error: "storage_unavailable" }` — never **201**

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after auth and validation.

## Rules

- **[INVITE-R1]** The invite link is how the contact reaches this catalog; it does not skip unlock (BH-UNLOCK)
- **[INVITE-R2]** Never log the contact in the URL of the invite as a query secret beyond what the link must carry
