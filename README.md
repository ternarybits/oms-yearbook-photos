# Old Mill School Yearbook Photo Collection

A small web app that lets families upload photos and video clips for possible
inclusion in the Old Mill School yearbook. No account, no login — scan a QR code
at a school event, pick photos, tap upload.

Uploads are stored in [Gumnut](https://gumnut.ai), where the yearbook committee
reviews and selects them. The app itself is upload-only: it never displays the
collection back to anyone.

## Status

Planning. See [`docs/PLAN.md`](docs/PLAN.md) for goals, supported and unsupported
use cases, privacy commitments, and the technical approach.

## Stack

- Cloudflare Workers (hosting + upload proxy)
- Gumnut (photo and video storage, organization, committee tooling)

## Contributing

This is a volunteer-maintained project for a single school. The code is public so
that it can be maintained by anyone the committee designates; no secrets live in
this repository.
