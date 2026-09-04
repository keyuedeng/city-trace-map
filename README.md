# City Trace Map

A self-contained, single-file interactive map for documenting surveillance and
tracking infrastructure encountered along a field research route in Melbourne.

Open [`index.html`](index.html) directly in a browser — no build step, no
server, no dependencies beyond the CDN scripts it loads (Leaflet + the
Leaflet.Draw CSS/JS, unused by default but included for future extension).
Survey data lives in [`data.json`](data.json), tracked in this repo alongside
the app — see **Saving your data** below.

## Modes

- **Viewer** — click "View map" on the entry screen. Clean, read-only map
  with a route-colour legend, a category legend, and a collapsible sidebar
  listing every point grouped by technology type. Hover (or tap) a marker for
  its photo, operator, and a short note on what it collects.
- **Admin** — click "Enter admin" and enter the password (hardcoded as
  `surveillance2024` near the top of the `<script>` block, search for
  `ADMIN_PASSWORD` — change it before sharing this file with anyone). Gives
  you two tools, toggled from the sidebar:
  - **Route builder** — pick a transport mode (preset colour, or override
    with the colour picker), then click the map to lay down vertices.
    Double-click, or press "Finish current segment", to end the segment.
    Drag any white vertex dot to reshape a finished segment. "Delete last
    segment" removes the most recently added one. Below the segment list,
    drag the **🚩 Start** / **🏁 Finish** chips onto the map to mark where
    the overall route begins and ends (independent of any segment's
    endpoints) — drag a placed sign again to move it, or click it on the map
    to remove it. Both show up in viewer mode too.
  - **Marker tool** — click the map to drop a new point, or click an
    existing one to edit/delete it. Fill in the name, technology type,
    operator, description, and an optional photo — either click the photo
    box to choose a file, or just paste an image (Ctrl/Cmd+V) straight from
    your clipboard while the form is open, e.g. a screenshot. Either way it's
    auto-compressed client-side to a max 800px-wide JPEG and embedded as
    base64 — nothing is uploaded anywhere.

While in admin mode your edits are also mirrored to `localStorage` in that
browser as a convenience, so a refresh mid-survey doesn't lose work — but
that's just a safety net, not the source of truth. See below for the real one.

## Saving your data

A browser tab can't silently write files to your disk — that's a security
boundary, not a bug — but Chrome and Edge have an API that gets close. In
admin mode there's a small **Data file** bar at the top of the sidebar:

1. Click **Connect**. A native file picker opens — choose
   [`data.json`](data.json) in this project folder.
2. From then on, every add/edit/delete/drag auto-saves straight to that file
   on disk. No exporting, no copy-paste, no manual backup step. Just edit
   the map and `git status` will show `data.json` changed.
3. Next time you open `index.html` and log in, it reconnects automatically
   if the browser still remembers granting permission — otherwise the bar
   shows **Reconnect**, a single click (no re-picking the file).
4. First time setting this up somewhere without a `data.json` yet? Click
   **New** instead of **Connect** to create one.

This needs Chrome or Edge — the bar will say "Auto-save needs Chrome or
Edge" and hide the buttons in other browsers, where it just quietly falls
back to the `localStorage`-only behaviour described above (use **Export data
(JSON)** to get your work out manually in that case).

Since `data.json` is a real file in this repo, committing it to git *is* your
backup and version history — no separate export step needed for that part
either.

## Publishing a finished map

Keep editing in `index.html` (this file) — it's your working copy and always
keeps its admin password. When you're ready to share what you have:

1. In admin mode, click **Download viewer HTML** in the top bar.
2. A new file, `city-trace-map-viewer.html`, downloads to your browser's
   downloads folder — a complete, standalone snapshot of the current map
   with today's data baked in.
3. Send *that* file to whoever you're sharing with. Opening it goes straight
   to the read-only map — no password prompt, no "View map" click, nothing
   admin-shaped visible at all.

`index.html` doesn't change when you do this, so you can keep adding points
and re-download a fresh viewer file any time — each download is a
point-in-time snapshot, not a link back to the live data.

If you just want the raw JSON (for a backup, or to hand-edit), use **Export
data (JSON)** instead — its **Import** tab pastes JSON back in to load it
into the current browser session, handy for iterating without hand-editing
the source each time.

## Notes on the basemap

CARTO's free anonymous "Dark Matter" raster tiles
(`{s}.basemaps.cartocdn.com/dark_all/...`) now require an API key — without
one every tile comes back stamped "API KEY REQUIRED". This file uses Esri's
free, no-key **World Dark Gray Canvas** basemap instead, which has a similar
dark, low-detail aesthetic. If you have (or get) a CARTO API key, the
`initMap()` function has a comment showing the one-line swap back.

## Tech

Vanilla JS, no framework. Leaflet 1.9.4 from cdnjs. `index.html` carries a
baked-in sample `DATA` object as a fallback/default (and as what gets
embedded into an exported viewer file, which stays single-file and
self-contained on purpose) — but in admin mode, connecting `data.json` via
the File System Access API takes over as the working copy, loaded fresh on
connect and written on every change. There is no backend either way.
