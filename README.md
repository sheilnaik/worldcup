# World Cup 2026 Pick'Em

A single-file, no-backend web app for predicting the entire 2026 FIFA World Cup:

1. **Order the Groups** — drag the 4 teams in each of the 12 groups into your predicted finishing order (1st–4th). 1st & 2nd advance directly.
2. **Rank the Thirds** — drag to rank all 12 third-place teams; only the top 8 advance.
3. **Fill the Bracket** — the Round of 32 auto-populates from your group orders using FIFA's **official Annex C** third-place allocation table (all 495 combinations). Tap any matchup to advance a team; winners flow forward automatically through the Round of 16, Quarter-finals, Semi-finals, and Final.
4. **Share** — your entire bracket is compressed into the page URL (the `#…` hash). Copy the link, save it, send it. Reopen it anytime and every pick is restored. Nothing is stored on any server.

The Final is July 19, 2026 at MetLife Stadium, East Rutherford, NJ.

## How sharing works

Every pick — 12 group orders, the third-place ranking, and all 31 knockout winners — is packed into a single big integer and base64url-encoded into the URL hash. A complete bracket is about **11 characters** long. There is no database and no analytics; the link *is* the save file.

## Running locally

It's one self-contained `index.html`. Either:

- **Open the file directly** — double-click `index.html` (works over `file://`), or
- **Serve it** — `python3 -m http.server 8000` then visit `http://localhost:8000`.

No build step, no dependencies. Fonts load from Google Fonts (needs internet for the exact display typeface; falls back gracefully offline).

## Deploying

### GitHub Pages
1. Push `index.html` to a repo.
2. Settings → Pages → deploy from the `main` branch, root folder.
3. Your app is live at `https://<you>.github.io/<repo>/`.

(All paths are relative, so it works from a project subpath.)

### Vercel
From this folder:
```bash
npx vercel        # or: vercel deploy
```
Accept the defaults — it's a static site, no framework or build config needed. Or drag the folder into the Vercel dashboard.

## Notes on the bracket rules

- Group **winners** are paired against **third-place** teams; **runners-up** play other runners-up (with four winner-vs-runner-up matches: C/F and H/J swaps), exactly per the official 2026 bracket.
- The specific third-place opponent for each group winner is resolved from FIFA's pre-set 495-row lookup table once the 8 advancing third-place groups are known — here, that's your top-8 ranked thirds.
- Changing a group order or third-place ranking re-seeds the bracket and clears knockout picks (since the matchups change). Changing a knockout winner clears only the now-invalid picks downstream of it.

*Unofficial fan project. Team list and bracket structure per the 2026 FIFA World Cup format.*
