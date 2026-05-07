# Life Expectancy Analysis — Project 1 (Power BI)

> End-to-end Business Intelligence project analysing WHO life expectancy data across 183 countries (2000–2015).
> Continued and extended in [Project 2 (Python)](../Project_2_LE_Python_Analysis/) with formal statistical modelling.

---

## 1. Project Overview

This project demonstrates a complete Power BI workflow starting from a raw, uncleaned dataset with significant quality issues. The project progresses through data cleaning in Power Query, external data integration, star schema data modelling, DAX measure development, exploratory analysis, and dashboard design — producing a fully analytical report answering eight research questions.

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

---

## 2. Tools

| Tool | Purpose |
|------|---------|
| Power BI Desktop | Main development environment |
| Power Query (M) | Data cleaning and transformation |
| DAX | Measures, calculated columns, what-if parameters |
| Power BI visuals | Charts, Key Influencers, Decomposition Tree, Maps |

---

## 3. Data Sources

- **WHO Life Expectancy** — [Kaggle](https://www.kaggle.com/datasets/kumarajarshi/life-expectancy-who) (193 countries, 2000–2015, 22 columns)
- **World Bank GDP per capita** — [data.worldbank.org](https://data.worldbank.org/indicator/NY.GDP.PCAP.CD) (replaced unreliable WHO GDP column)
- **World Bank Population** — [data.worldbank.org](https://data.worldbank.org/indicator/SP.POP.TOTL) (replaced unreliable WHO Population column)
- **WHO Region lookup** — manually compiled mapping of 193 countries to 6 WHO regions

---

## 4. Data Cleaning (Power Query)

All cleaning decisions documented in a cleaning log with rationale.

| Decision | Detail |
|----------|--------|
| Removed null LE rows | Target variable; rows without it have no analytical value |
| Removed South Sudan | Founded 2011; pre-2011 data historically invalid; post-2011 mostly missing |
| Removed 10 micro-nations | Cook Islands, Dominica, Marshall Islands, Monaco, Nauru, Niue, Palau, San Marino, Saint Kitts and Nevis, Tuvalu — single 2013 row, null LE |
| Alcohol 2015 fill | Forward-filled from 2014; 2015 missing across all countries |
| Status reclassification | 14 countries corrected Developing → Developed (Canada, Finland, France, Greece, Estonia, Israel, Republic of Korea, Qatar, UAE, Kuwait, Bahrain, Saudi Arabia, Russian Federation, Oman) |
| ICOR zeros → null | Non-recording, not genuine zero human development |
| Schooling zeros → null | No country has zero years of schooling |
| Polio / Diphtheria fill | Montenegro (2000–2005) and Timor-Leste (2000–2001) filled up from nearest later year |
| Hep B anomalies documented | Single-digit values (7, 8, 9) mid-series identified as likely data entry errors — flagged, left in place due to Power Query limitations. **Corrected in Project 2.** |
| BMI — Sudan | Missing entirely across all years; left as null; Sudan excluded from BMI analysis |
| Duplicates | Norway, Slovenia, Somalia, Sweden, Switzerland, Timor-Leste, Togo, United Kingdom — duplicate rows removed |
| World Bank integration | WHO GDP and Population columns replaced with World Bank WDI data; unpivoted from wide to long format; 27 country name mismatches reconciled |
| WHO Region merge | Lookup table merged on Country; retained in Dim_Countries |

---

## 5. Data Modelling — Star Schema

Built using Reference queries (not Duplicate — Reference inherits source changes automatically).

```
Dim_Countries ──┐
                ├── Fact_Life_Expectancy
Dim_Year ───────┘
```

| Table | Grain | Key columns |
|-------|-------|-------------|
| `Fact_Life_Expectancy` | One row per country per year | Country, Year, all numeric measures |
| `Dim_Countries` | One row per country | Country, Status, WHO_Region |
| `Dim_Year` | One row per year | Year, Decade (2000s / 2010s) |

- Relationships: one-to-many, single filter direction (dimension → fact)
- Staging query: Enable Load = Off
- Hierarchies: Geographic (WHO Region → Country), Time (Decade → Year)

---

## 6. DAX Measures

All 16 measures stored in a dedicated empty Measures table (no relationships — expected).

### Base Aggregations
- `Avg Life Expectancy`, `Avg Adult Mortality`, `Avg GDP Per Capita`, `Avg Health Expenditure`, `Avg Schooling`, `Avg ICOR`, `Avg HIV/AIDS` — standard AVERAGE measures responding to all slicer/filter context

### Advanced Measures
| Measure | Technique | Purpose |
|---------|-----------|---------|
| `LE vs Global Average` | CALCULATE + ALL() | Removes filter context for true global average; returns signed difference |
| `Developed vs Developing Gap` | VAR + CALCULATE with Status filter | Computes both group averages simultaneously — cannot be replicated with a slicer |
| `YoY LE Change` | FILTER on ALL(Dim_Year) | Finds MAX year minus 1; returns blank for 2000 |
| `Mortality Pressure Index` | Additive composite | Adult Mortality + Infant Deaths + Under-Five Deaths |
| `Immunization Coverage Avg` | AVERAGEX | Per-row (Polio + Diphtheria + Hep B) / 3 before aggregation |
| `Health Expenditure Efficiency` | DIVIDE | LE / Total Expenditure; DIVIDE handles zero denominators |
| `GDP Total` | SUMX | GDP Per Capita × Population row-by-row then summed |

### Calculated Columns (with ISBLANK guards)
- `Immunization Composite` — (Polio + Diphtheria + Hep B) / 3; ISBLANK guards prevent null-as-zero distortion
- `LE Band` — Low (<60) / Medium (60–74) / High (≥75)
- `GDP Band` — Low (<$1k) / Middle ($1k–$10k) / High (≥$10k)
- `Disease Burden Score` — (100−HepB) + (100−Polio) + (100−Diphtheria) + (Measles/1000). Range: 3–312. **Limitation: components unnormalised — corrected in Project 2.**
- `Lifestyle Score` — Schooling − Alcohol − ABS(BMI−22). **Limitation: components unweighted, BMI deviation dominated — corrected in Project 2.**

### What-If Parameters
| Parameter | Slider range | Scenario measure | Multiplier |
|-----------|-------------|-----------------|------------|
| Health Expenditure | 0–5 pp, increment 0.5 | LE + (slider × 0.8) | Estimated from correlation — **replaced by OLS coefficient in Project 2** |
| Immunization Impact | 0–30 pp, increment 1 | LE + (slider × 0.15) | Estimated from correlation — **replaced by subgroup regression in Project 2** |

---

## 7. EDA and Visualisations

- 10 scatter plots (LE vs GDP, Schooling, ICOR, immunization variables, HIV/AIDS etc.)
- 3 line charts (LE trend by region, global LE over time, YoY change)
- 5 bar charts (regional comparisons, disease rankings, status gap)
- Key Influencers (continuous: top factors driving LE; categorical: top factors for High LE band)
- Decomposition Tree (LE decomposed by Region → Status → GDP Band)
- Filled map (global LE choropleth)
- Q&A visual

---

## 8. Key Findings

| Question | Power BI Finding | Project 2 Update |
|----------|-----------------|-----------------|
| Q1 — Disease | HIV/AIDS most damaging (r ≈ −0.56) | Confirmed. OLS coeff = −0.39 |
| Q2 — Immunization | 3.5× greater impact in developing nations | Refined: 0.296 yrs/1% coverage increase |
| Q3 — ICOR | Strongest correlate with LE; Equatorial Guinea as resource curse example | Formally confirmed: ICOR explains 79% variance vs GDP's 34% |
| Q4 — Predictors | Key Influencers: Schooling #1 | **Overturned:** ICOR #1; Schooling absorbed by collinearity |
| Q5 — Expenditure | Estimated +0.8 yrs per 1% spending increase | **Reversed:** OLS shows −0.38 for low-LE countries (reactive spending) |
| Q6 — Mortality | Adult Mortality strongest | Confirmed |
| Q7 — Lifestyle | Lifestyle Score r = −0.59 (misleading direction) | Corrected to +0.61 after normalisation and weighting |
| Q8 — Schooling | ~0.886 yrs per additional year schooling (estimated) | Formal OLS: ~1.1 yrs; partial correlation confirms genuine effect |

---

## 9. Documented Limitations

These limitations were identified during Project 1 and used to motivate Project 2:

- **Hep B anomalous values** — single-digit values (7, 8, 9) mid-series are likely data entry errors. Power Query cannot apply context-aware row-level correction without complex M scripting. Fixed in Python in Project 2 (2 lines per column).
- **Unnormalised composite scores** — Disease Burden Score mixes raw counts and percentages; Lifestyle Score weights all components equally despite different scales. Both corrected in Project 2 using min-max normalisation.
- **Estimated what-if coefficients** — health expenditure (0.8) and immunization (0.15) multipliers estimated from visual correlation inspection, not formal regression. Replaced with OLS-derived coefficients in Project 2.
- **No confounding analysis** — correlations show association but Power BI cannot separate genuine effects from GDP confounding. Partial correlations applied in Project 2.
- **No formal significance testing** — no p-values, confidence intervals, or hypothesis tests available natively. All added in Project 2.
- **No logarithmic regression** — Preston Curve (log GDP vs LE) not directly implementable in Power BI. Formally tested in Project 2.
- **Simple regional averages** — unweighted; large-population countries and small countries treated equally. Population-weighted averages computed in Project 2.

---

## 10. Techniques Used

| Category | Techniques |
|----------|-----------|
| Data cleaning | Null profiling, fill up/down, duplicate detection, outlier review, data type validation |
| Data modelling | Star schema, Reference queries, one-to-many relationships, drill-down hierarchies |
| DAX patterns | CALCULATE/ALL, VAR, AVERAGEX, SUMX, DIVIDE, FILTER/RELATED, ISBLANK guards |
| Analysis visuals | Key Influencers, Decomposition Tree, Anomaly Detection, Forecasting, Smart Narrative |
| Statistical | Pearson correlation (visual), scatter plots with trend lines, subgroup comparison |

---

*Full methodology and cleaning log: [`Data_science_detailed_guide_powerbi_and_life_expectancy_analysis.docx`](./Data_science_detailed_guide_powerbi_and_life_expectancy_analysis.docx)*
