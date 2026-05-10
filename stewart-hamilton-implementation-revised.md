# Revised Stewart-Hamilton Implementation Plan

**Status:** Ready for implementation
**Supersedes:** `stewart-hamilton-implementation.md`

## Summary of Changes from Original

The original plan had a critical formula error and terminology mismatch. This revision:

1. Fixes the unit/formula error (original `CO = dye / AUC` produced L/s, not L/min, and the slider ranges gave clinically implausible values)
2. Switches to temperature-based thermodilution to match how PiCCO actually works (cold saline + thermistor, not optical dye)
3. Treats CO as a full peer variable so any one of the four can be the unknown (lock/unlock UI)
4. Resolves moderate implementation gaps: undefined `C0`, missing `deltaTime`, missing CDN pins, ambiguous chart annotation strategy

---

## Corrected Formula

The thermodilution Stewart-Hamilton equation:

```
CO (L/min) = [V_i(mL) / 1000 × K × (T_b − T_i) × 60] / AUC (°C·s)
```

- `V_i` = injectate volume (mL)
- `T_b` = blood temperature, **fixed at 37 °C**
- `T_i` = injectate temperature (°C)
- `K` = correction factor, **fixed at 1.08** (specific heat/density ratio for cold saline in blood)
- `AUC` = area under the temperature-time curve (°C·s)
- Multiply numerator by 60 to convert L/s → L/min

**AUC slider range calibration** — at default values (15 mL, 4 °C injectate → ΔT = 33 °C):

```
CO = (0.015 × 1.08 × 33 × 60) / AUC = 32.1 / AUC
```

- AUC = 4 °C·s → CO ≈ 8 L/min (upper normal)
- AUC = 8 °C·s → CO ≈ 4 L/min (lower normal)

**Slider AUC range: 3–12 °C·s** covers ~2.7–10.7 L/min across all slider combinations.

---

## Four Bidirectional Variables

The app treats CO as a full peer variable, not just an output. Any one of the four can be the unknown; the other three act as inputs.

| Variable | Valid Range | Step | Default | Units |
|---|---|---|---|---|
| Cardiac Output (CO) | 1–20 L/min | 0.1 | 5.0 | L/min |
| Injectate Volume (V_i) | 10–20 mL | 1 | 15 | mL |
| Injectate Temperature (T_i) | 0–8 °C | 0.5 | 4 | °C |
| AUC | 3–12 °C·s | 0.5 | 6.4 | °C·s |

**All four rearrangements:**

```javascript
const K = 1.08, T_b = 37;

function solveFor(unknown, { co, vi, ti, auc }) {
  const dT = T_b - ti;
  switch (unknown) {
    case 'co':  return (vi / 1000 * K * dT * 60) / auc;
    case 'vi':  return (co * auc * 1000) / (K * dT * 60);
    case 'ti':  return T_b - (co * auc * 1000) / (vi * K * 60);
    case 'auc': return (vi / 1000 * K * dT * 60) / co;
  }
}
```

**Valid ranges for impossibility detection:**

| Variable | Physically impossible if... |
|---|---|
| CO | < 0 or > 20 L/min |
| V_i | < 10 or > 20 mL |
| T_i | < 0 or > 8 °C (also must be < T_b = 37) |
| AUC | ≤ 0 °C·s |

---

## Lock / Unlock UI

Each variable row has a padlock icon (open/closed) at the right end:

- **Locked:** slider is active and user-adjustable
- **Unlocked:** slider is greyed out and replaced by a computed read-only value display; this is the variable being solved for
- **Only one variable can be unlocked at a time.** Unlocking a second variable automatically re-locks the previously unlocked one.
- Default: CO is unlocked (original behaviour — CO is calculated from the three input sliders)

**Visual treatment of the unlocked variable:**

- Slider replaced by a large bold computed value in a highlighted box
- Row background tinted to distinguish it from input rows
- Lock icon animated (subtle bounce) briefly when the solved value updates

**Impossibility state:**

- When the computed result is outside the valid range, the computed value box shows a red warning instead of a number:

  > "Impossible — adjust inputs (e.g. injectate temp cannot exceed blood temp 37 °C)"

- The chart and animation pause/grey out until a valid configuration is restored

---

## Animation Speed

MTT is **removed as a user-facing slider**. Animation speed is derived from CO (higher CO → shorter loop duration): `animDuration_s = 60 / CO`. When CO is the unknown, animation updates live as inputs change.

---

## Terminology Changes (throughout all labels, tooltips, educational text)

- "Dye" → "cold saline"
- "Concentration (mg/L)" → "Temperature Change (°C)"
- "AUC units" → "°C·s"
- "Concentration-Time Curve" → "Temperature-Time Curve"
- Vascular animation description: "cold saline bolus" instead of "tracer"
- Y-axis label: "ΔTemperature (°C)"

The educational panel should explain: blood temperature briefly drops as cold saline mixes, the thermistor on the arterial catheter records this drop, and the AUC reflects how much dilution occurred.

---

## Curve Generation Fix

The temperature-time curve uses an exponential decay shape in temperature units. The formula with a correct peak derivation:

```javascript
function generateCurve(volume_mL, T_injectate, auc_CCs) {
  const K = 1.08;
  const T_blood = 37;
  const deltaT_input = T_blood - T_injectate;
  const points = [];
  const timeMax = 60; // seconds
  const dt = 0.5;

  // For exponential decay: integral from 0 to infinity of C0 * exp(-t/tau) dt = C0 * tau
  // We want C0 * tau = auc_CCs; tau is a fixed shape parameter
  const tau = 10; // shape constant (seconds)
  const C0 = auc_CCs / tau; // ensures integral matches AUC slider

  for (let t = 0; t <= timeMax; t += dt) {
    const dTemp = C0 * Math.exp(-t / tau);
    points.push({ x: t, y: dTemp });
  }
  return points;
}
```

This resolves the undefined `C0` and `volume_of_distribution` stubs in the original.

---

## Animation Fix: deltaTime

The `deltaTime` gap in the original pseudocode is resolved by using RAF timestamps:

```javascript
let lastTimestamp = null;

function animate(timestamp) {
  if (!lastTimestamp) lastTimestamp = timestamp;
  const deltaTime = (timestamp - lastTimestamp) / 1000; // seconds
  lastTimestamp = timestamp;

  currentTime += deltaTime;
  const animDuration = 60 / cardiacOutput; // derived from CO
  if (currentTime > animDuration) currentTime = 0; // loop

  updateTracerPosition(currentTime / animDuration); // 0–1 progress
  updateTimeMarker(currentTime);

  if (isPlaying) animationId = requestAnimationFrame(animate);
}
```

---

## Chart.js Time Marker — Recommended Approach

Use a **custom canvas overlay** instead of the annotation plugin (avoids an additional CDN dependency):

```javascript
function drawTimeMarker(chartInstance, currentTime) {
  const xScale = chartInstance.scales.x;
  const xPixel = xScale.getPixelForValue(currentTime);
  const ctx = chartInstance.ctx;

  ctx.save();
  ctx.beginPath();
  ctx.moveTo(xPixel, chartInstance.chartArea.top);
  ctx.lineTo(xPixel, chartInstance.chartArea.bottom);
  ctx.strokeStyle = 'rgba(255, 100, 100, 0.8)';
  ctx.setLineDash([5, 5]);
  ctx.stroke();
  ctx.restore();
}
```

Call this inside a Chart.js `afterDraw` plugin hook — no extra CDN needed.

---

## CDN URLs to Include

```html
<!-- Chart.js (pinned version) -->
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>

<!-- Anime.js (optional, only if SVG path animation complexity warrants it) -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/animejs/3.2.1/anime.min.js"></script>
```

---

## Educational Panel Updates

Add a note explaining the K correction factor:

> "The correction factor K = 1.08 accounts for the difference in specific heat and density between cold saline and blood."

Display the full equation with values substituted in real time so learners see how each variable contributes. When the unlocked variable changes, the substituted equation should highlight the term being solved for.

---

## Items Unchanged from Original Plan

- Single `index.html` file, no build step
- GitHub Pages deployment
- SVG vascular schematic with `getTotalLength()` / `getPointAtLength()`
- Responsive layout (desktop: grid, mobile: stacked flex)
- Download graph as PNG using `canvas.toBlob()`
- Clinical CO interpretation (low/normal/high with colour coding)
- Reset button
- Educational disclaimer (for learning only, not clinical decision-making)
- Phase 1/2/3 implementation priority order

---

## Implementation Todo Checklist

- [ ] Implement corrected thermodilution CO formula with unit conversion and K=1.08 constant; implement all four rearrangements for bidirectional solving
- [ ] Build lock/unlock icon UI — locked variables are sliders, unlocked variable is read-only computed output; only one variable unlocked at a time
- [ ] Replace dye/AUC/MTT sliders with Volume/Injectate Temp/AUC/CO controls using calibrated ranges
- [ ] Implement impossibility detection — when computed result is outside valid range, block display and show explanatory message
- [ ] Replace all "dye" and "concentration" terminology with cold saline / temperature throughout
- [ ] Implement temperature-time curve with correct C0 derivation (C0 = AUC/tau)
- [ ] Fix animation loop to derive deltaTime from RAF timestamps; derive duration from CO
- [ ] Implement time marker using Chart.js afterDraw plugin hook (no annotation CDN needed)
- [ ] Pin CDN URLs for Chart.js 4.4.0 (and optionally Anime.js 3.2.1)
- [ ] Update educational panel to explain temperature-based thermodilution and display live equation substitution
