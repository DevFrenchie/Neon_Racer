# APEX LINE — Point-to-Drift

A single-file HTML5 arcade drift racing game. Point where you want the nose,
hold to throttle, and hang the tail through the corner. The brighter your
projected racing line burns, the more you're banking.

Built with zero dependencies — no CDNs, no external assets. Pure inline
CSS + JS on a 2D `<canvas>`, with synthesized WebAudio for sound.

## Files

| File | Description |
| --- | --- |
| `apex_line.html` | Current build — point-to-drift overhaul, slip-angle partial-bleed physics, 3 chassis, 4 circuits (incl. procedural), mini-turbo system, career rank + best score in `localStorage`. |
| `drift_racer_2d_ui_patched.html` | Older, larger build — WASD drive, UI patches applied. Kept as reference. |
| `sample1.txt` / `sample2.txt` / `sample3.txt` | Generation/design prompts used to build the game. |

## Play

Open `apex_line.html` in any modern browser (Chrome, Edge, Firefox, Safari).
No build step, no server required.

### Controls

| Input | Action |
| --- | --- |
| **Desktop** | Drag mouse to steer & throttle toward your cursor · `Shift` drift · `Space` fire stored turbo · `B` brake · `R` reset · `Esc` garage |
| **Touch** | Drag anywhere to steer & throttle · `DRIFT` / `BOOST` / `BRAKE` buttons |

### Scoring

Drift bank + time bonus + style bonus (combo) + clean-race bonus, minus wall
penalties. Earning drift angle banks points; chained combos multiply. Charge
the drift meter to store a mini-turbo (blue → orange → purple tiers), then
fire it for a speed surge.

Career rank (D → S) and best score persist via `localStorage`.

## Physics

The car is a kinematic body with hand-authored slip-angle lateral
partial-bleed friction (not a rigid-body / Box2D-style model), running on a
fixed 1/60 s timestep with soft-wall collision. Full constants live in the
`G` object at the top of the script — ported 1:1 from the shipped spec in
`sample3.txt`; do not "improve" the constants.

## Development

- All logic lives inside the single `<script>` in `apex_line.html`.
- Game loop: fixed timestep accumulator → `step(dt)` per tick → `render()`.
- No build tooling. To syntax-check the inline script:

```bash
node -e "const fs=require('fs');const h=fs.readFileSync('apex_line.html','utf8');const m=h.match(/<script>([\s\S]*?)<\/script>/);fs.writeFileSync('_chk.js',m[1])" && node --check _chk.js && rm -f _chk.js
```
