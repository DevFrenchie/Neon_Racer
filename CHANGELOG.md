# Changelog

All notable changes to **NEON APEX DRIFT** are documented here.

The format follows [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/),
and the project currently follows light **Calendar Versioning** (CalVer) rather than SemVer — major releases are signposted by feature arcs, not strict API breaks.

---

## [Unreleased]

### Changed

- **Traction-gated slip rotation.** `Slip → Spin` yaw now scales with drift
  state (`slipYaw = yawFromSlip × (0.15 + 0.85·driftBlend)`), and lateral
  scrub ramps up as `driftBlend → 0` — excess lateral velocity above the
  grip cap is bled off up to ~4.5× faster on exit, and residual lateral
  velocity in the grip regime decays ~3× faster. Drifts keep their full
  tail-swing; releasing a drift now plants the car straight instead of
  coasting through the leftover curve.

### Changed (stock tuning defaults)

- **New baked-in defaults** (global, apply to every chassis): `Slip → Spin`
  0.85 → **0.1**, `Steering Spin` 2.2 → **2.5**, `Air Drag` 0.0003 →
  **0.00005**, `Rolling Resistance` 0.02 → **0.034**. Slip-driven rotation is
  much gentler by default (wide, lazy drift arcs), steering does more of the
  turning, and top speed is trimmed slightly by extra rolling drag.

---

## [1.0.0] — 2026-08-01

The first properly-numbered release. Lands two commits that close out the
**camber-feature arc** (intuitive tuner, per-car chassis personality, smooth
drift state transitions) and ship the **Test Pad fast-iteration layer** for
chassis comparison. Total: 2 commits on `main`, +114 / −84 lines.

### Added

- **Per-car `camberSpread` slider ("Camber Push")** in the tune panel's
  _Rear Tire Grip_ group. Range **0.5 – 2.5**, default per-car (BRUTE / VECTOR
  / HALO ship at 1.8). Widen the lateral envelope of drift rather than
  post-multiply per-frame velocity (see _Fixed → Drift teleport_ below for why).

- **Per-car `driftHang` factor** that scales `G.exitTime` per chassis, giving
  each car its own drift-exit personality on top of the universal smoother:

  | Chassis | `driftHang` | Effective exit settle | Feel |
  | --- | ---: | ---: | --- |
  | BRUTE  | 1.5 | ~0.42 s | Loose brawler; rear walks the curve briefly after release |
  | VECTOR | 1.0 | ~0.28 s | Stock balance; smooth taper back to grip |
  | HALO   | 0.7 | ~0.20 s | Precision; snappy redirect after release |

- **`swapCarInTestPad(newId)` helper** + **keydown intercept for `1` / `2` / `3`**
  in Test Pad mode. Pressing a digit key mid-test:
  - Switches `carCfg = CARS.muscle / .street / .super`.
  - Calls `placeCarAtStart()` for a clean state reset (position, heading,
    velocity, wheel angle, weight transfer, lateral friction, drift blend,
    combo timer, charge meter, boost state, chain counter, wall cooldown).
  - Re-syncs the tune panel if it's visible, so the new chassis's camber,
    friction, engine, etc. show where the old ones were a moment ago.
  - Gated on `race.state === 'race' && track.testpad === true`. Outside Test
    Pad the keys no-op correctly.

- **Tune-panel copy rewrite**. Every slider, group, and desc string replaced
  with plain-English "car guy" language. Group names went from
  `Steering / Grip & Bleed / Yaw & Drag / Scoring / Mini-Turbo /
   Drift Speed Boost / Surface`
  to
  `Steering / Rear Tire Grip / Body Rotation / Drift Scoring /
   Mini-Turbo (Space) / Drift Top-Speed / Track Surface`.
  Knob names went from `Yaw from slip → Slip → Spin`, `Bleed (drift) →
  Drift Tire Slip`, `Yaw from wheel → Steering Spin`, etc.

- **Highlight-knob `quick:true` flag** so key sliders (`Engine Power`,
  `Top Speed`, `Max Stored Charge`) sit in the **Quick Tune** row at the
  top of the panel, separate from the per-section groups.

### Changed

- **Mini-turbo tier pacing rebalanced** from accelerating gaps to evenly
  spaced 1 / 2 / 3-second transitions: `mtBlue 0.4 → 1.0`, `mtOrange 1.0 →
  2.0`, `mtPurple 2.0 → 3.0`. At the default charge rate of 1.0/s, each tier
  fires exactly one beat of the build rate. Replaces the previous
  0.6 / 1.0 / 1.6-second gaps that made purple feel stranded.

- **Steering responsiveness raised** to match the camber-driven rear looseness:
  `rackRate 3.2 → 4.4`, `centerSpring 1.6 → 1.1`, `maxWheel 0.58 → 0.65`.
  Snappier turn-in, less self-centering, more lock.

- **Drift-rotation feel amplified**: `yawFromSlip 0.55 → 0.85`,
  `bleedDrift 0.92 → 0.96`. Slip angle and held drift both drive harder
  rotation now that the rear walks wider.

- **CARS camber-push default unified to 1.8** across BRUTE / VECTOR / HALO.
  The wider drift feel is now the baseline on every chassis. Per-car tuning
  remains available in the Rear Tire Grip slider.

- **Tune-panel font sizes bumped** for readability:
  panel title 14 → 18 px, slider value 9 → 14 px (bold, tabular-nums),
  description 8.5 → 11.5 px, group headers 10 → 13 px, action button labels
  9 → 11 px.

### Fixed

- **Drift teleport on countersteer / direction flip**. The original `newLat *=
  c.camberSpread` post-multiplier amplified the magnitude of `newLat` at the
  moment of its largest swing (sign reversal during a tight countersteer), and
  the doubled value fed both `vel.x = newFwd·ch − newLat·sh` and
  `heading += newLat · yawFromSlip · 0.012` in the same frame. Over a
  fraction of a second that compounded into tens-of-meters position jumps.
  Replaced with envelope scaling on `latCap` (`camberCap = latCap *
  c.camberSpread`). Sign flips now happen in the smooth-friction regime (|
  latVel| < `camberCap`), with no per-frame amplification.

- **Grip snap-back on drift release**. Three knobs snapped on `inp.drift`
  toggle: the `latCap` factor (0.95 / 1.05), the `camberCap` envelope, and the
  `bleed` coefficient. Replaced with a single state variable
  `car.driftBlend` (0 → 1) that smoothly tracks `inp.drift` over the existing
  `G.entryTime` / `G.exitTime` smoother — same path as `latFric`. Releasing
  drift now settles through 0.28 s of residual looseness (multiplied by each
  chassis's `driftHang`) rather than slamming back to 100% grip on key
  release.

---

## Earlier history (pre-1.0)

Pre-1.0 history lives in `git log`. For convenience:

```
216eb3e (HEAD) Add per-car driftHang factor and Test Pad 1/2/3 hotkey chassis swap
24f96d2 Add per-car Camber Push tunable, smooth drift exit, and per-chassis defaults
f2bee47 Remove stale UI-patched build + sample text files from repo
9c74eaf Add Test Pad mode, scroll-wheel zoom, reverse-steering inversion + polish
601bd30 Bump apex_line.html to NEON APEX DRIFT (full build)
5358336 Initial commit: APEX LINE drift racing game
```

Earlier versions (before 1.0.0) cover:
- Test Pad free-drive / tune-live mode with reference parking-lot image
- Scroll-wheel zoom + middle-click reset
- Reverse-steering inversion when driving backward
- Initial NEON APEX DRIFT build out of the earlier APEX LINE prototype
- Sample / experimental builds (removed from repo in `f2bee47`)

---
