# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

**全国散策マップ** — A single-page web application for discovering and diagnosing Japanese hiking/walking courses. Users filter courses by region/difficulty, take a diagnosis quiz, and get AI-powered recommendations via the Google Gemini API. Deployed at https://noramimiyuma.github.io/hiking-course-diagnosis/

## Development Commands

```bash
# Install test dependencies
npm install

# Run a specific Puppeteer test
node test_zoom.js
node test_null_latlng.js
node validate.js

# Validate course data
node validate.js
```

There is no build step — `index.html` is served directly as a static file.

## Architecture

### Single-file app
All application logic lives in `index.html` (CSS + JS embedded inline). There is no bundler or framework. The file contains:
- Leaflet.js map initialization
- Course filtering, search, and bookmarking logic
- A multi-step diagnosis quiz
- AI concierge using Google Gemini API (key stored in `localStorage`)
- Mobile bottom-sheet / desktop sidebar responsive layout

### Course data
Course data is split across multiple JS files that are loaded as `<script>` tags:
- `mountains.js` — master course database (~3MB)
- `new_courses_<region>.js` — regional course arrays (Hokkaido/Tohoku, Kanto, Chubu, Kansai, Chugoku, Shikoku, Kyushu)
- `new_batch_*.js`, `new_p_*.js` — incremental batch additions

Each course object carries: name, coordinates (`lat`/`lng`), difficulty, prefecture, duration, features, seasonal suitability info, tips, and equipment notes.

### Difficulty levels (color-coded)
`beginner` (green) → `intermediate` (yellow-orange) → `advanced` (orange-red) → `expert` (purple)

### Data pipeline (Python scripts)
The `*.py` scripts are one-off data preparation utilities — they geocode course locations, fix misplaced coordinates, and integrate new data into the JS course files. Outputs are cached in `*_cache.json` files. These are not part of the app runtime.

### Testing
Tests use Puppeteer (headless Chrome) to automate UI interactions. Each `test_*.js` file targets a specific behavior (zoom controls, null lat/lng handling, CSS zoom, etc.).

## Key Constraints

- **No build pipeline** — edits to `index.html` or the course JS files take effect immediately on page reload.
- **Gemini API key** — the AI concierge requires users to input their own Gemini API key, which is persisted in `localStorage`. The app currently uses Gemini 2.5 models (updated from 1.5 in v40).
- **Map tiles** — provided by CARTO (`light_only_labels`) or OpenStreetMap via Leaflet CDN; no API key needed for tiles.
- **Course data files are large** — be careful when editing `mountains.js` (~3MB); prefer the region-specific files for targeted changes.
