# Stewart-Hamilton Equation Browser Demonstration
## Implementation Instructions for Cursor AI

---

## Project Overview

Build a single-page, browser-based educational tool demonstrating the Stewart-Hamilton equation in the context of PiCCO thermodilution cardiac output monitoring. Target audience: ICU personnel learning about continuous cardiac output monitoring.

**Key Requirements:**
- Single HTML file (no build step, no dependencies beyond CDN libraries)
- Fully self-contained and client-side only
- Deployable to GitHub Pages
- Mobile-responsive (must work on phones, tablets, and desktops)
- Educational focus: visual, interactive, clinically grounded

---

## Technical Stack

- **HTML5** with semantic markup
- **CSS3** with Flexbox or Grid for responsive layout
- **Vanilla JavaScript** (ES6+)
- **Chart.js** (via CDN) for the concentration-time curve
- **SVG** or **Canvas** for the vascular transit animation
- Optional: **Anime.js** (via CDN) for smooth tracer animation if needed

**No frameworks, no bundlers, no npm.** Everything loads from CDN or is inline.

---

## File Structure

```
repo/
├── index.html          (single file containing all HTML, CSS, and JS)
└── README.md           (usage instructions, equation explanation, clinical context)
```

---

## Core Functionality

### The Stewart-Hamilton Equation

**Cardiac Output (CO) = Amount of Indicator Injected / Area Under the Concentration-Time Curve**

For this demo:
- **CO** is calculated and displayed (liters per minute)
- **Amount of dye** is user-adjustable (milligrams, range: 5-10 mg)
- **Area under curve (AUC)** is user-adjustable (mg·s/L, range: 10-50)
- **Mean transit time (MTT)** is user-adjustable (seconds, range: 10-30 s)

The concentration-time curve is synthesized mathematically from AUC and MTT using a gamma distribution or exponential decay model to match physiological thermodilution waveforms.

---

## Layout Structure

### Desktop Layout (≥768px width)
```
+----------------------------------------------------------+
|  Header: Title + Brief Explanation (collapsible)        |
+----------------------------------------------------------+
|  Control Panel (left, ~250px)  |  Visualization (right)  |
|  - Sliders for:                |  - Vascular Animation   |
|    • Dye amount                |  - Time marker          |
|    • AUC                       |  - Concentration Curve  |
|    • Mean Transit Time         |                         |
|  - Calculated CO display       |                         |
|  - Reset button                |                         |
|  - Download Graph button       |                         |
+----------------------------------------------------------+
```

### Mobile Layout (<768px width)
```
+---------------------------+
|  Header (collapsible)     |
+---------------------------+
|  Sliders (stacked)        |
+---------------------------+
|  Calculated CO            |
+---------------------------+
|  Vascular Animation       |
+---------------------------+
|  Concentration Curve      |
+---------------------------+
|  Download Graph button    |
+---------------------------+
```

---

## Component Specifications

### 1. Control Panel

**Three sliders with real-time value display:**

1. **Amount of Dye Injected**
   - Range: 5-10 mg
   - Step: 0.1 mg
   - Default: 7.5 mg
   - Label: "Dye Injected (mg)"

2. **Area Under Curve (AUC)**
   - Range: 10-50 mg·s/L
   - Step: 1
   - Default: 30 mg·s/L
   - Label: "Area Under Curve (mg·s/L)"

3. **Mean Transit Time (MTT)**
   - Range: 10-30 seconds
   - Step: 1 second
   - Default: 20 seconds
   - Label: "Mean Transit Time (s)"

**Calculated Output Display:**
- Large, prominent display showing: "Cardiac Output: X.X L/min"
- Color-coded based on clinical ranges:
  - Red: <4.0 L/min (low)
  - Green: 4.0-8.0 L/min (normal)
  - Orange: >8.0 L/min (high)
- Include text interpretation below CO value (e.g., "Low CO - consider fluid responsiveness")

**Buttons:**
- "Reset to Defaults" — returns all sliders to default values
- "Download Graph" — saves current concentration curve as PNG

**Mobile Considerations:**
- Slider touch targets: minimum 44×44 px
- Stack sliders vertically with adequate spacing (≥16px between)
- Labels and values: minimum 14px font size

---

### 2. Vascular Transit Animation

**Purpose:** Show the physical path of dye through the cardiovascular system during thermodilution measurement.

**Visual Elements:**

**Vascular Pathway (schematic, not anatomically precise):**
- Right Atrium (injection site)
- Right Ventricle
- Pulmonary Artery
- Pulmonary Capillaries (mixing chamber)
- Pulmonary Veins
- Left Atrium

Draw as simplified boxes or circles connected by tubes/lines. Label each chamber clearly but minimally.

**Tracer Animation:**
- Animated dot or small circle representing the dye bolus
- Starts at Right Atrium when animation begins
- Moves through the pathway at a speed determined by Mean Transit Time
- **Animation duration = MTT** (if MTT = 20s, tracer takes 20s to complete the circuit)
- Visual feedback as tracer disperses:
  - Single dot in RA/RV
  - Fragments into multiple dots or a cloud in pulmonary capillaries (representing mixing/dilution)
  - Reassembles or fades as it exits to LA

**Synchronization:**
- A play/pause button to control animation
- Animation loops continuously by default
- Updates in real-time when sliders change (MTT change = speed change)

**Implementation Approach:**
- Use SVG for the vascular schematic (clean, scalable, mobile-friendly)
- Animate tracer position using JavaScript `requestAnimationFrame` or Anime.js
- SVG paths define the route; tracer follows path coordinates over time

**Mobile Considerations:**
- SVG viewBox scales to container width
- Minimum tap target for play/pause: 44×44 px
- Chamber labels: readable at small sizes (12-14px)

---

### 3. Concentration-Time Curve

**Purpose:** Display the dye concentration curve and show calculated cardiac output.

**Graph Specifications:**

**Axes:**
- X-axis: Time (0-60 seconds)
- Y-axis: Dye Concentration (mg/L), range 0 to peak based on AUC and dye amount
- Labels: "Time (seconds)" and "Concentration (mg/L)"

**Curve Generation:**
Use a **gamma distribution** to model the concentration curve:
- Physiologically realistic (skewed left, exponential decay tail)
- Parameters derived from AUC and MTT
- Alternative: exponential decay if gamma distribution is too complex

**Formula approximation for exponential decay:**
```
C(t) = (Dye_amount / (volume_of_distribution * MTT)) * exp(-t / MTT)
```

Adjust scale factor so the area under the generated curve matches the AUC slider value.

**Visual Elements:**
- Smooth line chart
- Shaded area under curve (semi-transparent fill) to emphasize AUC
- Vertical time marker (dashed line) that moves left-to-right during animation, synced with vascular tracer position
- Annotation box showing current CO value prominently on graph

**Real-Time Updates:**
- Curve recalculates and redraws whenever any slider changes
- Time marker position updates during vascular animation playback

**Chart.js Configuration:**
- Line chart type
- Responsive: true
- MaintainAspectRatio: false (allows flexible height)
- Enable legend: false (single dataset)
- Use Chart.js plugins for annotations if needed, or draw custom overlay

**Mobile Considerations:**
- Canvas scales to container width (100%)
- Readable axis labels and tick marks
- Touch-friendly tooltip on hover/tap

---

### 4. Educational Context Panel

**Header Section (collapsible):**

**Title:** "PiCCO Thermodilution Cardiac Output Monitor - Stewart-Hamilton Equation"

**Brief Explanation (expandable/collapsible on mobile):**
```
The Stewart-Hamilton equation calculates cardiac output (CO) by measuring 
how much a known quantity of indicator (dye or cold saline) is diluted 
as it passes through the heart and lungs.

CO = Amount of Indicator / Area Under the Concentration-Time Curve

In PiCCO monitoring:
- A bolus of cold saline is injected into the right atrium
- Temperature sensors measure the dilution curve
- The area under this curve (AUC) reflects blood flow
- Larger AUC = slower flow = lower cardiac output

Normal CO Range: 4-8 L/min (for an adult at rest)

Use the sliders below to explore how changes in dye amount, dilution 
(AUC), and transit time affect the calculated cardiac output.
```

**Mobile Considerations:**
- Default to collapsed on small screens
- Expand/collapse button clearly visible
- Text: 14-16px, line-height ≥1.5 for readability

---

## Calculation Logic

### Cardiac Output Formula

```javascript
// User inputs from sliders
const dyeAmount = /* slider value in mg */;
const auc = /* slider value in mg·s/L */;

// Calculation
const cardiacOutput = dyeAmount / auc; // Result in L/min

// Display to 1 decimal place
const coDisplay = cardiacOutput.toFixed(1);
```

### Clinical Range Interpretation

```javascript
function interpretCO(co) {
  if (co < 4.0) {
    return { status: 'low', message: 'Low CO - consider fluid responsiveness', color: 'red' };
  } else if (co >= 4.0 && co <= 8.0) {
    return { status: 'normal', message: 'Normal cardiac output', color: 'green' };
  } else {
    return { status: 'high', message: 'High CO - check for sepsis or hyperdynamic state', color: 'orange' };
  }
}
```

### Curve Generation (Exponential Decay Model)

```javascript
// Simplified approach for concentration curve
function generateCurve(dyeAmount, auc, mtt) {
  const points = [];
  const timeMax = 60; // seconds
  const dt = 0.5; // time step
  
  // Scale factor to match desired AUC
  // This is a simplified model - adjust for physiological accuracy
  const k = 1 / mtt; // decay constant
  const C0 = /* calculate peak concentration */;
  
  for (let t = 0; t <= timeMax; t += dt) {
    const concentration = C0 * Math.exp(-k * t);
    points.push({ x: t, y: concentration });
  }
  
  // Normalize curve so integral (AUC) matches slider value
  // Implement numerical integration and scaling
  
  return points;
}
```

**Note to implementer:** Gamma distribution is preferred for realism, but exponential decay is acceptable for MVP. Ensure the generated curve's computed area (numerical integration) matches the AUC slider value.

---

## Edge Cases & Validation

**Prevent Invalid Calculations:**
- Do not allow AUC = 0 (would cause division by zero)
- Clamp slider minimum values to prevent this
- If any slider is at minimum, display a warning: "Invalid configuration"

**Rounding:**
- Cardiac Output: 1 decimal place (5.2 L/min)
- Slider values: as per step size

**Range Warnings:**
- Display clinical interpretation for all CO values (low/normal/high)
- Use color coding for immediate visual feedback

---

## Export Functionality

**Download Graph as PNG:**

```javascript
// Chart.js renders to HTML5 canvas
// Use canvas.toBlob() to export

function downloadGraph() {
  const canvas = document.getElementById('concentrationChart');
  canvas.toBlob(function(blob) {
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.download = 'stewart-hamilton-curve.png';
    link.href = url;
    link.click();
  });
}
```

**Button:**
- Labeled: "Download Graph"
- Position: below the concentration curve
- Mobile: full-width button, minimum 44px height

---

## Responsive Design

### CSS Media Queries

```css
/* Desktop: side-by-side layout */
@media (min-width: 768px) {
  .container {
    display: grid;
    grid-template-columns: 250px 1fr;
  }
}

/* Mobile: stacked layout */
@media (max-width: 767px) {
  .container {
    display: flex;
    flex-direction: column;
  }
  
  .slider-container {
    width: 100%;
    margin-bottom: 16px;
  }
  
  .button {
    width: 100%;
    min-height: 44px;
  }
}
```

### Viewport Meta Tag

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

### Touch-Friendly Targets

- All interactive elements: minimum 44×44 px
- Adequate spacing between controls (≥16px)
- Text: minimum 14px for body, 16px for labels

---

## Animation Synchronization

**Key Requirement:** The vascular animation and concentration curve time marker must stay in sync.

**Implementation Strategy:**

1. **Shared Time State:**
   - Maintain a global `currentTime` variable (0 to MTT seconds)
   - Updated by `requestAnimationFrame` during animation playback

2. **Vascular Tracer Position:**
   - Calculate position along SVG path based on `currentTime / MTT`
   - Use SVG `getTotalLength()` and `getPointAtLength()` for smooth path following

3. **Time Marker on Graph:**
   - Draw vertical line at x = `currentTime` on Chart.js canvas
   - Update using Chart.js annotation plugin or custom canvas overlay

4. **Play/Pause Control:**
   - Button toggles animation state
   - On pause: both visuals freeze at current time
   - On play: both resume from frozen position

**Pseudocode:**
```javascript
let currentTime = 0;
let isPlaying = false;
let animationId;

function animate() {
  if (isPlaying) {
    currentTime += deltaTime;
    if (currentTime > mtt) currentTime = 0; // loop
    
    updateTracerPosition(currentTime);
    updateTimeMarker(currentTime);
    
    animationId = requestAnimationFrame(animate);
  }
}

function playPause() {
  isPlaying = !isPlaying;
  if (isPlaying) animate();
}
```

---

## GitHub Pages Deployment

**Steps:**
1. Create a GitHub repository
2. Place `index.html` in the root directory or `/docs` folder
3. Enable GitHub Pages in repository settings:
   - Settings → Pages
   - Source: Deploy from a branch
   - Branch: main, folder: / (root) or /docs
4. Access at: `https://<username>.github.io/<repo-name>/`

**No additional configuration needed.** Single HTML file serves directly.

---

## Testing Checklist

**Functional:**
- [ ] Sliders update CO calculation in real time
- [ ] CO display shows correct value with 1 decimal place
- [ ] CO interpretation (low/normal/high) updates with color coding
- [ ] Concentration curve redraws when sliders change
- [ ] Vascular animation speed matches MTT slider
- [ ] Animation play/pause works correctly
- [ ] Time marker on curve syncs with vascular tracer
- [ ] Reset button returns all sliders to defaults
- [ ] Download button saves PNG of current graph

**Responsive:**
- [ ] Layout switches to stacked on mobile (<768px)
- [ ] All text is readable on small screens (≥14px)
- [ ] Touch targets are adequate (≥44×44 px)
- [ ] No horizontal scrolling on any screen size
- [ ] SVG vascular animation scales correctly
- [ ] Chart.js graph fills container width

**Cross-Browser:**
- [ ] Chrome (desktop and mobile)
- [ ] Firefox
- [ ] Safari (desktop and iOS)
- [ ] Edge

**Edge Cases:**
- [ ] All sliders at minimum values (but above zero)
- [ ] All sliders at maximum values
- [ ] Rapid slider changes (no lag, smooth updates)

---

## Optional Enhancements (Post-MVP)

- **Preset Scenarios:** Buttons for "Normal", "Septic Shock", "Cardiogenic Shock" that set sliders to typical values
- **CSV Import:** Allow upload of time/concentration data to compute AUC algorithmically
- **Comparison Mode:** Show two curves side-by-side for before/after comparison
- **Extended Explanation:** Link to full clinical guide or PiCCO documentation

---

## Clinical Context Notes

**Target Audience:** ICU nurses, intensivists, advanced clinical practitioners learning PiCCO monitoring

**Learning Objectives:**
- Understand the relationship between indicator dilution and cardiac output
- Visualize the physical path of thermodilution
- Recognize the effect of AUC and transit time on CO calculation
- Interpret CO values in clinical context (low/normal/high)

**Clinical Ranges (Adult at Rest):**
- Normal CO: 4-8 L/min
- Low CO (<4 L/min): suggests reduced cardiac function, hypovolemia
- High CO (>8 L/min): suggests hyperdynamic state (sepsis, fever, etc.)

**Typical PiCCO Values:**
- Injectate volume: 10-20 mL cold saline (not shown in this demo)
- Measurement time: ~60 seconds
- Curve shape: rapid rise, exponential decay

---

## References for Implementer

**Stewart-Hamilton Equation:**
- Stewart, G. N. (1897). "Researches on the Circulation Time and on the Influences which affect it"
- Hamilton, W. F., et al. (1932). "Studies on the circulation"

**PiCCO Monitoring:**
- Pulse Contour Cardiac Output (PiCCO) technology overview
- Thermodilution principles in critical care

**Chart.js Documentation:**
- https://www.chartjs.org/docs/latest/

**SVG Animation:**
- MDN Web Docs: SVG and Animation
- Anime.js documentation (if used): https://animejs.com/

---

## Implementation Priorities

**Phase 1 (MVP):**
1. Build HTML structure with sliders and displays
2. Implement CO calculation logic
3. Generate concentration curve using exponential decay
4. Render curve with Chart.js
5. Add basic vascular schematic (static boxes/lines)
6. Implement responsive CSS

**Phase 2:**
7. Add vascular tracer animation
8. Sync time marker on curve with tracer
9. Implement play/pause control
10. Add download graph functionality

**Phase 3:**
11. Refine curve generation (gamma distribution if needed)
12. Add educational context panel
13. Polish mobile experience
14. Cross-browser testing

---

## Final Notes

- Keep the code clean and well-commented for maintainability
- Use semantic HTML5 elements where appropriate
- Ensure accessibility: labels, ARIA attributes, keyboard navigation
- Optimize for performance: avoid unnecessary redraws, debounce slider events if needed
- Test on actual mobile devices, not just browser dev tools

This is an educational tool, not a diagnostic device. Include a disclaimer that it is for learning purposes only and should not be used for clinical decision-making.

---

**End of Implementation Instructions**
