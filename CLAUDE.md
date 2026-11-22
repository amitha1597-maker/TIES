# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A clickable MVP prototype of **TIES** (Tactical Intelligence Explainability System), built from a course product spec. A tactical commander picks a start and target on a map, gets two route alternatives with a risk trade-off matrix and an explainability card per threat, picks a route, and sends one-tap field feedback that updates indicator weights.

**Everything is mock data.** The map is a schematic, invented grid (no real place, no coordinates), and the unit, sources, threats and scenario ("חומת אש") are fictional. The footer says so. Keep the tactical detail at the level of the spec, and don't add real places or real intelligence.

Everything lives in `index.html`: inline `<style>` and `<script>`, no build step, no package manager, no tests, and **no external dependencies at all** (system fonts only), so it also works offline. The page is `lang="he" dir="rtl"`. Keep copy in Hebrew and use the Hebrew geresh/gershayim characters (`א׳`, `קמ״ן`) rather than ASCII quotes, which also keeps JS strings safe.

**Note:** the parent folder's `CLAUDE.md` and `אתר דמה/CLAUDE.md` describe other sites. Their content rules and subagents don't apply here.

## Preview

Python is the Windows Store stub and Node isn't installed. Use the parent folder's server (it serves subfolders):

```
powershell -NoProfile -ExecutionPolicy Bypass -File ..\serve.ps1 -Port 8765
```

Then open http://localhost:8765/TIES/index.html. Opening `index.html` directly also works, since there are no fetches.

## Repository and deploy

This folder is its own git repo (the parent folders aren't), pushed to the **public** GitHub repo `amitha1597-maker/TIES` on branch `main`. GitHub Pages serves `main` from the repo root at https://amitha1597-maker.github.io/TIES/, so every push to `main` goes live within a minute or two. Because the repo and site are public, everything committed here must stay mock data. `README.md` is the public Hebrew description: keep it in sync when features change.

## How `index.html` is organized

- **Data (top of `<script>`) is the source of truth:** `KINDS` (edge type → speed in m/min and exposure factor), the grid built from `COLS`/`ROWS` into `nodes` and `edges` (ids `h{row}{col}` horizontal, `v{row}{col}` vertical; row 0 = courtyards and the jump house `h02`, row 1 = main street, row 2 = southern alley), `windows` (dominant windows on the main street), `THREATS`, `PRIORS` (Beta(a,b) per indicator type), `FEED` (raw items for the ingestion step), `FB` (feedback buttons).
- **Engine:** `dijkstra()` gives route A (min time) and route B (time + 0.05·exposure, with a penalty on A's edges so it's distinct). `evalRoute()` computes length, time and threats; a threat applies if any of its `edges` is on the route. Threat risk = `sev × prob × conf`, route risk = `1 − Π(1 − risk)`, and the recommendation is the lower `risk + 0.01·time`. `probOf()` holds the special cases: sniper probability is combined over the route's windows, with the top `S.barrels` covered (`COVER_FACTOR`); the blue-barrel probability is the live Bayesian weight; engineering clearing cuts the door charge ×0.1 and adds 12 minutes.
- **Scenario tuning:** with the defaults (4 barrels, no clearing) TIES recommends route A (main street with cover), as in the spec's scenario. Dropping barrels to 0 and turning on clearing flips it to B. Keep that behavior if you retune numbers.
- **State:** `S` is session state (step, routes, selection). `P` is persisted in `localStorage` (`ties-state-v1`, wrapped in try/catch): online flag, sync queue, reports, indicator weights, KPI samples.
- **Rendering:** `render()` redraws the top bar plus the current view: `renderStepper`/`renderPanel` (`step1`…`step5`)/`renderMap` for the mission, `renderDebrief`, `renderMetrics`. All buttons use `data-act` and one delegated click handler.
- **Offline / Delta sync:** the top-bar toggle simulates link loss. Offline reports go to `P.queue`; going online calls `syncNow()`, which builds the delta JSON shown in the metrics tab.
- **Layout:** tokens on `:root`, breakpoints `900px` (single column, map not sticky) and `560px` (phone). Check there's no page-level horizontal scroll at 375px. Motion is disabled under `prefers-reduced-motion`.
