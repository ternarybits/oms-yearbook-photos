# Old Mill School Yearbook Photo Collection — Plan

**Status:** Draft for team review
**Author:** Ted
**Date:** 2026-09-04

---

## How to read this document

**Part 1** is written for the yearbook team and school administration. It covers what the app does, what it deliberately does not do, how family privacy is handled, what it costs, and what decisions we need from the team. Part 1 can be shared on its own.

**Part 2** is the technical appendix for the people who build and maintain the app. It also feeds Claude Design (screens and states) and Claude Code (architecture and build order).

---

## Decisions to confirm before building

These are the choices that affect how the yearbook team works, so they are worth confirming before building. Purely technical decisions — file size limits, how uploads are handled — are settled and live in Part 2.

| # | Decision | What it means for the team |
|---|---|---|
| **D1** | **The team shares one Gumnut login** to review photos | Everyone uses the same username and password for now, so there is no record of who did what. Gumnut is adding per-person logins soon; when it arrives, each team member gets their own and nothing about the app changes. |
| **D2** | **Each way of sharing the link gets its own access code** | The code is built into the QR codes and newsletter links, so nobody ever types it. Giving the posters, the newsletter, and each class parent a different code means one can be switched off without disrupting the others — useful if a link ends up somewhere it shouldn't. |
| **D3** | **Next school year is not being designed yet** | Albums and storage are settings, not code, so whether next year reuses this library or starts a fresh one can be decided when it arrives rather than guessed at now. |

---

# Part 1 — For the yearbook team

## 1. Goals

1. **Make it effortless for a parent to contribute photos.** Scan a QR code at a school event, pick photos from the camera roll, tap upload. No account, no password, no app to install.
2. **Get photos to the team already organized.** Photos arrive tagged with who sent them and which event/album they belong to, so the team isn't sorting an undifferentiated pile.
3. **Protect family privacy by default.** No parent can browse the collection — not other families' photos, and not their own. The app collects; it never displays.
4. **Be reusable.** The library and album list are configuration, not code, so a future team can point the app at new albums without a developer. How next year is actually structured is deliberately left open (D3).
5. **Be maintainable by someone other than the person who built it.** Public code repository, documented setup, accounts owned by the school/team rather than an individual.

## 2. Scope and non-goals

**This app does two things: collect photos and get them safely into storage.** Everything the team does *with* the photos afterwards happens in Gumnut, the photo library the app uploads into. Yearbook layout and printing are a separate effort entirely.

Not built in this app:

| Not in this app | Where it happens instead |
|---|---|
| Browsing and searching the collection | The Gumnut web app, where the team signs in |
| Comments, notes, and favorites on photos | Gumnut — the team can use these while selecting |
| Organizing, culling, and grouping photos | Gumnut |
| Yearbook layout, page design, printing | A separate effort, out of scope here |

Not built at all:

- **A public gallery.** There is no page, anywhere, where one family can see another family's photos.
- **Parent accounts or logins.** Contributing takes no sign-in.
- **Any destructive action from the upload app** — no delete, no edit, no overwrite, for anyone. See §5 for the reasoning; it is a design principle, not a missing feature.
- **Any use of the photos outside the yearbook.**

**Getting the photos back out** is already handled: Gumnut can download a whole album, or any selection, as a single ZIP of originals — from the web app, or via the API. So when it is time to lay out pages, the yearbook team exports the album and works from real files. See §15.6.

## 3. Who uses this

| Role | Who | What they do | How they access it |
|---|---|---|---|
| **Contributing family** | Any parent or family member | Uploads photos and clips from a phone or laptop | QR code or link. No login. |
| **Yearbook lead** | Kari | Owns the collection: creates albums, prints QR codes, reviews what comes in, handles removal requests | Signs into Gumnut (D1) |
| **Parent volunteers** | Several, assisting Kari | Review and select photos for pages | Signs into Gumnut (D1) |
| **App maintainer** | TBD | Keeps the site running, adds albums when asked | Cloudflare + GitHub accounts |

Kari and the volunteers currently **share one Gumnut login** (D1). That is the main practical cost of the shared-account approach: no record of who changed what. It resolves when Gumnut ships per-person library access.

## 4. What a parent can do (supported user stories)

**S1 — Upload photos and clips from an event.**
A parent scans the QR code posted at the Fall Festival. The page opens already knowing the upload is for the Fall Festival album. They tap "Choose photos or videos," select 12 photos and a short clip from their camera roll, type their name, add a note ("Third grade booth, ~2pm"), check the rights box, and tap Upload. A progress indicator shows each file completing. They get a clear confirmation.

**S2 — Upload without a QR code.**
A parent follows the link from the school newsletter or a class-parent email. That link carries the same access code the QR codes use, so the form opens normally — they just choose which album(s) apply from the published list rather than having one pre-selected. Someone who types the bare subdomain with no link sees a short page explaining where to get the link (newsletter, class parent, or Kari's email) rather than an error.

**S3 — Put one batch into more than one album.**
Photos from a single afternoon that belong in both "Field Day" and "Fifth Grade" are added to both, in one submission.

**S3b — Send a large batch.**
A parent who shot 200 photos at Field Day selects them all and uploads in one go. The browser works through them steadily; nothing caps the count.

**S4 — Understand exactly who can see their photos.**
Plain-language privacy text is on the upload page itself, not buried in a link, including the fact that this is a one-way submission.

**S5 — Send more photos later, from any device.**
A parent can come back as many times as they like, from a phone or a laptop, with no login and nothing to remember. Each visit is a fresh submission. Sending the same photo twice is harmless — Gumnut keeps one copy (§15.1).

**S6 — Get a clear answer when the window has closed.**
After the submission deadline, the page explains that collection is closed and who to contact.

## 5. What a parent cannot do, and why

| Not supported | Reason |
|---|---|
| See other people's photos | Core privacy commitment |
| **See their own photos after uploading** | The app is upload-only. Once the confirmation screen is dismissed, there is no gallery, no history, and nothing to come back to. Parents keep their own copies in their camera roll; the team has theirs. |
| **Delete a photo after uploading** | **By design, not a v1 shortcut.** Deleting is destructive and irreversible from the parent's side, so it must require *proof that you are the person who uploaded it*. "Same device" is not proof — phones get handed to kids, shared, lent, and resold. Without a real login there is nothing to check, so the app offers no delete at all. Email Kari (§7). |
| **Edit the name or note after submitting** | Same reasoning. Editing someone else's attribution is a smaller harm than deleting their photo, but it rests on the same unmet identity claim. |

**The principle behind this whole table:** the app performs **no action on a photo after it is uploaded** — not viewing it, not editing it, not deleting it — because it cannot verify who is asking. "Same device" is not proof of identity, and any feature built on it becomes an argument for the next one: a parent who can *see* their past uploads will reasonably ask to delete one. Rather than build that staircase and stop partway up, the app does not take the first step. Upload is a one-way door. If real accounts are ever added, that is the point at which any of this becomes discussable.

**This is the most important thing to communicate to parents,** and the app says it directly: *"Your photos go straight to the yearbook team. Nobody browsing this site can see them — including you, afterwards. Keep your own copies."*

That last clause matters. A parent who assumes the app is also a backup, and later deletes from their phone, has lost something. The confirmation screen should reinforce it (§19.4).

## 6. Privacy and consent

- **Nothing is public.** There is no public gallery. Uploads are visible only to Kari and the parent volunteers.
- **No parent-to-parent visibility.** The app never shows one family's photos to another family.
- **What we collect:** the photos and video clips, the uploader's name as typed, an optional free-text note, and which albums they chose. We do not ask for email, phone, or student names.
- **Rights attestation.** The upload form has a required checkbox: *"I have the right to share these photos, and I understand they may appear in the school yearbook."* Parents frequently upload photos containing other people's children, so this is stated plainly.
- **Face recognition and naming.** Gumnut automatically detects faces and groups photos of the same person together, and generates a written description of each photo so search works. On top of that, **the team intends to attach children's names to those face groups.**

  **Why:** to answer the question every yearbook has to answer — *do we have enough photos of each child?* Without it, checking coverage across hundreds of photos means a volunteer squinting at thumbnails. With it, it is a list.

  In practice: the library holds a **name-to-face index of students**, built by the team from photos other families contributed. It is visible only to Kari and the volunteers, never published, never shown to other families, and no names appear anywhere in the upload app. Gumnut groups faces; it does not know who anyone is until a person types a name.

- **The photo release policy, and how naming actually enforces it.** The school's existing photo release policy lets a family ask that their child not appear in the yearbook. Historically that has been close to unenforceable in practice: with hundreds of candid event photos, **there has been no realistic way to know whether a covered child is in a given shot.** Compliance has depended on someone recognizing a face at layout time.

  Naming faces changes that, so the team will **mark covered children directly in the index — appending a marker such as `(DNR)` to the person's name.** Every photo containing that child then surfaces with the marker attached, and the team can exclude them reliably rather than hopefully.

  This reframes the whole question. Face naming is not a privacy cost that the release policy has to tolerate — **it is the mechanism that makes the release policy work for the first time.** A family that has asked for their child to be excluded is materially better protected with this system than without it.

  Two things follow:
  - **The names must persist.** Deleting the index when the book ships would discard exactly the information needed to honor the policy on any future use of these photos (§8 retention).
  - **The policy governs the archive, not just the book.** These photos are intended for future use beyond this year's yearbook, so the release policy applies to that future use too — not only to what gets printed in the spring.

  **Action item for the team (§10):** read the school's photo release policy closely and confirm what it actually covers — yearbook only, or all school publications and archives; whether it is opt-in or opt-out; and how a family registers or changes a preference. The marker convention should match the policy's real terms rather than an assumed version of them.
- **Retention.** The photos are intended to outlive this year's yearbook and serve as a school archive. **The name index should be kept alongside them, not deleted** — it is what makes the release policy enforceable on any future use, and discarding it would return the school to the position of not knowing who is in which photo. *Open question for the team:* how long the archive is kept, and who is responsible for it once this year's team disbands.

## 7. Removal requests

A parent who wants a photo removed emails Kari at an address published on the site, and she removes it in Gumnut. Target: handled within one week. This is a human process in v1, deliberately — the volume does not justify building anything, and a real person reading the request is the better outcome anyway.

## 8. Ownership and continuity

For the app to outlive whoever builds it:

- **The target is institutional ownership** of the Cloudflare, GitHub, and Gumnut accounts, under an address like `yearbook@oldmillschool.org` rather than anyone's personal email.
- **This is not resolved yet.** Whether the school will grant a subdomain and an institutional account is an open question. If it declines, the **PTA** — or a similar parent body — is the fallback owner. Either is acceptable; a personal account as the permanent home is not.
- **Sequencing:** the project starts on a temporary domain and personal accounts, and **migrates as soon as an institutional owner exists.** That migration is a tracked task, not a someday intention — it is the difference between a maintainable school asset and a project that quietly belongs to one parent.
- Note that the Gumnut account is the one that matters most: it holds the photos, the archive, and the name index (§6). Domain and code hosting can be moved cheaply at any time; the photo library is the thing with gravity.
- Credentials live in a password manager the owning body controls.
- The code is public on GitHub. Secrets are never in the code.
- A `README` documents how to change what the app collects into: create an album in Gumnut, add it to the config file, deploy, print a QR code. That is the whole recurring operation.
- The runbook names **who watches remaining photo storage** (§9) and **who maintains the name index and its release-policy markers** (§6) — the two recurring duties the app cannot do for itself.

## 9. Costs

| Item | Cost | Notes |
|---|---|---|
| App hosting (Cloudflare Workers) | **$0, likely** | The free tier gives 100,000 requests/day and the same 100 MB upload limit. Its 10 ms CPU cap is *CPU only* — time waiting on the network doesn't count, and a streaming upload is nearly all waiting. Budget $5/mo as a fallback if M1 shows otherwise. |
| Domain | $0 | Uses a school subdomain |
| Code hosting (GitHub) | $0 | Public repository |
| Photo + video storage (Gumnut) | **$0 expected** | Gumnut's default 10 GB is expected to cover the year, and the limit can be raised if needed. Video is the swing factor: 500 photos ≈ 2 GB, but 150 phone clips could reach the cap on their own. |

**The whole thing is expected to run at $0.** The one number to watch is storage, because accepting video makes it unpredictable — a single minute of phone video costs more than a hundred photos, and nobody can say in advance how many clips parents will send.

**It matters because hitting the cap fails loudly and all at once.** Gumnut rejects *every* upload with a `507` when the library is full — including duplicates — so the symptom is every parent failing simultaneously, most likely the evening after a big event. Raising the limit fixes it, but only if someone notices. Kari should check remaining capacity after each major event; that is the one recurring operational duty this project has.
| **Total** | **~$5 / month + storage** | |

## 10. What we need from the school and team

- [ ] **Subdomain decision** and an IT request to point it at the app (e.g. `yearbook.oldmillschool.org`) — or, if the school declines, a PTA-owned domain instead (§8). *Lead time here is the main scheduling risk; the app runs on a temporary address until it resolves.*
- [ ] **Album list** for the year (e.g. Fall Festival, Field Day, Class Photos, Fifth Grade, Winter Concert…)
- [ ] **Submission deadline** for the collection window
- [ ] **Brand inputs:** school colors (hex values), logo file, mascot artwork, and any font or style guidance. *We will not guess these.*
- [ ] **Approval of the privacy language** in §6
- [ ] **Read the school's photo release policy closely** (§6) — what it covers (yearbook only, or all publications and the archive), opt-in or opt-out, and how a family registers or changes a preference. **The one item that may need the school office rather than a team meeting, so start it early.**
- [ ] **Agree the marker convention** for covered children (e.g. `(DNR)` appended to the name), matching the policy's actual terms
- [ ] **Explicit sign-off on naming children's faces** (§6)
- [ ] **Retention decision** (§6) — how long the archive is kept, and who owns it once this year's team disbands
- [ ] **Kari's email address** for the site and for removal requests

## 11. Build order

Each stage is independently useful and shippable.

**Stage 1 — Upload works.** Themed page, photo picker, name + note fields, rights checkbox, uploads land in one Gumnut album. Privacy text. Deployed to a temporary address. *This alone is enough to start collecting.*

**Stage 2 — Albums and QR codes.** Multi-select album list, QR links that pre-select an album, a printable QR sheet for Kari. Move to the school subdomain.

**Stage 3 — Deadline handling.** Collection window enforced, with a clear closed-for-the-year page.

**Stage 4 — Handoff.** Abuse protections, documentation, and transfer of account ownership to the team, so the app can run without its original author.

**Later, if wanted:** a team gallery in the app, real parent accounts, and the uploader-initiated deletion those would unlock (§5).

---

# Part 2 — Technical appendix

## 12. Architecture

```
Parent's browser                Cloudflare Worker              Gumnut
─────────────────               ─────────────────              ──────
static page + JS   ──POST──▶    /api/upload                    
 (one file per         file      ├─ verify access code
  request, 2–3        + fields   ├─ verify Turnstile token
  concurrent)                    ├─ check magic bytes, size, type
                                 ├─ assets.create ─────────────▶ POST /api/assets
                                 ├─ assets.updateAsset ────────▶ PATCH description
                                 ├─ albums.assetsAssociations.add ▶ album membership
                    ◀──JSON──    └─ {ok, isDuplicate}
```

The browser keeps nothing. Once a file is uploaded and confirmed, the app has no further relationship with it.

**Key properties:**

- The Gumnut API key exists **only** as a Worker secret. It is never sent to the browser and never in the repository.
- Thumbnails are rendered from Gumnut's **signed CDN URLs**, fetched directly by the browser. The Worker is not a proxy for image bytes — only for uploads.
- The Worker holds no database. All per-parent state is in that parent's browser.

**Stack recommendation:** Cloudflare Workers with static assets (Worker + `[assets]` binding), Hono for routing, `gumnut-sdk` (TypeScript) for the Gumnut calls, plain TypeScript + a light CSS approach for the frontend. Deliberately boring and small — a future maintainer should be able to read the whole thing in an afternoon. No framework is required for a single-page form.

## 13. Configuration

Everything the team might change lives in one committed config object. Library and album IDs are not secrets and belong in the public repo.

```ts
// src/config.ts
export const CONFIG = {
  displayName: "Old Mill School Yearbook",
  libraryId: "lib_...",
  closesAt: "2027-03-15T23:59:59-07:00",
  contactEmail: "yearbook@oldmillschool.org",
  albums: [
    { slug: "fall-festival", name: "Fall Festival", albumId: "album_..." },
    { slug: "field-day",     name: "Field Day",     albumId: "album_..." },
    { slug: "fifth-grade",   name: "Fifth Grade",   albumId: "album_..." },
  ],

  // Access codes, by SHA-256 hash — see §16. The codes themselves are never
  // committed; only their hashes, which are safe in a public repo.
  accessCodes: [
    { label: "Fall Festival posters",  hash: "a3f1…", active: true },
    { label: "October newsletter",     hash: "9c22…", active: true },
    { label: "Back-to-school night",   hash: "51de…", active: false }, // revoked
  ],
} as const;
```

**On future years (D3):** deliberately not designed now. Adding albums to this same library is an edit to `albums`. Moving to a separate library per year is an edit to `libraryId` plus a re-scoped API key. Both stay available because nothing outside this file knows the year. Do not build multi-year machinery until someone actually needs it.

**QR / deep links** use slugs, never raw Gumnut IDs:
`https://yearbook.oldmillschool.org/?a=field-day,fifth-grade&c=<access-code>`

## 14. Upload contract

`POST /api/upload` — **exactly one file per request**, sent as the **raw request body** rather than as a multipart form (§14.1 explains why). The browser runs a small concurrency pool (2–3 in flight) over the selected files.

Metadata travels in query parameters, not the body: `uploaderName`, `comments`, `albums` (comma-separated slugs), `deviceId`, `deviceAssetId`, `fileCreatedAt`, `fileName`, `turnstileToken`, `accessCode`. `Content-Type` and `Content-Length` describe the file itself.

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

1. The browser `POST`s the **raw file as the request body**, with metadata in headers or query params (`?name=…&deviceAssetId=…&albums=…`) rather than as multipart fields.
2. The Worker builds the outbound `multipart/form-data` body as a `ReadableStream`: a text prelude carrying the scalar fields and the file part header, then **`request.body` piped straight through**, then the closing boundary.
3. It sends that with a raw `fetch` to `POST /api/assets` with `Content-Type: multipart/form-data; boundary=…` and the `Authorization: Bearer` header — bypassing `assets.create` for this one call.

Memory then stays constant regardless of file size, and the limit becomes Cloudflare's account-level request-body cap: **100 MB on Free/Pro, 200 MB on Business.**

The SDK is still used normally for every *other* call (`updateAsset`, album association, retrieve) — those are small JSON requests where its ergonomics are worth having. Only the upload call is hand-rolled, and it should be one well-commented function with a test.

This also pairs with gap **G3**: if Gumnut ships scoped upload tokens, this hand-rolled function disappears entirely and the browser uploads directly.

**Per-file server-side sequence:**

1. Stream the file to `POST /api/assets` per §14.1, reading the HTTP status to distinguish new (201) from duplicate (200) — see §15.1.
2. `client.assets.updateAsset(id, { description })` where description is the attribution block (§15.1).
3. For each selected album: `client.albums.assetsAssociations.add(albumId, { asset_ids: [id] })`.

Steps 2 and 3 are best-effort: if they fail, still return the `assetId`, and log. A photo that landed without its note is recoverable; a photo that was rejected because its note failed to save is not.

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
  Notes: Third grade booth, around 2pm
  ```
- **A photo the team has trashed will silently "succeed" on re-upload** and return the trashed asset without restoring it. The parent sees success; the photo does not reappear. Acceptable, but the team should know: trashing is not the same as blocking.

### 15.2 Signed asset URLs are unguessable but do not expire

`asset_urls` on an asset response are HMAC-SHA256-signed URLs to `assets.gumnut.ai`, served by Gumnut's CDN Worker. The browser can fetch them directly with no API key. They carry **no expiry** — anyone who has the URL can view that image indefinitely.

**The app never hands one of these to a browser** — with no upload history, nothing renders an uploaded photo back to a parent. So this is documented here as context for the team's own use of Gumnut (a signed URL pasted into an email is a permanent public link to that photo), not as a property this app relies on.

### 15.3 Rate limits are per *account*, shared by every parent

Weighted token bucket: **400 capacity, refills 100/second.** An upload costs **20 tokens** — roughly 5 uploads/second sustained, 20 in a burst. Because the app uses one API key on one account, **every parent uploading at once shares this budget.** Realistic worst case is the evening after a big event.

Mitigations: 2–3 concurrent uploads per browser (not 10); the SDK's built-in backoff; surface a `429` to the client as an automatic retry with jitter, not as a failure. Watch `X-RateLimit-Remaining`. This is the app's scaling ceiling and should be revisited if the school grows the program.

### 15.4 No description or album at create time

There is no way to set a description or album membership in the create call — hence the three-step sequence in §14. Listed as a Gumnut gap.

### 15.5 Storage cap returns 507, checked before deduplication

`POST /api/assets` checks storage limits *first*, so an account at its cap returns **`507 Insufficient Storage` even for a byte-identical duplicate** that would consume no new storage. The Worker should map `507` to a distinct, non-retryable client error with its own copy (§19.7) — retrying makes it worse, and every parent hits it simultaneously. The lead-facing side of this is in §9.

### 15.6 The team's side is all Gumnut — we build none of it

Worth knowing when writing the handoff docs, because this is where Kari and the volunteers actually spend their time:

- **Browse, search, and view** the collection, including video playback with seeking.
- **Tag people** — face detection plus manual face boxes for anyone missed (§6).
- **Bulk download** — select any set of photos, or a whole album, and download them as a ZIP of originals. Available in the Gumnut web app and through the API. (Mechanically the zipping happens in the browser rather than on a server, which makes no difference to how it is used.)
- **Trash** — soft delete, recoverable for 90 days, which is the safety net behind removal requests (§7).

## 16. Abuse prevention

No login means no strong identity, so the goal is raising cost, not perfect prevention. In layers, cheapest first:

1. **Access codes in the link** (`?c=…`), checked server-side. Keeps the page out of drive-by and crawler traffic.

   **Multiple codes, individually revocable.** Rather than one shared secret, issue a distinct code per distribution channel — one for the Fall Festival posters, one for the October newsletter, one per class parent. Three things follow: a leaked code can be killed without disrupting everyone else, the label on the code tells you *which channel* leaked, and codes can be retired when their event is over.

   **How they live in a public repo.** The config holds only the **SHA-256 hash** of each code plus a label and an `active` flag (§13). The Worker hashes the submitted code and compares. The codes themselves are never committed, so the repo stays public and revocation is a one-line config edit plus a deploy — no secret rotation, no separate store.

   This works because the codes ride in URLs and QR codes rather than being typed, so they can be long and fully random (16+ random characters). Do not shorten them into something human-memorable; that is what would make the published hashes worth attacking.

   *If instant revocation ever matters more than simplicity,* move the list to a KV namespace so it changes without a deploy. Not worth it at this scale.
2. **Cloudflare Turnstile** on the form — invisible for nearly all real parents, blocks scripted submission.
3. **Rate limiting per IP and per `deviceId`** in the Worker (Cloudflare Rate Limiting rules, or a Durable Object / KV counter): e.g. 100 files per device per day, 300 per IP per day.
4. **Server-side file validation:** magic-byte check (don't trust `Content-Type` or extension), size cap (§14.1), and reject anything that isn't a real image or video.
5. **Collection window** enforced server-side, not just hidden in the UI.

Deliberately **not** doing: image content moderation, or blocking on ML classification. The team reviews everything in Gumnut before anything reaches a page, which is the real backstop.

## 17. Gumnut gaps and improvement opportunities

Ordered by how much they'd improve this app.

> **Open item on G2 — needs confirming with Gumnut.** Write-only API keys have been reported as already supported. Two sources currently say otherwise: the [API key guide](https://docs.gumnut.ai/guides/authentication/api-keys) ("`read` is always required, and at least one action must be set") and the [create-API-key reference](https://docs.gumnut.ai/api-reference/api-keys/create-api-key) (`actions` accepts `read`/`write`/`delete`/`delete_permanently`, and "`read` is required whenever any broader action is selected"). Possibly shipped-but-undocumented, or in flight. **Try creating a key with `actions: ["write"]` — that settles it in one call.** Either way the app needs no read access, so this only changes how tightly the key can be scoped.

| # | Gap | Impact here | What would fix it |
|---|---|---|---|
| G1 | **No library sharing between users; no roles** — *on Gumnut's near-term roadmap* | Forces the team onto one shared Gumnut login (D1) in the meantime: no audit trail of who trashed what, and a password that circulates as volunteers turn over. Adopting it later needs no change to this app. | Library membership with viewer/editor roles (planned) |
| G2 | **No write-only API key scope** — `read` is still required alongside `write` | **The app makes zero read calls by design** (§15.1), so it is ready to use a write-only key the moment one exists. Until then its key can read every photo and the named person index (§6) — a compromised Worker secret exposes the whole collection to buy nothing at all. **See the note below: this may already be resolved.** | Allow `actions: ["write"]` without `read` |
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
├── PLAN.md                   # this document
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

## 19. Screens and states

This is the list Claude Design should work from. Every state here is reachable in the real app.

1. **Landing / upload form** — school branding, one-sentence explanation, privacy statement inline, uploader name field, comments field, album multi-select (pre-selected from URL params), rights-attestation checkbox, large "Choose photos or videos" target. Mobile-first: most parents arrive from a QR code on a phone.
2. **Files selected** — thumbnail grid of chosen files with per-file remove, total count and size, Upload button enabled. Video tiles carry a play badge and duration. **Any clip over 100 MB is rejected here, before uploading**, with copy that says what to do (trim it in Photos and re-add) rather than just refusing.
3. **Uploading** — per-file progress, overall progress, files completing one by one. Must survive slow school-parking-lot cellular gracefully, and must stay legible for a large batch: a parent sending 200 photos should see a sane summary, not 200 progress bars.
4. **Upload complete** — clear success confirmation, count uploaded, "these are now with the yearbook team," and an option to send more. **This is the last time the parent sees these photos in the app**, so the confirmation has to carry real weight — this is the screen that has to feel like the photos arrived somewhere safe.
5. **Partial failure** — some succeeded, some failed, with retry for just the failed ones. Do not lose the successful work.
6. **Collection closed** — friendly, dated, with Kari's contact.
7. **Error states** — file too large, unsupported type, invalid or missing access code, rate limited (retrying), network lost, and **storage full** (Gumnut returns `507` when the account is at its cap — see §15.5). The storage-full state needs its own copy: it is not the parent's fault and it tells them to contact Kari rather than retry.
8. **QR / print page** — for Kari. Generates a printable sheet with a QR code per album, album name, and short URL. Designed to be printed on a home printer and taped to a table.

**Design constraints:** mobile-first; large tap targets; works one-handed; readable outdoors on a phone at a school event; accessible contrast; no reliance on hover. Brand inputs (colors, logo, mascot) are pending from the team (§10) — build with tokenized placeholders so they drop in cleanly.

## 20. Implementation milestones for Claude Code

**M1 — Upload path.** Worker + static page. One album, hard-coded. Name, comments, rights checkbox. Single-file-per-request **streaming** upload with concurrency pool. Magic-byte validation, size cap. stream to `POST /api/assets` → `updateAsset` → album add, with duplicate handling per §15.1 (description written only on 201). Deploy to `*.workers.dev`. Build the streaming upload path from §14.1 first — retrofitting it later means redoing the client/server contract. *Exit: a phone uploads 10 photos, a 90 MB file succeeds, and Kari sees correct attribution in Gumnut.*

**M2 — Albums and QR codes.** Config file, album multi-select, slug-based URL presets, QR generation page, collection window enforced server-side. Custom subdomain.

**M3 — Deadline and polish.** Collection window enforced server-side, closed-for-the-year page, full error-state coverage.

**M4 — Hardening and handoff.** Turnstile, access code, per-IP and per-device rate limits, error states, GitHub Actions deploy, README, account-ownership transfer.

Throughout: no secrets in the client bundle, no endpoint that lists assets to a browser, and every Gumnut call goes through the Worker.
