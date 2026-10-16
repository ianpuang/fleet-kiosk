# WeMAL Fleet Health kiosk

Static kiosk page served by Cloudflare Pages from a private repo. A GitHub Action pulls the published
maintenance sheet (CSV) every 3 hours, commits a snapshot to `data/`, and redeploys.
The open kiosk page re-reads the snapshot every 15 minutes, so it picks up each sync
without a manual refresh.

## Files
- `index.html` — the kiosk (two screens, rotates every 20 s, scales to any 16:9 display)
- `data/fleet.csv` — latest sheet snapshot (written by the Action)
- `data/updated.txt` — UTC time of the last sync (drives the "Synced X h ago" footer)
- `.github/workflows/sync-and-deploy.yml` — fetch + commit + deploy, every 3 h

## Setup
1. Cloudflare dashboard → My Profile → API Tokens → Create token with
   **Account → Cloudflare Pages → Edit**. Note your Account ID (Workers & Pages overview).
2. GitHub repo → Settings → Secrets and variables → Actions → add
   `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID`.
3. Actions tab → "Sync fleet sheet and deploy" → **Run workflow**. This creates the
   `fleet-kiosk` Pages project and deploys it to `https://fleet-kiosk.pages.dev`.
4. Cloudflare → Workers & Pages → fleet-kiosk → Custom domains → add `fleet.ianpuang.com`.
5. Optional: Cloudflare Zero Trust → Access → protect the hostname so only approved
   people/devices can view it (the page is otherwise public to anyone with the URL).
6. Open the URL on the kiosk, full screen (F11 / kiosk mode).

## Rules (edit `CONFIG` at the top of the script in index.html)
- Overdue: current mileage past next service (or past the date for date-only vehicles)
- Due soon: ≤ 2,000 km or ≤ 14 days left, or the sheet's status says DUE
- Check data: odometer more than 20,000 km past next service (likely a typo)
- No data: no current mileage entered

## Sheet rules
Don't rename the header row. Columns are matched by name, so reordering is fine,
renaming is not. If the header breaks, the Action fails and keeps the last good data;
the kiosk shows a "not synced" warning after 7 hours.
