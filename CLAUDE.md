# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

Zeehaven is a client-side web tool that converts [Zeeschuimer](https://github.com/digitalmethodsinitiative/zeeschuimer) NDJSON files into structured CSV format. It runs entirely in the browser (no server) and is deployed as a GitHub Pages site at https://publicdatalab.github.io/zeehaven/.

## Development

No build step or dependencies. Open `index.html` in a browser or serve it with any static file server (e.g. `python3 -m http.server`). Drag an NDJSON file onto the drop zone to test.

## Architecture

The app has two files that matter:

- **`index.html`** — Contains the drag-and-drop UI and the `dodrop()` entry point. When a file is dropped, it reads the NDJSON, detects the platform from the filename (the third segment of the hyphen-delimited Zeeschuimer filename, e.g. `twitter.com`), calls the appropriate parser in `datamodel.js`, and triggers a CSV download via a generated Blob URL.
- **`datamodel.js`** — All parsing logic. Platform-specific parsers extract structured fields matching the [4CAT](https://github.com/digitalmethodsinitiative/4cat) data model:
  - `parseTwitter()` — X/Twitter. Handles retweets, quote tweets, media extraction, mentions, hashtags, geolocation. Navigates the nested `data.core.user_results.result` structure using a `core`/`legacy` key heuristic.
  - `parseInstagram()` — Instagram. Handles photos, videos, carousels (`MEDIA_TYPE_*` constants), user tags, hashtag extraction via regex, location data.
  - `parseTiktok()` — TikTok. Handles music metadata, duets, stickers, effects, warnings, challenges, diversification labels, thumbnail URL expiry checking.
  - `parseAll()` — Generic fallback that flattens the entire JSON structure into columns.

Helper functions: `flatten()` (recursive object flattener), `escapeHTML()` (sanitizes strings for CSV by replacing newlines, escaping quotes, replacing commas with look-alike character `‚`), `parse_qs()` (query string parser for thumbnail URL expiry).

## Key patterns to know

- Platform detection relies on the Zeeschuimer filename convention: `{prefix}-{hash}-{platform.domain}-{...}.ndjson`. The third hyphen-separated segment is matched against `twitter.com`, `instagram.com`, `tiktok.com`.
- CSV is built manually (not via a library) — field values are joined with commas and string fields are wrapped in escaped quotes. The `escapeHTML()` function replaces literal commas with `‚` (U+201A) to avoid breaking CSV structure.
- Each parser reads the first data row to infer headers via `flatten()`, then iterates all rows with a try/catch so one bad row doesn't break the entire export.
- The Twitter/X parser has a dual-path user field access (`core` vs `legacy` key) because the API response structure varies.
