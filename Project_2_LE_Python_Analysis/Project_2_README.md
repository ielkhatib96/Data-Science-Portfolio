# Life Expectancy Analysis — Project 2 (Python / JupyterLab)

> Statistical analysis and regression modelling of WHO life expectancy data (2000–2015, 183 countries).
> Replication and extension of [Project 1 (Power BI)](../Project_1_PowerBI_analysis/) using formal statistical techniques not available in Power BI.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ielkhatib96/Data-Science-Portfolio/blob/main/Project_2_LE_Python_Analysis/LE_Data_Analysis_Project_2.ipynb)

---

## 1. Project Overview

This project takes the same WHO Life Expectancy dataset from Project 1 and applies formal statistical techniques — partial correlations, OLS regression, VIF multicollinearity testing, interaction terms, residual analysis, ARIMA forecasting, and population-weighted regional averages — to answer the same research questions with greater rigour and to answer new questions that Power BI could not address.

### Research Questions

| # | Question |
|---|----------|
| Q1 | Is there a disease where high death rate particularly lowers LE more than others? |
| Q2 | Is the impact of immunization coverage on LE greater in developing nations than developed ones? |
| Q3 | Do countries with higher ICOR have higher LE? (resource curse investigation) |
| Q4 | What are the predicting variables actually affecting LE? |
| Q5 | Should a country with low LE (<65) increase healthcare expenditure to improve lifespan? |
| Q6 | How do infant and adult mortality rates affect LE? |
| Q7 | Does LE correlate with eating habits, lifestyle, and alcohol? |
| Q8 | What is the impact of schooling on LE? |

### New Questions (enabled by Python)

- What are the strongest predictors of life expectancy across countries?
- Does immunization coverage have an independent effect on LE after controlling for GDP and region?
- Does the GDP–LE relationship differ between developed and developing countries?
- Which countries overachieve or underperform relative to their predicted LE?
- Can composite health and lifestyle scores be constructed that meaningfully predict LE?
- Is the GDP–LE relationship logarithmic (Preston Curve)?

---

## 2. Tools and Libraries

| Tool / Library | Purpose |
|----------------|---------|
| pandas | Data manipulation, cleaning, merging, grouping |
| numpy | Numerical operations |
| matplotlib / seaborn | Static visualisation |
| scipy.stats | Pearson correlation with p-values |
| statsmodels | OLS regression, VIF, LOWESS smoothing, ARIMA |
| sklearn | MinMaxScaler for normalisation, LinearRegression for partial correlations |
| plotly | Interactive choropleth maps |
| itables | Interactive scrollable data tables within JupyterLab |

---

## 3. Data Sources

- **WHO Life Expectancy** — [Kaggle](https://www.kaggle.com/datasets/kumarajarshi/life-expectancy-who) (193 countries, 2000–2015, 22 columns)
- **World Bank GDP per capita** — [data.worldbank.org](https://data.worldbank.org/indicator/NY.GDP.PCAP.CD) (replaces unreliable WHO GDP column)
- **World Bank Population** — [data.worldbank.org](https://data.worldbank.org/indicator/SP.POP.TOTL) (replaces unreliable WHO Population column)
- **WHO Region lookup** — manually compiled mapping of 183 countries to 6 WHO regions

27 country name mismatches reconciled between WHO and World Bank naming conventions.

---

## 4. Data Cleaning

All cleaning steps documented with rationale. Column name stripping must precede all other operations. Data sorted by Country + Year before any fill operations.

| Step | Action | Rationale |
|------|--------|-----------|
| Row removal | South Sudan removed | Founded 2011; pre-2011 data invalid |
| Row removal | 10 countries removed | Single 2013 row with null LE — no usable data |
| Column names | Stripped whitespace | Prevents KeyError failures on column references |
| Encoding | Côte d'Ivoire mojibake fixed | Corrected in both WHO and World Bank files before merge |
| Zero → null | ICOR zeros → NaN | Non-recording, not genuine zero human development |
| Zero → null | Schooling zeros → NaN | No country has zero years of schooling |
| Anomaly correction | Hep B, Polio, Diphtheria single-digit values replaced | Values below 10 mid-series are data entry errors; forward-filled within country |
| Null filling | Alcohol: forward-fill | 2015 missing across all countries; filled from 2014 |
| Null filling | BMI, thinness: backward-fill | Sparse early-year gaps |
| External data | WHO GDP/Population replaced with World Bank | WHO columns had order-of-magnitude jumps between years |
| GDP backfill | Year 2000 backfilled from 2001 | World Bank extract started at 2001 |
| Status correction | 14 countries reclassified Developing → Developed | Based on World Bank income classifications |

---

## 5. Feature Engineering

- **WHO Region** — mapped all 183 countries to one of 6 WHO regions
- **LE Band** — Low (<60), Medium (60–74), High (≥75)
- **GDP Band** — Low (<$1k), Middle ($1k–$10k), High (≥$10k)
- **Immunization Composite** — average of Polio, Diphtheria, Hepatitis B (r = 0.58 with LE)
- **Disease Burden Score** — all components min-max normalised before combining; correlation improved from −0.39 (Project 1, unnormalised) to −0.58 (+49%)
- **Lifestyle Score** — normalised and weighted (Schooling ×2, Alcohol ×0.5, BMI deviation ×0.5); corrected from −0.59 (misleading) to +0.61

---

## 6. Statistical Techniques

### 6.1 Pearson Correlation
Measures linear relationship strength (−1 to +1). With 2,900 data points, effect size (the r value) matters more than p-value significance. Used as a first scan to identify where to investigate deeper.

### 6.2 Partial Correlation
Correlates two variables after removing the influence of confounders (GDP per capita + WHO Region). Key finding: Alcohol's relationship with LE dropped 93.8% — almost entirely confounding. Immunization dropped only 22% and retained significance — confirming a genuine independent effect.

### 6.3 OLS Regression
Predicts LE from all variables simultaneously. Full model R² = 0.886 (88.6% of variance explained). ICOR had the largest coefficient (39.45 in reduced model). HIV/AIDS coefficient of −0.39 means each unit increase costs ~0.4 years of LE.

### 6.4 VIF (Multicollinearity)
Detected severe collinearity: Polio/Diphtheria VIF >500, ICOR/Schooling VIF >120. A reduced model using Immunization Composite instead of three separate columns maintained R² = 0.887 while producing interpretable coefficients.

### 6.5 Interaction Terms
Tested whether GDP's effect on LE differs by development status. Result: ~10× stronger for developing countries. Formally confirms the Preston Curve diminishing returns pattern.

### 6.6 Residual Analysis
Identifies countries performing better or worse than the model predicts. Top overachievers: Vanuatu (+5.7 yrs), Vietnam (+5.0), Nicaragua (+4.9). Top underperformers: Sierra Leone (−8.7), Angola (−8.1), Turkmenistan (−6.3).

### 6.7 Preston Curve
Log(GDP) explains 62% of LE variance vs 34% for linear GDP — nearly double. Confirms the foundational health economics finding of diminishing returns to wealth.

### 6.8 LOWESS Smoothing
Reveals actual relationship shapes without forcing linearity. Key insight: GDP vs LE shows clear diminishing returns; ICOR vs LE is near-linear throughout its range.

### 6.9 ARIMA Forecasting
ARIMA(1,1,1) fitted to global population-weighted LE trend. 2025 forecast: 73.88 years (95% CI: 72.00–75.76). Note: COVID-19 setback (2020–2022) not captured — actual 2025 figure likely at lower bound.

### 6.10 Population-Weighted Regional Averages
Corrects Project 1's simple averages. South-East Asia weighted average is 2.19 years lower than simple average — India pulls the true regional picture down significantly.

---

## 7. Key Findings

| Question | Finding |
|----------|---------|
| Q1 — Disease impact | HIV/AIDS dominates (r = −0.56, coeff = −0.39). Each unit increase costs ~0.4 years of LE. |
| Q2 — Immunization by status | 0.296 years per 1% coverage increase in developing nations. Effect ~2× stronger than in developed nations. |
| Q3 — Resource curse | ICOR (r = 0.89) explains 79% of LE variance vs GDP's 34%. Equatorial Guinea confirmed as resource curse case. |
| Q4 — Variable importance | ICOR > Schooling > Adult Mortality > Immunization. Model explains 88.6% of LE variance. |
| Q5 — Expenditure for low-LE | **Contradicts Project 1.** OLS coefficient for low-LE countries is −0.38 (controlling for GDP). Spending is reactive, not preventive. |
| Q6 — Mortality rates | Adult Mortality strongest (r = −0.69). Infant/under-5 deaths weaker (−0.20) — raw counts biased by country size. |
| Q7 — Lifestyle | Alcohol and BMI show positive raw correlations — wealth confounding. After partial correlation, both drop sharply. Lifestyle Score: r = +0.61. |
| Q8 — Schooling | r = 0.78, partial r = 0.54 (31% drop). Simple OLS: ~1.1 years LE per year of schooling. |

---

## 8. Project 1 vs Project 2 — Key Differences

| Topic | Project 1 (Power BI) | Project 2 (Python) |
|-------|---------------------|-------------------|
| Top predictor | Key Influencers: Schooling #1 | OLS: ICOR #1 (r = 0.89) — Schooling absorbed by collinearity |
| Confounding | Cannot separate | Partial correlations: Alcohol 94% confounding, Immunization only 22% |
| Disease Burden Score | r = −0.39 (unnormalised) | r = −0.58 after min-max normalisation (+49%) |
| Lifestyle Score | r = −0.59 (misleading — BMI dominated) | r = +0.61 after weighting + normalisation |
| Health expenditure | Estimated +0.8 years per 1% | Formal OLS: −0.38 for low-LE countries — finding reversed |
| Hep B anomalies | Documented, unfixable in Power Query | Fixed in 2 lines per column using Python |
| Regression | None | Full OLS + VIF + reduced model |
| GDP–LE relationship | Visual observation only | Preston Curve: log(GDP) explains 28% more variance than linear |
| Regional averages | Simple (unweighted) | Population-weighted — South-East Asia 2.19 yrs lower than simple |

---

## 9. Time Series

### ARIMA Forecast
- 2015 actual: 71.97 years
- 2025 forecast: 73.88 years
- 95% CI: 72.00–75.76 years
- Assumes continuation of 2000–2015 trend (0.275 years/year)

### Changepoint Detection
Only 2002 showed statistically significant below-average growth (0.11 years vs average 0.275). The "plateaus" observed visually in Project 1 (2004–2005, 2009) are largely noise — the trend is remarkably steady.

### Population-Weighted Regional Differences

| Region | Weighted − Unweighted |
|--------|-----------------------|
| South-East Asia | −2.19 yrs (India pulls down) |
| Eastern Mediterranean | −1.83 yrs (Pakistan, Egypt pull down) |
| African | −1.46 yrs (Nigeria, Ethiopia, DRC pull down) |
| European | −0.35 yrs |
| Americas | +2.11 yrs (US, Brazil push up) |
| Western Pacific | +2.44 yrs (China pushes up) |

---

## 10. Techniques Reference (for self-study)

| # | Technique | YouTube search |
|---|-----------|----------------|
| 1 | Pearson Correlation | "Pearson correlation coefficient explained" |
| 2 | P-Value | "p-value explained simply" |
| 3 | Partial Correlation | "partial correlation vs correlation statistics" |
| 4 | OLS Regression | "ordinary least squares regression explained" |
| 5 | R² and Adjusted R² | "R squared vs adjusted R squared regression" |
| 6 | Confidence Intervals | "confidence intervals explained statistics" |
| 7 | VIF / Multicollinearity | "multicollinearity VIF regression" |
| 8 | Interaction Terms | "interaction terms regression interpretation" |
| 9 | Residual Analysis | "regression residuals analysis interpretation" |
| 10 | LOWESS Smoothing | "LOWESS smoothing scatter plot explained" |
| 11 | Min-Max Normalisation | "min-max normalisation feature scaling machine learning" |
| 12 | Preston Curve | "Preston curve GDP life expectancy health economics" |
| 13 | Z-Score Standardisation | "z-score standardisation explained" |
| 14 | ARIMA Forecasting | "ARIMA model time series forecasting explained" |
| 15 | Changepoint Detection | "changepoint detection time series" |
| 16 | Population-Weighted Averages | "weighted average vs simple average statistics" |

---

## 11. How to Run

**Option 1 — Google Colab (no installation required)**

Click the badge at the top. The notebook automatically downloads all required data files from this repo.

**Option 2 — Local JupyterLab**

```bash
git clone https://github.com/ielkhatib96/Data-Science-Portfolio.git
cd Data-Science-Portfolio/Project_2_LE_Python_Analysis
jupyter lab
```

Open `LE_Data_Analysis_Project_2.ipynb` and run all cells (Kernel → Restart & Run All).

**Required libraries** (all included in Anaconda):
`pandas` · `numpy` · `matplotlib` · `seaborn` · `scipy` · `statsmodels` · `sklearn` · `plotly` · `itables`

---

## 12. Next Steps

- Summary dashboard notebook (code-hidden visual report for non-technical audience)
- LLM API integration for natural language forecast interpretation (Project 3)

---

*Full methodology and detailed findings: [`Project_2_README_v3.docx`](./Project_2_README_v3.docx)*
