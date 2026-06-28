# Impulse Responses to Monetary Policy Shocks: Local Projections and Proxy SVAR

Estimates the dynamic causal effects of US monetary policy shocks on real GDP and CPI
using two complementary approaches — Local Projections (Jordà, 2005) and a Proxy SVAR /
SVAR-IV (Stock & Watson, 2012; Mertens & Ravn, 2013) — on quarterly US data, with a
provided Romer–Romer style narrative shock series used as the external instrument.

## Requirements

- **Python 3.10+**
- **Jupyter** (JupyterLab or the VS Code/Notebook interface) to open and run the `.ipynb` files
- The following Python libraries must be installed in the environment used to run the
  notebooks:

  | Library | Used for | Note |
  |---------|----------|------|
  | `pandas` | data loading, transforms, merging | **version ≥ 2.2** (uses `pd.PeriodIndex.from_fields`) |
  | `numpy` | arrays, linear algebra (companion matrix, bootstrap) | |
  | `statsmodels` | OLS + Newey–West HAC SEs, VAR(4), IV2SLS | |
  | `scipy` | normal critical values for confidence bands | |
  | `matplotlib` | IRF plots | |

  Install everything at once:

  ```bash
  pip install "pandas>=2.2" numpy statsmodels scipy matplotlib jupyter
  ```

## How to run

Open each notebook and run it top-to-bottom ("Restart Kernel & Run All"), in this order:

1. **`Data_Merge.ipynb`** — builds `merged_data.csv` from the FRED CSVs + `shocks.csv`
2. **`part_A.ipynb`** — Local Projections → saves `irf_gdp_lp.png`, `irf_cpi_lp.png`
3. **`part_B.ipynb`** — Proxy SVAR → saves `irf_gdp_svar.png`, `irf_cpi_svar.png`

`part_A` and `part_B` both read `merged_data.csv`, so `Data_Merge` must be run first.

## Input files (expected in the same folder)

- `GDPC1.csv`, `CPIAUCSL.csv`, `FEDFUNDS.csv`, `UNRATE.csv` — downloaded from FRED
- `shocks.csv` — provided monetary policy shock series

## Assumptions & Judgment Calls

1. **Monthly → quarterly = period averaging.** CPI, FFR, and unemployment are averaged
   over the three months of each quarter (GDP is already quarterly). GDP and CPI are then
   logged; FFR and unemployment stay in percent.
2. **Sample period.** Macro data span 1969Q1–2019Q4, but the shock series ends in 2007Q4,
   which binds estimation. The effective estimation sample is 1970Q1–2007Q4 (the 1969
   quarters are consumed as initial lags).
3. **Post-2007 macro data retained, not truncated.** Because Local Projections regress the
   lead `y(t+h)` on the current shock, post-2007 macro observations are kept so long-horizon
   leads remain available. This holds the LP sample at a constant 152 observations across
   all horizons.
4. **Unemployment enters contemporaneously only.** Equation (2) lists `unrate_t` without a
   lag range, unlike the other regressors (four lags each); we follow the specification as
   written.
5. **Newey–West bandwidth = h+1** for the LP HAC standard errors (permitted by the task).
6. **2SLS implemented as a covariance ratio.** With one instrument and one endogenous
   regressor, `delta_j = cov(z, u^j)/cov(z, u^ffr)`; this equals IV2SLS (verified in
   `part_B.ipynb`) and is used inside the bootstrap for speed.
7. **Bootstrap design.** Residual bootstrap, 1000 replications, `seed=42` for
   reproducibility. Residuals and the instrument are resampled with the same index draws so
   each residual stays paired with its instrument value, preserving the `cov(z, u^ffr)`
   correlation on which identification depends.
8. **shocks.csv format.** The file ships with `year, quarter, mp_shock` columns (not the
   `YYYYQN` string the task sheet describes); a quarterly index is constructed from the
   `year` and `quarter` columns.

## Key results

- **First-stage F = 13.06** (> 10): the narrative shock is a strong instrument.
- **Real GDP:** both methods give a hump-shaped decline troughing at roughly −0.75% to
  −0.8% about 9–12 quarters after the shock — consistent across methods and with theory.
- **CPI:** a short-run price puzzle (prices rise initially). Under Local Projections it
  later reverses to about −1.1%; under the SVAR the puzzle persists through the horizon.
```

