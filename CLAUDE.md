# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A scoreboard for a casual 4-team football session (8-minute matches; win 3 / draw 1 / loss 0). It started as an Excel workbook (`football_table.xlsx`). `docs/index.html` is a web port of that workbook, meant to be served by GitHub Pages from the `docs/` folder. The UI text is in Thai.

There is no build, no dependencies, no tests and no linter. To develop, open `docs/index.html` in a browser, or serve it with something like `python3 -m http.server -d docs`.

## Architecture

`docs/index.html` holds everything in one file: inline CSS, an inline vanilla-JS IIFE (ES5 style: `var` and `function`, no modules), and Google Fonts (Bai Jamjuree) as the only external resource.

- **Team slots and colors:** the logic only works with slots A–D (indices 0–3). `state.teams[slot]` maps each slot to a key in `COLORS` (English key → Thai label plus bib colors). The schedule never mentions colors.
- **`FIX`:** the fixed 16-match schedule as `[slotIndex, slotIndex]` pairs, copied from Sheet1. Matches 1–14 are the main schedule. Matches 15–16 are optional extras, played only if time allows. Code depends on index 14 as that boundary: the rule banner is inserted there, the fallback "next match" search covers 0–13, and the head-to-head counts use `FIX.slice(0,14)`.
- **State:** `{teams, scores, last, rev}` is saved to `localStorage` under the key `football_table_v1`. `scores[i]` is `[goalsTeam1, goalsTeam2]` stored as strings, and `""` means not played yet. `last` is the index of the match whose result was most recently completed (it may be missing in older saved state). If you change the shape of the state in a way that is not backward compatible, bump the key.
- **Online sync:** the same state object is the whole value of the `football` key in keyjson-storage (`https://api.diewland.com/jsondb/`, docs at `/jsondb/docs/`). The write secret is in the page source, so anyone who opens the page can write. The API has no push and no partial updates:
  - Every page polls every 3 s while visible, reading only `rev` (`GET ?name=football&key=rev`). When it differs from the local `rev` (or can't be read), `pullFull()` fetches the whole value and adopts it. Polls are skipped while local edits are unsent (`dirty`).
  - Each edit calls `save(keys)` with the fields it touched (`"s<match>.<side>"`, `"t<slot>"`, `"last"`). About 400 ms later, `push()` re-reads the online value, writes only those fields onto it, and POSTs the result. This lets two devices edit different matches, though two pushes landing within the same round trip can still lose one.
  - If the online value isn't valid scoreboard state, the page publishes its own state.
  - `renderShared()` re-renders after online changes, but leaves the fixtures alone while a score input has focus.
  - Any change to the state shape must keep `valid()`, `getField`/`setField` and the stored online value compatible.
- **Next-match highlight:** it marks the first unplayed match after `last`, wrapping to the top after match 16, and marks nothing if every match is played. If `last` is unset or its result was cleared, it marks the first unplayed match among 1–14.
- **Rendering:** each section (`renderTeams`, `renderStandings`, `renderFixtures`, `renderH2H`) rebuilds its container's `innerHTML` and re-attaches its listeners. When a score is typed, only the standings re-render, so the input keeps focus. The fixtures re-render on `change`.
- **Standings sort order:** points, then goal difference, then slot order.
- **Theming:** color tokens are defined on `:root`, with dark-mode overrides under both `prefers-color-scheme` and `[data-theme="dark"]`. At ≤560px the grids switch to 2 columns.

## The workbook

`football_table.xlsx` is the source the page was ported from:
- **Sheet1:** the live template. Team names are in N4:Q4 (with fallback to the slot letters in N3:Q3). Scores are typed as `"x-y"` strings in column G. Points per team go in I:L, and the totals are in row 22.
- **backup:** a past session's results.
- **Sheet2:** scratch scores.

Sheet1's formulas parse scores with `MID(G,1,1)` and `MID(G,3,1)`, so they only handle single-digit scores. Match 9's row (row 12) has no points formula for team A. The web page computes everything from `FIX`, so neither limitation affects it. If you change the schedule, keep `FIX` in sync with Sheet1 columns E/F.
