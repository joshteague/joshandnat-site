# Subtext Capture Snippet — Install Report

Ran autonomously (headless mode). This report documents every decision and includes anything a human should follow up on.

## Step 1 — Pre-check

No existing Fullstory/Subtext snippet found. Checked (in order):

- `package.json` — does not exist. This repo has no build system or dependency manager.
- HTML entry points — no `fullstory.com/s/fs.js` or `_fs_script` string in any `.html` file.
- Init calls — no `init({ orgId` / `window['_fs_org']` in any `.js`/`.html` file.

Result: **not installed.** Proceeded with install.

## Step 2 — Explore

This repository is a **static mirror of a Squarespace site** (per `README`-less structure, `.nojekyll`, `CNAME`, and the Squarespace boilerplate/asset folders like `assets.squarespace.com`, `images.squarespace-cdn.com`). It's a wedding site (Josh and Natalie, June 2026), hosted on GitHub Pages via the `joshandnat.com` custom domain.

- **Framework:** None — plain static HTML, no framework, no `package.json`, no bundler. It's a multi-page site with **six independent HTML entry points**, each with its own `<head>` (no shared layout/include mechanism since it's a flat mirror):
  - `index.html`
  - `cart.html`
  - `registry.html`
  - `accommodations.html`
  - `thingstodo.html`
  - `welcomeparty.html`
- **CSP:** No `Content-Security-Policy` header or meta tag anywhere, and no hosting config (`netlify.toml`, `vercel.json`, `_headers`, `.htaccess`) that sets one. GitHub Pages does not let this repo set custom response headers either. **No CSP changes needed.**
- **Existing analytics tools:** None found. No `package.json` to check, and a grep across all HTML for common SDKs (GA/gtag, GTM, Segment, PostHog, Mixpanel, Amplitude, Sentry, Hotjar, Clarity) turned up no real matches (one false positive: `amplitudeY`/`amplitudeZ`, which are CSS animation properties, not the Amplitude SDK). No custom analytics wrapper module exists either (there's no `/lib/`, `/utils/`, or `/analytics/` — the repo is just the mirrored HTML plus vendor asset folders).

## Step 3 — Plan

1. **Snippet placement:** Insert the snippet as the first child of `<head>` in all six HTML files, immediately after the `<head>` tag and before the existing `<meta http-equiv="X-UA-Compatible">` line (identical boilerplate at line 3-4 in every file), so capture starts as early as possible.
2. **CSP changes:** None needed — no CSP is configured anywhere in this project or its hosting layer.
3. **User identification:** No auth/login system exists in this static repo — it's a public, unauthenticated wedding site. There is no place in the codebase where "the authenticated user becomes available." **Skipped** (see Open Questions).
4. **Analytics tool linkage:** No analytics tools detected in Step 2. **Skipped** — nothing to link into.
5. **Privacy tags:** Scanned all six files for form inputs, textareas, and PII-rendering elements (RSVP forms, checkout/payment fields, guest notes, etc.). **Zero found** in the static markup — see Open Questions, this is a mirroring artifact rather than evidence the live site has no PII surfaces.
6. **Open questions:** listed below.

Plan applied directly (autonomous mode — no human available to approve mid-run).

## Step 4 — Install

Inserted the exact snippet (unmodified — org id `o-256EN8-na1`, host `fullstory.com`, script `edge.fullstory.com/s/fs.js`) as the first element inside `<head>` in:

- `index.html`
- `cart.html`
- `registry.html`
- `accommodations.html`
- `thingstodo.html`
- `welcomeparty.html`

No CSP changes were needed or made.

## Step 5 — Identify users

**Skipped.** This is a public, unauthenticated static wedding site — there is no login, session, or account system in this codebase to hook a `setIdentity` call into. If you later add an RSVP account system or gated content, revisit this step and call `FS('setIdentity', ...)` from wherever that authenticated user becomes available.

## Step 6 — Link analytics

**Skipped.** No analytics, session-replay, error-monitoring, or feature-flag SDKs were found anywhere in the repo to attach a `subtext_url`/`subtextUrl` property to.

## Step 7 — Mask sensitive data

Searched all six HTML files for `<input>`, `<textarea>`, `<form>`, and PII-related attributes/keywords (email, phone, name, address, payment, billing, card, checkout, RSVP). **Found none** — this repo's `cart.html` and any RSVP flow (linked from `.../rsvp`, not part of this mirror) contain no static form markup.

This is expected for a Squarespace **static mirror**: Squarespace's real RSVP form and cart/checkout UI (name, email, guest count, shipping/billing address, payment fields) are injected at runtime by Squarespace's own hosted JavaScript (and likely rendered in a same-origin block or a Squarespace-hosted iframe), not present in the pre-rendered HTML that was mirrored into this repo. `masked_count: 0` is accurate for the files as they exist here, but it does **not** mean the live, deployed site has no PII surfaces.

## Open questions / follow-ups for a human

1. **RSVP form and cart/checkout PII.** The live site (once deployed) almost certainly renders an RSVP form (name, email, guest names) and a Squarespace commerce cart/checkout (shipping address, payment details) via client-side JS that isn't present in this static mirror. Recommend: after deploying this install, use a Subtext session review (`subtext:live` / `subtext:review`) against the **live** `/cart` and `/rsvp` pages to see what actually renders, then add `.fs-mask` / `.fs-exclude` classes via Squarespace's custom CSS/code injection (Settings → Advanced → Code Injection) targeting those elements — this repo mirror can't be edited to add those classes since the real forms don't live in this HTML.
2. **No user identification is possible** in this repo's current form — flag if a future account/RSVP-login feature is added.
3. **GitHub Pages cannot serve a `Content-Security-Policy` response header** — if a CSP is ever added via a `<meta>` tag in these HTML files, it will need the `edge.fullstory.com`, `rs.fullstory.com` allowances described in the reference docs.
4. Since this repo is a **periodically-regenerated mirror** of the live Squarespace site, confirm whether it's regenerated in a way that would overwrite these six `<head>` edits on the next sync/export. If so, the snippet insertion should instead be added at the Squarespace source (Settings → Advanced → Code Injection → Header) so it survives re-mirroring, and this repo's copies would just reflect that.

## Step 8 — Explain

The Subtext snippet is installed. Once deployed, sessions will start capturing on your next page load — DOM snapshots, clicks, scrolls, network requests, and console output.

No privacy tags (`.fs-mask`/`.fs-exclude`) were added in this pass, since no PII-rendering markup exists in the static mirror — see the RSVP/checkout follow-up above. Any privacy tags added in the future only take effect on **new sessions captured after deploy**; they do not retroactively mask sessions already captured.
