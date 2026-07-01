# UT Buzz 🤘

**Live campus crowd levels for UT Austin — see where the seats are before you walk there.**

[**Live demo →**](https://stevenwimberly.github.io/buzz/) · [Proposal (PDF)](#) · Built by [Steven Wimberly](https://stevenwimberly.github.io)

UT Austin has ~55,000 students competing for a finite supply of study, dining, and gym space. The campus doesn't lack capacity — it lacks *visibility* into where capacity exists. Students routinely lose 15+ minutes a day walking between full buildings looking for a seat; across the student body that's roughly **2.5 million student-hours per year**. Buzz answers one question: *where can I actually get a seat right now?*

> **Status:** interactive prototype with simulated occupancy data. Built to demonstrate the full product experience for a proposed pilot partnership with UT administration. See [Roadmap](#roadmap).

## Features

| Feature | Description |
|---|---|
| **Live map** | Leaflet map of campus with color-coded occupancy pins (green → red) that update in real time |
| **Building detail** | Occupancy %, live headcount vs. capacity, hours, walk time from your location |
| **Predictions** | Trend indicator (filling up / clearing out) and a computed "best time to go" for the next 8 hours |
| **Hourly chart + weekly heatmap** | Per-building busyness by hour of day and day of week, with a live NOW marker |
| **Smart alternatives** | When a space is full, Buzz ranks nearby same-category spaces by a combined crowd + walk-time score |
| **Buzz reports** | Crowdsourced status updates ("Level 6 has open desks") with tagged statuses and aging timestamps |
| **Alerts** | Arm a below-50% threshold on any building; a banner fires when the occupancy simulation crosses it |
| **Saved spaces** | Favorite buildings, sorted by availability, with a suggestion feed |
| **Search & filters** | Free-text search plus category filters (study / dining / gym) synced between map and list |

## Architecture

Single-file vanilla HTML/CSS/JS — no framework, no build step. The only dependency is [Leaflet 1.9.4](https://leafletjs.com/) (CDN) with OpenStreetMap tiles.

```
index.html
├── Data model        buildings[] — id, coords, type, capacity, hours, base occupancy
├── Pattern engine    deterministic hourly/weekly busyness curves per building
├── Simulation loop   7s tick: drift toward time-of-day target + noise
├── Renderers         map pins, list cards, detail modal, saved, profile
└── State             in-memory: favorites, alerts, reports, filters
```

### How the simulated data works

Occupancy isn't random — it's generated so the demo behaves like a real campus:

- **Busyness curves.** Each building gets a 24-hour curve per day of week: a Gaussian falloff around type-specific peak hours (dining peaks at 12/18h, gyms at 7/17/19h, study spaces at 14/19/21h), a weekend damping factor (academic spaces drop to 55%, gyms only to 85%), late-night damping, and deterministic noise from a string hash — so PCL's Tuesday curve is identical on every page load, but no two buildings look alike.
- **Live drift.** Every 7 seconds, each building's occupancy moves 12% of the way toward its current-hour target plus ±2% noise, clamped to [2, 99]. Pins recolor, lists re-sort, and an open detail page updates its numbers in place. Armed alerts fire when a building crosses below 50%.
- **Walk times** are computed from a fixed user location near Speedway using flat-earth degree→feet conversion at Austin's latitude and a 280 ft/min pace.

In production, the pattern engine becomes the *prediction* layer (trained on historical counts) and the simulation loop is replaced by real data — see below.

### Production data plan

The pilot architecture (detailed in the [full proposal](#)) aggregates three sources, none of which involve personal data:

1. **Card-swipe counts** — dining halls and gyms already meter entry; UT Facilities exposes aggregated, anonymized totals via API
2. **Infrared door counters** — commodity sensors at study-space entrances counting anonymous transits
3. **Crowdsourced reports** — the Buzz Reports layer, for intra-building granularity sensors can't see (e.g., which floor of PCL has seats)

**Privacy is a design constraint, not a feature:** building-level aggregation only, no identity-linked data anywhere in the pipeline, raw sensor data purged on a 30-day rolling basis, FERPA/ISO review before any real data collection.

## Running locally

No toolchain needed:

```bash
git clone https://github.com/stevenwimberly/ut-buzz.git
cd ut-buzz
python3 -m http.server 8000   # or just open index.html
```

Then open `http://localhost:8000`. Requires internet access for Leaflet + OSM tiles.

## Demo tips

- Watch any pin for ~30 seconds — occupancy drifts live
- Open **Gregory Gym** (usually ~97%) → check *Try instead* and *Best time to go*
- Arm the **notify below 50%** toggle on a busy building, then wait for the banner
- Post a **Buzz report** from any building's detail page
- Star buildings and check the **Saved** tab badge

## Roadmap

- [x] **v0.9 — Interactive prototype** (this repo): full UX with simulated data
- [ ] **Phase 1 — Software pilot**: real occupancy from existing UT data sources (Wi-Fi AP counts / card swipes) at 2–3 buildings, no new hardware
- [ ] **Phase 2 — Full pilot**: 10 buildings, infrared counters at study spaces, trained prediction model, iOS/Android wrappers
- [ ] localStorage persistence for favorites/alerts
- [ ] Floor-level granularity for multi-story buildings
- [ ] Accessibility pass (screen-reader labels on map pins, full keyboard nav)

## Tech

`HTML` `CSS` `JavaScript (ES6)` `Leaflet` `OpenStreetMap` — 1 file, 0 build steps, ~1,500 lines

## License

© Steven Wimberly. Not affiliated with or endorsed by The University of Texas at Austin — "UT" is used descriptively. Map data © OpenStreetMap contributors.
