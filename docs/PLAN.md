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

1. **Make it effortless to contribute photos.** Scan a QR code at a school event, pick photos from the camera roll, tap upload. No account, no password, no app to install.
2. **Get photos to the team already organized.** Photos arrive tagged with who sent them and which event/album they belong to, so the team isn't sorting an undifferentiated pile.
3. **Protect family privacy by default.** No parent can browse the collection — not other families' photos, and not their own. The app collects; it never displays.
4. **Be reusable.** The library and album list are configuration, not code, so a future team can point the app at new albums without a developer. How next year is actually structured is deliberately left open (D3).
5. **Be maintainable by someone other than the person who built it.** Public code repository, documented setup, and accounts owned by an institution rather than an individual — so the maintainer role can pass to any parent with a technical background.

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
- **Any use of the photos outside the school's own purposes.** They are collected for the yearbook and kept as a school archive (§6); they are not shared, sold, or handed to anyone else.

**Getting the photos back out** is already handled: Gumnut can download a whole album, or any selection, as a single ZIP of originals — from the web app, or via the API. So when it is time to lay out pages, the yearbook team exports the album and works from real files. See §15.6.

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

Uploads go to **Gumnut**, the photo library the yearbook team works in. Gumnut's privacy policy — **<https://gumnut.ai/privacy>** — is the authoritative document for how files are stored, how they are processed, how long they are kept, and how they can be exported or deleted. It is considerably more thorough than anything we would write here, and the upload page links to it directly.

The rest of this section covers only what is **specific to this app and this project** — the things Gumnut's policy cannot know about.

### Specific to this app

- **Nothing is public.** There is no public gallery, and no page anywhere that displays the collection.
- **No parent-to-parent visibility.** The app never shows one family's photos to another family. It never shows a contributor their own photos either — it collects and does not display (§5).
- **What the app asks for:** the photos and clips, the uploader's name as typed, an optional note, and which albums they chose. It does not ask for email, phone number, or student names.
- **Rights attestation.** The upload form carries a required checkbox: *"I have the right to share these photos, and I understand they may appear in the school yearbook and be kept in the school's photo archive."* Contributors routinely upload photos containing other people's children, so this is stated plainly rather than buried. **The exact wording needs the team's sign-off (§10)** — it must match what §6 says the photos are actually for, including archive use, not just this year's book.

### Specific to this project

- **The school's photo release policy.** Families can ask that their child not appear in the yearbook. Historically this has been difficult to honor reliably: across hundreds of candid event photos, there has been no practical way to know whether a covered child is in a given shot, so compliance depended on someone recognizing a face during layout.

  Because the team can now identify who appears in which photos, covered children can be **marked in Gumnut and excluded reliably** rather than by eye. The mechanics are in §15.6. Two consequences worth stating up front:
  - **The policy governs the archive, not just the book.** These photos are meant for use beyond this spring, so the release policy applies to that future use too.
  - **The identifying information has to persist** for that to keep working. Discarding it later would return the school to not knowing who is in which photo.

  **Action item (§10):** read the release policy closely and confirm what it actually covers — yearbook only, or all school publications and archives; whether it is opt-in or opt-out; and how a family registers or changes a preference. The team's handling should follow the policy's real terms rather than an assumed version of them.

- **Retention.** The photos are intended to outlive this year's yearbook and serve as a school archive. *Open question for the team:* how long it is kept, and who is responsible for it once this year's team disbands.

## 7. Removal requests

A contributor who wants a photo removed emails the yearbook lead at an address published on the site, who removes it in Gumnut. Target: handled within one week. This is a human process in v1, deliberately — the volume does not justify building anything, and a real person reading the request is the better outcome anyway.

## 8. Ownership and continuity

For the app to outlive whoever builds it:

- **The target is institutional ownership** of the Cloudflare, GitHub, and Gumnut accounts, under an institutional address rather than anyone's personal email.
- **Two candidate homes**, and it is not yet settled which:
  - **`mvschools.org`** — the school district. Carries the most weight, and makes the app unambiguously a school asset. Also the slower path: it means a district IT request, and district IT may reasonably decline to host a volunteer-built app.
  - **`oldmillpta.org`** — the PTA. Almost certainly faster and more likely to say yes, and the PTA is a durable body that outlives any one family. A perfectly good permanent home, not merely a fallback.

  Either is acceptable. A personal account as the *permanent* home is not.
- **Sequencing:** the project starts on a temporary domain and personal accounts, and **migrates as soon as an institutional owner exists.** That migration is a tracked task, not a someday intention — it is the difference between a maintainable school asset and a project that quietly belongs to one parent.
- Note that the Gumnut account is the one that matters most: it holds the photos, the archive, and the team's record of who appears in them (§15.6). Domain and code hosting can be moved cheaply at any time; the photo library is the thing with gravity.
- Credentials live in a password manager the owning body controls.
- The code is public on GitHub. Secrets are never in the code.
- A `README` documents how to change what the app collects into: create an album in Gumnut, add it to the config file, deploy, print a QR code. That is the whole recurring operation.
- The runbook names **who watches remaining photo storage** (§9) and **who maintains the person tags and release-policy markers** (§15.6) — the two recurring duties the app cannot do for itself.

## 9. Costs

| Item | Cost | Notes |
|---|---|---|
| App hosting (Cloudflare Workers) | **$0, likely** | The free tier gives 100,000 requests/day and the same 100 MB upload limit. Its 10 ms CPU cap is *CPU only* — time waiting on the network doesn't count, and a streaming upload is nearly all waiting. Budget $5/mo as a fallback if M1 shows otherwise. |
| Domain | $0 | A subdomain of a district or PTA domain the school community already owns |
| Code hosting (GitHub) | $0 | Public repository |
| Photo + video storage (Gumnut) | **$0 expected** | Gumnut's default 10 GB is expected to cover the year, and the limit can be raised if needed. Video is the swing factor: 500 photos ≈ 2 GB, but 150 phone clips could reach the cap on their own. |
| **Total** | **$0 expected**, $5/mo worst case | Nothing here requires a purchase order |

**The whole thing is expected to run at $0.** The one number to watch is storage, because accepting video makes it unpredictable — a single minute of phone video costs more than a hundred photos, and nobody can say in advance how many clips parents will send.

**It matters because hitting the cap fails loudly and all at once.** Gumnut rejects *every* upload with a `507` when the library is full — including duplicates — so the symptom is every parent failing simultaneously, most likely the evening after a big event. Raising the limit fixes it, but only if someone notices. The yearbook lead should check remaining capacity after each major event; that is the one recurring operational duty this project has.

## 10. What we need from the school and team

- [ ] **Domain decision** — `yearbook.mvschools.org` (district) or `yearbook.oldmillpta.org` (PTA), plus whoever administers DNS pointing it at the app (§8). *Lead time here is the main scheduling risk; the app runs on a temporary address until it resolves, so **ask early and take whichever answer comes back first**.*
- [ ] **Album list** for the year (e.g. Lapathon, Field Day, Class Photos, Fifth Grade, Winter Concert…)
- [ ] **Submission deadline** for the collection window
- [ ] **Brand inputs:** school colors (hex values), logo file, mascot artwork, and any font or style guidance. *We will not guess these.*
- [ ] **Approval of the app's privacy language** in §6, and specifically the **wording of the rights-attestation checkbox** — it is the one sentence contributors actually agree to, and it needs to describe archive use, not just the yearbook
- [ ] **Read the school's photo release policy closely** (§6) — what it covers (yearbook only, or all publications and the archive), opt-in or opt-out, and how a family registers or changes a preference. **The one item that may need the school office rather than a team meeting, so start it early.**
- [ ] **Agree how covered children are marked** in Gumnut (e.g. a `(DNR)` suffix), matching the policy's actual terms
- [ ] **Retention decision** (§6) — how long the archive is kept, and who owns it once this year's team disbands
- [ ] **The yearbook lead's email address** for the site and for removal requests

## 11. Build order

Each stage is independently useful and shippable.

**Stage 1 — Upload works.** Themed page, photo picker, name + note fields, rights checkbox, uploads land in one Gumnut album. Privacy text. Access code and collection window enforced from the start — both are a few lines each (§16), and Stage 1 is already collecting real photos of real children at a public URL, so it should not run wide open while the rest is built. Deployed to a temporary address. *This alone is enough to start collecting.*

**Stage 2 — Albums and QR codes.** Multi-select album list, QR links that pre-select an album, a printable QR sheet for the yearbook lead. Move to the permanent subdomain.

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

### 15.6 The team's side is all Gumnut — we build none of it

Worth knowing when writing the handoff docs, because this is where the yearbook team actually spends its time:

- **Browse, search, and view** the collection, including video playback with seeking.
- **Tag people** — Gumnut detects and groups faces automatically; the team puts names to those groups, adding manual face boxes for anyone missed. Two things depend on this:
  - **Coverage.** *Do we have enough photos of each child?* is a list rather than a squint through thumbnails.
  - **Release-policy handling (§6).** Children covered by the policy are marked in the person's name — a suffix such as `(DNR)` — so every photo they appear in surfaces with the marker attached and can be excluded reliably. This information needs to persist for as long as the archive does; deleting it would return the school to not knowing who is in which photo.
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

1. **Landing / upload form** — school branding, one-sentence explanation, the app's short privacy statement inline with a link out to Gumnut's full policy (§6), uploader name field, comments field, album multi-select (pre-selected from URL params), rights-attestation checkbox, large "Choose photos or videos" target. Mobile-first: most parents arrive from a QR code on a phone.
2. **Files selected** — thumbnail grid of chosen files with per-file remove, total count and size, Upload button enabled. Video tiles carry a play badge and duration. **Any clip over 100 MB is rejected here, before uploading**, with copy that says what to do (trim it in Photos and re-add) rather than just refusing.
3. **Uploading** — per-file progress, overall progress, files completing one by one. Must survive slow school-parking-lot cellular gracefully, and must stay legible for a large batch: a parent sending 200 photos should see a sane summary, not 200 progress bars.
4. **Upload complete** — success shown in place, not on a separate screen: the count that went through, "these are now with the yearbook team," and an easy way to send more.
5. **Partial failure** — some succeeded, some failed, with retry for just the failed ones. Do not lose the successful work.
6. **Collection closed** — friendly, dated, with the yearbook lead's contact.
7. **Error states** — file too large, unsupported type, invalid or missing access code, rate limited (retrying), network lost, and **storage full** (Gumnut returns `507` when the account is at its cap — see §15.5). The storage-full state needs its own copy: it is not the parent's fault and it tells them to contact the yearbook lead rather than retry.
8. **QR / print page** — for the yearbook lead. Generates a printable sheet with a QR code per album, album name, and short URL. Designed to be printed on a home printer and taped to a table.

**Design constraints:** mobile-first; large tap targets; works one-handed; readable outdoors on a phone at a school event; accessible contrast; no reliance on hover. Brand inputs (colors, logo, mascot) are pending from the team (§10) — build with tokenized placeholders so they drop in cleanly.

## 20. Implementation milestones for Claude Code

**M1 — Upload path.** Worker + static page. One album, hard-coded. Name, comments, rights checkbox. Single-file-per-request **streaming** upload with concurrency pool. Magic-byte validation, size cap, header-field length limits. Stream to `POST /api/assets` → `updateAsset` → album add, with duplicate handling per §15.1 (description written only on 201). Access-code hash check and collection-window check — a hash compare and a date compare, cheap enough to belong here rather than at the end. `LICENSE` and a first `README`. Deploy to `*.workers.dev`. Build the streaming upload path from §14.1 first — retrofitting it later means redoing the client/server contract. *Exit: a phone uploads 10 photos, a 90 MB file succeeds, a bad access code is rejected, and the yearbook lead sees correct attribution in Gumnut.*

**M2 — Albums and QR codes.** Config file, album multi-select, slug-based URL presets, QR generation page. Permanent subdomain.

**M3 — Polish and error handling.** Closed-for-the-year page, full error-state coverage (§19.7), `429` retry with `Retry-After` (§14.1), `507` storage-full handling, partial-failure retry.

**M4 — Hardening and handoff.** Turnstile, per-device and per-IP rate limits, GitHub Actions deploy, full README, account-ownership transfer.

Throughout: no secrets in the client bundle, no endpoint that lists assets to a browser, and every Gumnut call goes through the Worker.
