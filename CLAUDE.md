# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file, zero-dependency, no-backend web app for predicting the entire 2026 FIFA World Cup (group orders → third-place ranking → 31-match knockout bracket). The entire app — HTML, CSS, and JS — lives in `index.html`. There is no build step, no framework, and no server.

## Running & deploying

- **Run locally:** open `index.html` directly (`file://` works), or `python3 -m http.server 8000`.
- **Deploy:** static host. GitHub Pages (deploy `main`/root) or `npx vercel` (accept defaults). All asset paths are relative, so it works from a subpath.
- There are **no tests, no lint, and no build**. Editing means editing `index.html` and reloading the browser.

## Architecture (`index.html`)

The file is one `<style>` block, the HTML sections, and one `<script id="appjs">`. The app is a 5-section single-page flow (`hero → groups → thirds → bracket → share`), navigated by `go(id)` which toggles `.active` on `<section>`s. State lives in one module-global `picks` object:

```
picks = {
  groups:  {A:[t0,t1,t2,t3], ...},      // each group ordered 1st..4th
  thirds:  [groupLetter, ...],          // 12 entries, ranked best..worst (top 8 advance)
  winners: {matchNo: "top"|"bottom"},   // 31 knockout picks
}
```

Any pick change re-renders the affected section and calls `updateHash()`.

### The bracket engine (the core logic)

This is the part that requires reading several functions together:

- `GROUPS` / `GROUP_LETTERS` (A–L) — the 48 teams. `FLAG` maps team → emoji.
- `THIRD_TABLE` — FIFA's **official Annex C** lookup: all 495 combinations of which 8 third-place groups advance → which group winner plays which third. Key is the 8 advancing third-groups sorted+joined (e.g. `"ABCDEFGH"`); value is the assignment string read in `THIRD_WINNER_ORDER` (`A,B,D,E,G,I,K,L`). `thirdAssignments()` resolves winner-group → third-group from it.
- `R32` (Round-of-32 match list with slot refs like group winners/runners-up/thirds) + `TREE` (`R16/QF/SF/F`, each match's `from:[matchNo,matchNo]`) define the whole 104→103 match tree. Match numbers follow the official FIFA scheme (knockouts are 73–103).
- `buildBracket()` walks R32 then `TREE` in order, resolving each slot. `resolveSlot()` / `slotSeed()` turn a slot ref into an actual team; `advancerOf(no, M)` returns a match's chosen winner.
- Editing a group order or thirds ranking re-seeds the bracket and clears knockout picks; editing a knockout winner clears only downstream picks via `clearDownstream(no)` (uses a feeder map built from `TREE`).

### URL-hash serialization (the "save file")

There is **no storage** — the entire bracket is packed into the URL hash and that *is* the save file. `encodePicks()` folds everything into one BigInt by mixed radix, in this exact order: each group permutation (radix 24 = 4!), then the thirds permutation (radix 12!), then the 31 knockout picks (radix 3: 0=unset/1=top/2=bottom, in `KO_ORDER`). The BigInt is base64url-encoded with a 1-char `VERSION` prefix (`"A"`). `decodePicks()` reverses it **in exactly reverse order**. Permutations convert via `permToIndex`/`indexToPerm` (Lehmer-code style).

**Critical invariant:** if you change *what* is encoded, the order it's encoded in, the radices, or the `VERSION`, `encode`/`decode` must stay mirror images or every existing shared link breaks. Bump `VERSION` for any incompatible change.

### Drag-and-drop

`enableSortable(container, itemSel, dragClass, onReorder)` is a small custom (no-library) reordering helper; `makeSortable` (groups) and `makeThirdsSortable` wrap it and write the new order back into `picks` on drop.

## `results.json` and the results skill

`results.json` is a separate data file (an `{ "games": [...] }` object of *finished* World Cup matches) — it is **not** read by `index.html`; it's produced/updated by the bundled `worldcup-results` skill, which fetches from the worldcup26.ir API and appends only newly-finished, not-already-saved games (dedup keyed on match `id`). A finished game has `finished: "TRUE"` (a string). To update it, invoke the skill rather than hand-editing. See `.claude/skills/worldcup-results/` (`SKILL.md`, `references/api.md`, `scripts/save_results.py`).
