# Stomatal Model Interactive Dashboards

Three interactive Jupyter notebooks implementing the combined stomatal model from Merilo et al. (2018), each exploring a different experimental protocol.

## Live dashboards

A browser-hosted version of all three dashboards (no install required) is published at:

**https://desai-sahil.github.io/stomatal-conductance-model/**

The dashboards run entirely client-side via Pyodide — first load takes ~10–30 s while the scientific stack downloads, then slider interaction is near-instant. See [`DEPLOYMENT.md`](DEPLOYMENT.md) for how the build is wired up (GitHub Action + `panel convert`).

| Notebook | Description |
|----------|-------------|
| `merilo_transients_dashboard.ipynb` | VPD step-response transients (80% → 60% RH) with experimental data overlay |
| `oscillations_dashboard.ipynb` | Three-step humidity oscillation protocol (80% → 55% → 30% RH) |
| `hysteresis_dashboard.ipynb` | Drydown / rewatering hysteresis cycle (ψ_xyl steps) |

Each notebook generates publication-quality static figures **and** five interactive Panel dashboards (Hydraulics, V_stress, ABA Biosynthesis, ABA Signaling, Electrophysiology).

---

## Folder Structure

```
├── README.md
├── DEPLOYMENT.md                ← how the GitHub Pages build works
├── requirements.txt
├── hysteresis_dashboard.ipynb
├── oscillations_dashboard.ipynb
├── merilo_transients_dashboard.ipynb
├── .github/
│   └── workflows/
│       └── deploy.yml           ← Action that publishes to GitHub Pages
└── data/
    ├── merilo_2018_wildtype_vpd_change.csv
    ├── merilo_2018_ost1_vpd_change.csv
    └── merilo_2018_aao32_vpd_change.csv
```

> **Important:** Keep `data/` in the same directory as the notebooks. The transients notebook reads CSVs from `data/`.

---

## Option A — Terminal (pip + venv)

Use this if you have Python installed and prefer working from the command line.

### 1. Open a terminal and navigate to this folder

```bash
cd path/to/this/folder
```

### 2. Create a virtual environment (recommended)

```bash
python3 -m venv venv
```

### 3. Activate the virtual environment

**macOS / Linux:**
```bash
source venv/bin/activate
```

**Windows (Command Prompt):**
```cmd
venv\Scripts\activate
```

**Windows (PowerShell):**
```powershell
venv\Scripts\Activate.ps1
```

### 4. Install dependencies

```bash
pip install -r requirements.txt
```

### 5. Launch Jupyter Notebook

```bash
jupyter notebook
```

This opens a browser window. Click on any of the three dashboard notebooks to open and run it.

### 6. Run all cells

Inside the notebook, use **Cell → Run All** (or press `Shift+Enter` through each cell). The interactive dashboards will appear inline after the static figures.

### 7. Deactivate when done

```bash
deactivate
```

---

## Option B — Anaconda / Conda

Use this if you have Anaconda or Miniconda installed.

### 1. Open Anaconda Prompt (Windows) or Terminal (macOS/Linux)

### 2. Create a new conda environment

```bash
conda create -n stomatal-dashboards python=3.11 -y
```

### 3. Activate the environment

```bash
conda activate stomatal-dashboards
```

### 4. Install dependencies

```bash
pip install -r requirements.txt
```

Or install individually via conda + pip:

```bash
conda install numpy scipy pandas matplotlib bokeh -y
pip install panel jupyter
```

### 5. Launch Jupyter Notebook

**From the terminal:**
```bash
jupyter notebook
```

**Or from Anaconda Navigator:**
1. Open Anaconda Navigator
2. Switch the environment dropdown (top) to **stomatal-dashboards**
3. Click **Launch** under Jupyter Notebook
4. Navigate to this folder and open any dashboard notebook

### 6. Run all cells

Use **Cell → Run All** or step through with `Shift+Enter`. The interactive Panel dashboards will render below the static matplotlib figures.

### 7. Deactivate when done

```bash
conda deactivate
```

---

## Troubleshooting

- **"No module named panel"** — Make sure you installed requirements in the correct environment. Run `pip install panel` in the same environment where Jupyter is running.
- **FileNotFoundError for CSVs** — Make sure the `data/` folder is in the same directory as the notebooks, and that you launched Jupyter from this folder.
- **Dashboards not displaying** — Panel widgets require a running Jupyter kernel. If you see blank output, try restarting the kernel and running all cells again.
- **Bokeh version warnings** — Safe to ignore. The dashboards work with Bokeh 3.x.
