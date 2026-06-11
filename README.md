# Fiscal Policy and Employment in Mexico's Northern Border: A Synthetic Control Analysis

Python replication of my master's thesis: **"Impacts of Fiscal Policy Cuts and Minimum Wage Increases in the North Frontier of Mexico: a case study of the IMMEX sector in the North Frontier Economic Free Zone"** (2022). The original analysis was implemented in Stata using the `allsynth` package; this repository translates the full empirical pipeline into Python (`pandas`, `numpy`, `scipy`, `matplotlib`).

## The policy question

In January 2019, the Mexican federal government created the **North Frontier Economic Free Zone (NFZ)** — a policy package applied to 43 municipalities within 25 km of the US border:

- **VAT reduction** from 16% to 8%
- **Income tax (ISR) reduction** from 30% to 20%
- **Minimum wage increase of 100%** (from 88.36 to 176.72 MXN/day)

**Research question:** What was the effect of this policy package on employment in the IMMEX sector (Industria Manufacturera, Maquiladora y de Servicios de Exportación) in the treated municipalities?

## Main findings

| Municipality | Effect | RMSPE Ratio |
|---|---|---|
| Acuña | **Negative** — steep employment decline, not recovered by 2022 | 5.10 |
| Reynosa | Mild positive | 4.17 |
| Tijuana | Mild positive | 2.16 |
| Tecate, Mexicali, Juárez, Nogales, Matamoros, Nuevo Laredo, Ensenada | Null / minimal | 1.25–2.05 |

The policy had **null or minimal effects on IMMEX employment in 8 of 10 municipalities**. The government's minimal goal — not harming employment while raising wages and cutting taxes — was largely achieved, with Acuña the notable exception. Post-2020 gaps partly reflect the COVID-19 shock and are interpreted alongside the gap figures.

## Methodology

**Synthetic Control Method** (Abadie, Diamond & Hainmueller, 2010). For each treated municipality, a synthetic counterfactual is constructed as a weighted average of donor-pool (untreated) municipalities that best matches the treated unit's pre-treatment characteristics. Weights are non-negative and sum to one (no extrapolation). The post-treatment gap between the actual unit and its synthetic counterpart is the estimated treatment effect.

Implementation details:

- **Predictor matrix** — point values of the outcome at evenly spaced pre-treatment dates plus standardized pre-period averages of ten economic covariates (work hours, establishments, national/international income, payments, etc.), mirroring the Stata `allsynth` specification.
- **Nested optimization** — an inner SLSQP solve for donor weights W given predictor weights V, and an outer Nelder-Mead search (with 5 random restarts) for the V that minimizes pre-treatment outcome fit.
- **Inference** — post/pre RMSPE ratio: the size of the post-treatment divergence relative to pre-treatment fitting noise.
- **Outcome models** — the primary outcome is `sop` (IMMEX employment as a share of the economically active population). Acuña uses log-employment, as its SOP trajectory cannot be matched by any convex combination of donors; this per-unit model selection mirrors the thesis.

## Data

- **Source:** INEGI (Banco de Información Económica), ENOE (Encuesta Nacional de Ocupación y Empleo), and IMMEX program data.
- **Panel:** 41 municipalities × monthly observations, July 2007 – January 2022.
- **Treated units (10):** Acuña, Ensenada, Ciudad Juárez, Matamoros, Mexicali, Nogales, Nuevo Laredo, Reynosa, Tecate, Tijuana.
- **Donor pool (31):** IMMEX-relevant municipalities across the rest of Mexico.
- **Treatment date:** January 2019.

## Repository structure

```
nfz_scm_app/
├── data/
│   ├── raw/                  # original panel (Excel)
│   └── processed/            # clean parquet + results CSV
├── notebooks/
│   ├── 01_data_inspection.ipynb   # load, clean, derive variables, save panel
│   └── 02_scm_exploration.ipynb   # SCM implementation, all 10 municipalities, results
└── README.md
```

## How to run

```bash
conda create -n nfz_scm python=3.11 pandas numpy scipy matplotlib pyarrow openpyxl jupyter
conda activate nfz_scm
```

Run the notebooks in order. Notebook 01 produces `data/processed/nfz_panel_clean.parquet`; notebook 02 runs the full SCM analysis (the 10-municipality loop takes roughly 30–40 minutes due to the nested optimization with restarts) and saves `scm_results_summary.csv`.

## References

- Abadie, A., Diamond, A., & Hainmueller, J. (2010). Synthetic Control Methods for Comparative Case Studies. *JASA*.
- Abadie, A. (2021). Using Synthetic Controls: Feasibility, Data Requirements, and Methodological Aspects. *Journal of Economic Literature*.
- Wiltshire, J. (2021). allsynth: Synthetic control bias-correction utilities for Stata. (Methodology of the original thesis's stacked estimator; not replicated here.)

## Possible extensions

Placebo (permutation) inference across the donor pool, the Wiltshire (2021) stacked bias-corrected estimator, and a Streamlit app for interactive exploration of the gap figures.

---

**Author:** Hector Alejandro Vazquez Reyes
