# Lean Canvas

A single-file, offline-capable editor for Ash Maurya's Lean Canvas — the nine-box
adaptation of the Business Model Canvas. Fill in the boxes, then export the canvas
as an editable PowerPoint slide or a PNG.

Everything lives in `lean-canvas.html`. There is no build step, no bundler, and no
server-side component.

## Using it

Type directly into any box.

| Key | Does |
| --- | --- |
| `Enter` | Start the next point |
| `Shift`+`Enter` | Line break inside the current point |
| `Backspace` on an empty point | Delete it and move up |
| `Escape` | Leave the field |

Drag the bullet dot next to a point to move it into a different box — handy when
something you wrote as a solution turns out to be a channel.

**Guide me** dims everything except the current step and walks through Maurya's
fill order, which is deliberately not the reading order of the grid:

> problem → customer segments → unique value proposition → solution → channels
> → revenue streams → cost structure → key metrics → unfair advantage

**Tabs** hold multiple canvases. Maurya's advice is one canvas per customer
segment rather than one canvas listing several segments in box 2.

On a narrow screen the grid collapses to a single column in fill order, so the
phone layout becomes the guided path.

## Exports

- **PowerPoint slide** — one 16:9 slide built from real shapes and text boxes, so
  it stays editable in PowerPoint or Keynote. Not a screenshot.
- **Image** — 2400 × 1350 PNG.
- **Canvas file** — JSON of every canvas, round-trips through Import.
- **Copy as text** — plain outline to the clipboard.

The slide and the image are drawn from the same layout function, measured in
inches, so they match each other and always render the full nine-box grid — even
if you built the canvas on a phone.

## Storage

Canvases autosave to `localStorage` under the key `leancanvas.v1`. That is
per-browser and per-origin: nothing is uploaded anywhere, and nothing syncs
between machines. Use the JSON export to move work around or to back it up.

## Serving it locally

`localStorage` is scoped by origin. Opening the file directly with `file://`
gives you an opaque origin — some browsers hand out no storage at all there, and
others share one bucket across every local file you open. Serve it over
`http://localhost` instead, which gives it a stable origin of its own:

```sh
# Python (already installed on macOS and most Linux)
python3 -m http.server 8000

# or Node
npx serve .

# or PHP
php -S localhost:8000
```

Then open <http://localhost:8000/lean-canvas.html>.

Any static host works for a permanent home — GitHub Pages, Netlify, Caddy, an
S3 bucket. There is nothing to configure.

## Downloads in two environments

The page detects at load whether it is running inside the Claude artifact
runtime:

```js
const IN_ARTIFACT = !!(window.claude && typeof window.claude.use === "function");
```

Inside that runtime, plain `<a download>` links are inert, so files go through
the platform's `downloads` capability and the viewer confirms each save.
Anywhere else, the page falls back to an object URL and a synthetic anchor
click. The export buttons behave the same either way.

`Copy as text` has a matching fallback: `navigator.clipboard` needs a secure
context, so on `file://` it drops to a selection-based copy, and failing that
offers the outline as a `.txt` download.

## Network

Two requests at load, both from CDNs:

- Google Fonts — Bricolage Grotesque and Public Sans
- cdnjs — PptxGenJS 3.12.0, used only for the PowerPoint export

Block both and the tool still runs: you get system fonts, and the PowerPoint
option reports that the library failed to load. The PNG export uses the Canvas
2D API directly and has no dependency at all.

## Browser support

Any current Chrome, Firefox, Safari, or Edge. Uses CSS Grid, `canvas.toBlob`,
and the HTML5 drag-and-drop API. Respects `prefers-reduced-motion` and
`prefers-color-scheme`; the canvas sheet stays paper-white in dark mode, since
it is a document, while the surrounding chrome darkens.
