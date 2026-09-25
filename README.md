# Data-Scientist-Udacity-Project1
(Owned by Ms. Chingmuankim Naulak)

Based on the 2025 Stack Overflow Developer Survey, the following is the Salary Analysis Report
#  Developer Salary Prediction – 2025 Stack Overflow Survey

**A CRISP-DM case study in explaining and predicting software developer compensation.**

---
## What this project is

The 2025 Stack Overflow Developer Survey is the largest public census of the software profession:
**49,128 respondents**, 170 fields, 170+ countries. This project uses it to answer six business
questions about what actually drives developer pay — and to build a model good enough to price a
developer profile in any market.

The analysis follows **CRISP-DM**, the Cross-Industry Standard Process for Data Mining: six named
phases, each with its own deliverables, ending in a deployment artefact and an honest limitations
section.


## Headline results

| Metric | v1 (first submission) | **v2 (current submission)** |
|---|---|---|
| Records used | 4,076 *(a data-loading bug silently destroyed 92% of the rows)* | **23,026** |
| Final model | Random Forest on 4 features | **XGBoost (tuned) on 136 features** |
| Cross-validated R² | — | **0.588** |
| Test R² | **−0.043** *(worse than predicting the average)* | **0.602** |
| Median absolute % error | ~118% | **26.7%** |
| Baseline (predict the median) | −0.043 | −0.046 |
| Business questions answered | 4 (one unanswerable as posed) | **6 (H1–H6)** |


The final model explains **60% of the variance in log-salary** (test R² 0.602; 5-fold CV 0.588),
against a median baseline that scores *below zero*. In salary terms, the typical prediction is
within **27%** of the true figure, and 47.7% of predictions land within ±25%.

---

## The six business questions

| ID | Question | Short answer |
|---|---|---|
| **H1** | What factors influence developers' salaries? | Country dominates (66.8% of signal), then experience (14.1%); every tool family combined is 5.7% |
| **H2** | Top three drivers, in detail | Geography, experience, and employment type — plus employer size as the next actionable lever (2.4×) |
| **H3** | Does tool utilisation lead to higher earnings? | **Yes, conditionally.** 94 of 115 tools carry a significant adjusted premium or penalty (median +5.2%); cloud/container/data tooling pays, older enterprise tooling carries a penalty |
| **H4** | Does higher education necessarily pay? | **No.** The raw education gradient (+0.109 ρ) collapses to **−0.031** once country and experience are held constant; 0 of 8 major markets show a monotonic gradient |
| **H5** | Which model best suits the data? | Tuned gradient boosting (XGBoost) — CV R² 0.588 / test R² 0.602 vs baseline −0.046 |
| **H6** | Same profile, different regions? | A frozen 8-year profile ranges **6.1×** by location alone ($23,489 → $142,666) |

---

## The dataset

| File | Size | Role |
|---|---|---|
| `survey_results_public.csv` | 49,128 × 170 | Survey responses, one row per respondent |
| `survey_results_schema.csv` | 139 × 6 | Official question dictionary (`qname` → `question` → `type`) |

The schema file is used twice: in **Data Understanding** to attach human-readable question text to
every column, and in **Data Preparation** to classify each column. Profiling the raw file classifies
its 170 fields as:

| Kind | Count | Treatment |
|---|---|---|
| Single-choice categorical | 63 | One-hot encoding (top-N + rare-category floor) |
| Multi-select (`;`-delimited) | 56 | Split into per-item binary presence flags |
| Numeric | 51 | Median imputation + standardisation |

Of the 56 multi-select fields, **12** are used to build the 136-feature model matrix — the ten
technology-stack questions plus `AILearnHow` and `AIAgent_Uses`. The remainder are ranking questions
(`TechEndorse_*`, `TechOppose_*`, `JobSatPoints_*`) and AI-attitude fields, which are excluded under
missingness policies P5/P6 with the reason documented in notebook §3.1.3.

> **Why this matters.** One-hot encoding a multi-select field as a whole string — which the v1
> notebook did — creates thousands of near-empty combination columns. Splitting into per-item flags
> is what makes categories like "which languages do you use" analysable at all.

---

## Missing values: the analysis the first submission lacked

169 of 170 columns are incomplete; `AIAgentObsWrite` is the worst at **99.5%** missing. The reviewer
required a thorough analysis *and* a justified treatment. Rather than patching in passing,
missingness is classified by **mechanism**:

| Band | Columns |
|---|---|
| 0% complete | 1 |
| 0–5% | 5 |
| 5–30% | 21 |
| 30–60% | 90 |
| 60–90% | 23 |
| >90% | 30 |

Three mechanisms are distinguished, with evidence:

| Mechanism | Fields | Why it is missing | Policy |
|---|---|---|---|
| **M1 — Conditional / structural** | `OrgSize`, `ICorPM`, `Industry`, `RemoteWork` | Students, hobbyists and the unemployed have no employer to describe — the notebook *proves* this with an employment crosstab (students are **7.8×** more likely to be missing `OrgSize`) | Encode `"Unknown"` as an explicit category; missingness is itself information |
| **M2 — Multi-select not shown** | every `*HaveWorkedWith` field | The gate question was not answered | No imputation; per-item presence flags only |
| **M3 — Genuine item non-response** | `EdLevel` (2.1%), `YearsCode` (12.5%), `WorkExp` (12.8%), `Age` (2.1%) | Low-rate, spread across all respondent types | Median imputation **fitted inside the pipeline**, plus a `*_was_missing` flag per field |

The target (`ConvertedCompYearly`) is **51.3% missing** and is therefore dropped rather than
imputed — imputing the quantity being predicted would fabricate the outcome. The cleaning funnel
keeps 23,026 of 23,928 salary reporters (96.2%); the filtered median is **$76,570**.


---

## CRISP-DM walkthrough

| Phase | Section | What it produced |
|---|---|---|
| **1. Business Understanding** | §1 | 6 falsifiable hypotheses, named stakeholders, business + technical success criteria set *before* modelling |
| **2. Data Understanding** | §2 | Schema join (every field traced to its survey question), column classification, 5 documented quality checks |
| **3. Data Preparation** | §3 | Missingness taxonomy + 6 justified policies; target cleaning funnel; ordinal/top-N/multi-label encoding; leakage-free pipeline |
| **4. Modeling** | §4 | 6 model families, 5-fold CV, bounded randomised tuning |
| **5. Evaluation** | §5 | Answers to H1–H5, permutation importance, confounder-adjusted tool and education analyses, 3 robustness checks |
| **6. Deployment** | §6 | What-if regional pricing tool (H6), persisted artefacts, model card, limitations, recommendations |

---

## Key findings

### H1 — What drives developer salaries?

Permutation importance is used rather than the model's built-in gain importance, because gain
importance rewards a feature simply for having many split points. Permutation importance asks the
business question directly: *if we lost this information, how much worse would predictions get?*


| # | Feature family | Share of signal | Importance (ΔR²) | Strongest single column |
|---|---|---|---|---|
| 1 | **Country / region** | 66.8% | 0.469 | `Country_top` |
| 2 | **Experience** | 14.1% | 0.099 | `WorkExp_num` |
| 3 | **Employment type** | 4.9% | 0.034 | `Employment` |
| 4 | **Employer size** | 2.9% | 0.021 | `OrgSize_ord` |
| 5 | **Role (DevType)** | 1.8% | 0.013 | `DevType` |
| 6 | **Age** | 1.4% | 0.010 | `Age_num` |

**Country is not a factor among factors — it is the factor.** The model loses 0.469 R² when
`Country_top` is shuffled, against 0.099 for the entire experience block. Every tool family combined
accounts for just 5.7%.

### H2 — The top three, in detail

**#1 Geography.** Median pay across countries with ≥40 respondents spans from
`Nigeria` ($3,830) to `United States of America` ($150,000) — a **39×** range. The contrast is not
only rich-vs-poor: the USA versus the Netherlands is still ~1.8×.


**#2 Experience.** Pay rises **8.8×** from the 0–2 year band ($12,470) to 30+ years ($110,214), but
the curve is concave — the steepest returns are in the first decade, and the marginal value of
another year flattens sharply after ~15 years.


**#3 Employer size** *(the next actionable driver after employment type, which is partly
definitional — a student's "salary" is a part-time stipend)*. Pay climbs monotonically with employer
scale: solo freelancers earn $41,267 median against $100,000 at firms with 10,000+ employees — a
**2.4×** gap on employer size alone. Employment type itself: employed $80,000 vs students $16,374
(**4.9×**).

Crucially, the experience gradient **survives inside every focus country** — so experience is a real
driver, not a proxy for living somewhere expensive. In India pay rises **8.0×** from the 0–5 year
band to 20+ years ($6,974 → $55,795); in the United States only **2.2×** ($79,000 → $170,000).


### H3 — Does tool utilisation lead to higher earnings?

**The naive comparison is unreliable.** Tool usage is confounded by geography, experience and role,
so a raw "median salary of users" ranking partly measures *who uses the tool and where they live*
rather than the tool itself. Adjusting for those confounders flips the sign of **5** of the 42 larger
gaps (|raw premium| ≥ 15%) and reorders the rest, even though the mean size of that group barely
moves (+11.6% raw → +12.0% adjusted). The raw ranking simply cannot be trusted.

The adjusted method removes the confounders by **residualisation**: fit a control model on
experience, age, education, employer size and country only (that control model alone explains
R² = 0.430); then compare the mean residual of users against non-users for each tool. Significance is
tested with a Mann–Whitney U test, and only tools with ≥200 users are reported.

**94 of 115 tools (82%)** show a statistically significant adjusted premium or penalty, and the
median adjusted premium across all 115 tools is **+5.2%**. The largest single effect is
**MacOS at +39.6%**; the largest penalty is **Windows at −18.4%**.

**Largest adjusted pay advantages:**

| Tool | Family | Users | Median pay (users) | Raw premium | **Adjusted premium** |
|---|---|---|---|---|---|
| `MacOS` | Operating system | 7,782 | $93,972 | +47.2% | **+39.6%** |
| `Dynamodb` | Database | 1,939 | $102,575 | +46.2% | **+36.2%** |
| `Kubernetes` | Cloud / platform | 5,411 | $93,611 | +46.5% | **+34.4%** |
| `Homebrew` | Cloud / platform | 4,816 | $98,244 | +48.4% | **+33.7%** |
| `Amazon Web Services (AWS)` | Cloud / platform | 8,006 | $92,812 | +39.8% | **+32.0%** |
| `Cursor` | Dev environment | 3,446 | $83,714 | +9.1% | **+29.8%** |
| `Elasticsearch` | Database | 3,301 | $87,550 | +29.7% | **+29.1%** |
| `Redis` | Database | 5,497 | $84,691 | +16.6% | **+29.0%** |
| `Docker` | Cloud / platform | 12,897 | $81,870 | +29.9% | **+27.5%** |
| `Ruby` | Programming language | 1,492 | $104,413 | +43.4% | **+27.2%** |

**Largest adjusted penalties:**

| Tool | Users | Raw premium | **Adjusted penalty** |
|---|---|---|---|
| `Windows` | 10,707 | −12.1% | **−18.4%** |
| `C` | 3,907 | −5.8% | **−15.4%** |
| `PHP` | 3,934 | −25.2% | **−15.1%** |
| `C++` | 4,423 | −1.3% | **−11.8%** |
| `Dart` | 1,135 | −34.2% | **−11.0%** |
| `Oracle` | 1,896 | −6.2% | **−10.4%** |



The pattern is coherent: **the tools that pay are the ones attached to infrastructure, platform and
data** — cloud providers, container orchestration, databases, and modern AI-assisted editors. The
tools that carry a penalty are older enterprise tooling and the Windows OS.

> **Caveat, stated plainly:** this is an adjusted *association*, not a causal effect. Developers who
> adopt cloud tooling may already have been on a higher-paying track. Selection is a real
> alternative explanation and this design cannot rule it out.

### H4 — Does higher education necessarily lead to greater earning potential?

**No — and this is the most interesting result in the project.**

On the raw numbers education looks like it pays: median compensation rises from $54,527 (secondary
school) to $88,492 (professional/doctoral), a 1.62× spread, with a Spearman ρ of **+0.109**.

But three tests dismantle the naive reading:

1. **The correlation collapses under adjustment.** Controlling for country, experience, age and
   employer size, ρ falls from +0.109 to **−0.031** — a weak *negative* residual association.
2. **The gradient is not monotonic.** In the adjusted ranking, an *Associate* degree scores **−3.4%**
   — below *some college* at **+2.8%**. A Master's adds only **+3.2pp** over a Bachelor's.
3. **0 of 8 major markets show a monotonic gradient.** Within every one of the USA, UK, Germany,
   Canada, France, Australia, Brazil and India, more education does not reliably mean more money.

At **equal experience**, the ordering breaks down completely — among developers with 15+ years, a
Bachelor's ($109,438) out-earns both a Master's ($95,955) and a professional/doctoral degree
($98,612).

**Conclusion:** education is the weakest of the major drivers, and most of its apparent effect is a
proxy for country and experience. It is a credential, not a pay lever.

### H5 — Which model best suits the data?

Six candidate model families were compared under identical 5-fold cross-validation on the training
set, with the test set touched exactly once.

| Model | CV R² | CV SD | Test R² | RMSE (log) | Median abs. % error |
|---|---|---|---|---|---|
| XGBoost | 0.591 | 0.017 | 0.599 | 0.681 | 26.8% |
| XGBoost (tuned) | 0.588 | — | **0.602** | 0.679 | **26.7%** |
| Hist Gradient Boosting | 0.587 | 0.016 | 0.595 | 0.685 | 27.6% |
| Random Forest | 0.569 | 0.018 | 0.578 | 0.699 | 27.8% |
| Ridge (alpha=1) | 0.545 | 0.012 | 0.556 | 0.717 | 31.0% |
| Linear Regression | 0.545 | 0.012 | 0.556 | 0.717 | 31.0% |
| Decision Tree | 0.432 | 0.029 | 0.441 | 0.805 | 33.5% |
| Baseline (median) | −0.046 | 0.000 | −0.050 | 1.102 | 48.2% |


**Three conclusions:**

1. **The problem is non-linear.** Linear and Ridge regression plateau at R² ≈ 0.55 while boosted
   trees reach ≈ 0.60. Pay is not an additive function of its drivers — the value of
   experience *depends on* the country, and vice versa.
2. **Regularisation alone does not help.** Ridge ≈ OLS, so the feature matrix is not badly collinear.
3. **The gap to the baseline is enormous**: −0.046 → 0.588 on cross-validation, or **+0.634 R²**.


### H6 — Simulation: the same developer, placed across different regions

One profile is **frozen** and only `Country` varies. Everything else is identical: 8 years coding, 8
years work experience, age 25–34, Bachelor's degree, back-end developer, 100–499 person employer,
remote, individual contributor, identical tool stack.

| Region | Predicted salary | Observed market median | vs global median |
|---|---|---|---|
| United States of America | $142,666 | $150,000 | +86% |
| Switzerland | $115,224 | $141,972 | +51% |
| United Kingdom | $86,926 | $95,299 | +14% |
| Australia | $85,734 | $97,514 | +12% |
| Canada | $85,494 | $87,550 | +12% |
| Netherlands | $74,398 | $81,210 | −3% |
| Germany | $72,556 | $81,210 | −5% |
| Sweden | $68,386 | $71,017 | −11% |
| Spain | $62,579 | $62,648 | −18% |
| Poland | $48,713 | $60,395 | −36% |
| Brazil | $29,536 | $29,347 | −61% |
| India | $23,489 | $17,436 | −69% |


The same person earns **6.1×** more in the United States than in India — an annual difference of
**$119,177**. Predicted values track observed market medians with a correlation of **0.985**, which
is a useful external sanity check: the model is not inventing geography, it is reproducing a real
pattern. (The ratio, the difference and the correlation are computed from the 12 printed
prediction/median pairs of notebook §6.1; the "vs global median" column is computed against the
modelling population's median of $76,570.)

---

## Robustness

The main result was stress-tested three ways (notebook §5.7):

| Check | Result |
|---|---|
| **R1 — Salary filter sensitivity** | Test R² spans only **0.579–0.617** (spread 0.0371) across four plausibility filters |
| **R2 — Geography held constant** | Refitting on the 5,064 US respondents *lowers* test R² to **0.473** (less variance left to explain); the remaining drivers are experience (0.148), employment type (0.113) and age (0.043) |
| **R3 — Split stability** | Test R² standard deviation across five seeds: **0.0074** (mean 0.6044, range 0.0149) |

> The R1 figures use the **untuned** model configuration so that all four filters are compared on
> identical terms and the check stays cheap. That is why its "Main filter" row reads 0.599 rather
> than the headline 0.602.

| Alternative filter | n | Test R² |
|---|---|---|
| Main filter ($1k–$500k) | 23,026 | 0.599 |
| Stricter low cut ($5k–$500k) | 22,046 | 0.617 |
| Stricter high cut ($1k–$300k) | 22,666 | 0.592 |
| No high cut ($1k–$10M) | 23,200 | 0.579 |

---


## Limitations

A salary survey is observational data. These limits are real and are stated rather than buried:

1. **Self-report bias.** Salaries are unverified. Over- and under-reporting are both plausible and
   neither is measurable from this file.
2. **No equity or bonus data.** Total compensation in the survey excludes equity grants, which are a
   large component of senior pay in some markets. The model is closer to a base-salary model.
3. **Currency conversion is not purchasing power.** `ConvertedCompYearly` uses one conversion rate, so
   a $40k salary in Warsaw and a $40k salary in San Francisco are not the same life.
4. **Correlation is not causation.** H3 and H4 are adjusted *associations*. Selection effects cannot
   be ruled out without an instrument or natural experiment.
5. **The 51.3% who did not report a salary are excluded.** They differ systematically (students,
   retirees, the unemployed). The model describes the salary-reporting population only.
6. **R² ≈ 0.60 leaves 40% unexplained.** Individual pay depends on negotiation, timing, firm
   profitability and luck — none of which is in the file. This is a market-level tool, not an
   individual-level oracle.

## Future work

* Add purchasing-power-parity indices to convert the geographic effect into a *real* income comparison.
* Deploy the what-if calculator as a small interactive web app.

---

## Tech stack

`pandas` · `numpy` · `scikit-learn` · `XGBoost` · `matplotlib` · `seaborn` · `scipy` · `joblib` · `Jupyter`


## Acknowledgements

Survey data: [Stack Overflow Developer Survey 2025](https://survey.stackoverflow.co/2025/).
Process framework: [CRISP-DM](https://www.datascience-pm.com/crisp-dm-2/) — *Evaluating CRISP-DM for
Data Science*, Data Science PM, 2025.

---
