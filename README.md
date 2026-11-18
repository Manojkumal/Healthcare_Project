# Healthcare Expenditure and Life Expectancy Across Countries: An Extended Panel Data Analysis

This project asks how healthcare spending relates to life expectancy across countries, using World Bank data for 186 countries from 2000 to 2023. It tests whether the relationship has changed over time, whether it holds after accounting for GDP per capita and out-of-pocket financing, and whether it is nonlinear. France and the United States are highlighted as examples. All results are statistical associations, not causal effects.

**Main deliverable:** [`Healthcare_Spending_Life_Expectancy.ipynb`](Healthcare_Spending_Life_Expectancy.ipynb). All cells are executed and saved with their outputs, so the notebook can be read on GitHub without running it.

## Repository structure

```
.
├── Healthcare_Spending_Life_Expectancy.ipynb   # Main notebook: data collection → preprocessing → EDA → models → inference
├── README.md
├── requirements.txt                            # Python packages
├── dashboard_template.html                     # Page layout for the dashboard (the notebook fills in the data)
├── FPb project.pdf                             # Approved project proposal
├── assets/
│   ├── plotly-2.35.2.min.js                    # Charting library, built into the dashboard so it works offline
│   └── world_110m.json                         # World map shapes for the dashboard map
├── data/
│   ├── raw/                                    # Cached API responses (one JSON per indicator)
│   │   ├── <INDICATOR_CODE>.json, countries.json, WHO_UHC_INDEX_REPORTED.json
│   │   ├── wdi_long.csv                        # The 10 main indicators in long format (iso3, year, indicator, value)
│   │   └── country_meta.csv                    # Country name, region, income group
│   └── processed/
│       ├── panel_full.csv                      # All 217 economies × 2000–2023, with quality flags
│       └── panel_analytic.csv                  # Analytic sample: 186 countries, 4,404 country-years
├── outputs/
│   ├── figures/                                # All charts (PNG), including static choropleth maps
│   ├── maps/                                   # Interactive choropleth maps (HTML)
│   ├── tables/                                 # Every result table (CSV): preprocessing, EDA, models, robustness, inference
│   └── dashboard.html                          # Interactive dashboard (open in a browser)
└── report/
    └── final_report.md                         # Written report of results
```

## Notebook contents

| Section | Content |
|---|---|
| Overview | Research question, motivation, approach, notebook map |
| 1. Setup | Libraries, paths, indicator definitions, plotting style |
| 2. Data collection | World Bank API download of 10 indicators and country metadata, with caching |
| 3. Preprocessing | Removing aggregates, uniqueness check, country-year panel, study window, validity checks (failed records excluded), missing values, outlier flags, feature engineering, analytic sample |
| 4. EDA | Descriptive statistics, distributions, trends, scatterplots, boxplots, correlations, choropleth maps, France vs US with context indicators |
| 5. Panel regression | Pooled OLS → country fixed effects → two-way fixed effects → + GDP → + out-of-pocket share → nonlinear, with a flexible check; Hausman test |
| 6. Robustness | Alternative spending measures (incl. constant prices), balanced panel, COVID and outlier exclusions, lags, long differences, between estimator, income groups, regions, periods, why the relationship weakened, other outcomes |
| 7. Inference | Permutation tests and bootstrap CIs (high vs low out-of-pocket burden), sensitivity to single countries, country-cluster bootstrap |
| 8. Country performance | Over- and under-performers relative to spending; how model-dependent the US gap is |
| 9. Dashboard | Generates the interactive HTML dashboard |
| 10. Summary and conclusions | Supported findings, model-dependent results, hypotheses, policy implications, future research, limitations |

## Data

The data come from the [World Bank World Development Indicators](https://databank.worldbank.org/source/world-development-indicators) API. The health-expenditure series originate from the WHO Global Health Expenditure Database.

| Indicator | Code |
|---|---|
| Health expenditure per capita, PPP (main spending measure) | SH.XPD.CHEX.PP.CD |
| Health expenditure, % of GDP | SH.XPD.CHEX.GD.ZS |
| Out-of-pocket expenditure, % of health expenditure | SH.XPD.OOPC.CH.ZS |
| Life expectancy at birth (outcome) | SP.DYN.LE00.IN |
| GDP per capita, PPP, constant 2021 $ | NY.GDP.PCAP.PP.KD |
| Secondary indicators (US$ spending, government/private shares, under-5 and maternal mortality) | SH.XPD.CHEX.PC.CD, SH.XPD.GHED.CH.ZS, SH.XPD.PVTD.CH.ZS, SH.DYN.MORT, SH.STA.MMRT |
| Context for the France–US comparison only (not used in any model) | SI.POV.GINI, SH.STA.OWAD.ZS, SH.PRV.SMOK, SP.POP.65UP.TO.ZS, and the WHO UHC service coverage index (UHC_INDEX_REPORTED, WHO Global Health Observatory) |

The raw data are already included in `data/raw/`, so the notebook runs **without internet access**. To download fresh data, set `REFRESH = True` in the notebook's setup cell.

## How to run the notebook

Requires Python 3.10 or later.

```bash
# 1. Clone the repository and enter it
git clone <repository-url>
cd <repository-folder>

# 2. Create a virtual environment and install packages
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# 3. Open the notebook
jupyter notebook Healthcare_Spending_Life_Expectancy.ipynb
```

Then choose **Kernel → Restart & Run All**. The full notebook runs in about 2–3 minutes. Alternatively, execute it from the command line:

```bash
jupyter nbconvert --to notebook --execute --inplace Healthcare_Spending_Life_Expectancy.ipynb
```

**Viewing the dashboard:** open `outputs/dashboard.html` in any web browser. The charting library and map shapes are built into the file, so it works **offline**. The separate interactive maps in `outputs/maps/` have the charting library built in but download the country shapes from the internet; static PNG versions are in `outputs/figures/`.

## Key findings

- **Across countries:** 10% more health spending goes with about 0.5 more years of life expectancy. This compares countries; it is not the effect of one country raising its spending.
- **Within countries over time:** same-year spending is not associated with life expectancy once global trends and GDP are included. Returns diminish strongly: within-country gains appear only at low spending levels.
- **Delayed effects:** spending lagged five years is positively associated with life expectancy (weaker once GDP is controlled for).
- **Over time:** the association weakened. Much of this is diminishing returns as spending rose; the rest is concentrated in 2020–2023.
- **Out-of-pocket burden:** high-burden countries live about 4 years less, but this gap is explained by lower national income.
- **France and the US:** the US spends twice as much per person as France but lives 4.4 years less, about 3–7 years below what its spending predicts depending on the comparison.
