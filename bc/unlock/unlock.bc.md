# Unlock Behaviour Contract

Implements **BH-UNLOCK** — `blog.design.md` "Subscriber OTP from header Login or catalog view/unlock; optional remember this device. Does not create a publisher".

Iteration 2. Observable behaviour only. No class names. If design, tests, or code disagree with this file, this BC wins (unless the product changed — then update the design doc and this file first).

Scope: subscriber OTP session. Same operations whether started from header Login → subscriber or catalog view/unlock.
Out of scope: publisher password/passkey (BH-AUTH), creating grants (BH-INVITE / BH-REQUEST), remember-device **lifetime**.

## Surface — `{site}/auth/subscribers`

Tests call these HTTP routes on the website host. JSON, camelCase.

A subscriber session is `Authorization: Bearer {subscriberToken}`. It is not a publisher session.

### POST /auth/subscribers/otp

- Body: `{ email?, phone? }` — exactly one of `email` or `phone`
- **[UNLOCK-01]** Success: **204**. An OTP is sent to that contact. The OTP is not in the response. A subscriber account exists for that contact after a later successful session if it did not already
- **[UNLOCK-02]** Fail (both or neither, or invalid contact): **400** `{ error: "validation_error" }`
- **[UNLOCK-03]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **204**

### POST /auth/subscribers/session

- Body: `{ email?, phone?, otp, rememberDevice? }` — `rememberDevice` is optional boolean
- **[UNLOCK-04]** Success (correct OTP): **200** `{ token, remembered: false }` when `rememberDevice` is omitted or false
- **[UNLOCK-05]** Success (correct OTP, `rememberDevice: true`): **200** `{ token, remembered: true }`. Presenting this token later skips OTP on this device until the session is gone. No duration is specified
- **[UNLOCK-06]** Success (valid remembered subscriber token presented instead of a new OTP): **200** `{ token, remembered: true }` — view/unlock does not require a new OTP
- **[UNLOCK-07]** Fail (wrong OTP, unknown contact with bad otp, or invalid remembered token): **401** `{ error: "invalid_credentials" }`; no usable session
- **[UNLOCK-08]** Fail (invalid body): **400** `{ error: "validation_error" }`
- **[UNLOCK-09]** Fail (store unreachable): **503** `{ error: "storage_unavailable" }` — never **200** or **401**

Until this BC is done, a request that would succeed returns **501** `{ error: "not_implemented" }` after validation.

## Rules

- **[UNLOCK-R1]** Header Login → subscriber and catalog view/unlock are this surface. Do not invent a second OTP flow
- **[UNLOCK-R2]** Success does not create a publisher and does not grant this catalog
- **[UNLOCK-R3]** Never log the OTP or the session token
- **[UNLOCK-R4]** Do not invent a remember-this-device clock
