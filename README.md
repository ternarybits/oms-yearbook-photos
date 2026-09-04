# Old Mill School Yearbook Photo Collection

A small web app that lets families upload photos and video clips for possible
inclusion in the Old Mill School yearbook. No account, no login — scan a QR code
at a school event, pick photos, tap upload.

Uploads are stored in [Gumnut](https://gumnut.ai), where the yearbook team
reviews and selects them. The app itself is upload-only: it never displays the
collection back to anyone.

## Status

Planning.

- **[Plan overview](https://claude.ai/design/p/97ac4aa0-49a4-41c6-8130-845f42fc224f?file=Yearbook+Photo+Collection+-+Part+1.dc.html)**
  — a friendlier read of Part 1, written for the yearbook team: goals, what a
  parent experiences, what the app deliberately does not do, privacy, and the
  decisions we need from the team.
- **[`docs/PLAN.md`](docs/PLAN.md)** — the full plan, including the technical
  appendix (architecture, upload contract, Gumnut behaviors, milestones).

Both cover the same project; the first is the one to send to someone who is not
going to build it.

## Stack

- Cloudflare Workers (hosting + upload proxy)
- Gumnut (photo and video storage, organization, committee tooling)

## Contributing

This is a volunteer-maintained project for a single school. The code is public so
that it can be maintained by anyone the team designates; no secrets live in
this repository.
