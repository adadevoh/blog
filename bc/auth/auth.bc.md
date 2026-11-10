# Auth Behaviour Contract

Implements **BH-AUTH** — `blog.design.md` "Publisher signs up and signs in with username + password or passkey and reaches their control panel".

Iteration 1. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: publisher signup and session.
Out of scope: subscriber OTP (BH-UNLOCK), OAuth, B2C, logout, subscriber grants.

## Surface — `{site}/auth/publishers`

Tests call these HTTP routes on the website host. JSON, camelCase.

A publisher session is `Authorization: Bearer {publisherToken}`. It authorises **this** publisher’s control panel only. It is not a subscriber session.

### POST /auth/publishers

- Body: `{ username, password }`
- **[AUTH-01]** Success: **201** `{ username }`. A publisher and empty catalog exist. No session yet
- **[AUTH-02]** Fail (username taken): **409** `{ error: "username_taken" }`
- **[AUTH-03]** Fail (missing/invalid username or password): **400** `{ error: "validation_error" }`
- **[AUTH-04]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **201**

### POST /auth/publishers/session

- Body: `{ username, password }` **or** a passkey assertion that identifies that username
- **[AUTH-05]** Success (password): **200** `{ token }` — a publisher session
- **[AUTH-06]** Success (passkey): **200** `{ token }` — same session shape as AUTH-05
- **[AUTH-07]** Fail (unknown username, wrong password, or failed passkey): **401** `{ error: "invalid_credentials" }`; no token
- **[AUTH-08]** Fail (invalid body): **400** `{ error: "validation_error" }`
- **[AUTH-09]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **200** or **401**

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after validation.

## Rules

- **[AUTH-R1]** A publisher session never authorises subscriber read of another catalog by itself
- **[AUTH-R2]** Never log passwords, passkey material, or the session token
- **[AUTH-R3]** Control-panel operations without a valid publisher token are **404** `not_found`, not **403**
- **[AUTH-R4]** Signup does not create a subscriber or a grant on this catalog
