# DELIVERY — Fragile Cargo v1.0.0 (2026-11-01)

Delivery note for the LOB7 edition. Honest account of what was tested, tuned
and what remains approximate.

## Specification interpretation

- **"Simple deterministic balance model"** — cargo torque is Σ(weight × mean
  arm from the centre column); wave tilt is scripted per crossing and added
  in degrees; the tilt angle follows its target through a fixed-step (1/60 s)
  exponential smoothing (τ = 0.55 s). No physics library, no randomness in
  the engine (RNG only in cosmetic particles).
- **"Weight → draft, distribution → tilt"** — total weight sets the hull's
  draught (LOAD gauge, hull visually sinks). Overload beyond capacity
  amplifies waves (+1.2 per unit of overload); beyond 125 % of capacity the
  crossing is refused at departure. Distribution sets the tilt needle.
- **"Dangerous prolonged tilt"** — ≥ 18° for 1.4 s (cumulative, decays 1.6×
  faster outside the zone) drops the most exposed parcel on the low side
  (max of weight × arm × tilt sign), with a 0.5 s grace after each fall.
- **"One parcel at a time"** — during the crossing a slide (translation
  only, no rotation) locks further moves for its 280 ms animation.
- **Bonus parcels** may be left on the dock (less weight, fewer points).

## Tests (all against the real index.html, loaded in a Node vm)

16/16 green:

- shape rotation/normalization; placement collisions, bounds, occupancy;
- torque proportionality and symmetry; draft ratio and wave amplification;
  departure refusal (missing required / overload);
- wave announcement ≥ 1.5 s before impact; envelope ramps; event order;
- sustained-danger fall (most exposed object, required ⇒ run fails);
  bonus fall ≠ failure; danger-meter grace;
- crossing moves: collision rejection, one-at-a-time lock, translation only;
- **frame-rate independence**: identical final state fed through the
  fixed-step accumulator at 30, 60 and 144 fps (levels 1, 4, 6);
- score computation and stars;
- **solvability**: a deterministic captain-bot clears all six crossings with
  every bonus aboard (greedy balanced placement + one-move tilt corrections);
- difficulty curve (wave peaks non-decreasing; tutorials ≤ 7°, finale 21°);
- i18n key parity EN/FR/ES; level integrity (manifest fits deck, weights
  within capacity rules).

Difficulty probe: a *passive* captain (good placement, no mid-crossing
moves) survives levels 1–3 but loses required cargo on 4 and 6 (level 5
survives only with the bonus acting as ballast) — mid-crossing moves are
what the late game demands.

## Real browser play (in-app browser, 1280×720 and 390×740)

- Home screen, level select, unlock persistence (localStorage).
- Level 1 played end-to-end with **real pointer events** (drag from dock,
  rotate mid-drag via button, tap-select + tap-green-cell move), wave banner
  countdown → "hitting now!", arrival screen with score/stars, unlock.
- Failure flow: everything stacked on one edge → barrel overboard → fail
  screen → retry back to placement.
- Languages switched live (EN/FR/ES), portrait layout verified (deck grid
  fully above the waterline, gauges readable).
- Pixel probes confirmed the illustration renders (cyan sea, vermillion
  hull, deck, parcel colours) even when the test panel throttles rAF; the
  embedded watchdog timer keeps the simulation running at reduced rate.
- Audio implemented (WebAudio synth) but not critically ear-tested
  end-to-end; mute available.

## Known limits

- The in-app test browser throttles rAF when unfocused: simulated time runs
  at reduced rate there (the watchdog keeps it alive); on a normal browser
  the accumulator runs at full rate. Final states are provably
  frequency-independent (see tests).
- Difficulty tuned against the captain-bot and manual runs; no large-sample
  playtesting. Late waves (19–21°) require reading the banner and reacting
  within ~2.5 s — intentional.
- No gamepad support; rotation during the crossing is deliberately absent
  (orientation is a pre-departure decision).

## Tooling

- Built entirely with Z-Code (GLM). Test harness + cover generator live in
  the workshop (`games/cargaison-fragile-dev/`), outside this repository
  (static package rules: no build scripts shipped).
