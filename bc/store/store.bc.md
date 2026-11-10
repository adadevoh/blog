# Store Behaviour Contract

Implements **BH-STORE** — `blog.design.md` "Publishers, subscribers, catalogs, works, analog refs, grants, invites, requests, and sessions survive a restart. A store failure is never an empty catalog, an empty panel, or a successful write".

Iteration 1. Observable behaviour only. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: process health, store readiness, durability of domain data, how store failure looks.
Out of scope: schema, provider, migrations (spec when the engine is chosen).

## Surface — `{site}`

Tests call these HTTP routes on the website host.

### GET /health

- **[STORE-01]** Success: **200**. Process is up. Does not touch the database

### GET /health/ready

- **[STORE-02]** Success: **200** when the database is reachable
- **[STORE-03]** Fail (database not reachable): **503** `{ error: "storage_unavailable" }`

Until this BC is done, `/health/ready` may return **501** `{ error: "not_implemented" }` instead of probing the store. `/health` is real.

## Rules

- **[STORE-R1]** After a process restart, publishers, subscribers, catalogs, works, analog references, grants, invites, requests, publisher sessions, and remember-this-device subscriber sessions that were saved are still there
- **[STORE-R2]** A store outage on a read that would return a list is **503** `storage_unavailable`, never **200** with `[]`
- **[STORE-R3]** A store outage on signup, session, invite, request, write, or grant delete is **503** `storage_unavailable`, never a success
- **[STORE-R4]** Missing connection string: the process does not boot
- **[STORE-R5]** Public listing is per publisher catalog, not one global mix of all authors
