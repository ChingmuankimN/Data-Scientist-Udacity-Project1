Using 2025 Stack Overflow Survey Data, this study aims to uncover insights and patterns on the key factors that drives Developers' Job and Salary Satisfaction to help companies and individuals make better decisions in the hiring process and the compensation offer to attract more competent employees. To discover this valuable information, a study has been conducted using machine learning model following CRISP-DM process in analyzing the data. The study conducted are as below:
Exploring key drivers:
1. What are the most important features of the dataset, and how do they drive developers' salaries?
2. What unusual or creative insights can we uncover from the survey data?
3. How accurate is a machine learning model at predicting salaries from this information?
4. What happens in a "what-if" scenario - if we create a hypothetical developer profile, what salary would the model predict?
Key drivers of Developers' Salaries: What Matters Most?
When the model was asked to extract Top 15 drivers of Salary, the following four features stood out as the strongest drivers of salary:
Years of Coding: Years of coding has the biggest influence. More years generally means higher compensation, though the effect levels off after about 20 years. Fig. 2 shows a histogram that is skewed towards the right, suggesting that most developers have approximately 20 years of experience in coding. The peak appears around 10 years. After 20 years, the count declines steadily, with relatively few developers reporting more than 40 years of coding experience. This suggests the developer population is concentrated in the early to mid-career range, with far fewer very senior professionals.
Country: Location has the second biggest influence. Developers in the U.S. tend to earn significantly more than those in emerging markets.
AI Tool Usage: Those who embrace AI assistants or machine learning in their workflows often earn more, showing a "skills premium" for early adopters.
Programming Languages: Developers who reported using modern languages like Python, TypeScript, or Go were associated with higher pay than those sticking to older stacks.

Experience in coding and Geography set the baseline, while skill choices (languages, AI use) can give developers a competitive edge.
Digging Deeper into the Data: Uncovering creative Insights
Developers who reported using AI tools earned more, but some also reported lower satisfaction. Productivity gains may come with new frustrations.
Developers combining Python + TypeScript consistently reported higher salaries than those specializing in just one language. Versatility is rewarded.
Performance of the Model: How accurate does it predict?
The experimental result shows about 25% of the variance are in developers' salaries which is not so good. The RMSE measures the average error in predictions. Result shows that model is able to capture few meaningful patterns, leaving a room for improvement. Performance can be boosted by:
- Adding more features (e.g., job role, education level, company type)
- Using feature engineering (e.g., grouping countries by income level)
- Trying more advanced models or tuning hyperparameters.
While an Accuracy of ~69% is obtained, which means the model correctly predicts job satisfaction for about 69% of the test cases and F10 of ~80%. It means the model has a good balance of precision and recall for the positive class (those marked as "satisfied"). The F1 score being higher than accuracy suggests the model is good at finding the positives.
Hypothetical Scenario : Using a What-if?
Developers from two different locations are compared to see the salary impact with the same level of experience. The result shows quite a low wage for a person living in India with an experience of 5 years than a person in the U.S.
Conclusion:
While Experience and Geography tops the list in key factors that drives Developers' Salary, the data also confirms that it is not necessary to switch locations, you can change the tools you learn and the technologies you adopt, that can pay off big for enhancing the salary. Skilled individuals of around 5–20 years of experience are found to be most highly recruited with few senior developers'. This data may help both individuals and HR's in selection and recruitment procedures.
This post is part of a data science project following the CRISP-DM process. Full code, notebook, and details are available on my GitHub.
Keep exploring!
