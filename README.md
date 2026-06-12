# World Cup 2026 Pick'Em

A single-file, no-backend web app for predicting the entire 2026 FIFA World Cup:

1. **Order the Groups** — drag the 4 teams in each of the 12 groups into your predicted finishing order (1st–4th). 1st & 2nd advance directly.
2. **Rank the Thirds** — drag to rank all 12 third-place teams; only the top 8 advance.
3. **Fill the Bracket** — the Round of 32 auto-populates from your group orders using FIFA's **official Annex C** third-place allocation table (all 495 combinations). Tap any matchup to advance a team; winners flow forward automatically through the Round of 16, Quarter-finals, Semi-finals, and Final.
4. **Share** — your entire bracket is compressed into the page URL (the `#…` hash). Copy the link, save it, send it. Reopen it anytime and every pick is restored. Nothing is stored on any server.

The Final is July 19, 2026 at MetLife Stadium, East Rutherford, NJ.

## Live scoring

A **Live Scorecard** (its own stepper step, and shown again on the Share screen) scores your picks against real results as the tournament unfolds. It reads `results.json` — a file of finished matches sitting next to `index.html` — and awards points **only for outcomes that have actually been decided**, so your tally grows game by game. Nothing is scored for undecided matches; a group is only scored once all 6 of its games are final.

The points schedule:

**Part 1 — Group order** (per team, once the group is complete)
- Exact 1st → **4**, exact 2nd → **3**, exact 3rd → **2**, exact 4th → **2**
- Right side but wrong slot (you had them advancing and they did, or eliminated and they were) → **1**

**Part 2 — Top-8 third-place picks** (once the 8 advancing thirds are known)
- Each team you ranked top-8 that really advanced → **4**
- A team you ranked top-8 that finished 3rd but missed the cut → **2** (consolation)
- All 8 exactly right → **+5** bonus

**Part 3 — Knockout bracket** (per correctly-picked winner of a decided match)
- R32 → **2**, R16 → **4**, QF → **8**, SF → **16**, Final → **32**
- Champion correct → **+32** (stacks on the Final, so nailing the champion's last game is worth 64)

Knockout games are matched by the two teams your bracket placed in each slot, so the results feed's match numbering never has to line up with the app's. The scorecard updates the moment a group locks or a knockout is decided.

### Keeping `results.json` fresh

`results.json` is the live feed. Refresh it from the World Cup API with the bundled `worldcup-results` skill (or any source that writes the same shape — an `{ "games": [...] }` object of finished matches with English team names matching the app, string scores, and `finished: "TRUE"`), then commit and push. The app fetches it with a cache-busting query param, so a browser refresh always reflects the newest file even behind GitHub Pages' CDN cache.

> **Note:** scoring needs the page served over **http(s)** — GitHub Pages, Vercel, or a local server all work. Opening `index.html` straight off disk (`file://`) blocks the fetch in most browsers, and the scorecard will say the feed is unavailable; everything else still works offline.

## How sharing works

Every pick — 12 group orders, the third-place ranking, and all 31 knockout winners — is packed into a single big integer and base64url-encoded into the URL hash. A complete bracket is about **11 characters** long. There is no database and no analytics; the link *is* the save file.

## Running locally

It's one self-contained `index.html`. Either:

- **Open the file directly** — double-click `index.html` (works over `file://`), or
- **Serve it** — `python3 -m http.server 8000` then visit `http://localhost:8000`.

No build step, no dependencies. Fonts load from Google Fonts (needs internet for the exact display typeface; falls back gracefully offline).

## Deploying

### GitHub Pages
1. Push `index.html` **and `results.json`** (side by side) to a repo.
2. Settings → Pages → deploy from the `main` branch, root folder.
3. Your app is live at `https://<you>.github.io/<repo>/`.

(All paths are relative, so it works from a project subpath.) To update scores mid-tournament, refresh `results.json` and push again — Pages redeploys and players see new scores on their next refresh.

### Vercel
From this folder:
```bash
npx vercel        # or: vercel deploy
```
Accept the defaults — it's a static site, no framework or build config needed. Or drag the folder into the Vercel dashboard. Deploying the folder ships `results.json` alongside `index.html`, so scoring works out of the box.

## Notes on the bracket rules

- Group **winners** are paired against **third-place** teams; **runners-up** play other runners-up (with four winner-vs-runner-up matches: C/F and H/J swaps), exactly per the official 2026 bracket.
- The specific third-place opponent for each group winner is resolved from FIFA's pre-set 495-row lookup table once the 8 advancing third-place groups are known — here, that's your top-8 ranked thirds.
- Changing a group order or third-place ranking re-seeds the bracket and clears knockout picks (since the matchups change). Changing a knockout winner clears only the now-invalid picks downstream of it.

*Unofficial fan project. Team list and bracket structure per the 2026 FIFA World Cup format.*
