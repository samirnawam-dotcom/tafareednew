# Tafareed — scroll-film landing page

A single-page storefront for [tafareed.com](https://tafareed.com), built on the
"Celestial Paper" redesign direction: cream paper, ink navy, champagne gold,
Italiana over IBM Plex Sans Arabic, Arabic by default with an English toggle.

## The hero

The page opens on a full-bleed film that is **scrubbed by scroll** rather than
played. On load the clip is decoded once into still frames held in memory; scrolling
then swaps bitmaps on a canvas, so the scroll stays smooth regardless of how the
MP4 is keyframed. Frame height follows the viewport (capped at 1600px, doubled
for retina) and the frame count is then sized to a ~260MB budget, so a sharper
frame simply means fewer of them. If frame extraction fails the page falls back to
seeking the video directly, and on coarse-pointer devices it holds the poster
frame (iOS Safari cannot seek frame by frame).

Three collection chapters — Letterline, Rings, Celestia — fade in over the film
in sequence as it scrubs.

## Files

```
index.html                              the whole page: markup, styles, script
assets/tafareed-hero.mp4                1080x1920, keyframe every 12, used by the page
assets/hero-source-1080x1920.mp4        original Google Flow render, 1080x1920
assets/hero-poster.jpg                  first frame, shown before the clip loads
assets/logo-dark.png / logo-light.png   wordmark for paper and for the film
assets/shop/*.jpg                       12 product shots, toned to the cream panel
```

## Replacing the film

Render a new clip at 1080x1920, then re-encode it with tight keyframes so the
decode pass can seek through it quickly:

```sh
ffmpeg -i new-render.mp4 -an \
  -c:v libx264 -crf 21 -preset slow -g 12 -keyint_min 12 -sc_threshold 0 \
  -pix_fmt yuv420p -movflags +faststart assets/tafareed-hero.mp4

ffmpeg -ss 0.3 -i new-render.mp4 -frames:v 1 -vf scale=1080:-2 -q:v 2 assets/hero-poster.jpg
```

Use `-g 1 -x264-params keyint=1:scenecut=0` instead if you ever disable the
decode pass and scrub the video element directly — that path needs every frame
to be a keyframe, at roughly three times the file size.

The clip wants one unbroken camera move, fixed lighting and constant exposure —
cuts and exposure jumps read as glitches when someone scrubs slowly.

## The catalogue

Products live in the `CATALOGUE` array near the top of the script: name, Arabic
and English category, price in SAR, SKU, product URL, and an image key matching
`assets/shop/<key>.jpg`. Filters and sorting are driven off that array, so adding
a piece is one line plus one image.

Prices and names were taken from the live Salla store and are a snapshot — this
page does not read Salla at runtime. To keep it in sync, the markup below the
film should become a Twilight theme section fed by Salla's own product data.

## Known limits

- The bag is a counter, not a cart; checkout belongs to Salla.
- The newsletter form validates and confirms but posts nowhere.
