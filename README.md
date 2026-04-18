# AI-DOPTERS VISUALIZER

A single-file, demoscene-caliber audio visualizer wrapped in an LCARS (Star-Trek computer) skin. Drop any MP3 onto the page and watch it come alive: a pulsing tube-tunnel of reactive cells, TRON-style "bits" whizzing through, octahedron glyphs rotating in the distance, a text banner painted onto the wall of the tube, and a chromatic-aberration + grain post-pass tying the whole thing together.

Originally built for **"The Moons of Saturn · Energy"** (a Suno-generated track bundled with this repo), but works with anything you throw at it.

Live: `index.html` — open it in a modern browser, no build step, no server required (though serving over HTTP is recommended for some browsers; see below).

---

## Controls

| Control | What it does |
|---|---|
| **▶ ENGAGE** (boot) | Starts playback of the bundled track. Required once — browsers block autoplay without a user gesture. |
| **DROP ANY .MP3 HERE** (boot) | Drop a file, or tap to browse. Replaces the bundled track. Custom tracks run on live FFT only. |
| **SCROLLER MESSAGE** (boot) | The text painted on the inside of the tube. Edit at boot or live. |
| **PAUSE / PLAY** | Toggles audio + visual. |
| **RESTART** | Seeks to 0:00 and plays. |
| **MUTE / UNMUTE** | Audio only — the visual keeps running. |
| **VOL** slider | 0–100. Dragging up from zero auto-unmutes. |
| **Drag & drop anywhere** (post-engage) | Drops a new MP3 in mid-flight. |

---

## How it works

One HTML file. Everything is inline.

- **Audio graph**: `<audio>` → `MediaElementSource` → `AnalyserNode` (+ splitter to destination). The analyser supplies an 2048-bin FFT every frame.
- **Band split**: SUB (20–80 Hz), BASS (80–250 Hz), MID (250–4 kHz), AIR (4–16 kHz). Each runs through an envelope follower (fast attack, slow release).
- **Pre-computed bands** (`bands.json`): for the bundled track only, we also read offline-computed band levels and `max()` them with live FFT. This keeps the visual consistent on quieter system volume and adds a track-specific motion-feel. Custom tracks skip this and run on live FFT alone.
- **Onset detector**: bass-over-baseline lifts a `kick` envelope that drives the laser transients.
- **Camera drift**: mid-band onsets kick a critically-damped spring, so every strong mid transient nudges the tunnel off-axis and springs back.
- **Shader**: single GLSL fragment shader renders a polar tunnel (`depth = 1/r`, `a = atan2(uv.y, uv.x)`), composites tunnel cells, rainbow wobble, signal-pulse sweeps, TRON tracer bits, TRON octahedron glyphs, a text banner CanvasTexture mapped onto a depth band, the central "GANYMEDE SEED" gaussian, lasers, stars, and a starfield-drift background.
- **Post-processing**: UnrealBloomPass (0.50 / 0.42 / 0.35) → chromatic-aberration + grain ShaderPass → OutputPass.
- **Scene timeline** (bundled track only): six scene labels timed to the original track (INTRO → VERSE → MOVEMENT → BUILD → DROP → OUTRO). The SCENE pill in the header auto-updates. Custom tracks collapse to a single `CUSTOM TRACK · LIVE FFT` label.

---

## Running it

### Locally, no server
Open `index.html` directly in Chrome, Edge, Firefox, or Safari. Works in most cases. Some browsers restrict CORS on `file://` — if the bundled MP3 fails to load, serve over HTTP instead.

### Locally, with a server
Any static server works:

```bash
# Python 3
python -m http.server 8000

# Node
npx serve .
```

Then open `http://localhost:8000/`.

### Android tablet / phone
Works on modern Android Chrome. Touch is wired through:
- **Tap ENGAGE** to start (autoplay is blocked without a gesture).
- **Tap the drop zone** to open the file picker — drag-and-drop won't fire on touch, but the picker does.
- Portrait layouts fold the left rail to 96 px (or 72 px on phones) via media queries so the scene label and chips don't collide.
- `touch-action:none` on the canvas prevents the page from scrolling or pinch-zooming while you interact with it.

Serving the page from a local network is easiest: `python -m http.server 8000 --bind 0.0.0.0` on your desktop, then open `http://<desktop-ip>:8000/` on the tablet.

---

## Files

```
index.html                    Everything. Three.js via CDN import-map.
moons-of-saturn-energy.mp3    The bundled track.
bands.json                    Offline band analysis for the bundled track.
cover.jpg                     Cover art (not used at runtime; source material).
analysis/                     Notes and analysis scratch.
```

---

## Credits

- Track: **"The Moons of Saturn · Energy"** — generated with Suno.
- Engine: Three.js (r150+) via `esm.sh` import map, UnrealBloomPass, EffectComposer.
- Visual language: LCARS (*Star Trek: The Next Generation* / Michael Okuda), TRON (Lisberger/Syd Mead), demoscene tunnel effects.
- Built with Claude Code.

---

## License

No license declared — treat as "all rights reserved" until the repo owner states otherwise.
