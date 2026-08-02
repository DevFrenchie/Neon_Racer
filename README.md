# NEON APEX DRIFT

A single-file HTML5 arcade drift racer. Open the HTML file in any modern browser — no server, no build step, no external assets. Fully offline, all audio synthesized via WebAudio.

**File:** `apex_line.html` (the canonical game file — the in-repo filename, the title in the page is **NEON APEX DRIFT**).

Other HTML files in the repo (`drift_racer_2d_ui_patched.html`, `sample1.txt`–`sample3.txt`) are earlier experimental builds and reference material kept for history.

## How to play

| Action | Key / input |
| --- | --- |
| Drive | **W A S D** or **Arrow keys** |
| Drift | **Shift** (or right-mouse) |
| Fire stored mini-turbo | **Space** |
| Brake | **B** |
| Restart race | **R** |
| Pause / garage | **Esc** |
| Mute / unmute | in-race **SOUND** button |
| **Point-to-drive** | **hold left mouse** (or hold finger on mobile) — direction aims the nose, distance = throttle |
| **Camera zoom** | **scroll wheel** on the canvas (race mode) |
| **Reset zoom** | **middle-click** on the canvas (race mode) |

On touch devices, an on-screen pad appears (▲ ▼ ◀ ▶ + DRIFT / BRAKE / BOOST).

## Screens

1. **Title** — entry point with career rank and best score.
2. **Test Pad** — **TEST PAD · TUNE LIVE** from the title enters a free-drive parking lot (the reference image from `Image_assets/test_track.png`, inlined as a `data:` URL for offline portability) with the tune panel pre-opened, no countdown and no auto-finish — for feeling out a tune change without race pressure.
3. **Garage** — pick chassis (BRUTE / VECTOR / HALO), circuit (SUNSET OVAL / NEON EIGHT / RIDGE RUN / MIDNIGHT SEED), lap count (1–9, default 3).
4. **Race** — countdown 3-2-1-GO, drift bank accumulates across all laps, time bonus at the end.
5. **Results** — 1-3 stars based on final score, breakdown of bank / time / style / clean / wall penalties. Career rank auto-updates (D → C → B → A → S).
6. **Pause** — Esc or **P** during a race. Resume / Restart / Garage. Also auto-fires on window blur.
7. **Track editor** — see below.
8. **Tune panel** — see below.

## Controls quick-reference (also on-screen in the garage)

- Hold **left mouse** or finger and drift in any direction; let go to coast.
- The dashboard's slip-angle needle turns **green** in the scoring window — hold there for big bank.
- Mini-turbo auto-banks while drifting; press **Space** to fire it. Three tiers — blue → orange → purple — based on charge level.
- **Drift chain**: flip drift sides (without losing momentum) to multiply bank and charge.

## Track editor

Open from **Garage → Track editor** (or `gEditor`).

- **Click** to add control points (anywhere on the map).
- **Drag** points to move them. **Right-drag** to pan, **wheel** to zoom (in the editor's own camera).
- **Width** and **gates** selects control ribbon width and checkpoint count.
- **Set start** → click near the line to mark the starting sample position.
- **Save** / **Load** / **Delete** — tracks persist in `localStorage` (`neonApexTracks_v1`).
- **Export** downloads a `.json` track file. **Import** loads one back.
- **Race it** jumps straight into a race on the custom track.

## Tune panel

Open from **Garage → Tune setup** (or `gTune`).

Sliders edit **per-car** stats (shown on the currently selected chassis) and **global** physics constants. The file ships with all sliders at their stock values; you can tune live without persisting, or **Set as default** to write them to `localStorage` (`neonApexTuning_v2`), where they auto-load on every boot.

Slider groups: **Quick Tune** (the handful that change feel the most), **Current Car**, **Steering**, **Grip & Bleed**, **Yaw & Drag**, **Scoring**, **Mini-Turbo (Space)**, **Drift Speed Boost**, **Drift Chain & Exit**, **Surface**.

A live "WHAT THESE KNOBS DO" readout translates the current slider values into the felt numbers (top speed, drift-boost cap, exit kick, chain bonus, blue/orange/purple turbo powers).

- **Apply** — session-only.
- **Set as default** — session + persisted (auto-loads next launch).
- **Restore saved** — re-load the persisted defaults without changing them.
- **Restore stock** — return every slider to the shipped values and clear any saved overrides for the current car.

## Scoring & physics

- **Drift model** — kinematic slip-angle with *partial* lateral bleed (no hard clamp), so cars drift predictably and recover smoothly.
- **Physics step** — fixed 60 Hz internal tick with accumulator and dt clamp (`dt > 0.1 → 0.1`). All `step()` ordering follows a reference spec (surface → steering → weight transfer → drag → speed grip loss → world→local → gradual lat-friction → partial bleed → yaw → integrate → soft wall).
- **Drift detection** — `inp.drift && speed > 60 && driftAngle > car.thr`. `driftAngle` is the heading-vs-velocity delta.
- **Slip window** — `optLow` (0.4) → `optHigh` (0.8) gives a 1.3× score multiplier. **Countersteer** adds another `counterBonus` (1.15×). **Over-rot** (>1.0 rad) drops to `overPenalty` (0.5×). A **feint** (a swing of the wheel sign before entering a drift) grants `feintBonus`.
- **Drift chain** — slip-direction reversal carries bonus `chainBonus × chainLevel` and tops up the mini-turbo charge.
- **Exit kick** — releasing a sustained drift (`combo > 0.3s, speed > 60`) gets a brief forward impulse (`exitBoostPow / exitBoostDur`).
- **Drift speed boost** — a passive top-speed bonus builds while sliding (`dsbRate → dsbMax`) and decays (`dsbDecay`) when off-throttle. Active mini-turbo further raises the cap.
- **Surface** — grass multiplier (`grassMult`, default 0.45) for off-track slow-down and a soft-wall inward push (`wallPush / wallDamp`).
- **Reduce-motion** — particle / shake FX disabled when `prefers-reduced-motion: reduce`.

## Test Pad & live tuning

A free-drive parking lot accessed from the **Title** screen via **TEST PAD · TUNE LIVE**.

- Renders the same reference image used during development — `Image_assets/test_track.png` (1376 × 768 px). The file is also inlined as a `data:image/png;base64,…` URL inside `apex_line.html` so the build remains a single self-contained HTML file offline; if you don't need that, swap the loader to `'Image_assets/test_track.png'` and the HTML shrinks by roughly 80%.
- The image is drawn at `IMAGE_SCALE = 1.62` so it reads at near real-scale. The dashed orange outline on top of the image is the actual cp perimeter — the soft-wall is keyed off `closestOnTrack`, not the image bounds.
- The **tune panel** is opened automatically at entry (`tuneOpen()` → `tuneBuild()` + `tuneSyncPanel()`), so every slider populates from live tune state and dragging changes the car in real time.
- No countdown, no finish line, no progress tracking. Drive in circles if you want.

## Camera zoom + driving feel

- **Mouse wheel** anywhere on the canvas (when not in the track editor) zooms in/out at 1.12× per tick, clamped to `[0.25, 4.0]`.
- **Middle-click** snaps the zoom back to 1.0× (default).
- The player zoom lives in `camera.userZoom`, separate from the per-frame auto-zoom lerp that gives speed feedback — so a value you set with the wheel persists across frames rather than snapping back every tick.
- In **track editor** the editor's own wheel handler (cursor-pivot zoom, `[0.15, 5.0]`) takes over.
- **Reverse-steering inversion** — when the car's local-forward velocity is below `−10` (≈ 2.5 mph rearward), the keyboard branch of `readInput()` flips the sign of `steer`, so A/D inputs invert in the natural way you expect when driving the car backward. The `-10` deadband prevents flapping at zero crossings.
- **Speed display** — the on-screen MPH readout is `Math.round(spd * 0.25 * 0.7)`, a display-only 30% attenuation on top of the internal×MPH conversion so the top speed reads ~105 MPH instead of the literal ~150. Physics, scoring, drift and mini-turbo are unaffected (all keyed off the internal units).

## Persistence

Everything is stored in `localStorage` under:
- `neonApexDrift_v1` — career: `{best, races, rank}`.
- `neonApexTracks_v1` — saved custom tracks.
- `neonApexTuning_v2` — persisted tuning per-car + per-global.

All three are guarded against sandboxed embeddings (try/catch + a `mem` memory fallback) — the game degrades gracefully if `localStorage` is unavailable.

## Validation one-liner

```bash
node -e "const fs=require('fs');const h=fs.readFileSync('apex_line.html','utf8');const m=h.match(/<script>([\\s\\S]*?)<\\/script>/);fs.writeFileSync('_chk.js',m[1]);" && node --check _chk.js && rm -f _chk.js && echo 'SYNTAX OK'
```

## Version control

Standard git. The repo was initialised with `git init -b main`, `.gitignore` excludes Freebuff/Hermes internals and local AI cleanup artifacts; the game file, this README, and the design/reference files are tracked.
