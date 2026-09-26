# Auth Behaviour Contract

Implements **BH-AUTH** — `blog.design.md` "Poster signs in with OTP to their phone or email and reaches their control panel. Today only the owner’s contact may succeed; other contacts do not create an account".

Iteration 3. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: request OTP, open a session, what that session authorises today.
Out of scope: public signup, creating a second poster or diary, passwords, OAuth, B2C, logout, refresh tokens.

## Surface — `{site}/auth`

Tests call these HTTP routes on the website host. JSON, camelCase.

A session is `Authorization: Bearer {token}`. It is a **poster** session. Today the only poster is the configured owner. The token authorises **that poster’s** control panel (BH-POST, BH-MANAGE, BH-NOTIFY write path). It does not put a poster id in the path.

### POST /auth/otp

- Body: `{ email?, phone? }` — exactly one of `email` or `phone`
- **[AUTH-01]** Success (contact is the owner’s): **204**. An OTP is sent to that contact. The OTP is not in the response
- **[AUTH-02]** Fail (contact is not the owner): **401** `{ error: "invalid_credentials" }`. No OTP. No account is created
- **[AUTH-03]** Fail (both or neither, or invalid contact): **400** `{ error: "validation_error" }`
- **[AUTH-04]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **204** or **401**

### POST /auth/session

- Body: `{ email?, phone?, otp }` — the same single contact plus the OTP
- **[AUTH-05]** Success (owner contact, correct OTP): **200** `{ token }`. A session is created
- **[AUTH-06]** Fail (unknown contact, not the owner, or wrong OTP): **401** `{ error: "invalid_credentials" }`; no token; no account is created
- **[AUTH-07]** Fail (missing otp or invalid body): **400** `{ error: "validation_error" }`
- **[AUTH-08]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **200** or **401**

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after validation. **AUTH-02** and **AUTH-06** still run for real (no signup).

## Rules

- **[AUTH-R1]** A contact that is not the owner never becomes a poster
- **[AUTH-R2]** The session authorises only this poster’s diary. A later second poster’s token must not manage this diary
- **[AUTH-R3]** Never log the OTP or the session token
- **[AUTH-R4]** Control-panel operations without a valid token are **404** `not_found`, not **403**
