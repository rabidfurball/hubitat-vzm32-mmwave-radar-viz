# mmWave Radar Visualizer

Single-file HTML/JS tool for visualizing mmWave target data from
Inovelli VZM32-SN switches in real time. Connects directly to your
Hubitat hub over WebSocket, plots detected targets on a 2D canvas,
and supports rectangular zone definition + per-zone occupancy
detection.

## Features

- **Real-time target plot** — connects to Hubitat's event WebSocket
  and renders detected targets as soon as the switches publish
  `targetInfo` events
- **Multi-device** — show targets from multiple VZM32-SNs on the
  same canvas; each device gets its own color palette + per-device
  position offset so you can lay out the whole house
- **Floor plan overlay** — load a PNG/JPG/WebP of your floor plan
  and align it underneath the radar plot. Manual sliders OR
  **calibrate-by-walking** (click two spots on the floor plan
  while standing at each — solves scale + rotation + offset)
- **Rectangular zones** — define up to N rectangles per device,
  visualize them on the canvas, see live occupancy as targets pass
  through
- **JSON import/export** — paste / copy zone configs to use them in
  the [zone-aware driver][zone-driver] (see *Pairing with the
  zone driver* below)
- **Trail plotting** — show last N positions of each target so you
  can see motion paths
- **Heatmap mode** — accumulate target positions over time to find
  hot spots
- **Cursor coords** — hover the canvas to read coordinates in mm
  (useful for picking zone bounds)

[zone-driver]: https://github.com/rabidfurball/hubitat-vzm32-mmwave-zone-driver

## Requirements

- A Hubitat hub on your local network
- One or more **Inovelli VZM32-SN** Blue Series mmWave Dimmers paired
  to the hub
- **Parameter 107** (`mmWave Target Info`) **enabled** on each
  switch you want to visualize. Enable it in the device's preferences
  in Hubitat. Note that this generates a steady stream of events
  while motion is present (~1 event/sec per target).
- A modern browser (Chrome / Firefox / Safari recent enough to
  support WebSocket + Canvas2D — basically anything from the last
  several years)

## Install / use

There's nothing to install. Download the HTML file and open it
locally:

```sh
curl -O https://raw.githubusercontent.com/rabidfurball/hubitat-vzm32-mmwave-radar-viz/main/mmwave-radar.html
open mmwave-radar.html   # macOS
xdg-open mmwave-radar.html   # Linux
# or just double-click on Windows
```

Or just open the raw URL in your browser
([raw link](https://raw.githubusercontent.com/rabidfurball/hubitat-vzm32-mmwave-radar-viz/main/mmwave-radar.html))
— though saving locally is friendlier for repeat use.

### Connecting to the hub

1. Enter your Hubitat IP at the top of the side panel (default
   `10.0.4.20`, change to whatever yours is)
2. Click **Connect** — the status pill should flip to a green
   "Connected" once the WebSocket handshake completes
3. Switch on a few VZM32-SNs that have parameter 107 enabled, then
   click the device chips that appear under "Devices" to select
   which ones to plot
4. Walk around in front of one of the switches — targets should
   appear on the radar within ~1 second

### Loading a floor plan

In the **Floorplan Overlay** section:
- **Load Image** — file picker for a local PNG/JPG/WebP, OR
- **URL** — paste a URL (must be CORS-accessible from your browser)

Then either:
- **Calibrate by walking** (recommended) — click `Calibrate by Walking`,
  stand in the field of view of one of your switches, and click the
  spot on the floor plan where you're physically standing. Walk to a
  visibly different spot, click again. The tool computes scale,
  rotation, and offset from those two (click, target) pairs and
  applies them. Re-run any time to refine.
- **Manual** — use the opacity / scale / offset / rotation sliders to
  align the image by eye. Slower and less accurate but doesn't
  require walking around.

### Defining zones

The **Zones** section lets you add rectangular zones with `xMin /
xMax / yMin / yMax` bounds (in mm; coordinate origin = the switch).
Each zone shows live occupancy ("active" / "empty") next to its
name as targets enter and leave.

Workflow for picking good bounds:

1. Walk to a corner of the area you want to capture as a zone
2. Hover the cursor over your position on the radar — the bottom
   of the canvas shows mm coords
3. Note `(x, y)` for each corner of the zone
4. Click **+ Add Zone** and enter the rectangle bounds with a bit
   of margin

## Pairing with the zone driver

If you're using the [hubitat-vzm32-mmwave-zone-driver][zone-driver]
fork, the zone JSON shape is **the same**. Workflow:

1. Define zones interactively in this visualizer
2. Click **Export** to copy the JSON
3. Paste into the **Zone Config (JSON)** preference of the
   corresponding device on your hub
4. Save preferences — the driver will start emitting `area0..area3`
   events as targets cross those rectangles

A future version of this tool may wire up direct push/pull to the
device preference via Maker API (no copy/paste). For now it's a
manual handoff.

## Data, privacy

This tool is **fully client-side**. The HTML page makes a single
WebSocket connection to your Hubitat hub on your local network.
Nothing is sent to any external server. Your floor plan images stay
local — the file picker uses a `FileReader` and never uploads.

The *one* exception: if you choose to load a floor plan via URL,
the browser will fetch that URL directly. If it's a public URL,
that's a normal browser request you'd be making anyway.

## License

MIT — see [LICENSE](LICENSE).

## Acknowledgements

- Built with [Claude Code](https://claude.com/claude-code).
- mmWave hardware: Inovelli's VZM32-SN Blue Series 2-in-1 Dimmer.
