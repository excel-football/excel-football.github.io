# Excel Football

A live scoreboard for a casual 4-team football session: 8-minute matches, with 3 points for a win, 1 for a draw and 0 for a loss. It started as an Excel sheet (`football_table.xlsx`) and is now a single web page that everyone at the pitch can open on their phone.

**Live page:** https://excel-football.github.io/

## Features

- **Team colors:** pick a shirt color (bib) for each of the 4 teams.
- **Fixed schedule:** 14 matches, plus 2 optional extra matches if time allows. If there is only time for one more match, pick two teams with rock-paper-scissors.
- **Points table:** standings update as scores are entered. Teams are ranked by points, then goal difference, and each team's wins, draws and losses are shown.
- **Next match:** the first unplayed match after the one just entered is highlighted. After the last row, it wraps back to the top.
- **Head-to-head:** a count of how many times each pair of teams meets.
- **Online sync:** everyone with the page open sees the same scores within a few seconds. Scores are also kept on the device, so the page still works offline and catches up when the connection returns.

## How it works

Everything is in `docs/index.html`: plain HTML, CSS and JavaScript, with no build step and no dependencies. GitHub Pages serves it from the `docs/` folder.

Scores are stored online in [keyjson-storage](https://api.diewland.com/jsondb/docs/) under the key `football`. Each page checks for changes every 3 seconds. When it saves, it re-reads the latest data and changes only the fields that were edited, so two people entering different matches don't overwrite each other.

> The write secret is in the page source, so anyone who opens the page can change the scores. This is fine for a casual scoreboard; don't store anything private there.

## Run locally

```sh
python3 -m http.server -d docs
```

Then open http://localhost:8000. The local page syncs with the same online data as the live page.

## Files

| File | Purpose |
|---|---|
| `docs/index.html` | The scoreboard web page |
| `football_table.xlsx` | The original Excel version (Sheet1 is the template, `backup` holds a past session's results) |
