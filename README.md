# Stewart-Hamilton PiCCO Demonstrator

Single-page educational browser demo for exploring the Stewart-Hamilton equation in PiCCO-style cold saline thermodilution.

## Open Locally

Open `index.html` directly in a browser. The page is self-contained apart from Chart.js, which loads from jsDelivr:

```html
https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js
```

## What It Demonstrates

The demo uses the temperature-based Stewart-Hamilton relationship:

```text
CO = [V_i / 1000 * K * (T_b - T_i) * 60] / AUC
```

- `CO`: cardiac output in L/min
- `V_i`: cold saline injectate volume in mL
- `K`: correction factor, fixed at 1.08
- `T_b`: blood temperature, fixed at 37 C
- `T_i`: injectate temperature in C
- `AUC`: area under the temperature-time curve in C*s

The lock controls let learners choose any one of the four variables as the calculated unknown while adjusting the other three.

## Deploy to GitHub Pages

1. Commit `index.html` and `README.md` to the repository root.
2. In GitHub, open repository Settings > Pages.
3. Choose "Deploy from a branch".
4. Select the main branch and root folder.
5. Open the published Pages URL.

## Clinical Disclaimer

This is an educational model only. It is not a diagnostic tool and must not be used for clinical decision-making.
