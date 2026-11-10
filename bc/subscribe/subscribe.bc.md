# Subscribe Behaviour Contract

Implements **BH-SUBSCRIBE** — `blog.design.md` "The grant after invite or accept: whole catalog or 1..N topics. Uniqueness per publisher, subscriber, catalog-or-topic".

Iteration 3. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: the grant record that unlock uses. Created only by BH-INVITE or accepted BH-REQUEST.
Out of scope: open self-serve POST-to-subscribe, sending notifications (BH-NOTIFY), listing/deleting grants (BH-SUBS).

## Surface

No public “subscribe me now” route. Grants appear as a result of invite (BH-INVITE) or accept (BH-REQUEST). Tests observe a grant as `{ publisher, subscriber, catalog: true }` or `{ publisher, subscriber, topics: [ ... ] }`.

- **[SUBSCRIBE-01]** After a successful invite or accept for **whole catalog**: a grant exists with `catalog: true` and no topic restriction
- **[SUBSCRIBE-02]** After a successful invite or accept for **topics**: a grant exists with those topic names (1..N) for this publisher
- **[SUBSCRIBE-03]** Fail (would grant this publisher a subscriber grant on **their** catalog): **409** `{ error: "self_subscribe" }` — no grant
- **[SUBSCRIBE-04]** Fail (same publisher, same subscriber, same catalog-or-topic set already): **409** `{ error: "already_subscribed" }`
- **[SUBSCRIBE-05]** Fail (store unreachable while creating the grant): **503** `{ error: "storage_unavailable" }` — never a successful invite/accept

Until invite/request and this BC are done, those operations may return **501** `{ error: "not_implemented" }`.

## Rules

- **[SUBSCRIBE-R1]** A grant is not a publisher session and does not open the control panel
- **[SUBSCRIBE-R2]** Whole-catalog and topic grants for the same subscriber on the same publisher are distinct; topic grants do not imply other topics
- **[SUBSCRIBE-R3]** Uniqueness of (publisher, subscriber, catalog-or-topic) is in the store
- **[SUBSCRIBE-R4]** No grant without invite or accept
