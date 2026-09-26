# Blog — product design

Method: [BCDD](https://github.com/adadevoh/bcdd/blob/main/bcdd.md). This file is the product. Do not copy `bcdd.md` into this repo. Observable behaviour lives in `bc/**/*.bc.md`. Mechanism lives in `*.spec.md` only when *how* needs agreeing.

If this file, a BC, and the code disagree: this file wins on product, the BC on behaviour, the spec on how. Amend the document first, then the code.

The Blazor template pages (Home / Counter / Weather) are scaffold, not this product.

## 1. What it is

This is a personal website and a **shareable diary**. A poster writes entries, marks each **private or not**, and shares the public ones. Viewers read the public diary and can subscribe to that poster’s posts — all of them, or filtered by topic — and receive a **“new post”** notification.

**Today:** one poster (the site owner). The site home *is* that diary. Login opens that poster’s control panel.

**Later (not this product until this file is amended):** anyone can sign up, get an account, compose, and share *their* diary. Viewers subscribe to that poster. Domain and BCs are already per-poster and per-diary so that change is an amendment, not a rewrite.

Two surfaces, same for one poster or many:

1. **Shareable diary** — public entries as a Twitter-like feed. Read, subscribe (all or by topic). No compose.
2. **Control panel** — that poster’s write, visibility, and manage-all for **their** diary. Viewers never see it.

Resume and Projects stay in the header as this site’s destinations; their interiors are unspecified.

Value: a diary you can share without making every entry public, and without becoming a social network.

## 2. What it is not

Do not build these unless this document is updated to allow them:

- Public signup, a second poster, or a second shareable diary (including paths like `blog/{someone}`). Leave **room** in the model (a diary belongs to a poster); do not **ship** a second account
- Compose, publish, or manage on the shareable diary. That work is the control panel
- A login wall on home. Home is the shareable diary. Login opens the control panel
- Password accounts, OAuth, or Azure AD B2C. Auth is OTP to email or phone
- Resume or Projects *content*
- Comments, likes, follows, profiles, search, images, markdown, or more than one tag per post
- Topics as their own product. Topics are tags on entries; a subscription may filter on one tag
- Private posts on the shareable diary, or in subscriber notifications
- Unsubscribe, double opt-in, an on-site inbox, or sending the full post body as the notification. The notification is **“new post”**
- A separate public API app or a Next.js app. This product is one website
- Staff roles, moderation queues, or a site-wide admin over other people’s diaries
- Features that belong to a later iteration, “while you are here”
- Invented clocks. The only specified page size is the first **5** posts on the diary feed

## 3. Domain

**Poster** — a person with an account and one diary. Identified by email; OTP may go to that email or phone. **Today** only the site owner is a poster: that contact is already known; completing OTP on it opens a session. A different contact does not create an account.

**Diary** — one poster’s shareable feed of **public** posts. **Today** there is one diary, and it is the site home. A later amendment adds a reachable name per diary; do not invent that name now.

**Post** — written content, one tag (topic), a post date, the poster, and **private or not**. Public posts appear on that poster’s shareable diary and trigger a “new post” notification to matching subscribers of **that diary**. Private posts are visible only to that poster, in the control panel, and never notify.

**Topic** — a tag on a post. Categorizes the entry. A viewer may subscribe to one tag on a diary.

**Viewer** — anyone without a session for that poster. Reads the shareable diary, may subscribe. No control panel.

**Subscriber** — an email *or* a phone, tied to **one diary**, optionally **one tag** on that diary. All: no tag, “new post” for each new public post on that diary. Filtered: only when the new public post has that tag. Delivery uses the channel they gave. A subscription is not an account.

Bounded contexts: **Reading**, **Publishing**, **Identity**, **Subscriptions**. A later BC belongs in one of these. Do not invent a fifth context.

## 4. Promotion (later amendment)

When this file is updated to allow multi-poster:

- A new contact that completes OTP **creates** a poster and their diary
- That poster composes and manages **only** their posts
- Their diary is shareable under a diary name
- Viewers subscribe to **that** diary (all or by topic)

Until then: signup stays closed. Persistence and operations are still “a poster’s diary”, never “the one global blog row.”

## 5. Flows

### Chrome

Header: **Blog**, **Resume**, **Projects** on the left; **Login** on the right. Opening the site lands on Blog — the (today, only) shareable diary. Resume and Projects are this site’s destinations; this document does not specify what they show.

**Login** is OTP. Success opens **that poster’s control panel**.

### Viewer

1. Land on the shareable diary. See **public** posts as a lazy-loading card feed. First page **5**, more on scroll, newest first.
2. Click a card: scrollable modal with the whole public entry.
3. Subscribe to **this diary**: all posts, or filter by one topic. Modal: email or phone; optional tag matched against tags already used on this diary’s **public** posts.
4. When a **public** post is written on this diary, matching subscribers receive a **“new post”** notification on the channel they gave.

Viewers do not see private posts. They cannot obtain a session unless they are the (today, only) poster.

### Poster (today: the owner)

1. Click **Login**. OTP to the owner’s phone or email. Completing it signs them in and opens the control panel. Any other contact: no session, no account.
2. Write a post, tag it, mark **private or not**. Public: it appears on the shareable diary and notifies matching subscribers. Private: control panel only, no notification.
3. See **all** of their posts (public and private) and manage them (including private or not).
4. They may open Blog and see what viewers see. They do not compose there.

## 6. Behaviours

Each behaviour has a stable ID (`BH-…`). That ID is the unit of a change list, an iteration, and a release. Minor behaviours live inside the BC, not here.

| ID | Behaviour | BC | Status |
| --- | --- | --- | --- |
| **BH-FEED** | Shareable diary: public posts as a card feed, 5 then more on scroll, newest first | `bc/feed/feed.bc.md` | BC written |
| **BH-READ** | Click a card: scrollable modal with the full public entry | `bc/read/read.bc.md` | BC written |
| **BH-SUBSCRIBE** | On a diary: subscribe with email or phone to all public posts, or filter by one tag used on that diary’s public posts | `bc/subscribe/subscribe.bc.md` | BC written |
| **BH-NOTIFY** | When a public post is added to a diary, matching subscribers of that diary receive a “new post” notification on the channel they gave | `bc/notify/notify.bc.md` | BC written |
| **BH-AUTH** | Poster signs in with OTP to their phone or email and reaches their control panel. Today only the owner’s contact may succeed; other contacts do not create an account | `bc/auth/auth.bc.md` | BC written |
| **BH-POST** | Signed-in poster, in their control panel: write a post, tag it, mark private or not. Public posts go on that diary | `bc/post/post.bc.md` | BC written |
| **BH-MANAGE** | Signed-in poster, in their control panel: see all of their posts and manage them (including private or not) | `bc/manage/manage.bc.md` | BC written |
| **BH-STORE** | Posters, diaries, posts, tags, and subscriptions survive a restart and are shared across instances. A store failure is never reported as an empty diary, an empty panel list, or a successful write | `bc/store/store.bc.md` | BC written |

A BC with no row here is wrong. A row with no BC is a planned slice, not an implemented one.

## 7. Surfaces

One website:

- **Shareable diary** — Blog, and the site home. Public feed, read, subscribe. No compose.
- **Control panel** — Login (OTP). Write, set private or not, manage that poster’s posts.
- **Header** — Blog, Resume, Projects (left); Login (right). Resume and Projects are unspecified.

The Blazor app *is* the website. Operations in each BC are what the UI and tests observe on this host — not a separate API project. Exact inputs and result codes live in each BC.

## 8. Unimplemented signal

Until a slice is done, every operation in its BC exists and is reachable, with real shapes, and returns this signal — never invented success data:

- HTTP: **501** with stable error `not_implemented`

Auth, authorisation, and input validation on that operation still run for real. A behaviour is not done while this signal remains on its surface.

## 9. Stack

Locked until this document says otherwise:

| Layer | Tech |
| --- | --- |
| Website | ASP.NET Core Blazor in this repo (`net8.0`, interactive server) |
| Domain | C# |
| Persistence | A database. Engine undecided until a spec |
| Poster auth | OTP to email or phone. Passwords, OAuth, and B2C are not in this product unless a later amendment says so |

Display name is the personal site. Code names stay `Blog.*`.

## 10. Guardrails

These bind humans and agents. If a change would violate them, update this file first, then the BC, then the code.

### Product invariants

- Today signup is closed: only the configured owner may obtain a session. Do not ship a second poster. Do not encode “the store can only hold one poster” as if promotion were impossible — a diary belongs to a poster.
- Two surfaces: shareable diary and control panel. Do not compose on the diary. Do not subscribe on the panel.
- Home is the shareable diary of public posts. Login opens the control panel.
- A session is a **poster** session and authorises **that poster’s** diary only. Identity comes from the session, not from a poster id in the URL.
- Auth is OTP to email or phone. Do not add a password, OAuth, or B2C.
- A post has exactly one tag. Tags categorize entries and may filter a subscription. Do not build a topics product.
- **Private** posts are not on the shareable diary and do not notify. **Public** posts are on that poster’s diary and may notify that diary’s subscribers.
- The notification is **“new post”**, not the post body, and never for a private post.
- Subscribe is to **a diary**: all, or one tag. Email *or* phone. Delivery uses that channel. A subscription is not an account. Do not add unsubscribe, double opt-in, or an on-site inbox until this file says so.
- The diary feed is newest first. First page **5** posts; more on scroll. Do not invent another page size or clock.
- Resume and Projects are header tabs only. Do not invent their interiors.
- Do not invent comments, likes, follows, profiles, search, images, markdown, staff roles, or moderation.

### Configuration security

- Never commit real secrets (SQL, mail, SMS, signing keys). Local placeholders only.
- Fail to boot rather than start without the store connection.
- Do not log OTPs, session secrets, or contact details in URLs.

### Operational

- Control-panel operations require that poster’s session.
- A viewer asking for a private post or the control panel does not get it. **404** `not_found`, not “forbidden with details.”
- Uniqueness that matters (poster email or phone; a subscriber’s contact plus diary plus optional tag) is enforced in the store, not only in application checks.
- A store outage is a failure. Never report it as an empty diary, an empty panel list, or a successful write.
- Schema changes ship as versioned migrations with the model change.

### Test

- Tests live in the solution. A dedicated test project may appear when the first BC lands. A run is self-contained.
- Tests declare the minor-behaviour ID they prove. Do not skip, filter, or delete tests to make a change look green.
- A done BC still returning `not_implemented` fails the build.
- Do not point tests at a shared or production database.

### Engineering path

- Implement the current iteration’s `BH-…` IDs only. Smallest change that lands that behaviour.
- Public operations are whatever the current BCs list. New behaviour needs a row in §6 and a BC before code.
- Do not invent public signup, a second diary’s URL, compose on the shareable diary, clocks, resume/projects interiors, or delivery of the full post body. The only specified page size is the first 5 posts.
- After a product change: this file → BC → tests and code.

### Data and privacy

- Keep each post’s content, tag, date, poster, diary, and private-or-not.
- Keep each poster’s contact (email, and phone if used for OTP) and that they have one diary.
- Keep who subscribed, to which diary, on which channel, with which optional tag.
- Minimize contact details in logs. Do not put email or phone in public paths. Today the public identifier is the site itself; later it is the diary name.

## 11. Deployment

The intended public site is the personal domain (for example jadadevoh.com). Destination and pipeline are not chosen yet. Skip a full strategy until they are.

Local development is the laptop.

## 12. Iterations

An iteration is a set of behaviours. Slices small enough to finish beat slices that look tidy.

**Iteration 0 — this document and the BCs**

- Product, domain, promotion note, behaviour catalogue, guardrails, stack, BCs

**Iteration 1 — shareable diary**

- **BH-FEED**
- **BH-STORE** (enough to keep a poster, a diary, and posts)

**Iteration 2 — read and subscribe**

- **BH-READ**
- **BH-SUBSCRIBE**
- **BH-NOTIFY**

**Iteration 3 — poster can get in**

- **BH-AUTH**

**Iteration 4 — control panel**

- **BH-POST**
- **BH-MANAGE**

A BC is done when every case in it has a passing test, no unimplemented signal remains, and the documents and code agree. A behaviour is done when its BC is done. An iteration is done when every behaviour in it is done and this file says so.
