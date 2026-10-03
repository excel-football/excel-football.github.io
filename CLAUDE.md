# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A scoreboard for a casual 4-team football session (8-minute matches; win 3 / draw 1 / loss 0). It started as an Excel workbook (`football_table.xlsx`). `docs/index.html` is a web port of that workbook, meant to be served by GitHub Pages from the `docs/` folder. The UI text is in Thai.

There is no build, no dependencies, no tests and no linter. To develop, open `docs/index.html` in a browser, or serve it with something like `python3 -m http.server -d docs`.

## Architecture

`docs/index.html` holds everything in one file: inline CSS, an inline vanilla-JS IIFE (ES5 style: `var` and `function`, no modules), and Google Fonts (Bai Jamjuree) as the only external resource.

- **Team slots and colors:** the logic only works with slots A–D (indices 0–3). `state.teams[slot]` maps each slot to a key in `COLORS` (English key → Thai label plus bib colors). The schedule never mentions colors.
- **`FIX`:** the fixed 16-match schedule as `[slotIndex, slotIndex]` pairs, copied from Sheet1. Matches 1–14 are the main schedule. Matches 15–16 are optional extras, played only if time allows. Code depends on index 14 as that boundary: the rule banner is inserted there, the "next match" search covers 0–13, and the head-to-head counts use `FIX.slice(0,14)`.
- **State:** `{teams, scores}` is saved to `localStorage` under the key `football_table_v1`. `scores[i]` is `[goalsTeam1, goalsTeam2]` stored as strings, and `""` means not played yet. If you change the shape of the state, bump the key.
- **Rendering:** each section (`renderTeams`, `renderStandings`, `renderFixtures`, `renderH2H`) rebuilds its container's `innerHTML` and re-attaches its listeners. When a score is typed, only the standings re-render, so the input keeps focus. The fixtures re-render on `change`.
- **Standings sort order:** points, then goal difference, then slot order.
- **`BACKUP`:** hard-coded results from the workbook's `backup` sheet, shown read-only in a `<details>` block.
- **Theming:** color tokens are defined on `:root`, with dark-mode overrides under both `prefers-color-scheme` and `[data-theme="dark"]`. At ≤560px the grids switch to 2 columns.

## The workbook

`football_table.xlsx` is the source the page was ported from:
- **Sheet1:** the live template. Team names are in N4:Q4 (with fallback to the slot letters in N3:Q3). Scores are typed as `"x-y"` strings in column G. Points per team go in I:L, and the totals are in row 22.
- **backup:** a past session's results.
- **Sheet2:** scratch scores.

Sheet1's formulas parse scores with `MID(G,1,1)` and `MID(G,3,1)`, so they only handle single-digit scores. Match 9's row (row 12) has no points formula for team A. The web page computes everything from `FIX`, so neither limitation affects it. If you change the schedule, keep `FIX` in sync with Sheet1 columns E/F.
