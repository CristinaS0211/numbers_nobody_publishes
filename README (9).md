Am rescris textul într-o engleză mai simplă și mai naturală, ca și cum l-ai fi scris tu. Am păstrat cifrele, rezultatele și explicațiile importante, dar am folosit propoziții mai scurte și cuvinte mai ușoare.



\# Numbers Nobody Publishes – Quant Route



The code is in `numbers\_nobody\_publishes.ipynb`, where I explained each step. The final file for the client is `client\_file.xlsx`.



\## How I understood the problem



The client does not just need revenue and employee numbers for each company. They want to know which companies are in the wrong price category and have enough evidence to explain their decisions to an auditor.



For this reason, I think it is important to know which company each number belongs to, where the information comes from, and how accurate the estimate is.



I chose the quant route because I wanted to build a model using data from many companies instead of researching every company separately. This also allows me to measure how accurate my predictions are.



\## Part 1 – Matching the companies



I used all 100 company names from the list. Since the matching process is automatic, checking 100 companies is not much different from checking 10.



For each company, I searched for its name in an official business register from the country mentioned in the dataset. I used sources such as ONRC for Romania, the government register for France, Brønnøysund for Norway, PRH for Finland, and Companies House for the UK.



I cleaned the company names to make them easier to compare. Then I calculated a similarity score between 0 and 100.



I accepted a match if the score was at least 75 and there was only one possible company above this value. At first, I used 90, but I reduced it to 75 because I wanted to match more companies.



This score only shows how similar the names are. It is not a probability that the match is correct. I also kept the source link and the score for each company.



The process was more difficult than expected because a brand name is not always the same as the legal company name. Also, several companies can have similar names, and the country in the dataset is not necessarily the country of the parent company.



\*\*Results for the 100 companies:\*\*



\- 9 companies were matched.

\- 4 had multiple possible matches, so I could not choose one.

\- 1 company was not found in the register.

\- 1 name was too short to compare after cleaning (LV=).

\- 14 companies had no country.

\- 71 companies were from countries where I could not use an official register to search by name.



Only the company that was not found in the register might not exist. For the other cases, the problem was that I could not identify the company with the available information.



For 85 companies, I could not even start comparing names because the country was missing or I did not have a suitable official source.



For example, the list includes 19 companies from the US and 12 each from Germany and the UK, but I could not use a suitable free official name search for all of them.



Out of the 14 companies that reached the matching stage, 9 were matched.



My conclusion is that using only official registers from the given country provides more reliable matches, but it also limits how many companies I can identify.



\## Part 2 – Building the prediction model



\### Data preparation



I used financial data reported by Romanian companies in 2024 and combined it with information from the trade register using the fiscal code (CUI).



I used two target variables:



\- \*\*I13:\*\* annual net revenue.

\- \*\*I20:\*\* average number of employees.



At first, I made a mistake when joining the data. I used all authorised activity codes, which increased the dataset from 4.1 million to 19.6 million rows. This meant the same company could appear in both the training and test datasets.



To solve this, I removed that join and used only the main activity code from the financial data.



From the original 822,292 financial reports, I removed:



\- 323,272 companies with missing revenue or employee information.

\- 176 companies with negative values.

\- 12,292 companies with both zero revenue and zero employees.



I considered companies with both values equal to zero as not having economic activity for this analysis.



After cleaning, I had \*\*485,955 companies\*\* for the model.



I also noticed that I had initially used I3 instead of I13 for revenue. I corrected this after checking the variable descriptions.



\### Variables used



I kept the following variables:



\- Company age.

\- Industry category.

\- First two digits of the activity code.

\- Whether the company is in the capital city.

\- Legal form of the company.



I also tested other options, but I decided not to use them.



For example, I tried using industry categories as separate binary variables, but 26 of them were almost always zero, and the model had an R² of about 0.02.



I also tried grouping registration years into five-year periods, but this removed useful information.



For revenue, I tested the Yeo-Johnson transformation, but the distribution was still skewed. I decided to use `log(1 + x)` instead.



I could not include a franchise variable because being part of a franchise is based on a contract and is not directly available in the registers.



I also avoided using other financial variables because, if I already had that financial information, I would probably also have the company's revenue.



\### Variable importance



The importance of the variables was different for the two models.



| Variable                | Employees | Revenue |

| ----------------------- | --------- | ------- |

| Industry category       | 38.7%     | 29.3%   |

| Company age             | 29.9%     | 49.0%   |

| Two-digit activity code | 26.1%     | 13.3%   |

| Capital city            | 2.7%      | 5.4%    |

| Legal form              | 2.5%      | 2.9%    |



The most important variables were industry and company age. The capital city and legal form variables had very little importance.



\### Method



I divided the dataset into training and test data.



I kept 20% of the companies for testing and used the other 80% for training.



On the training data, I applied 5-fold cross-validation. I used CatBoost and trained a separate model for revenue and employees.



I used the test dataset only for the final evaluation.



\### Results



The R² results on the log scale were:



| Model     | Train R² | CV R² | Test R² |

| --------- | -------- | ----- | ------- |

| Employees | 0.121    | 0.118 | 0.118   |

| Revenue   | 0.083    | 0.080 | 0.081   |



The results are similar for training, cross-validation, and testing. This suggests that the models are stable and are not overfitting.



However, the prediction performance is still weak.



For revenue, the typical prediction error is around a factor of 2.4. Only 42% of the companies have predictions within a factor of 2 of their actual revenue.



For employees, 83% of the predictions are within a factor of 2. However, this result can be misleading because most companies have only one or two employees.



I compared the model with a simple prediction based on the median number of employees. The median actually performed slightly better, with a mean log error of 0.398 compared to 0.406 for CatBoost.



This means the employee model does not really improve on a very simple method.



\### Prediction intervals



I also calculated a 95% prediction interval using the errors from the test data.



For employees, the actual value can be between 0.39 and 4 times the predicted value.



For revenue, the actual value can be between 0.027 and 14 times the prediction.



I also calculated intervals using cross-validation errors. The results were very similar:



\- Employees: 0.40 to 4.1 times the prediction.

\- Revenue: 0.029 to 13.8 times the prediction.



For employees, 94.9% of the test companies were inside the interval calculated from cross-validation. This is very close to the expected 95%.



For every estimate, I included the prediction interval, the reported 2024 value when available, the unit and year, the company name and registration number, the source link, the matching score, and the number of missing model inputs.



An important limitation is that the model can only be used for companies that were successfully matched. Without identifying the company, I cannot get its age, industry, or legal form.



\## Part 3 – My decision



\*\*I would not recommend using these predictions for pricing decisions.\*\*



There are two main reasons.



First, only 9 out of the 100 companies could be matched and receive an estimate.



Second, the models have low predictive power. The employee model explains only around 12% of the variation between companies, while the revenue model explains around 8%.



Also, the employee model does not perform better than simply predicting the median.



The prediction intervals are too wide, especially for revenue. An interval from 0.027 to 14 times the estimated value is too large to decide whether a company belongs to a small-company or enterprise price category.



I would give the client the matching results, including the sources and reasons why some companies were not matched. I would also include the actual financial information for companies where it was available.



However, I would explain that the model estimates are not accurate enough to use for pricing.



There are also some problems that my analysis cannot fully identify, such as a company being matched incorrectly, a small local company belonging to a large international group, or the 2024 financial data no longer representing the company's current situation.



As a next step, I would try to add variables that are more related to company size, such as the number of branches, share capital, and employee size categories when available.



I would also try to identify the parent company of each business.



\## Part 4 – What I would tell the client



One important thing I noticed is that 10 companies in the list have the Siemens name.



They appear under 10 different legal names. Seven have a country mentioned (Mexico, China, the UK, Brazil, Spain, Singapore, and Canada), while three have no country.



In the dataset, they look like separate customers, but they may belong to the same larger group.



If the client sets prices based only on the size of each local company, they might charge a small subsidiary a lower price even if it belongs to a very large international business.



However, I cannot group companies only because they have similar names. For example, Siemens Energy and Siemens Healthineers are listed separately from Siemens AG.



The correct grouping should be based on ownership, not just the company name.



This is important because financial statements usually show information for individual legal entities, not the entire group.



In the Romanian register, the variable for a foreign parent company is missing for more than 99% of companies. This makes it difficult to identify ownership relationships.



A subsidiary may report low revenue or few employees, even if the software is used by a much larger group.



For this reason, \*\*I think identifying the ultimate parent company would be more useful for the client than estimating revenue or employees with the current model.\*\*



That would be my main priority.



\## Part 5 – Romanian company register



For this part, I defined an active company as one whose latest status code is 1048.



I then selected companies that had the information needed by the model: registration date, legal form, county, and an activity code that could be assigned to an industry category.



This is only a basic check. It means the model has enough information to make a prediction, not that the prediction will be accurate.



From the 3,924,396 companies with a fiscal code, 1,620,179 were active.



Of these, 1,433,312 had all the required variables, representing \*\*88.5% of the active companies\*\*.



The largest industries were:



\- Retail trade: 204,122 companies.

\- Construction: 154,295 companies.

\- Agriculture, forestry and fishing: 137,569 companies.



\## What I did not complete



Most of my time was spent cleaning and joining the datasets and testing different versions of the variables.



I focused more on Part 2, where I built and evaluated the prediction models. For Part 1, I kept a strict matching method based on official registers.



Some things I did not manage to complete were:



\- Adding more countries, such as Germany, the Netherlands, and the US, where I could not use a suitable free official register search.

\- Using GLEIF as an alternative source, because it is not the official national business register.

\- Grouping companies by their parent company.

\- Testing more CatBoost settings to improve the models.



Revenue was measured in RON, and I used one average exchange rate for conversion to EUR.



\## Debate topics



\### Where I would not trust AI



I would not completely trust AI when it gives information about a specific company or when it identifies variables from a dataset.



For example, I initially used I3 instead of I13 for revenue. This showed me how easy it is to make a mistake if I do not check the meaning of each variable.



To avoid these problems, I would compare the variables with the official documentation and manually check a few companies.



\### How I would check if a large dataset is correct



I would use different methods to identify possible problems.



For example, I would compare the number of companies in each industry with national statistics and check whether revenue per employee has reasonable values.



I would also compare information for the same company across different years to identify unusual changes.



Finally, I would manually check a sample of a few hundred rows every month.



\### How I would check the prediction intervals



I would calculate the prediction intervals using errors from companies that were not used during training.



I would also evaluate the intervals separately for different industries and company sizes.



After one year, I would compare the predictions with the new financial reports and calculate how many actual values fall inside the intervals.



If the result is very different from 95%, it would mean that the intervals are not reliable and need to be adjusted.



Make the opening and conclusion more directUse a consistent client-facing tone

