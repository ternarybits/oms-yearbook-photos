# Old Mill School Yearbook Photo Collection — Plan

**Status:** Draft for team review
**Author:** Ted
**Date:** 2026-09-04
**Updated:** 2026-09-11 — yearbook team meeting

---

## How to read this document

**Part 1** is written for the yearbook team and school administration. It covers what the app does, what it deliberately does not do, how family privacy is handled, what it costs, and what decisions we need from the team. Part 1 can be shared on its own.

**Part 2** is the technical appendix for the people who build and maintain the app. It also feeds Claude Design (screens and states) and Claude Code (architecture and build order).

---

## Background and decisions from the September 11 team meeting

### How the yearbook has worked

Roni has been the Yearbook Lead for the past seven years. Kari is taking over this year. Roni's relationships across the school and district let her contact teachers, the principal, and other staff directly. That personal network will not transfer to Kari, and relying on each lead to rebuild it is not a sustainable approach as the role changes each year. Kari will handle communication and lead handoff outside the scope of this plan.

Roni collected photos onto her phone in an ad-hoc way: families emailed and texted photos or shared Apple Photos and Google Photos albums. She then uploaded them to [Jostens Yearbook Avenue](https://yearbookavenue.jostens.com/), the yearbook management system she has been using. There she organized photos into albums by event, photo submitter, and whatever other groupings helped her work.

Student names are pre-populated in Yearbook Avenue, but each photo needs to be manually tagged with the students who appear in it. Roni knew most of the students and did all of that tagging herself. Student identification and tagging are outside the scope of this plan. One hypothesis is to import the school yearbook photos, which are already clearly tagged with each child's name, grade, and teacher. This is a possibility to explore outside this plan, not a committed import or tagging feature.

As reported in the meeting, Yearbook Avenue supports cropping and resizing, but not other photo editing. Roni rarely edited photos; when she did, she edited them on her phone outside Yearbook Avenue and uploaded the edited versions again. These are descriptions of the team's existing workflow, not a new requirement to build editing into the upload app.

### Decisions recorded

- **The PTA is responsible for the yearbook.** The upload page will have a home on the PTA's site/domain, likely `yearbook.oldmillpta.org`. Ted will handle the exact subdomain and DNS setup outside the scope of this plan.
- **Temporary hosting:** use an assigned Cloudflare Worker URL (`*.workers.dev`) until the PTA subdomain is ready.
- **District-policy disclaimer:** the team agreed to add a disclaimer on the upload page saying that uploaded photos are covered by the school district's photo release policy. Draft wording and policy-analysis status are in §6.

Historically, the team has given limited attention to data privacy. The disclaimer is this year's agreed step; it does not establish what the policy permits or demonstrate compliance. The relevant policies in the [Mill Valley School District Board Policy Manual](https://simbli.eboardsolutions.com/Policy/PolicyListing.aspx?S=36030331) have been reviewed. Findings, source links, and remaining evidence gaps are in [the district-policy analysis](DISTRICT-POLICY-ANALYSIS.md), summarized in §6.

The upload app remains the proposed central collection point, with Gumnut for review and organization and a file export to Yearbook Avenue for production. No automated integration or transfer of student tags into Yearbook Avenue has been agreed.

---

## Decisions to confirm before building

These are the choices that affect how the yearbook team works, so they are worth confirming before building. Purely technical decisions — file size limits, how uploads are handled — are settled and live in Part 2.

| # | Decision | What it means for the team |
|---|---|---|
| **D1** | **The team shares one Gumnut login** to review photos | Everyone uses the same username and password for now, so there is no record of who did what. Gumnut is adding per-person logins soon; when it arrives, each team member gets their own and nothing about the app changes. |
| **D2** | **Each way of sharing the link gets its own access code** | The code is built into the QR codes and newsletter links, so nobody ever types it. Giving the posters, the newsletter, and each class parent a different code means one can be switched off without disrupting the others — useful if a link ends up somewhere it shouldn't. |
| **D3** | **Next school year is not being designed yet** | Albums and storage are settings, not code, so whether next year reuses this library or starts a fresh one can be decided when it arrives rather than guessed at now. |
| **D4** | **What happens to the photos after the yearbook is printed** | Submissions are for yearbook consideration only. Decide the retention period and responsible owner, subject to applicable record-retention obligations (§6), and disclose the decision before collection. Do not assume the whole library can be deleted immediately after printing. This is the decision that is hardest to revisit later, so it is worth settling first. See §6. |

---

# Part 1 — For the yearbook team

## 1. Goals

1. **Make it effortless to contribute photos.** Scan a QR code at a school event, pick photos from the camera roll, tap upload. No account, no password, no app to install.
2. **Get photos to the team already organized.** Photos arrive tagged with who sent them and which event/album they belong to, so the team isn't sorting an undifferentiated pile.
3. **Protect family privacy by default.** No parent can browse the collection — not other families' photos, and not their own. The app collects; it never displays.
4. **Be reusable.** The library and album list are configuration, not code, so a future team can point the app at new albums without a developer. How next year is actually structured is deliberately left open (D3).
5. **Be maintainable by someone other than the person who built it.** Public code repository, documented setup, and accounts owned by an institution rather than an individual — so the maintainer role can pass to any parent with a technical background.

## 2. Scope and non-goals

**This app does two things: collect photos and get them safely into storage.** The proposed review and organization workflow happens in Gumnut, the photo library the app uploads into. Yearbook layout and printing remain in Jostens Yearbook Avenue, outside this app.

Not built in this app:

| Not in this app | Where it happens instead |
|---|---|
| Browsing and searching the collection | The Gumnut web app, where the team signs in |
| Comments, notes, and favorites on photos | Gumnut — the team can use these while selecting |
| Organizing, culling, and grouping photos | Gumnut |
| Yearbook layout, page design, printing | Jostens Yearbook Avenue, out of scope here |

Not built at all:

- **A public gallery.** There is no page, anywhere, where one family can see another family's photos.
- **Parent accounts or logins.** Contributing takes no sign-in.
- **Any destructive action from the upload app** — no delete, no edit, no overwrite, for anyone. See §5 for the reasoning; it is a design principle, not a missing feature.
- **Any use of the photos outside the yearbook.** Uploading a photo puts it in front of the yearbook team for possible inclusion in the book. It is not permission to use it in the newsletter, on the school website, in fundraising materials, or anywhere else. A broader use would have to be asked for separately (§6).

**Getting the photos back out** is already handled: Gumnut can download a whole album, or any selection, as a single ZIP of originals — from the web app, or via the API. So when it is time to lay out pages, the yearbook team exports the album and uploads the selected files to Yearbook Avenue. Student identification, tagging, and any transfer of tags are outside this plan; no transfer capability is assumed. See §15.6.

## 3. Who uses this

| Role | Who | What they do | How they access it |
|---|---|---|---|
| **Contributor** | Any parent, family member, or staff member | Uploads photos and clips from a phone or laptop | QR code or link. No login. |
| **Yearbook lead** | Kari | Owns the collection: creates albums, prints QR codes, reviews what comes in, handles removal requests | Signs into Gumnut (D1) |
| **Parent volunteers** | Several, assisting the lead | Review and select photos for pages | Signs into Gumnut (D1) |
| **App maintainer** | Ted, this year | Keeps the site running, adds albums when asked | Cloudflare + GitHub accounts |

The lead and the volunteers currently **share one Gumnut login** (D1). That is the main practical cost of the shared-account approach: no record of who changed what. It resolves when Gumnut ships per-person library access.

**The maintainer role is meant to be handed off.** Ted holds it this year and may again next year, but the job is deliberately scoped so that **any parent with a technical background can take it over** — add an album to a config file, deploy, occasionally check storage. That constraint is why the stack is small and boring (§12), why nothing depends on its original author, and why the accounts belong to an institution rather than a person (§8). A maintainer who has to understand a clever architecture is a maintainer the project cannot replace.

## 4. What a contributor can do (supported user stories)

**S1 — Upload photos and clips from an event.**
A parent scans the QR code posted at the Lapathon. The page opens already knowing the upload is for the Lapathon album. They tap "Choose photos or videos," select 12 photos and a short clip from their camera roll, type their name, add a note ("Third graders on the back stretch, ~2pm"), check the rights box, and tap Upload. A progress indicator shows each file completing, and the page confirms how many went through.

**S2 — Upload without a QR code.**
A parent follows the link from the school newsletter or a class-parent email. That link carries the same access code the QR codes use, so the form opens normally — they just choose which album(s) apply from the published list rather than having one pre-selected. Someone who types the bare subdomain with no link sees a short page explaining where to get the link (newsletter, class parent, or the yearbook lead's email) rather than an error.

**S3 — Put one batch into more than one album.**
Photos from a single afternoon that belong in both "Field Day" and "Fifth Grade" are added to both, in one submission.

**S4 — Send a large batch.**
A parent who shot 200 photos at Field Day selects them all and uploads in one go. The browser works through them steadily; nothing caps the count.

**S5 — Understand exactly who can see their photos.**
Plain-language privacy text is on the upload page itself, not buried in a link, including the fact that this is a one-way submission.

**S6 — Send more photos later, from any device.**
A parent can come back as many times as they like, from a phone or a laptop, with no login and nothing to remember. Each visit is a fresh submission. Sending the same photo twice is harmless — Gumnut keeps one copy (§15.1).

**S7 — Get a clear answer when the window has closed.**
After the submission deadline, the page explains that collection is closed and who to contact.

## 5. What a contributor cannot do, and why

| Not supported | Reason |
|---|---|
| See other people's photos | Core privacy commitment |
| **See their own photos after uploading** | The app is upload-only. There is no gallery and no history — the originals stay in the contributor's camera roll, where they already were. |
| **Delete a photo after uploading** | **By design, not a v1 shortcut.** Deleting is destructive and irreversible from the parent's side, so it must require *proof that you are the person who uploaded it*. "Same device" is not proof — phones get handed to kids, shared, lent, and resold. Without a real login there is nothing to check, so the app offers no delete at all. Email the yearbook lead (§7). |
| **Edit the name or note after submitting** | Same reasoning. Editing someone else's attribution is a smaller harm than deleting their photo, but it rests on the same unmet identity claim. |

**The principle behind this whole table:** the app performs **no action on a photo after it is uploaded** — not viewing it, not editing it, not deleting it — because it cannot verify who is asking. "Same device" is not proof of identity, and any feature built on it becomes an argument for the next one: a parent who can *see* their past uploads will reasonably ask to delete one. Rather than build that staircase and stop partway up, the app does not take the first step. Upload is a one-way door. If real accounts are ever added, that is the point at which any of this becomes discussable.

**This is the most important thing to communicate to parents,** and the app says it directly: *"Your photos go straight to the yearbook team. Nobody browsing this site can see them."*

## 6. Privacy

### Where the photos are stored

Uploads go to **Gumnut**, the photo library the yearbook team works in. The upload page links to Gumnut's privacy policy — **<https://gumnut.ai/privacy>** — for provider practices. That policy does not by itself establish district authorization or satisfy an applicable student-record storage contract. The district-policy analysis below identifies that distinction; provider agreements have not been audited in this review.

The rest of this section covers only what is **specific to this app and this project** — the things Gumnut's policy cannot know about.

### Specific to this app

- **Nothing is public.** There is no public gallery, and no page anywhere that displays the collection.
- **No parent-to-parent visibility.** The app never shows one family's photos to another family. It never shows a contributor their own photos either — it collects and does not display (§5).
- **What the app asks for:** the photos and clips, the uploader's name as typed, an optional note, and which albums they chose. It does not ask for email, phone number, or student names.
- **Rights attestation.** The upload form carries a required checkbox: *"I have the right to share these photos, and I understand they may appear in the school yearbook."* Contributors routinely upload photos containing other people's children, so this is stated plainly rather than buried. The exact wording needs the team's sign-off (§10).
- **What uploading does *not* grant.** The scope of that checkbox is deliberately narrow, and the rest of the project has to respect it. Uploading a photo stores it so the yearbook team can consider it for the book. It is **not** consent to use it in any other school material — not the newsletter, not the website, not fundraising, not social media. A parent handing over candid shots of other people's children for a yearbook should not thereby be granting the school an open-ended license, and if a broader use is ever wanted, the right move is to ask for it then rather than to have quietly collected it now.

### Specific to this project

- **District-policy review completed September 11.** See [District policy analysis](DISTRICT-POLICY-ANALYSIS.md) for the relevant BP/AR provisions, revision dates, source links, and evidence gaps. The manual contains directory-information, website, student-record, PTA, volunteer, retention, and AI rules; the review has not established a single release that automatically authorizes every yearbook submission or use.

- **Disclaimer decision retained; wording clarified.** Proposed upload-page text for review:

  > Photos and videos submitted here are for consideration in the Old Mill School yearbook. Their use is subject to applicable Mill Valley School District photo-release permissions and privacy policies. Uploading does not override a family's release restrictions. Submissions go to the PTA yearbook team for private review and are not displayed on this site.

  Link “privacy policies” to the [district policy manual](https://simbli.eboardsolutions.com/Policy/PolicyListing.aspx?S=36030331), and add the actual family photo-release form when obtained. Keep the uploader's rights checkbox separate: it does not grant permission on behalf of every pictured child's parent or replace district notice/consent requirements.

- **Collection and publication are different.** BP/AR 5125.1 governs district directory-information release, annual notice, and written opt outs. BP 1113 requires prior written consent for an individual student's photo accompanied by their name or other personally identifying information on district/school websites. Neither establishes the exact printed-yearbook permission for this project. The app stays upload-only; private review does not authorize later public sharing. The actual family release form and its yearbook scope remain to be confirmed.

- **PTA ownership does not settle student-record status.** BP/AR 1230 treats the PTA as a separate school-connected organization. If the collection is maintained for the district, BP/AR 5125 student-record access and third-party contract requirements may apply to its storage and processing. Confirm the existing authority, record classification, and applicable agreements for the Cloudflare → Gumnut → Yearbook Avenue data path. This is an unresolved applicability question, not a finding that these vendors are prohibited or approved. The shared login in D1 remains a proposal; it provides no per-person audit trail and should be assessed against applicable access requirements.

- **Automated processing.** BP 0441's AI privacy and accountability principles are relevant to Gumnut's proposed face grouping. This review does not establish approved processing or vendor terms, and it does not impose a new no-AI design. The portrait-import hypothesis and student-tagging procedures remain outside this plan.

- **Retention remains a decision, subject to applicable rules.** Yearbook consideration is the stated project purpose. Choose a defined retention period and responsible owner and disclose them before collection. If the collection is district records, AR 3580 and AR 5125 classification/retention requirements may limit when it can be destroyed. The reviewed policies do not specify a universal deletion date for yearbook submissions. Consider originals, derived media, exports, and provider copies when settling the schedule; deleting from Gumnut does not establish deletion from Yearbook Avenue or downloaded files.

## 7. Removal requests

A contributor or parent/guardian with a removal concern emails the yearbook lead at an address published on the site. Target: respond and review the request within one week. Removal from Gumnut, exported copies, and publication use must follow the applicable release and retention requirements; the app must not promise immediate deletion of every copy or removal from already printed books. This is a human process in v1, deliberately — the volume does not justify building anything, and a real person reading the request is the better outcome anyway.

## 8. Ownership and continuity

For the app to outlive whoever builds it:

- **The target is institutional ownership** of the Cloudflare, GitHub, and Gumnut accounts, under an institutional address rather than anyone's personal email.
- **PTA home confirmed September 11:** the PTA is responsible for the yearbook, and the upload page will live on the PTA's site/domain, likely **`yearbook.oldmillpta.org`**. Ted will handle the exact hostname and DNS setup outside this plan.
- **Sequencing:** start at an assigned **`*.workers.dev`** URL until the PTA subdomain is ready. Domain setup is outside this plan; choosing the PTA domain does not mean institutional account ownership is already in place. Temporary personal accounts must still be transferred to the PTA for continuity.
- Note that the Gumnut account is the one that matters most: it holds the photos and the team's record of who appears in them (§15.6). Domain and code hosting can be moved cheaply at any time; the photo library is the thing with gravity.
- Credentials live in a password manager the owning body controls.
- The code is public on GitHub. Secrets are never in the code.
- A `README` documents how to change what the app collects into: create an album in Gumnut, add it to the config file, deploy, print a QR code. That is the whole recurring operation.
- The runbook names **who watches remaining photo storage** (§9). Student identification and tagging procedures are outside this plan.

## 9. Costs

| Item | Cost | Notes |
|---|---|---|
| App hosting (Cloudflare Workers) | **$0, likely** | The free tier gives 100,000 requests/day and the same 100 MB upload limit. Its 10 ms CPU cap is *CPU only* — time waiting on the network doesn't count, and a streaming upload is nearly all waiting. Budget $5/mo as a fallback if M1 shows otherwise. |
| Domain | $0 | A subdomain of the PTA domain the school community already owns |
| Code hosting (GitHub) | $0 | Public repository |
| Photo + video storage (Gumnut) | **$0 expected** | Gumnut's default 10 GB is expected to cover the year, and the limit can be raised if needed. Video is the swing factor: 500 photos ≈ 2 GB, but 150 phone clips could reach the cap on their own. |
| **Total** | **$0 expected**, $5/mo worst case | Nothing here requires a purchase order |

**The whole thing is expected to run at $0.** The one number to watch is storage, because accepting video makes it unpredictable — a single minute of phone video costs more than a hundred photos, and nobody can say in advance how many clips parents will send.

**It matters because hitting the cap fails loudly and all at once.** Gumnut rejects *every* upload with a `507` when the library is full — including duplicates — so the symptom is every parent failing simultaneously, most likely the evening after a big event. Raising the limit fixes it, but only if someone notices. The yearbook lead should check remaining capacity after each major event; that is the one recurring operational duty this project has.

## 10. What we need from the school and team

- [x] **PTA domain direction and temporary hosting agreed September 11** — PTA home, likely `yearbook.oldmillpta.org`; assigned `*.workers.dev` URL until ready (§8).
- [ ] **Album list** for the year (e.g. Lapathon, Field Day, Class Photos, Fifth Grade, Winter Concert…)
- [ ] **Submission deadline** for the collection window
- [ ] **Brand inputs:** school colors (hex values), logo file, mascot artwork, and any font or style guidance. *We will not guess these.*
- [ ] **Approval of the app's privacy language** in §6, and specifically the **wording of the rights-attestation checkbox** — it is the one sentence contributors actually agree to, and it is deliberately scoped to the yearbook alone
- [x] **Upload-page district-policy disclaimer agreed September 11** (§6).
- [ ] **Finalize disclaimer wording and policy link** — review the proposed sentence in §6 alongside the rights attestation.
- [x] **District-policy analysis completed** (§6) — [findings and evidence gaps](DISTRICT-POLICY-ANALYSIS.md).
- [ ] **Resolve policy applicability and release evidence** — actual family photo/yearbook release, existing PTA/yearbook authority and record classification, applicable provider/access requirements, and permitted automated processing. The policy text alone does not establish these facts; this does not add communication, tagging, or DNS work to the plan.
- [ ] **Retention decision** (§6) — **choose a retention period consistent with the collection's record classification and applicable obligations.** The yearbook-only purpose, any continued need, responsible owner, and retention period must be reflected on the upload page *before* collection starts. Decide this early; it is much harder to add after the fact.
- [ ] **The yearbook lead's email address** for the site and for removal requests

## 11. Build order

Each stage is independently useful and shippable.

**Stage 1 — Upload works.** Themed page, photo picker, name + note fields, rights checkbox, uploads land in one Gumnut album. Privacy text and the agreed district-policy disclaimer (§6). Access code and collection window enforced from the start — both are a few lines each (§16), and Stage 1 is already collecting real photos of real children at a public URL, so it should not run wide open while the rest is built. Deployed to a temporary address. *This completes the technical intake path; use with real student photos also depends on resolving the applicable release and data-handling questions in §6.*

**Stage 2 — Albums and QR codes.** Multi-select album list, QR links that pre-select an album, a printable QR sheet for the yearbook lead. Use the PTA subdomain when Ted has set it up outside this plan; that setup is not a milestone requirement.

**Stage 3 — Deadline handling.** Collection window enforced, with a clear closed-for-the-year page.

**Stage 4 — Handoff.** The remaining abuse protections (Turnstile, rate limiting), documentation, and transfer of account ownership to the team, so the app can run without its original author.

**Later, if wanted:** a team gallery in the app, real parent accounts, and the uploader-initiated deletion those would unlock (§5).

---

# Part 2 — Technical appendix

## 12. Architecture

```
Contributor's browser           Cloudflare Worker              Gumnut
─────────────────────           ─────────────────              ──────
static page + JS   ──POST──▶    /api/upload
 (one file per      raw bytes    ├─ check access code hash
  request, 2–3      as the body  ├─ check collection window
  concurrent)       + headers    ├─ verify Turnstile token
                                 ├─ check magic bytes, size, type
                                 ├─ stream body ───────────────▶ POST /api/assets
                                 │   (§14.1 — never buffered)
                                 ├─ set metadata.description ──▶ PATCH asset (201 only)
                                 ├─ albums.assetsAssociations.add ▶ album membership
                    ◀──JSON──    └─ {ok, isDuplicate}
```

The browser keeps nothing. Once a file is uploaded and confirmed, the app has no further relationship with it.

**Key properties:**

- The Gumnut API key exists **only** as a Worker secret. It is never sent to the browser and never in the repository.
- The Worker is a **streaming proxy, not a store**. File bytes pass through it to Gumnut without ever being buffered or written anywhere (§14.1). It never serves image bytes back out — the app has no gallery.
- The Worker's only persistent state is **rate-limit counters** (KV or a Durable Object, §16), keyed by `deviceId` and IP. Nothing there identifies a contributor, and it expires on its own.
- The browser stores exactly one thing: a random `deviceId` UUID, used only for rate limiting.

**Stack recommendation:** Cloudflare Workers with static assets (Worker + `[assets]` binding), Hono for routing, `gumnut-sdk` (TypeScript) for the Gumnut calls, plain TypeScript + a light CSS approach for the frontend. Deliberately boring and small — a future maintainer should be able to read the whole thing in an afternoon. No framework is required for a single-page form.

## 13. Configuration

Everything the team might change lives in one committed config object. Library and album IDs are not secrets and belong in the public repo.

```ts
// src/config.ts
export const CONFIG = {
  displayName: "Old Mill School Yearbook",
  libraryId: "lib_...",
  closesAt: "2027-03-15T23:59:59-07:00",
  contactEmail: "yearbook@oldmillpta.org",
  albums: [
    { slug: "lapathon",      name: "Lapathon",      albumId: "album_..." },
    { slug: "field-day",     name: "Field Day",     albumId: "album_..." },
    { slug: "fifth-grade",   name: "Fifth Grade",   albumId: "album_..." },
  ],

  // Access codes, by SHA-256 hash — see §16. The codes themselves are never
  // committed; only their hashes, which are safe in a public repo.
  accessCodes: [
    { label: "Lapathon posters",       hash: "a3f1…", active: true },
    { label: "October newsletter",     hash: "9c22…", active: true },
    { label: "Back-to-school night",   hash: "51de…", active: false }, // revoked
  ],
} as const;
```

**On future years (D3):** deliberately not designed now. Adding albums to this same library is an edit to `albums`. Moving to a separate library per year is an edit to `libraryId` plus a re-scoped API key. Both stay available because nothing outside this file knows the year. Do not build multi-year machinery until someone actually needs it.

**QR / deep links** use slugs, never raw Gumnut IDs:
`https://yearbook.oldmillpta.org/?a=field-day,fifth-grade&c=<access-code>`

## 14. Upload contract

`POST /api/upload` — **exactly one file per request**, sent as the **raw request body** rather than as a multipart form (§14.1 explains why). The browser runs a small concurrency pool (2–3 in flight) over the selected files.

Metadata travels in **request headers**, not the body and not the query string: `X-Uploader-Name`, `X-Comments`, `X-Albums` (comma-separated slugs), `X-Device-Id`, `X-Device-Asset-Id`, `X-File-Created-At`, `X-File-Name`, `X-Turnstile-Token`, `X-Access-Code`. `Content-Type` and `Content-Length` describe the file itself.

**Headers rather than the query string, deliberately.** Cloudflare's request logs and analytics record the full request URL but not arbitrary headers. A contributor's name and free-text comment are exactly the personal data §6 promises to keep to the yearbook team — putting them in a URL would copy them into logging infrastructure nobody in this project controls or thinks about. Header values must be encoded (RFC 2047 or base64) since names and comments can contain non-ASCII characters. The *page* URL is a different matter: `?a=…&c=…` there carries only album slugs and a semi-public access code, both of which are printed on posters anyway.

**Validate both text fields server-side**: cap `X-Uploader-Name` at 100 characters and `X-Comments` at 1,000, strip control characters, and reject anything longer rather than truncating silently. These strings are written into Gumnut where the team will read them.

Response: `{ ok: true, isDuplicate }` or a typed error. **No asset ID or URL is returned to the browser** — it has no use for either, and not returning them keeps the client incapable of referring to a stored photo at all.

**Limits:** **100 MB per file** — the Cloudflare account request-body ceiling, and the real constraint — and **no cap on the number of files** in a submission.

**Why one file per request:** constant Worker memory, a clean per-file progress bar, per-file retry without redoing the batch, and a natural mapping onto Gumnut's rate limiter. It is also what removes any cap on batch size — a parent can send 300 photos from an event and the browser simply works through the queue.

### 14.1 The upload must stream — do not buffer

This is the requirement that sets the 100 MB per-file limit, and it is the single easiest thing to get wrong.

**What the SDK actually does** (verified in `src/internal/uploads.ts`):

- `addFormValue` appends a `File`/`Blob` **by reference** — `form.append(key, value, name)`. For a `File` input the SDK adds **no copy**. This is the path to use.
- Passing a `Response` instead hits `makeFile([await value.blob()], …)` — **buffers the whole body.** Same for an async iterable. Avoid both on Workers.

So the SDK is not the problem. **`request.formData()` on the inbound side is** — it materializes the uploaded file in isolate memory before the SDK ever sees it. With a 128 MB isolate shared across every concurrent request, that is the real ceiling.

**Therefore the Worker must never call `request.formData()` for the file.** Instead:

1. The browser `POST`s the **raw file as the request body**, with metadata in the request headers listed in §14 (`X-Uploader-Name`, `X-Device-Asset-Id`, `X-Albums`, …) rather than as multipart fields.
2. The Worker builds the outbound `multipart/form-data` body as a `ReadableStream`: a text prelude carrying the scalar fields and the file part header, then **`request.body` piped straight through**, then the closing boundary.
3. It sends that with a raw `fetch` to `POST /api/assets` with `Content-Type: multipart/form-data; boundary=…` and the `Authorization: Bearer` header — bypassing `assets.create` for this one call.

Memory then stays constant regardless of file size, and the limit becomes Cloudflare's account-level request-body cap: **100 MB on Free/Pro, 200 MB on Business.**

The SDK is still used normally for the two *other* calls (`updateAsset` and album association) — small JSON requests where its ergonomics are worth having. Only the upload call is hand-rolled, and it should be one well-commented function with a test.

**Because it is hand-rolled, that function does not inherit the SDK's retry behavior** (§15.3). The upload is both the most expensive call in Gumnut's rate limiter (20 tokens) and the one most likely to be throttled during an after-event rush, so it must handle `429` itself: honor `Retry-After`, back off exponentially with jitter, and cap at two or three attempts before surfacing a retryable error to the browser. It must also recognize `507` (library full) and fail immediately without retrying, since retrying cannot help.

This also pairs with gap **G3**: if Gumnut ships scoped upload tokens, this hand-rolled function disappears entirely and the browser uploads directly.

**Per-file server-side sequence:**

1. Stream the file to `POST /api/assets` per §14.1, reading the HTTP status to distinguish new (201) from duplicate (200) — see §15.1.
2. `client.assets.updateAsset(id, { description })` where description is the attribution block (§15.1).
3. For each selected album: `client.albums.assetsAssociations.add(albumId, { asset_ids: [id] })`.

Steps 2 and 3 are best-effort: if they fail, still return `{ ok: true }` to the browser, and log the asset ID together with the attribution and album slugs that failed to attach. A photo that landed without its note is recoverable — the team can fix it in Gumnut from that log line. A photo rejected because its note failed to save is simply lost.

`file_created_at` and `file_modified_at` are required by the API — use the browser's `File.lastModified` for both. Gumnut derives real capture time from EXIF server-side, so this is only a fallback.

`deviceId` is a UUID generated once per browser and kept in `localStorage`; `deviceAssetId` is a fresh UUID per file. These exist to populate Gumnut's provenance fields and to support per-device rate limiting (§16) — **they are not an identity mechanism** and must never be used to grant access to anything. This is the only thing the app stores in the browser.

Accept `image/*` plus explicit `image/heic,image/heif` in the file input so iOS offers the original HEIC rather than transcoding. **Gumnut stores and renders HEIC natively** (confirmed: `pillow-heif` registered server-side, HEIC decode handled at the CDN edge).

### 14.2 Video

Also accept `video/*`. Nothing about the upload path changes — the streaming approach in §14.1 is indifferent to what the bytes are, which is precisely why allowing video is cheap now and would have been expensive under a buffering design.

- Gumnut stores video natively and generates poster frames (`thumbnail_image` / `small_image` / `preview_image`), so clips appear like photos for the team in Gumnut, with no work on our side.
- The CDN serves video by passthrough with HTTP Range support, so team playback seeks properly.
- **Enforce the 100 MB cap client-side first**, before the upload starts — `File.size` is known immediately. Failing a 300 MB clip after uploading 100 MB of it over school-parking-lot cellular is a genuinely bad experience. Check server-side too, via `Content-Length`, since the client check is only a courtesy.
- iPhone clips arrive as HEVC in a `.mov`. These upload and store fine.
- **No transcoding, client- or server-side.** Browser-side video transcoding (WebCodecs, ffmpeg.wasm) is slow, memory-hungry, and unreliable on exactly the mobile Safari most parents will use. Recorded as a stretch goal against the 100 MB cap; not planned. If clip size becomes a real complaint, the better answer is gap **G3** (direct upload) rather than transcoding on a phone.

## 15. Gumnut behaviors that shape the design

These were verified against the SDK, API docs, and backend source. They are the non-obvious constraints Claude Code should not have to rediscover.

### 15.1 Duplicate uploads return the *existing* asset

`POST /api/assets` dedupes on SHA-256 within a library. A byte-identical re-upload returns **200 OK with the existing asset** (a new one returns 201) and *keeps the existing asset's metadata*.

Two consequences:

- **Two parents can upload the same photo** (AirDropped between families, forwarded in a group chat). A naive `updateAsset({description})` would **overwrite the first parent's attribution.**

  **The rule: write the description only on `201`, never on `200`.** The create call's own HTTP status says which happened, so first-uploader attribution is preserved and the second submission still succeeds and still joins its albums. The second uploader's name is simply not recorded against that file — an acceptable trade, and the first contributor arguably has the better claim anyway.

  This deliberately avoids read-then-append, which would have meant retrieving the asset to merge descriptions. **The result is that the app issues no read calls at all** (see G2 in §17) — a property worth protecting in review, not just a happy accident.

  Description format — one block, written once, structured enough to parse and readable in Gumnut's UI:
  ```
  Uploaded by: Jane Smith (2026-10-12)
  Notes: Third graders on the back stretch, around 2pm
  ```
- **A photo the team has trashed will silently "succeed" on re-upload** and return the trashed asset without restoring it. The parent sees success; the photo does not reappear. Acceptable, but the team should know: trashing is not the same as blocking.

### 15.2 Signed asset URLs are unguessable but do not expire

`asset_urls` on an asset response are HMAC-SHA256-signed URLs to `assets.gumnut.ai`, served by Gumnut's CDN Worker. The browser can fetch them directly with no API key. They carry **no expiry** — anyone who has the URL can view that image indefinitely.

**The app never hands one of these to a browser** — with no upload history, nothing renders an uploaded photo back to a parent. So this is documented here as context for the team's own use of Gumnut (a signed URL pasted into an email is a permanent public link to that photo), not as a property this app relies on.

### 15.3 Rate limits are per *account*, shared by every parent

Weighted token bucket: **400 capacity, refills 100/second.** An upload costs **20 tokens** — roughly 5 uploads/second sustained, 20 in a burst. Because the app uses one API key on one account, **every parent uploading at once shares this budget.** Realistic worst case is the evening after a big event.

Mitigations: 2–3 concurrent uploads per browser (not 10); retry on `429` with jitter rather than surfacing a failure; watch `X-RateLimit-Remaining`. **Note that the SDK's built-in backoff does not cover the call that matters** — the upload is hand-rolled precisely so it can stream (§14.1), so it must implement its own `Retry-After` handling. That is called out as a requirement in §14.1. This is the app's scaling ceiling and should be revisited if the school grows the program.

### 15.4 No description or album at create time

There is no way to set a description or album membership in the create call — hence the three-step sequence in §14. Listed as a Gumnut gap.

### 15.5 Storage cap returns 507, checked before deduplication

`POST /api/assets` checks storage limits *first*, so an account at its cap returns **`507 Insufficient Storage` even for a byte-identical duplicate** that would consume no new storage. The Worker should map `507` to a distinct, non-retryable client error with its own copy (§19.7) — retrying makes it worse, and every parent hits it simultaneously. The lead-facing side of this is in §9.

### 15.6 Gumnut review and export — we build none of it

Worth knowing when writing the handoff docs, because this is where the yearbook team actually spends its time:

- **Browse, search, and view** the collection, including video playback with seeking.
- **People features (context only; identification and tagging are out of scope).** Gumnut detects and groups faces automatically. The school-photo import hypothesis is recorded in the meeting background; no import implementation or tagging procedure is required by this plan. Potential uses include:
  - **Coverage.** *Do we have enough photos of each child?* is a list rather than a squint through thumbnails.
  - **Release-policy handling (§6), subject to the applicable permissions.** A marker such as `(DNR)` is a proposed review aid, not an agreed compliance mechanism. The team must verify identities and applicable release preferences before selecting photos; missed or incorrect face tags remain possible. The policy analysis identifies applicable rules and remaining evidence gaps; the team's identification and tagging procedures remain outside this plan.
- **Bulk download** — select any set of photos, or a whole album, and download them as a ZIP of originals. Available in the Gumnut web app and through the API. (Mechanically the zipping happens in the browser rather than on a server, which makes no difference to how it is used.)
- **Trash** — soft delete, recoverable for 90 days, which is the safety net behind removal requests (§7).

## 16. Abuse prevention

No login means no strong identity, so the goal is raising cost, not perfect prevention. In layers, cheapest first:

1. **Access codes in the link** (`?c=…`), checked server-side. Keeps the page out of drive-by and crawler traffic.

   **Multiple codes, individually revocable.** Rather than one shared secret, issue a distinct code per distribution channel — one for the Lapathon posters, one for the October newsletter, one per class parent. Three things follow: a leaked code can be killed without disrupting everyone else, the label on the code tells you *which channel* leaked, and codes can be retired when their event is over.

   **How they live in a public repo.** The config holds only the **SHA-256 hash** of each code plus a label and an `active` flag (§13). The Worker hashes the submitted code and compares. The codes themselves are never committed, so the repo stays public and revocation is a one-line config edit plus a deploy — no secret rotation, no separate store.

   This works because the codes ride in URLs and QR codes rather than being typed, so they can be long and fully random (16+ random characters). Do not shorten them into something human-memorable; that is what would make the published hashes worth attacking.

   *If instant revocation ever matters more than simplicity,* move the list to a KV namespace so it changes without a deploy. Not worth it at this scale.
2. **Cloudflare Turnstile** on the form — invisible for nearly all real parents, blocks scripted submission.
3. **Rate limiting per `deviceId`, and a burst limit per IP** in the Worker (Cloudflare Rate Limiting rules, or a Durable Object / KV counter). Suggested starting points: **500 files per device per day**, and **60 uploads per minute per IP** rather than a daily per-IP cap.

   Both numbers need to clear the legitimate cases, or the limit becomes the bug. S4 describes a parent sending 200 photos in one sitting and §14 describes 300 from an event, so any per-device daily cap below ~500 would reject exactly the user we designed for. The per-IP rule is worse: at an indoor event on the school's guest Wi-Fi, *every* parent shares one egress IP, so a daily per-IP cap would cut off the whole gym partway through the Winter Concert. A per-minute burst limit still stops a script while leaving a crowd of real parents alone.
4. **Server-side file validation:** magic-byte check (don't trust `Content-Type` or extension), size cap (§14.1), and reject anything that isn't a real image or video.
5. **Collection window** enforced server-side, not just hidden in the UI.

Deliberately **not** doing: image content moderation, or blocking on ML classification. The team reviews everything in Gumnut before anything reaches a page, which is the real backstop.

## 17. Gumnut gaps and improvement opportunities

Ordered by how much they'd improve this app.

> **Open item on G2 — needs confirming with Gumnut.** Write-only API keys have been reported as already supported. Two sources currently say otherwise: the [API key guide](https://docs.gumnut.ai/guides/authentication/api-keys) ("`read` is always required, and at least one action must be set") and the [create-API-key reference](https://docs.gumnut.ai/api-reference/api-keys/create-api-key) (`actions` accepts `read`/`write`/`delete`/`delete_permanently`, and "`read` is required whenever any broader action is selected"). Possibly shipped-but-undocumented, or in flight. **Try creating a key with `actions: ["write"]` — that settles it in one call.** Either way the app needs no read access, so this only changes how tightly the key can be scoped.

| # | Gap | Impact here | What would fix it |
|---|---|---|---|
| G1 | **No library sharing between users; no roles** — *on Gumnut's near-term roadmap* | Forces the team onto one shared Gumnut login (D1) in the meantime: no audit trail of who trashed what, and a password that circulates as volunteers turn over. Adopting it later needs no change to this app. | Library membership with viewer/editor roles (planned) |
| G2 | **No write-only API key scope** — `read` is still required alongside `write` | **The app makes zero read calls by design** (§15.1), so it is ready to use a write-only key the moment one exists. Until then its key can read every photo and every person tag (§15.6) — a compromised Worker secret exposes the whole collection to buy nothing at all. **See the note above: this may already be resolved.** | Allow `actions: ["write"]` without `read` |
| G3 | **No short-lived, scoped upload tokens** | The Worker must sit in the data path for every byte. A token minted per submission, scoped to one album and a few minutes, would let the browser upload directly to Gumnut — removing the Worker size limits, the concurrency tuning, and most of the rate-limit pressure. **This is the single biggest architectural simplification available.** | Mint-a-scoped-upload-token endpoint |
| G4 | **No description or album membership at create time** | 3 API calls per photo instead of 1, and three chances to half-succeed. Cheap in rate-limit terms (20 + 1 + 1 tokens, not 3× 20) but it is the main source of partial-failure states in §14. | Optional `description` and `album_ids` on `POST /api/assets` |
| G5 | **No structured provenance field** | Minor. The user-set `metadata.description` is a separate field from the AI caption, so nothing overwrites the uploader's name — but it is still free text, so "which photos did Jane send?" is not a query the team can run. | A structured `contributed_by` field, or arbitrary key-value metadata |
| G6 | **Signed asset URLs never expire and can't be revoked** | The app no longer relies on this, but it affects the team: a photo URL pasted into an email is a permanent public link to a child's photo, with no way to withdraw it. | Optional expiry and a revocation path |
| G7 | **Rate limit is per account, not per credential** | All parents share one bucket (§15.3). A per-API-key bucket, or higher limits for ingest, would remove the ceiling. | Per-key rate limit budgets |
| G8 | **Trashed assets silently absorb re-uploads** | Team trashes a photo; the same file re-uploaded reports success and vanishes. Confusing for both parent and team. | Distinguish "matched a trashed asset" in the response |

## 18. Repository and deployment

```
oms-yearbook-photos/          # public GitHub repo, NOT under the gumnut org
├── README.md                 # setup, how to add albums, account ownership
├── LICENSE                   # MIT — see below
├── docs/
│   └── PLAN.md               # this document
├── wrangler.toml
├── src/
│   ├── index.ts              # Worker: routes + static assets
│   ├── config.ts             # library, albums, deadline (committed, no secrets)
│   ├── upload.ts             # upload handler
│   └── client/               # frontend
└── .github/workflows/deploy.yml
```

**Secrets** (via `wrangler secret put`, never committed, never in a client bundle):

- `GUMNUT_API_KEY` — scoped to the one library, minimum viable permissions (write-only if G2 allows)
- `TURNSTILE_SECRET_KEY`

Access codes are deliberately **not** secrets: only their hashes are committed (§16), which is what makes revoking one a normal reviewable code change rather than a credential rotation.

**Deployment:** GitHub Actions on push to `main`, using a `CLOUDFLARE_API_TOKEN` repository secret. Because the repo is public, treat every pull request as untrusted — do not expose secrets to PR builds.

**Note on the public repo:** library IDs, album IDs, access-code hashes, school branding, and the whole app are fine in public. The threat model is only the two secrets above.

**A license is required, not optional.** Goal 5 is that anyone with a technical background can pick this up and run it. Public code with no license file is *not* open source — by default nobody else has permission to modify or redeploy it, which would defeat the whole point. MIT is the right choice: it is short, it is what a future volunteer will expect, and it imposes nothing on the school. Add it in M1.

## 19. Screens and states

This is the list Claude Design should work from. Every state here is reachable in the real app.

1. **Landing / upload form** — school branding, one-sentence explanation, the app's short privacy statement and district photo-release disclaimer inline, with links to Gumnut's full policy and the district policy manual (§6), uploader name field, comments field, album multi-select (pre-selected from URL params), rights-attestation checkbox, large "Choose photos or videos" target. Mobile-first: most parents arrive from a QR code on a phone.
2. **Files selected** — thumbnail grid of chosen files with per-file remove, total count and size, Upload button enabled. Video tiles carry a play badge and duration. **Any clip over 100 MB is rejected here, before uploading**, with copy that says what to do (trim it in Photos and re-add) rather than just refusing.
3. **Uploading** — per-file progress, overall progress, files completing one by one. Must survive slow school-parking-lot cellular gracefully, and must stay legible for a large batch: a parent sending 200 photos should see a sane summary, not 200 progress bars.
4. **Upload complete** — success shown in place, not on a separate screen: the count that went through, "these are now with the yearbook team," and an easy way to send more.
5. **Partial failure** — some succeeded, some failed, with retry for just the failed ones. Do not lose the successful work.
6. **Collection closed** — friendly, dated, with the yearbook lead's contact.
7. **Error states** — file too large, unsupported type, invalid or missing access code, rate limited (retrying), network lost, and **storage full** (Gumnut returns `507` when the account is at its cap — see §15.5). The storage-full state needs its own copy: it is not the parent's fault and it tells them to contact the yearbook lead rather than retry.
8. **QR / print page** — for the yearbook lead. Generates a printable sheet with a QR code per album, album name, and short URL. Designed to be printed on a home printer and taped to a table.

**Design constraints:** mobile-first; large tap targets; works one-handed; readable outdoors on a phone at a school event; accessible contrast; no reliance on hover. Brand inputs (colors, logo, mascot) are pending from the team (§10) — build with tokenized placeholders so they drop in cleanly.

## 20. Implementation milestones for Claude Code

**M1 — Upload path.** Worker + static page. One album, hard-coded. Name, comments, rights checkbox, privacy text, and district-policy disclaimer (§6). Single-file-per-request **streaming** upload with concurrency pool. Magic-byte validation, size cap, header-field length limits. Stream to `POST /api/assets` → `updateAsset` → album add, with duplicate handling per §15.1 (description written only on 201). Access-code hash check and collection-window check — a hash compare and a date compare, cheap enough to belong here rather than at the end. `LICENSE` and a first `README`. Deploy to `*.workers.dev`. Build the streaming upload path from §14.1 first — retrofitting it later means redoing the client/server contract. *Exit: a phone uploads 10 photos, a 90 MB file succeeds, a bad access code is rejected, and the yearbook lead sees correct attribution in Gumnut.*

**M2 — Albums and QR codes.** Config file, album multi-select, slug-based URL presets, QR generation page. Use the PTA subdomain (likely `yearbook.oldmillpta.org`) when available; Ted handles setup outside this plan, and M2 does not depend on it.

**M3 — Polish and error handling.** Closed-for-the-year page, full error-state coverage (§19.7), `429` retry with `Retry-After` (§14.1), `507` storage-full handling, partial-failure retry.

**M4 — Hardening and handoff.** Turnstile, per-device and per-IP rate limits, GitHub Actions deploy, full README, account-ownership transfer.

Throughout: no secrets in the client bundle, no endpoint that lists assets to a browser, and every Gumnut call goes through the Worker.
