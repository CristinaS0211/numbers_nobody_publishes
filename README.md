
# Numbers Nobody Publishes – Quant Route

The code is in `numbers_nobody_publishes.ipynb`, with explanations before each step. The final file for the client is `client_file.xlsx`.

## My approach

I chose the quant route because I have a background in statistics and wanted to focus mainly on machine learning. My goal was to estimate company revenue and number of employees using information such as industry, company age, legal form and location.

I decided to focus more on data preparation, feature engineering and model evaluation rather than trying to complete every task.

## Part 1 – Company Matching

I used **fuzzy matching on official business registers** to identify the 100 companies. I cleaned the names and calculated similarity scores between 0 and 100. I accepted matches with a score of at least 75, but only when there was one clear match.

Only 9 companies were successfully matched. Four had ambiguous matches, 14 had no country information, 71 did not have a suitable official register available, one was not found and one name was too short to compare.

I preferred to keep the matching process strict rather than risk assigning information to the wrong company.

## Part 2 – Machine Learning

### 1. Data preparation

I used Romanian financial data from 2024, joined with the trade register using the fiscal code (CUI). My two target variables were:

- **I13:** annual net revenue
- **I20:** average number of employees

One of the first challenges was preparing the dataset correctly. Initially, I joined all authorised activity codes, which increased the dataset from around 4.1 million to 19.6 million rows because some companies appeared multiple times.

This was a problem because the same company could end up in both the training and test sets. I corrected it by using only the main activity code.

I also noticed that I had initially used I3 instead of I13 for revenue, which I corrected after checking the variable definitions.

After removing missing values, negative observations and companies with both zero revenue and zero employees, I had **485,955 companies** for training.

### 2. Exploratory Data Analysis and transformations

Before training the models, I analysed the distributions of the variables and checked for outliers.

I noticed that both target variables were strongly **right-skewed**. Most companies had relatively low revenue and a small number of employees, while a few companies had extremely high values.

I thought this could make prediction more difficult, so I decided to transform the target variables.

My first idea was to apply a logarithmic transformation. However, some companies had negative revenue, so I tried the **Yeo-Johnson transformation**, which can also handle negative values.

Unfortunately, the results were not satisfactory, and some versions of the model showed signs of overfitting.

I finally decided to remove the negative observations and apply `log(1 + x)` to both targets. This also allowed me to keep companies with zero values for one of the targets.

I used the logarithmic scale for training and evaluation because it reduced the influence of extremely large companies.

### 3. Feature engineering and model selection

I initially tried **LightGBM**, but I was not satisfied with the results.

This made me reconsider some of my preprocessing choices. For example, I had created dummy variables for categorical features and discretised some numerical variables.

One example was grouping registration years into five-year intervals. I realised that this could remove useful information because companies registered in different years would be treated as belonging to the same category.

I also started questioning whether creating dummy variables was necessary for the algorithm I wanted to use.

For this reason, I decided to try **CatBoost**, a gradient boosting algorithm known for handling categorical variables effectively.

I thought CatBoost could be more suitable for my dataset because several predictors were categorical. It also allowed me to work with these variables without manually creating dummy variables for all of them.

The final predictors were:

- **Company age:** I thought older companies might have had more time to grow.
- **Industry category:** companies from different industries can have very different revenues and numbers of employees.
- **Two-digit activity code:** to include more detailed industry information.
- **Legal form:** different legal structures might be associated with different company sizes.
- **Capital city indicator:** I thought that being located in Bucharest could influence company size because of the different economic environment and business opportunities.

I also considered including other financial variables, but I decided not to use them because my goal was to predict revenue and employees when financial information was not available.

### 4. Model training and evaluation

I trained two separate **CatBoost models**, one for revenue and one for the number of employees.

I split the data into 80% for training and 20% for testing. Then, I applied **5-fold cross-validation** on the training set.

I wanted to check whether the models could generalise to new companies rather than simply learn patterns specific to the training data.

The results on the log scale were:

| Model | Train R² | CV R² | Test R² |
|---|---:|---:|---:|
| Employees | 0.121 | 0.118 | 0.118 |
| Revenue | 0.083 | 0.080 | 0.081 |

The first thing I noticed was that the results were very similar across training, cross-validation and testing.

This was encouraging because it suggested that **the final models did not show strong signs of overfitting**.

However, the main problem was their predictive performance.

The employee model explained only around 12% of the variation, while the revenue model explained around 8%, on the transformed scale.

I also compared the employee model with a simple baseline that predicted the median. The baseline actually performed slightly better, with a mean log error of 0.398 compared to 0.406 for CatBoost.

This showed me that a more complex machine learning algorithm does not necessarily produce better predictions.

I also calculated **95% prediction intervals** using the errors from the test data.

For employees, the actual value could be between 0.39 and 4 times the predicted value. For revenue, the interval was much wider, between 0.027 and 14 times the prediction.

The cross-validation errors gave similar intervals, which suggested that the uncertainty estimates were relatively stable. However, the intervals were too wide to be useful for the client's pricing decisions.

### 5. What I would improve

After analysing the results, I started thinking about why the models performed so poorly and what I could do to improve them.

I think the main limitation is that **the variables I used do not provide enough information about a company's actual size**, which makes it difficult to predict revenue and the number of employees accurately.

For example, two companies from the same industry, with similar ages and legal forms, can still have completely different revenues.

If I continued this project, I would first focus on adding more relevant predictors, such as share capital, number of branches, employee size categories or information about parent companies.

I also considered whether **regularisation** could improve the results. However, since the final models did not show strong signs of overfitting, I do not think regularisation alone would solve the problem.

What I regret the most is that **I did not have enough time for hyperparameter tuning**.

I would have liked to use GridSearchCV or RandomizedSearchCV to test different CatBoost parameters, such as tree depth, learning rate, number of iterations and regularisation parameters.

I cannot say that tuning would have solved the problem, but I think it would have helped me understand whether the weak results came mainly from the model configuration or from the limited information available in the predictors.

## Part 3 – Would I use the models?

**No, I would not use these models in their current form.**

Their predictive performance is extremely weak, and I think they clearly need additional and more relevant variables for training.

Even though the cross-validation results suggest that the models are relatively stable, they explain very little of the differences between companies. The prediction intervals are also too wide to make reliable pricing decisions.

Another limitation is that only 9 out of the 100 companies were successfully matched.

I would provide the client with the matching results, official sources and reported financial information where available, but I would not recommend using the model predictions to decide pricing categories.

I see these models as a starting point, not a final solution.

## Part 4 – An important observation

I noticed that 10 companies in the list had the Siemens name but appeared under different legal names and in different countries.

This made me realise that a company with low local revenue or few employees could still belong to a much larger international group.

However, companies should be grouped based on ownership, not just similar names. For example, Siemens Energy and Siemens Healthineers should not automatically be grouped with Siemens AG.

I think identifying the ultimate parent company could be more useful for the client's pricing decisions than estimating each local company's size separately.

## Part 5 – Romanian Business Register

I also analysed how many active Romanian companies had the variables needed by the model.

Using status code 1048, I identified 1,620,179 active companies. Of these, **1,433,312 (88.5%)** had all the required predictors.

However, this only means the model can generate predictions, not that they are reliable.

## What I did not complete and final thoughts

Most of my time went into cleaning and joining large datasets, exploratory data analysis, feature engineering, testing transformations and building the machine learning models.

I did not have enough time for hyperparameter tuning, testing additional features, improving international company matching or identifying parent companies.

I preferred to focus on the ML part because it is closer to my statistics background, rather than trying to cover everything superficially.

Even though the results were not good, I am happy that I managed to build a starting point, especially because this problem was outside my comfort zone.

The most important thing I learned is that **a model can be stable without being useful**. In this case, I think improving the input data and adding more relevant predictors would be much more important than simply trying a more complex algorithm.
