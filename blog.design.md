# Blog — product design

Method: [BCDD](https://github.com/adadevoh/bcdd/blob/main/bcdd.md). This file is the product. Do not copy `bcdd.md` into this repo. Observable behaviour lives in `bc/**/*.bc.md`. Mechanism lives in `*.spec.md` only when *how* needs agreeing.

If this file, a BC, and the code disagree: this file wins on product, the BC on behaviour, the spec on how. Amend the document first, then the code.

The Blazor template pages (Home / Counter / Weather) are scaffold, not this product.

## 1. What it is

This is a **gated publishing catalog**. Two **account** types:

- **Publisher** — creates written works (type, dictate, or scan), marked **private or not**, filed under **topics**. Has a control panel for their works and for who may subscribe.
- **Subscriber** — reads another publisher’s **public** works according to a **grant** (whole catalog, or 1..N topics) and gets **“new post”** notifications. Read-only except they may **list and delete their own subscriptions**.

A person may hold **both** kinds of account. Those are two doors, not one merged session. A publisher **cannot subscribe to their own** catalog. They **can** subscribe to **another** publisher.

Subscribers find a catalog by **invite** or a **direct link**. There is no directory. The catalog page always runs **view/unlock** unless **remember this device** already holds a subscriber session.

Value: publish under topics; let invited or accepted readers in; keep private works off the catalog and off notifications.

## 2. What it is not

Do not build these unless this document is updated to allow them:

- A mixed home feed of every publisher’s works
- A publisher directory or search
- Open self-serve subscribe (no invite and no accepted request)
- A publisher granting themselves a subscriber grant on **their** catalog
- Compose, invite, or manage works on the catalog page. That is the control panel
- Subscriber compose or any write except deleting their own grants
- OTP as **publisher** login. Publishers are username + password or passkey
- Password as **subscriber** login. Subscribers are email or phone + OTP
- OAuth, Azure AD B2C, or a third account type
- Resume or Projects *content*
- Comments, likes, follows, profiles, images as the article itself, markdown, or more than one topic per work
- Topics as their own product (graphs, trending). Topics categorize works and shape grants
- Private works on the catalog, in blur lists as full body, or in notifications — even for a whole-catalog grant
- Sending the full work body as the notification. The notification is **“new post”**
- A remember-this-device **clock**. Remembered vs not is specified; a lifetime is not
- A separate public API app or a Next.js app. This product is one website
- Staff roles or a site-wide admin over other publishers’ catalogs
- Features that belong to a later iteration, “while you are here”
- Invented clocks. The only specified page size is the first **5** works on a catalog feed

## 3. Domain

**Publisher** — an account identified by **username**. Signs in with **password or passkey**. Owns one **catalog**. Writes, dictates, or scans works; invites and accepts subscribers; never holds a subscriber grant on their own catalog.

**Subscriber** — an account identified by **email or phone**. Signs in with **OTP** (the same flow from header Login or catalog view/unlock). Optional **remember this device**. Holds **grants** to other publishers’ catalogs. Read-only plus delete-own-grants.

**Catalog** — one publisher’s page of works, reached by invite or direct link (the publisher’s public name). Not a site-wide mix of authors.

**Work / article** — title, written content, one topic, post date, the publisher, **private or not**. Dictate and scan also keep the **analog source** (audio or document) as reference. Public works may appear on that catalog for covering grants and may notify. Private works exist only in that publisher’s control panel.

**Topic** — a tag on a work. A grant may be the whole catalog or 1..N topics on that catalog.

**Grant** — after invite or accepted request: this subscriber may read (and be notified of) this publisher’s **public** works, either all of them or only listed topics. Uniqueness: publisher + subscriber + catalog-or-topic.

**Invite** — the publisher adds a subscriber contact to catalog or topic(s) and an invite link is sent.

**Request** — a visitor asks for catalog or topic(s); the publisher accepts or rejects.

Bounded contexts: **Reading**, **Publishing**, **Identity**, **Subscriptions**. A later BC belongs in one of these. Do not invent a fifth.

## 4. Flows

### Chrome

Header: **Blog**, **Resume**, **Projects** on the left; **Login** on the right. Resume and Projects are unspecified.

**Login** is a chooser: **subscriber** or **publisher**.

- Subscriber: the **same** flow as catalog **view/unlock** (contact → OTP → session; optional remember this device).
- Publisher: username + password or passkey → **that** publisher’s control panel.

### Catalog (invite or direct link)

1. Open a publisher’s catalog. If there is no subscriber session with a grant, the feed is **blurred** (titles, maybe a preview). Full bodies are not shown.
2. **View/unlock** runs the subscriber OTP flow (unless remember-this-device already has a subscriber session).
3. Without a grant: they may **request** catalog or topic(s). They cannot read full public works until accepted (or invited).
4. With a grant and unlocked: card feed of **public** works in that grant. First page **5**, more on scroll, newest first. Topic grant: only those topics. Whole-catalog grant: all public works of **this** publisher. **Private never.**
5. Click a card: full public entry only if the grant covers it.

A visiting **publisher** uses Login → publisher (their panel) or Login → subscriber (to read **this** catalog, which must not be their own).

### Publisher

1. Login as publisher. Control panel: type, dictate, or scan a work; topic; private or not. Dictate/scan keep analog as reference.
2. Public: on this catalog for covering subscribers; matching grants get **“new post”**. Private: panel only; no notify.
3. See all of **their** works and manage them (including private or not).
4. Invite a contact to catalog or topic(s), or accept/reject requests. Cannot grant themselves.

### Subscriber

1. Unlock (header or view/unlock). Remember this device optional.
2. Read public works in their grants. Manage a **list of subscriptions** and **delete** what they do not want. No compose.

## 5. Behaviours

Each behaviour has a stable ID (`BH-…`). That ID is the unit of a change list, an iteration, and a release. Minor behaviours live inside the BC, not here.

| ID | Behaviour | BC | Status |
| --- | --- | --- | --- |
| **BH-FEED** | Per-publisher catalog: blurred without a covering subscriber grant; otherwise public works in the grant, 5 then more, newest first. Private never listed | `bc/feed/feed.bc.md` | BC written |
| **BH-READ** | Full public work only with a covering grant; otherwise not the full body. Private is not found | `bc/read/read.bc.md` | BC written |
| **BH-AUTH** | Publisher signs up and signs in with username + password or passkey and reaches their control panel | `bc/auth/auth.bc.md` | BC written |
| **BH-UNLOCK** | Subscriber OTP from header Login or catalog view/unlock; optional remember this device. Does not create a publisher | `bc/unlock/unlock.bc.md` | BC written |
| **BH-INVITE** | Publisher invites a contact to this catalog or topic(s); invite link sent. Fails if that would be self-subscribe | `bc/invite/invite.bc.md` | BC written |
| **BH-REQUEST** | Visitor requests this catalog or topic(s); publisher accept/reject. Fails if requester is this publisher | `bc/request/request.bc.md` | BC written |
| **BH-SUBSCRIBE** | The grant after invite or accept: whole catalog or 1..N topics. Uniqueness per publisher, subscriber, catalog-or-topic | `bc/subscribe/subscribe.bc.md` | BC written |
| **BH-SUBS** | Signed-in subscriber lists their grants and deletes their own | `bc/subs/subs.bc.md` | BC written |
| **BH-NOTIFY** | When this publisher pushes a **public** work, covering grants get a “new post” notification. Never private | `bc/notify/notify.bc.md` | BC written |
| **BH-POST** | Publisher types a work, topic, private or not. Public goes on this catalog | `bc/post/post.bc.md` | BC written |
| **BH-DICTATE** | Publisher dictation becomes work text; analog audio kept as reference | `bc/dictate/dictate.bc.md` | BC written |
| **BH-SCAN** | Publisher document becomes the corresponding work; analog document kept as reference | `bc/scan/scan.bc.md` | BC written |
| **BH-MANAGE** | Publisher sees all of their works and manages them (including private or not) | `bc/manage/manage.bc.md` | BC written |
| **BH-STORE** | Publishers, subscribers, catalogs, works, analog refs, grants, invites, requests, and sessions survive a restart. A store failure is never an empty catalog, an empty panel, or a successful write | `bc/store/store.bc.md` | BC written |

A BC with no row here is wrong. A row with no BC is a planned slice, not an implemented one.

## 6. Surfaces

One website:

- **Publisher catalog** — invite or direct link. Blur or granted public feed, read, view/unlock, request. No compose.
- **Control panel** — publisher session. Write, dictate, scan, manage works, invite, accept/reject.
- **Subscriber list** — subscriber session. Grants they hold; delete own.
- **Header** — Blog, Resume, Projects (left); Login chooser (right): subscriber or publisher.

The Blazor app *is* the website. Operations in each BC are what the UI and tests observe on this host. Exact inputs and result codes live in each BC.

## 7. Unimplemented signal

Until a slice is done, every operation in its BC exists and is reachable, with real shapes, and returns this signal — never invented success data:

- HTTP: **501** with stable error `not_implemented`

Auth, authorisation, and input validation on that operation still run for real. A behaviour is not done while this signal remains on its surface.

## 8. Stack

Locked until this document says otherwise:

| Layer | Tech |
| --- | --- |
| Website | ASP.NET Core Blazor in this repo (`net8.0`, interactive server) |
| Domain | C# |
| Persistence | A database. Engine undecided until a spec |
| Publisher auth | Username + password or passkey |
| Subscriber auth | Email or phone + OTP |

Display name is the site. Code names stay `Blog.*`.

## 9. Guardrails

These bind humans and agents. If a change would violate them, update this file first, then the BC, then the code.

### Product invariants

- Two account types. Do not merge publisher and subscriber into one session. Do not let a publisher hold a grant on **their** catalog.
- Catalog is per publisher. Do not mix publishers on one feed. Find by invite or direct link; do not invent a directory.
- Header Login chooses subscriber or publisher. Subscriber Login **is** view/unlock (same OTP flow). Publisher Login is password or passkey.
- Unlock/view **always** runs on a catalog unless remember-this-device already has a subscriber session. Do not invent how long remember lasts.
- Grants exist only after invite or accepted request. Do not open-subscribe.
- Subscriber is read-only except deleting their own grants. Do not give them compose.
- Compose, analog capture, and work manage are the publisher control panel. Do not put them on the catalog.
- A work has exactly one topic and a title. Dictate/scan keep analog reference. Do not invent summarization.
- **Private** is panel-only and never notifies, regardless of topic or grant. **Public** is catalog-visible only to covering grants.
- Notification is **“new post”**, not the body, and only to covering grants of **that** publisher.
- Catalog feed is newest first. First page **5**; more on scroll. Do not invent another page size or clock.
- Resume and Projects are header tabs only. Do not invent their interiors.
- Do not invent comments, likes, follows, profiles, markdown, staff roles, or OAuth.

### Configuration security

- Never commit real secrets (SQL, mail, SMS, signing keys, passkey config). Local placeholders only.
- Fail to boot rather than start without the store connection.
- Do not log OTPs, passwords, passkey material, session secrets, or contact details in URLs.

### Operational

- Control-panel operations require **that** publisher’s session. Identity from the session, not a publisher id in the body.
- Subscriber read of full works requires a subscriber session **and** a covering grant.
- Private work, missing grant, or another publisher’s admin: **404** `not_found`, not **403**.
- Uniqueness that matters (publisher username; subscriber contact; grant of publisher + subscriber + catalog-or-topic) is in the store.
- A store outage is a failure. Never an empty feed, empty panel, or successful write/invite/accept.
- Schema changes ship as versioned migrations with the model change.

### Test

- Tests live in the solution. A dedicated test project may appear when the first BC lands. A run is self-contained.
- Tests declare the minor-behaviour ID they prove. Do not skip, filter, or delete tests to make a change look green.
- A done BC still returning `not_implemented` fails the build.
- Do not point tests at a shared or production database.

### Engineering path

- Implement the current iteration’s `BH-…` IDs only. Smallest change that lands that behaviour.
- Public operations are whatever the current BCs list. New behaviour needs a row in §5 and a BC before code.
- Do not invent a mixed catalog, a directory, self-subscribe, a remember-device TTL, compose on the catalog, or full-body notifications.
- After a product change: this file → BC → tests and code.

### Data and privacy

- Keep each work’s title, content, topic, date, publisher, private-or-not, and analog reference when captured by dictate or scan.
- Keep publisher username and credential material as required to sign in (never in logs).
- Keep subscriber contact, grants, invites, requests, and remember-this-device sessions.
- Minimize contact in logs. Public identifier of a catalog is the publisher username.

## 10. Deployment

The intended public site is the personal domain (for example jadadevoh.com). Destination and pipeline are not chosen yet. Skip a full strategy until they are.

Local development is the laptop.

## 11. Iterations

An iteration is a set of behaviours. Slices small enough to finish beat slices that look tidy.

**Iteration 0 — this document and the BCs**

- Product, domain, behaviour catalogue, guardrails, stack, BCs

**Iteration 1 — store and publishers**

- **BH-STORE**
- **BH-AUTH**

**Iteration 2 — gated catalog**

- **BH-UNLOCK**
- **BH-FEED**
- **BH-READ**

**Iteration 3 — grants**

- **BH-INVITE**
- **BH-REQUEST**
- **BH-SUBSCRIBE**
- **BH-SUBS**
- **BH-NOTIFY**

**Iteration 4 — control panel**

- **BH-POST**
- **BH-DICTATE**
- **BH-SCAN**
- **BH-MANAGE**

A BC is done when every case in it has a passing test, no unimplemented signal remains, and the documents and code agree. A behaviour is done when its BC is done. An iteration is done when every behaviour in it is done and this file says so.
