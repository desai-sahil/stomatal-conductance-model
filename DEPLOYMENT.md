# Deploying the dashboards to GitHub Pages with Pyodide

This guide takes the three existing notebooks and gets them running in the browser
via [`panel convert`](https://panel.holoviz.org/how_to/wasm/convert.html), served
as static files from GitHub Pages. No backend, no Colab redirect.

The final URL will be:
**`https://desai-sahil.github.io/stomatal-conductance-model/`**

---

## Step 1 — Make three small changes to each notebook

### 1a. First code cell: load the Panel extension

Add this near the top of each notebook (above any `pn.Column`, `pn.Row`,
`pn.widgets.*` use). It tells Panel which renderer the page needs:

```python
import panel as pn
pn.extension("bokeh")
```

If any dashboard also uses `pn.widgets.Tabulator`, add `"tabulator"` to the
extension call: `pn.extension("bokeh", "tabulator")`.

### 1b. CSV reads must use a URL (only matters for `merilo_transients_dashboard.ipynb`)

Pyodide does not have a normal filesystem when served from GitHub Pages, but
pandas can read directly from a URL. Replace each `pd.read_csv("data/...")`
with the raw GitHub URL:

```python
DATA_BASE = (
    "https://raw.githubusercontent.com/"
    "desai-sahil/stomatal-conductance-model/main/data/"
)

df_wt    = pd.read_csv(DATA_BASE + "merilo_2018_wildtype_vpd_change.csv")
df_ost1  = pd.read_csv(DATA_BASE + "merilo_2018_ost1_vpd_change.csv")
df_aao32 = pd.read_csv(DATA_BASE + "merilo_2018_aao32_vpd_change.csv")
```

This works identically when running the notebook locally and when running
inside Pyodide in the browser.

### 1c. Last cell: mark what should be served

Panel only renders what is explicitly marked `.servable()`. At the very bottom
of each notebook, build a single layout containing everything you want the
visitor to see, and mark it servable. Something like:

```python
pn.Column(
    "# Stomatal oscillations dashboard",
    "Three-step humidity oscillation protocol (80% → 55% → 30% RH).",
    pn.Tabs(
        ("Hydraulics",      hydraulics_dashboard),
        ("V_stress",        vstress_dashboard),
        ("ABA biosynthesis", aba_biosynthesis_dashboard),
        ("ABA signaling",   aba_signaling_dashboard),
        ("Electrophysiology", electrophysiology_dashboard),
    ),
).servable()
```

If you also want to keep the static matplotlib figures visible in the
deployed page, wrap each one in a Matplotlib pane:

```python
pn.pane.Matplotlib(fig_oscillations, tight=True, dpi=120)
```

…and slot it into the `pn.Column` above. (Cells that just *display* a
matplotlib figure inline won't appear in the served app — only `.servable()`
content does.)

---

## Step 2 — Drop in the GitHub Action

Create the file `.github/workflows/deploy.yml` in the repo (the contents are
in the `deploy.yml` file alongside this guide). On every push to `main` it
will:

1. Install your `requirements.txt`.
2. Run `panel convert` on the three notebooks with `--to pyodide-worker`
   (Python runs in a Web Worker, so the UI stays responsive during heavy
   ODE integration).
3. Generate an `index.html` listing the three dashboards (`--index`).
4. Publish the result to GitHub Pages.

---

## Step 3 — Enable GitHub Pages

In the repo on github.com:

1. **Settings → Pages**
2. **Source**: select **GitHub Actions** (not "Deploy from branch")
3. Push any commit to `main` to trigger the first build.

The first run takes ~3–5 minutes (Panel + scientific stack install). Subsequent
runs are faster thanks to the pip cache.

---

## Step 4 — Try it locally first (optional but recommended)

Before pushing, you can dry-run the build:

```bash
pip install -r requirements.txt
panel convert hysteresis_dashboard.ipynb \
              oscillations_dashboard.ipynb \
              merilo_transients_dashboard.ipynb \
  --to pyodide-worker \
  --out site \
  --index \
  --requirements numpy scipy pandas matplotlib bokeh panel

# Serve the result
python -m http.server -d site 8000
```

Then open `http://localhost:8000` in your browser.

The first load will take 10–30 seconds while Pyodide downloads the scientific
stack (numpy, scipy, pandas, matplotlib all together are ~20 MB). After that,
slider interaction is near-instant. The bundle is cached by the browser, so
return visits skip the cold start.

---

## What to expect performance-wise

| Action                          | Time          |
|---------------------------------|---------------|
| First load (cold)               | 10–30 s       |
| First load (warm/cached)        | 1–2 s         |
| Slider drag → plot update       | <100 ms typically; up to ~1 s if a heavy `solve_ivp` runs |
| ODE integration in browser      | ~3–5× slower than native CPython, but still interactive for stomatal-model timesteps |

If a particular dashboard turns out to be too slow inside Pyodide (most likely
candidate: anything doing very long `solve_ivp` integrations on every slider
move), the fixes in order of effort are:

1. **Throttle the slider** with `pn.bind(..., throttled=True)` — only triggers
   on mouse-up, not on every drag tick.
2. **Cache the heavy compute** with `@pn.cache` so identical parameter sets
   are reused.
3. **Lower the default time-grid resolution** for the live preview, and add a
   "high-resolution" toggle.

If those don't get you there, deploy that specific notebook to a Hugging Face
Space (real CPython, free, ~30 s cold start) and link to it from the index.

---

## Troubleshooting

**Build fails in `panel convert` with a Bokeh version mismatch.**
Pin `bokeh` and `panel` to a known-good pair in `requirements.txt`. As of
April 2026, `panel==1.5.*` with `bokeh==3.6.*` is reliable.

**Dashboard appears but sliders do nothing.**
Almost always means `pn.extension("bokeh")` is missing or runs *after* the
widgets are constructed. It must be in the very first executed cell.

**Blank page, browser console says "Pyodide not initialized".**
Check that the `.html` file is being served from a real HTTP server (or
GitHub Pages), not opened with `file://`. Pyodide's worker needs cross-origin
isolation that `file://` doesn't provide.

**CSVs fail to load (only in transients dashboard).**
The URL from Step 1b must be the `raw.githubusercontent.com` host, not
`github.com/.../blob/...`. The latter returns HTML, not CSV.
