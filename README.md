# Data Science Portfolio — Ibrahim Elkhatib

A structured portfolio demonstrating end-to-end data science skills across three progressive projects. Each project builds on the last — from certified BI tool competency through to Python statistical modelling and AI integration.

---

## Projects

### Project 1 — Life Expectancy Analysis (Power BI)
**Tools:** Power BI Desktop · Power Query · DAX · Star Schema

A full end-to-end Business Intelligence project analysing WHO life expectancy data across 183 countries (2000–2015). Covers data ingestion, cleaning in Power Query, star schema data modelling, DAX measure development, and interactive dashboard design.

**Key deliverables:**
- Data cleaning pipeline (null strategy, duplicate removal, World Bank data integration)
- Star schema: `Fact_Life_Expectancy`, `Dim_Countries`, `Dim_Year`
- 14 DAX measures including CALCULATE/ALL, AVERAGEX, SUMX, VAR patterns
- Composite scores: Disease Burden Index, Lifestyle Score, Immunization Composite
- What-if parameters for health expenditure and immunization scenarios
- Interactive dashboard with drill-down hierarchies and slicers

📁 [`Project_1_PowerBI_analysis/`](./Project_1_PowerBI_analysis/)

---

### Project 2 — Life Expectancy Analysis (Python)
**Tools:** Python · pandas · statsmodels · sklearn · scipy · matplotlib · seaborn · plotly

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/ielkhatib96/Data-Science-Portfolio/blob/main/Project_2_LE_Python_Analysis/LE_Data_Analysis_Project_2.ipynb)

Replication and extension of Project 1 using Python and formal statistical modelling — applying techniques not possible in Power BI. The same dataset is taken further with partial correlations, OLS regression, VIF testing, interaction terms, residual analysis, ARIMA forecasting, and population-weighted regional averages.

**Key deliverables:**
- Full data cleaning pipeline in pandas (replicating and extending Power Query decisions)
- Anomaly detection: threshold-based and neighbour-comparison within country time series
- Partial correlations controlling for GDP and WHO Region — separating genuine effects from confounding
- OLS regression (R² = 0.886) with VIF multicollinearity testing and reduced model
- Interaction term: GDP × Development Status (formally confirming Preston Curve)
- Residual analysis identifying overachievers (Vanuatu +5.7 yrs) and underperformers (Sierra Leone −8.7 yrs)
- ARIMA forecast: global LE projected at 73.88 years by 2025 (95% CI: 72.00–75.76)
- Population-weighted regional averages (correcting Project 1's simple averages)
- Interactive choropleth map (plotly)
- Min-max normalised composite scores — improving Disease Burden correlation from −0.39 to −0.58

**Research questions answered:**
- What are the strongest predictors of life expectancy? *(ICOR r=0.89, beats GDP r=0.58)*
- Does immunization have an independent effect after controlling for wealth? *(Yes — partial r=0.45, p≈0)*
- Does GDP's effect on LE differ by development status? *(~10× stronger in developing nations)*
- Which countries over/underperform relative to their predicted LE?
- Is the GDP–LE relationship logarithmic? *(Log GDP explains 62% vs 34% for linear)*

📁 [`Project_2_LE_Python_Analysis/`](./Project_2_LE_Python_Analysis/)
📄 [`Project_2_README_v3.docx`](./Project_2_LE_Python_Analysis/Project_2_README_v3.docx) — full methodology and findings

---

### Project 3 — LLM API Integration *(coming soon)*
**Tools:** Python · Anthropic API · ARIMA

Adding a natural language interpretation layer to the Project 2 ARIMA forecast — allowing users to query forecast outputs in plain English via an LLM API call.

---

## Dataset

**WHO Life Expectancy** — [Kaggle](https://www.kaggle.com/datasets/kumarajarshi/life-expectancy-who)
183 countries · 2000–2015 · 22 variables

Supplemented with World Bank World Development Indicators (GDP per capita, Population) to replace unreliable WHO source columns.

---

## Tech Stack

| Category | Tools |
|----------|-------|
| BI & Dashboarding | Power BI Desktop, DAX, Power Query |
| Data Manipulation | pandas, numpy |
| Statistical Modelling | statsmodels (OLS, ARIMA, VIF), scipy.stats |
| Machine Learning | sklearn (MinMaxScaler, LinearRegression) |
| Visualisation | matplotlib, seaborn, plotly |
| Version Control | Git, GitHub, GitHub Desktop |
| Environments | JupyterLab, Google Colab, Anaconda |

---

## How to Run Project 2

**Option 1 — Google Colab (no installation required)**
Click the "Open in Colab" badge above. The notebook automatically downloads all required data files from this repo on first run.

**Option 2 — Local JupyterLab**
```bash
git clone https://github.com/ielkhatib96/Data-Science-Portfolio.git
cd Data-Science-Portfolio/Project_2_LE_Python_Analysis
jupyter lab
```
Then open `LE_Data_Analysis_Project_2.ipynb` and run all cells.

**Required libraries** (all included in Anaconda):
`pandas` · `numpy` · `matplotlib` · `seaborn` · `scipy` · `statsmodels` · `sklearn` · `plotly` · `itables`

---

*Portfolio in active development — Project 3 in planning.*
