# Data-Scientist-Udacity-Project1
(Owned by Ms. Chingmuankim Naulak)

Based on the 2025 Stack Overflow Developer Survey, the following is the Sales Analysis Report
#  Developer Salary Prediction – 2025 Stack Overflow Survey

##  Motivation
The goal of this project is to explore the **2025 Stack Overflow Developer Survey** dataset and answer business-driven questions such as:

- What factors most influence developer salaries?  
- How do experience, country, languages, and AI tool usage impact outcomes?  
- Can we build a model to predict compensation from survey responses?  
- What insights can be communicated to both technical and non-technical stakeholders?  

By following the **CRISP-DM process**, we move from data exploration → cleaning → modeling → evaluation → communication.

---

##  Libraries Used
This project was developed in Python with the following key libraries:

- **pandas** – data manipulation and cleaning  
- **numpy** – numerical operations  
- **matplotlib & seaborn** – exploratory data analysis (EDA) and visualization  
- **scikit-learn** – machine learning (Random Forests, preprocessing pipelines, metrics)  
- **jupyter notebook** – interactive coding, documentation, and reproducible analysis  

---

## Repository Contents
- **notebooks/SalesAnalysis.ipynb**  
  Main Jupyter Notebook containing data exploration, cleaning, modeling, and evaluation with explanatory comments.

- **data/survey_results_public.csv** (Due to large size of the data, this dataset can't be uploaded in this github repo)
  Raw survey responses (as provided by Stack Overflow).

- **data/survey_results_schema.csv**  
  Schema mapping column names to human-readable questions.

- **README.md**  
  This file. Describes project motivation, structure, results, and acknowledgments.

- **blog_post.md**  
  A blog-style write-up with non-technical explanations and visuals, designed for a wider audience.



##  Summary of Analysis & Results
1. **Most Important Features**  
   - **Country** – regional differences dominate salary predictions.  
   - **Years of Coding Experience** – more years generally correlate with higher compensation.  
   - **Languages Worked With** – using Python, TypeScript, Go, or Rust increases salary potential.  
   - **AI Tool Usage** – regular use of AI coding assistants is associated with higher salaries.  
   - **Work Hours** – only a minor effect; working 60+ hours doesn’t guarantee more pay.

2. **Unusual / Creative Insights**  
   - Overwork doesn’t pay: many developers working 40 hours earn nearly the same as those working 60+.  
   - The “AI paradox”: AI users earn more but sometimes report lower satisfaction.  
   - Language combinations (e.g., Python + TypeScript) outperform single-language profiles.

3. **Model Accuracy**  
   - **R² (log compensation):** ≈ 0.60  
   - **RMSE (original units):** ≈ $20,000–25,000  
   → Good at capturing trends across groups, less precise for individual prediction.

4. **Creative Scenario**  
   - A **5-year experienced developer in India**, working 40 hours/week with **Python + TypeScript** and **AI tools**, is predicted to earn **~$X USD/year**, about 20–30% above the regional average.



##  Acknowledgments
- Dataset: [Stack Overflow Developer Survey 2025](https://insights.stackoverflow.com/survey)  
- Tools: Python, Jupyter, and the open-source data science community  
- Inspiration: CRISP-DM methodology for structured data science projects  



*This project demonstrates the end-to-end data science workflow: exploring raw survey data, cleaning it, building predictive models, evaluating them, and communicating actionable insights to both technical and non-technical audiences.*
