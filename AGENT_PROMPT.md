# Task: migrate 142 iW99 UI images off ImageKit → iw99-ui-assets repo

## 1. What to download

Every row in `MIGRATION_MAP.csv` in this repo. Columns: `category`,
`new_path`, `imagekit_url`, `used_in`, `action`, `note`. Download the exact
URL in `imagekit_url` for every row — 142 rows total.

## 2. Format conversion rule — read carefully, this is not just "convert everything"

Check the `action` column for each row, it already tells you what to do:

- `convert-to-webp` → download the file, convert it to `.webp`, save it.
  This covers `.png`, `.jpg`/`.jpeg`, and `.avif` sources.
- `keep-as-is (gif)` → download and save the raw bytes exactly as-is.
  **Do not** convert, re-encode, or touch this file in any way — it's an
  animated GIF and any re-encode risks flattening it to a single frame.
- `keep-as-is (already webp)` → same as above: download and save the raw
  bytes exactly as-is, no re-encoding. Some of these are animated webp —
  running them through a converter (even webp→webp) can silently strip
  animation. Treat this the same as the gif rule.

If you're not sure whether a specific `.webp` or `.gif` source is animated
or static, that doesn't matter — the rule is the same either way: don't
re-encode it, just save the original bytes.

## 3. File naming and placement

Save each file at the exact path in the `new_path` column, relative to the
repo root (e.g. `new_path = ui/vip/5.webp` → save to `ui/vip/5.webp` in this
repo). Do not rename, do not flatten folders, do not change case, even if a
name looks wrong to you (see the `note` column — a couple of names are
flagged as placeholder/unconfirmed on purpose, leave them as given).

## 4. Git workflow — read this carefully, this is a production-adjacent repo

- Clone `ccorryxx-bot/iw99-ui-assets`.
- Create a **new branch** off `main`, e.g. `agent/imagekit-migration`.
  **Never commit directly to `main`.**
- Add only the 142 files described above, each at its exact `new_path`. Do
  not modify, delete, or move any existing file (`wrangler.toml`,
  `README.md`, `MIGRATION_MAP.csv`, `.gitignore`, etc.) — new files only.
- Commit normally (no `--amend`), push the branch, **open a pull request**
  into `main`. **Do not merge the PR.** The repo owner reviews and merges
  it manually.
- **Never force-push** (`git push --force` / `-f`), never rewrite history
  (`rebase -i`, `filter-branch`), never delete branches other than your own
  working branch.
- This repo is wired to auto-deploy to a live Cloudflare Worker on push to
  `main`. Because you're only ever pushing to a feature branch and opening
  a PR, nothing you do can reach production without a human merging it —
  stay on that branch/PR workflow and that safety holds automatically.

## 5. If anything is ambiguous or a download fails

Skip that row, note it, keep going — don't guess. List any skipped/failed
rows in the PR description along with the reason (404, redirect, wrong
content-type, ambiguous format, etc.) so they can be followed up
individually. In particular, don't guess a "better" name for the two rows
already flagged as unconfirmed in the `note` column — leave those exactly
as given in `new_path`.
