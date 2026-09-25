# Compensation Analytics for Workforce Optimization
Using the 2025 Stack Overflow Developer Survey (49,128 respondents across 170+ countries), this study aims to uncover the key factors that drive developer compensation — so that companies can make defensible offers and individuals can make better-informed career decisions. The analysis follows the CRISP-DM process (six phases, from business understanding to deployment) and models 23,026 salary-reporting respondents on a matrix of 136 features with an XGBoost ensemble. The questions the study set out to answer are listed below.
## Exploring key drivers:
#### What factors influence developers' salaries, and how much does each one matter?
#### Among those drivers, what are the top three — in detail?
#### Does utilization of specific tools lead to higher earnings among developers?
#### Does obtaining higher education necessarily lead to greater earning potential?
#### Which machine learning model best suits this data, and how accurate is it?
#### What happens in a "what-if" scenario — if we freeze one developer profile and change only their location, what salary does the model predict?

## Key drivers of Developers' Salaries: What Matters Most?
The model explains the variation in pay by shuffling each feature family and measuring how much worse the predictions get. Country, experience and employment type stand out, and the ranking is unusually lopsided:
### 	Driver	Share of explained signal	Effect size
1	Country / region	66.8%	$3,830 (Nigeria) → $150,000 (USA) — a 39× range
2	Experience	14.1%	8.8× from the 0–2 year band ($12,470) to 30+ years ($110,214)
3	Employment type	4.9%	Employed $80,000 vs students $16,374 — 4.9×
4	Employer size	2.9%	Solo freelancer $41,267 → 10,000+ employees $100,000 — 2.4×
5	Role (DevType)	1.8%	
6	Age	1.4%	
#### Country is not a factor among factors — it is the the factor. Shuffling country alone costs the model 0.469 R², against 0.099 for the entire experience block. Every tool family in the survey combined — languages, databases, cloud platforms, frameworks, editors, operating systems — accounts for just 5.7%. Where a developer lives explains far more about their pay than what they know.
#### Experience is the strongest thing a developer controls. Pay rises 8.8× across a career, but the curve is concave: the steepest returns come in the first decade, and the marginal value of another year flattens sharply after about 15 years. Crucially, the gradient holds inside every one of the eight focus markets, so experience is a genuine driver and not a proxy for living somewhere expensive.
#### Employment type is real but partly definitional. A student's "salary" is a part-time stipend, not a full-time wage, so this family partly acts as a population filter. The next driver a person can actually act on is employer size: moving from solo freelancing to a large firm is worth about 2.4× on its own.

##### Digging Deeper into the Data: Uncovering creative Insights
Four countries pay far above the rest. Led by the United States at a median of $150,000 — close to twice the global median of $76,570 — followed by Switzerland ($141,972), Israel ($141,188) and Ireland ($116,015). The spread is not only rich-versus-poor: even between two wealthy markets, the USA versus the Netherlands, the gap is still ~1.8×.
The return on experience is wildly unequal across countries. Measured from the 0–5 year band to the 20+ year band, pay rises about 8.0× in India ($6,974 → $55,795) but only about 2.2× in the United States ($79,000 → $170,000). Early-career developers in lower-paying markets have the steepest room to climb.
Tool utilisation does pay — conditionally. A naive "median salary of users of tool X" ranking is unreliable, because tool usage is confounded by country, seniority and role. After adjusting for those confounders, 94 of 115 tools show a statistically significant premium or penalty. The winners are infrastructure, platform and data tooling — MacOS (+39.6%), DynamoDB (+36.2%), Kubernetes (+34.4%), Homebrew (+33.7%), AWS (+32.0%) and the AI-assisted editor Cursor (+29.8%). The penalties fall on older enterprise tooling and Windows (−18.4%). Adjusting does not shrink the effects so much as reorder them — it flips the sign of 5 of the 42 larger gaps, which is exactly why the raw ranking cannot be trusted. And this is an adjusted association, not proof that learning Kubernetes causes a raise.
More education does not necessarily mean more money. On the raw numbers it looks like it pays: median compensation rises from $54,527 (secondary school) to $88,492 (professional/doctoral), a 1.62× spread, with a rank correlation of +0.109. Three checks dismantle that reading. The correlation collapses to −0.031 once country, experience, age and employer size are held constant. The adjusted ranking is not even monotonic — an Associate degree scores −3.4%, below some college at +2.8%, and a Master's adds only +3.2 percentage points over a Bachelor's. And 0 of 8 major markets (USA, UK, Germany, Canada, France, Australia, Brazil, India) show a monotonic education gradient. At equal experience the ladder breaks at the top: among those with 15+ years, a Bachelor's ($109,438) out-earns both a Master's ($95,955) and a professional/doctoral degree ($98,612). Education is a credential, not a pay lever.
The salary-reporting population is not the developer population. The 51.3% of respondents who did not report a salary differ systematically — they are students, retirees and the not-employed — so every number here describes salaried developers only.
#### Performance of the Model: How accurate does it predict?
Six candidate model families were compared under identical conditions — 5-fold cross-validation on the training set, with the hold-out test set touched exactly once. Two linear models (OLS and Ridge), a single decision tree, a random forest, histogram gradient boosting and XGBoost:
Model	Cross-validated R²	Test R²	Median absolute % error
XGBoost (tuned)	0.588	0.602	26.7%
XGBoost	0.591	0.599	26.8%
Histogram gradient boosting	0.587	0.595	27.6%
Random forest	0.569	0.578	27.8%
Ridge regression (α = 1)	0.545	0.556	31.0%
Linear regression	0.545	0.556	31.0%
Single decision tree	0.432	0.441	33.5%
Baseline (predict the median)	−0.046	−0.050	48.2%
The final model explains about 60% of the variance in pay — an improvement of +0.634 R² over the median baseline, which scores below zero. On the log scale the RMSE is 0.679, a multiplicative error of ×1.97, and the typical prediction lands within 27% of the true salary; 47.7% of predictions are inside ±25%. Three conclusions follow:
The problem is non-linear. Linear and Ridge regression plateau around R² ≈ 0.55 while boosted trees reach ≈ 0.60. Pay is not an additive function of its drivers — the value of experience depends on the country, and vice versa.
Regularisation alone does not help. Ridge ≈ OLS, so the feature matrix is not badly collinear; the gain came from the model class, not from tuning. The tuned model wins on the test set (0.602) even though the untuned one has a marginally higher CV score (0.591) — a useful reminder that CV differences of 0.003 are noise.
The result is stable. Refitting under four different salary-plausibility filters moves test R² only between 0.579 and 0.617, and across five random train/test splits the standard deviation is 0.0074. Holding geography constant (a US-only refit) lowers R² to 0.473, which is expected — with country removed there is less variance left to explain — and inside the US, experience and employment type become the dominant drivers.
The remaining 40% is real-world noise that no survey field captures: negotiation, bonus timing, equity, firm profitability and luck. This is a market-level tool, not an individual-level oracle.
#### Hypothetical Scenario : Using a What-if?
One profile is frozen — 8 years of coding, 8 years of work experience, age 25–34, Bachelor's degree, back-end developer, 100–499 person employer, remote, individual contributor, identical tool stack — and only the country changes:
Region	Predicted salary	Observed market median
United States of America	$142,666	$150,000
Switzerland	$115,224	$141,972
United Kingdom	$86,926	$95,299
Canada	$85,494	$87,550
Germany	$72,556	$81,210
India	$23,489	$17,436
The same person earns 6.1× more in the United States than in India — an annual difference of $119,177 — for identical work. Predicted values track the observed market medians with a correlation of 0.985, a useful external sanity check: the model is not inventing geography, it is reproducing a real pattern. (It does, however, run slightly high in low-paying markets and slightly low in rich ones, which is the usual mild regression to the mean.)

#### Conclusion:
Location is the top driver of developers' salary, making up about two-thirds (66.8%) of everything the model can explain. Experience comes next, with an 8.8× rise in pay across a career that flattens after roughly 15 years, followed by employment type and employer size. Education, contrary to the folklore, is not a pay lever at all: its apparent effect is a proxy for country and experience, and the ladder breaks at the top. Tools matter, but conditionally — the premium attaches to infrastructure, platform and data tooling, and even then it is an association that selection effects could partly explain.
The practical reading is that developers cannot rewrite where they live, but they can act on the things that remain: accumulate experience deliberately, move toward infrastructure and data tooling, and treat employer scale as a real lever rather than a detail. For hiring managers and HR teams, the model's value is narrower but sharper — it prices a comparable profile in 12 markets within about 27% of the true figure, which is enough to sanity-check an offer and far better than comparing raw averages across countries.
Future work is to add purchasing-power-parity indices so the geographic effect becomes a real income comparison, extend the same pipeline to job satisfaction as a second outcome, and test causal identification for the tool premium with an instrument or a natural experiment.
Data: Stack Overflow Developer Survey 2025. Process framework: CRISP-DM. Full notebook, figures and artefacts: this repository.
