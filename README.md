# WeMAL Fleet Health kiosk

Static kiosk page served by GitHub Pages. A GitHub Action pulls the published
maintenance sheet (CSV) every 5 minutes, commits a snapshot to `data/` when the sheet
changed, and redeploys. The open kiosk page reads the published sheet directly every 2 minutes
(Google refreshes the published CSV about every 5 minutes) and falls back to the
`data/` snapshot if Google can't be reached. Sheet edits show on the kiosk within
about 2-7 minutes. The snapshot is only a backup, so GitHub's late scheduled runs don't matter.

## Files
- `index.html` — the kiosk (two screens, rotates every 20 s; portrait 1080x1920 layout for the standing kiosk, landscape 1920x1080 otherwise)
- `data/fleet.csv` — latest sheet snapshot (written by the Action)
- `data/updated.txt` — UTC time of the last sync (drives the "Synced X h ago" footer)
- `.github/workflows/sync-and-deploy.yml` — fetch + commit (on change) + deploy, every 5 min

## Setup
Live at https://ianpuang.github.io/fleet-kiosk/ (public repo, GitHub Pages via Actions).
Open it on the kiosk full screen (F11 / kiosk mode). For a manual refresh:
Actions tab → "Sync fleet sheet and deploy" → **Run workflow**.

## Rules (edit `CONFIG` at the top of the script in index.html)
- Overdue: current mileage past next service (or past the date for date-only vehicles)
- Due soon: ≤ 2,000 km or ≤ 14 days left, or the sheet's status says DUE
- Check data: odometer more than 20,000 km past next service (likely a typo)
- No data: no current mileage entered

## Sheet rules
Don't rename the header row. Columns are matched by name, so reordering is fine,
renaming is not. If the header breaks, the Action fails and keeps the last good data;
the kiosk shows a "not synced" warning after 2 hours.
