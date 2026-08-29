# iw99-ui-assets

iW99 UI assets (VIP badges, avatars, payment icons, banners, provider logos,
etc.) — Cloudflare Worker static assets, replaces ImageKit for these files.

This is intentionally a **separate repo/Worker/domain from `game-assets2`**
(which only serves game thumbnails) and from `imbet69-ui-assets` (iMBET69's
own UI assets — different site, different account, kept isolated on
purpose). Keeping this iW99-only traffic on its own Worker means a bad
file/deploy here can never affect game thumbnails or another site's UI
assets, and vice versa.

## Structure

All files live under `ui/`, one folder per category:

```
ui/
  vip/          51 files  — VIP tier badges, 0.webp .. 50.webp
  avatar/       40 files  — account avatar picker, 1.webp .. 40.webp
  agent/         5 files  — agent-referral avatar picker
  provider/      5 files  — game provider brand logos (pp/jili/jdb/pg + 1 unidentified)
  category/      6 files  — home page category filter icons
  payment/       4 files  — deposit/withdraw method icons
  fab/           4 files  — home page floating action buttons
  banner/        3 files  — home page banner carousel
  social/        5 files  — footer/account social icons
  spinwheel/     3 files  — spin wheel modal graphics
  account/       5 files  — account page quick-action icons
  badge/         1 file   — animated "hot" badge overlay (GIF, do not touch)
  promo/         6 files  — promotions page / daily sign-in cards
  brand/         3 files  — site logo (default, agent dashboard, sidebar)
  app/           1 file   — splash screen logo
```

`MIGRATION_MAP.csv` is the source of truth for what goes where — see that
file for the exact original ImageKit URL each file replaces and which
component(s) use it.

## Conversion rule

- `.png` / `.jpg` / `.jpeg` / `.avif` source → convert to `.webp` on upload
- `.gif` source → keep as `.gif`, **never** re-encode (breaks animation)
- source already `.webp` → keep as-is, **never** re-encode (some of these
  are animated webp — re-encoding can silently flatten them to one frame)

## Known open items (see MIGRATION_MAP.csv notes column)

- `ui/provider/UNKNOWN-license-badge-5.webp` — file is real and in use
  (HomePage.vue footer strip) but nothing in the code identifies which
  provider/license it is. Rename after a visual check.
- `ui/social/messaging-icon.webp` — same source file is used as "WA"
  (WhatsApp) in one place and "Viber" in another. Likely a copy-paste
  mismatch in the app code, not an asset problem — flagged for the app repo,
  not fixed here.
- Several source files are `.avif`, which the original conversion rule
  (png/jpg → webp, gif/webp → keep) didn't cover. Defaulted to
  convert-to-webp for consistency with everything else. Flag if that's wrong
  for any specific file.

## Deploy

Wired to Cloudflare Workers static assets (`wrangler.toml`, `directory =
"./"`). Push to `main` → auto-deploy. iW99's `vercel.json` proxies
`/img/ui/*` to this Worker's domain so the app never hits it directly (same
reasoning as the `/img/thumb/*` → `game-assets2` proxy already in place).
