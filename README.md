# The Missing Middle: A Statistical Investigation of the Exoplanet Radius Gap

**Jovana Mickovikj**

A statistical study of the *radius gap* — the observed shortage of planets with radii
between roughly 1.5 and 2.0 Earth radii, first resolved by Fulton et al. (2017). Using a
quality-filtered sample of 2,587 planets from the NASA Exoplanet Archive, the project
addresses four questions:

1. Does the radius distribution show statistical evidence of two populations, and where
   does the boundary between them sit?
2. Do formal tests of multimodality support that structure?
3. Setting radius itself aside to avoid circular reasoning, can orbital and stellar
   properties predict which side of the gap a planet falls on?
4. Is the period-dependence predicted by atmospheric mass-loss theory borne out, both
   classically and under Bayesian estimation?

## Results

A two-component Gaussian mixture is strongly preferred over one (ΔAIC ≈ −242,
ΔBIC ≈ −224), and a kernel density estimate locates a density minimum at **1.76 R⊕**,
stable across smoothing bandwidths and across four independent filtering variants. The
minimum is shallow — about 2% below the adjacent peak. Of the two formal modality tests,
Silverman's critical bandwidth test rejects unimodality (p = 0.028) while Hartigan's dip
test does not (p = 0.52); a power simulation at this sample size shows the dip test has
essentially no sensitivity to a valley this shallow.

Classification on orbital and stellar parameters alone — with every radius-derived
quantity excluded — reaches ROC-AUC 0.77–0.79, with the period–insolation axis carrying
most of the predictive signal. The above-gap fraction is 0.455 for short-period planets
against 0.790 for long-period ones (Δ = 0.335, 95% credible interval [0.299, 0.369]),
a result stable across priors, period thresholds, and a cluster bootstrap over host
systems.

This period-dependence is confounded with transit detection bias, which predicts the same
signature and which a composite archive sample cannot correct for. The report treats this
as its principal limitation.

## Repository layout

```
.
├── README.md
├── requirements.txt
│
├── code/
│   ├── 01_data_preparation.ipynb    exploration, missingness, filtering cascade (6319 → 2587)
│   ├── 02_gap_detection.ipynb       GMM comparison, KDE gap location, bootstrap, modality tests
│   ├── 03_classification.ipynb      logistic regression, random forest, feature importance
│   ├── 04_hypothesis_testing.ipynb  KS, Mann-Whitney, Kruskal-Wallis, chi-square, Fisher
│   ├── 05_bayesian_analysis.ipynb   Beta-Binomial posterior for the above-gap proportion
│   └── 06_robustness_checks.ipynb   test power, valley depth, detection bias, clustering
│
├── data/
│   ├── raw_nasa_archive_2026-07-15.csv       raw download, 6,319 planets
│   ├── radius_provenance_lookup.csv          measured vs derived radius, per planet
│   ├── filtered_main_sample.csv              main analysis sample, 2,587 planets
│
├── report/
│   └── The_Missing_Middle.pdf
│
└── presentation/
    └── The_Missing_Middle.pptx
```

Notebooks are numbered in execution order. Each one after 01 reads
`data/filtered_main_sample.csv` and is otherwise self-contained. They locate `data/` by
walking up from the working directory, so they run whether Jupyter is started from the
project root or from `code/`.

## Data

All data comes from the **NASA Exoplanet Archive**, Planetary Systems Composite
Parameters (PSCompPars) table, downloaded **15 July 2026**.

| File | Rows | Description |
|---|---|---|
| `raw_nasa_archive_2026-07-15.csv` | 6,319 | Raw download. One row per confirmed planet, 84 columns of orbital, physical and stellar parameters with uncertainties. Unmodified. |
| `radius_provenance_lookup.csv` | 6,319 | Per-planet record of whether the archive's radius was measured or derived from the Chen & Kipping (2017) mass–radius relation. Used by filter step 3. |
| `filtered_main_sample.csv` | 2,587 | Main analysis sample, produced by notebook 01. |

PSCompPars is a *composite* catalogue: different columns for the same planet may come
from different literature sources and are not guaranteed to be mutually consistent. This
is noted as a limitation in the report.

### Filtering cascade

Every cleaning and transformation step is executed and printed in notebook 01, so the
sample size can be traced at each stage:

| Step | Filter | N remaining | N removed |
|---|---|---|---|
| 0 | Start (raw PSCompPars) | 6319 | – |
| 1 | Has `pl_rade` and `pl_orbper` | 5929 | 390 |
| 2 | Not controversial (`pl_controv_flag = 0`) | 5879 | 50 |
| 3 | Radius measured, not derived | 4713 | 1166 |
| 4 | Radius not a limit value | 4710 | 3 |
| 5 | Period not a limit value | 4710 | 0 |
| 6 | Radius in 0.5–4 R⊕ | 3512 | 1198 |
| 7 | Relative radius error < 20% | **2587** | 925 |

Step 3 matters most. The archive derives some radii from a mass–radius relation rather
than measuring them, and a deterministic relation cannot reproduce fine distributional
structure — including those radii would smooth away the feature under investigation.

## Running

```bash
pip install -r requirements.txt
jupyter lab
```

Run the notebooks in numerical order (`Kernel → Restart & Run All` in each).

Notebook 01 reads `data/radius_provenance_lookup.csv` if it is present, and otherwise
downloads it from the NASA Exoplanet Archive TAP service and caches it there. Since the
cache is included in this repository, all six notebooks run offline and reproduce the
figures reported in the paper exactly. Random seeds are fixed throughout — bootstraps,
train/test splits and model fitting.

Re-running notebook 01 regenerates `filtered_main_sample.csv` and
`filtered_transit_only_sample.csv` with identical contents.

## Requirements

See `requirements.txt` — pandas, numpy, scipy, scikit-learn, matplotlib, seaborn,
statsmodels, and `diptest` for Hartigan's dip test.

## References

- Fulton et al. (2017), *The California-Kepler Survey. III. A Gap in the Radius
  Distribution of Small Planets*, AJ 154, 109. doi:10.3847/1538-3881/aa80eb
- Weiss et al. (2018), *The California-Kepler Survey. V. Peas in a Pod*, AJ 155, 48.
  doi:10.3847/1538-3881/aa9ff6
- Chen & Kipping (2017), *Probabilistic Forecasting of the Masses and Radii of Other
  Worlds*, ApJ 834, 17. doi:10.3847/1538-4357/834/1/17
