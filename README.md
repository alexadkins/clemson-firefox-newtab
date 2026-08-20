# Clemson Firefox New Tab

A single-file, fully custom Firefox "new tab" page: an animated Clemson-themed
skyline wallpaper that reacts to window size, cycles sky color with real
sunrise/sunset times, layers in live weather effects (Open-Meteo), and shows
a small set of personal bookmark shortcuts pulled from a CSV file.

There's no build step and no dependencies — it's one `index.html` (HTML +
CSS + vanilla JS) plus a folder of image assets and a `bookmarks.csv` file
you edit by hand.

## Why this exists

Firefox's built-in New Tab page supports a background image and a bookmarks
toolbar, but:

- You can't animate or react to window size with it.
- Once you override the new tab with a custom page (via an extension),
  Firefox's "Only show the bookmarks toolbar on New Tab" setting stops
  applying, and a plain hosted webpage has no access to the WebExtensions
  bookmarks API to render its own toolbar.
- WebExtensions also has no API to auto-focus/select the address bar on a
  new tab ([Mozilla bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1345920)),
  so you lose "just start typing a search" muscle memory.

This project works around both: bookmarks are defined in a simple CSV file
and rendered as grouped link cards on the page itself, and an autofocused,
pre-selected "Search Google" box in the center of the page replaces the
address-bar-typing habit (use `Cmd+L` / `Ctrl+L` if you want the real address
bar).

## Features

- **Reactive layout** — the Clemson logo, tiger paw, and mountain skyline
  layers resize and re-nest based on window width/height so nothing crops
  off-screen; layers compress together ("squash") as the window gets short.
- **Time-of-day sky** — background sky color cycles through
  night → dawn → day → dusk → night, anchored to your browser's real
  geolocation (falls back to Clemson, SC if location access is denied).
- **Live weather** — fetches current conditions from the free
  [Open-Meteo](https://open-meteo.com/) API and animates matching effects:
  drifting clouds (cloudy/fog), rain, snow, and darker fast-moving storm
  clouds — plus a small time + weather text widget.
- **Bookmarks section** — reads `bookmarks.csv`, groups links by category,
  and renders them as cards with favicons (Google's favicon service, with a
  Clemson paw fallback for domains that don't resolve one). Cards hide
  automatically if the window gets too short to fit them without colliding
  with the search box, and stack into a single scrollable column on narrow
  windows.
- **Search box** — autofocused "Search Google" input in place of relying on
  the (new-tab-inaccessible) address bar.

## Requirements

- Firefox
- The [New Tab Override](https://addons.mozilla.org/firefox/addon/new-tab-override/)
  extension (or similar) to point new tabs at this page's URL
- Somewhere to host the files over `http(s)://` — see [Hosting](#hosting) below

## Setup

### 1. Get the files somewhere they can be served over HTTP(S)

Firefox blocks `fetch()` (used to load `bookmarks.csv`) under `file://`, so
this page needs to be served, not opened directly from disk. Any static web
host works — GitHub Pages, a university web space, Netlify, a local dev
server, etc.

Clone the repo onto that host:

```bash
git clone <this-repo-url>
```

#### Hosting

Any static file host is fine as long as the whole folder (not just
`index.html`) is uploaded, so `resources/` and `bookmarks.csv` stay
alongside it with the same relative paths.

For local testing (not for the extension — Firefox still needs a real
`http://` URL for it to treat as the new tab — but useful for iterating):

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/index.html
```

### 2. Edit your bookmarks

Open `bookmarks.csv` in any spreadsheet app or text editor. It's a plain
3-column CSV:

```csv
category,title,url
Advising,CUNavigate,https://clemson.campus.eab.com/home
CPSC 1050,Canvas,https://clemson.instructure.com/courses/294963
```

- `category` groups links into their own card. Cards appear in the order
  their first link is encountered.
- `title` is the link text shown on the page.
- `url` is the full destination URL.

Favicons are pulled automatically from Google's favicon service. Some
domains (e.g. anything under `clemson.edu` or `eab.com`) don't resolve real
favicons through that service and are shown with the Clemson paw icon
instead — see `NO_FAVICON_DOMAIN_SUFFIXES` near the top of the bookmarks
script in `index.html` if you want to add more domains to that list.

### 3. Point Firefox's new tab at your hosted page

1. Install [New Tab Override](https://addons.mozilla.org/firefox/addon/new-tab-override/).
2. In its settings, choose "Custom URL" and enter the URL where you hosted
   `index.html` (e.g. `https://your-host.example.com/index.html`).
3. Open a new tab — you should see the page load with the search box
   autofocused.

### 4. (Optional) Allow location access

The first time the page loads, your browser may prompt for location
permission — allow it to get accurate local sunrise/sunset times and
weather. If you deny it, the page falls back to Clemson, SC's coordinates.

## Project structure

```
index.html              All markup, styles, and logic (single file, no build step)
bookmarks.csv            Your editable bookmark links, grouped by category
resources/
  ClemsonUniversity_RGB__Orange.png   Clemson wordmark logo
  Paw_RGB__Orange.png                 Tiger paw graphic
  paw-orange.png                      Small paw icon (favicon + bookmark fallback)
  Footer_Layer1.png ... Footer_Layer4.png   Mountain/skyline layers (back to front)
  zoomBG.png                          Original static background this project replaced
  clouds/
    cloud1.png, cloud2.png                    Source cloud art (low opacity)
    cloud1-fx.png, cloud2-fx.png               Alpha-boosted, upscaled versions used for cloudy/fog
    cloud1-storm.png, cloud2-storm.png         Darker, purple-tinted versions used for storms
```

## Customizing

Everything lives in `index.html` as CSS custom properties and small,
independent JS functions — there's no build step, so just edit and reload.
A few starting points:

- **Sky colors** — see the keyframe tables near `buildSkyKeyframes()`.
- **Mountain squash behavior** — `updateLayout()` and the `LAYERS` table.
- **Weather effects** — `spawnClouds()`, `spawnRain()`, `spawnSnow()`,
  `applyWeatherFX()`, and the `CLOUD_IMAGES`/`STORM_CLOUD_IMAGES` arrays.
- **Testing time-of-day changes quickly** — open the browser console and
  call `startTimeDemo()` to rapidly cycle through a full day (`stopTimeDemo()`
  to stop), instead of waiting for real time to pass.
- **Bookmark card styling** — `.bookmark-category`, `.bookmark-link`,
  `.bookmark-category-title` in the `<style>` block.

## Known limitations

- Requires being served over `http(s)://` — `bookmarks.csv` won't load
  under `file://`.
- Weather and astronomical calculations depend on browser geolocation;
  without it, both fall back to Clemson, SC.
- On very short windows, bookmark cards intentionally sink behind the front
  mountain layer as a visual effect — content covered that way is not
  reachable by scrolling. This is a deliberate tradeoff, not a bug.

## License

Copyright (C) 2026 Alex Adkins

This program is free software: you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation, either version 3 of the License, or (at your option) any later
version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with
this program. If not, see <https://www.gnu.org/licenses/>.

### Trademarks and brand assets

The license above covers the code in this repository. It does **not** grant any
rights to Clemson University's trademarks or brand assets, which are not the
author's to license:

- `resources/ClemsonUniversity_RGB__Orange.png` — Clemson University wordmark
- `resources/Paw_RGB__Orange.png`, `resources/paw-orange.png` — Clemson tiger paw

These remain the property of Clemson University. Anyone redistributing a
modified version of this project should substitute their own artwork.
