# OTBE Release Workflow — Step-by-Step

You can literally go down this file and tick off steps.

## 1. Prepare GitHub

1. Create a new repository, e.g. `oauth-trust-binding-extension`.
2. Unzip this package and copy all files into the repo.
3. Commit and push:
   - `README.md`
   - `draft/`
   - `docs/`
   - `OUTREACH/`

After that, you will have a permanent `<GITHUB_LINK>`.

## 2. Submit Internet-Draft to IETF

1. Go to https://datatracker.ietf.org/
2. Create an account, if you don’t have one.
3. Choose “Submit New Internet-Draft”.
4. Use filename: `draft-fulz-oauth-trust-binding-00`.
5. Upload `draft/draft-fulz-oauth-trust-binding-00.md` (Markdown) or the XML if preferred.
6. Complete metadata (your name, email, etc.).

After submission you’ll get an `<IETF_LINK>` for the draft.

## 3. Contact OAuth WG

1. Open `OUTREACH/email_oauth_wg.txt`.
2. Replace `<IETF_LINK>` and `<GITHUB_LINK>`.
3. Send to:
   - `oauth@ietf.org`

## 4. Contact Industry Identity Teams

1. Open `OUTREACH/email_industry.txt`.
2. Replace `<IETF_LINK>` and `<GITHUB_LINK>`.
3. Send to:
   - Google: security@google.com, oidc-feedback@google.com
   - Microsoft: secure@microsoft.com
   - Auth0 / Okta: security@okta.com
   - GitHub: security@github.com
   - Cloudflare: security@cloudflare.com

(You can adjust list as you like.)

## 5. Contact Regulators

1. Open `OUTREACH/email_regulators.txt`.
2. Replace `<IETF_LINK>` and `<GITHUB_LINK>`.
3. Send to:
   - European Data Protection Supervisor (EDPS) contact form
   - EDPB (European Data Protection Board) – general contact
   - National data protection authority (e.g. BfDI in Germany)

## 6. Contact Media

1. Open `OUTREACH/email_media.txt`.
2. Replace `<IETF_LINK>` and `<GITHUB_LINK>`.
3. Send to:
   - Heise Security: redaktion@heise.de
   - Ars Technica: tips@arstechnica.com
   - Schneier on Security: public contact (blog submissions)
   - Others as you see fit

## 7. Follow-Up

- Watch for responses on:
  - IETF datatracker (comments, reviews)
  - OAuth WG mailing list
  - GitHub issues
- Iterate draft version: `-01`, `-02`, etc.
- Optionally, submit a conference talk proposal using the docs.
