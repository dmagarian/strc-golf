# PAR — STRC on the course

A self-contained HTML proof of concept: a stylized Michael Saylor takes a golf shot every minute, with STRC's simulated price controlling accuracy and relative trading volume controlling power. The 3D course has a fairway, rough, bunkers, water, trees, and a 320-yard pin.

**Live demo:** <https://dmagarian.github.io/strc-golf/>

## Publishing

GitHub Pages publishes the root of the `main` branch. Push changes to `index.html` to update the demo automatically. `.nojekyll` tells Pages to serve the files directly. The public site requires no GitHub sign-in and uses simulated market data.

## Open it

Double-click `index.html`, or drop it into a modern browser. It contains the application, Three.js, both fonts, and third-party licenses. No install, build, API key, or internet connection is required. WebGL must be available.

To serve it locally:

```sh
python3 -m http.server 8080
```

Then open <http://localhost:8080>.

## Play

- Change price and volume with the sliders or number fields, then click **Take the shot** or press **Space**.
- Try **The dip**, **Below par**, **At par**, **Volume surge**, and **Whiff** presets.
- The automatic-shot switch enables a shot every 60 seconds. Each automatic shot makes a small simulated change to the current price and volume. A manual shot restarts the countdown. Returning from a hidden tab starts a fresh minute instead of replaying missed shots.
- Use the buttons in the course to follow the ball, inspect the golfer, switch to an overhead view, or enable synthesized sound effects. Drag the course horizontally to rotate the overview.
- The scorecard records the last five completed shots. Data resets on reload.

## The formula

Price and volume are sampled at tee-off, so changing controls during flight affects the next shot.

```text
precision = exp(-(abs(price - 100) / 10)^2)
power     = clamp(log2(1 + volume) / 2, 0, 1.4)
carry     = min(320, 210 + 45 * log2(1 + volume))
distance  = round(carry * (0.55 + 0.45 * precision))
apex      = 12 + 15 * power
```

| Simulated price | Landing |
| --- | --- |
| ≤ $68 | Missed swing |
| > $68 and < $73 | Water |
| $73 to < $83 | Trees |
| $83 to < $93 | Bunker |
| $93 to < $99 | Rough |
| $99 to $103 | Fairway; hole-in-one if volume ≥ 3× |
| > $103 to $110 | Rough |
| > $110 | Bunker |

Hazard shots target the corresponding physical feature on the course, with the displayed distance calculated from that target. Volume still changes their flight height. Hole-in-one shots target the actual cup. This is an intentionally playful formula, not a prediction model or a golf physics engine.

## Market data and character

**This version uses simulated data, not live STRC quotes.** No current stock price is claimed. Volume is a multiplier against an assumed average one-minute trading volume, not an actual traded share count. The character is an unofficial low-poly caricature, not a photorealistic model.

For a live-data version, connect a market-data provider to the internal `setMarket(price, volumeMultiplier)` function and remove the random market changes in `updateCountdown`. Keep API credentials on a server; derive the multiplier from the latest minute's traded shares and a comparable average. Select the provider and its market-hours/delayed-data policy before wiring that in.

## Implementation

All editable application code and styles are in `index.html`. The named `shot-model` script holds the pure formula; `golf-app` holds the UI, course, golfer, animation, and timer. The `three-library` script embeds pinned Three.js 0.160.1's classic build so direct `file://` opening works. Its upstream deprecation warning is expected; a future build-based application should use ES modules. DM Sans and Space Grotesk's Latin variable fonts are embedded as WOFF2.

Third-party notices are embedded in the HTML and retained in `vendor/`. Three.js is MIT licensed; both fonts use SIL OFL 1.1. Renderer reference: [official Three.js documentation](https://threejs.org/docs/pages/WebGLRenderer.html).
