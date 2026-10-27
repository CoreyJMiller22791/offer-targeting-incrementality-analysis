# Offer Targeting and Incrementality Analysis

## Executive Summary

This project demonstrates how customer and campaign data can be transformed into a data-driven marketing targeting strategy. Six synthetic raw datasets were loaded into DuckDB, SQL was used to build the analytical population and engineer historical features, and Python was used for modeling, validation, and business analysis.

The first phase compared Logistic Regression with Gradient Boosting to predict which treated customer-campaign opportunities were most likely to activate. Logistic Regression was selected because it produced stronger and more stable holdout performance. The second phase measured the campaign’s average treatment effect and used separate treated and control Logistic Regression models to estimate how much each opportunity’s activation probability changed because of treatment.

Treatment increased activation by 6.9 percentage points across the holdout population. Uplift targeting provided its strongest prioritization value within the top 30% of opportunities, generating approximately 54 more incremental activations than random targeting and 92 more than response targeting at the same targeting depth.


## Business Problem

The business needed to determine whether the marketing treatment increased activation and how eligible customer-campaign opportunities should be prioritized. The analysis compared two targeting strategies: selecting opportunities most likely to activate when treated and selecting those most likely to activate because of treatment. The final decision was to identify which strategy produced the greatest incremental benefit at the same targeting depth.

## Data and Analytical Grain

The analysis used six synthetic source datasets representing customers, campaigns, campaign eligibility, treatment assignments, transactions, and offer-activation events. The final analytical grain was one customer-campaign-observation-date record per row. Each row represented one customer’s eligibility for a specific campaign at a defined observation date, with historical predictors measured before that date and activation outcomes measured afterward.

## SQL Feature Engineering and QA

SQL was used to engineer historical predictors using only information available before each observation date, when campaign targeting occurred. These predictors included historical order count, historical revenue, most recent transaction date, and days since the customer’s last transaction.

The binary activation outcome captured whether a customer activated from the observation date through day 30. A 90-day revenue outcome also captured category-matched transaction revenue associated with the retailer category promoted by each campaign.

Quality checks confirmed that the final model population preserved the intended customer-campaign-observation-date grain and matched the eligibility population’s row count. Additional checks validated treatment and activation values, negative revenue, missing values, and recency consistency. Missing transaction recency was expected only for customers without prior transaction history and was handled during model preprocessing.

## Modeling Approach

A preprocessing pipeline handled numerical and categorical predictors separately. Numerical missing values were filled with zero and accompanied by a missing-history indicator, numerical features were standardized for Logistic Regression, and categorical features were one-hot encoded.

The response-modeling phase compared Logistic Regression with Gradient Boosting using treated customer-campaign opportunities. A chronological training and holdout split, separated by a 30-day outcome-maturity gap, was used to evaluate performance on later campaigns. Logistic Regression was selected because it produced stronger and more stable holdout performance.

The uplift phase used separate Logistic Regression models for the treated and control populations. Both models scored every opportunity in the full holdout population, and estimated uplift was calculated as the predicted activation probability under treatment minus the predicted probability without treatment. This allowed response targeting and uplift targeting to be compared at identical population depths.

## Key Results

- **Response-model selection:** Logistic Regression produced stronger and more stable holdout results than Gradient Boosting. Its holdout ROC AUC was 0.5442 compared with the no-skill baseline of 0.5000, while average precision was 0.2109 compared with 0.1858. Log loss and Brier score also showed small improvements over baseline, and the alignment between training and holdout performance indicated no substantial overfitting.

- **Response-targeting performance:** The top 10% of Logistic Regression scores captured 12.02% of observed activations and had an activation rate 20% higher than the overall holdout rate. Expanding targeting to the top 20% captured 23.47% of activations and maintained an activation rate 17% above the overall rate.

- **Average treatment effect:** Treatment increased activation by 6.91 percentage points across the holdout population. The 95% confidence interval ranged from 5.27 to 8.56 percentage points and did not include zero. Treatment lift was positive across all four holdout campaigns, ranging from 6.00 to 8.39 percentage points.

- **Control-model performance:** The control Logistic Regression model also outperformed its no-skill baseline. Its holdout ROC AUC was 0.5777, and its average precision was 0.1561 compared with a baseline of 0.1166. This supported its use for estimating activation probability without treatment.

- **Uplift estimation:** Average predicted uplift was 5.49 percentage points compared with the observed holdout treatment lift of 6.91 percentage points. The models underestimated the overall effect by approximately 1.4 percentage points but remained reasonably aligned at the population level.

- **Uplift-targeting performance:** Uplift targeting produced approximately 17 more incremental activations than random targeting at 10% targeting, 42 more at 20%, and 54 more at 30%. Its advantage became less consistent beyond the top 30%.

- **Strategy comparison:** Compared with response targeting, uplift targeting generated approximately 73 additional incremental activations at 10% targeting, 78 at 20%, and 92 at 30%. The largest advantage occurred at the 30% targeting depth, demonstrating that ranking opportunities by estimated treatment benefit was more effective than ranking them solely by their probability of activating when treated.

## Business Recommendation

Use the uplift model to prioritize the top 30% of customer-campaign opportunities. This segment produced the strongest gain over random targeting and the largest advantage over response targeting. Model-driven performance became inconsistent beyond the top 30%, so additional targeting should be evaluated based on campaign economics and operational goals.

## AI Decision Assistant

The campaign decision assistant makes the strategy comparison accessible
through questions such as:

- Which strategy should I use when targeting 30% of customers?
- How do the recommendations change between 30% and 60%?
- How much does the leading strategy outperform the alternatives at 30% targeting?

The assistant uses a local language model to interpret each question.
Python retrieves the existing campaign estimates and calculates the
comparisons. The model then explains the results, while Python handles
unsupported targeting depths separately.

The final workflow passed all 17 questions in the evaluation suite,
including questions with multiple depths, written shares, and unsupported
values. It also passed a separate check for requests exceeding the
three-depth limit.

The assistant compares estimated incremental activations. It cannot
determine profit without campaign cost and customer value information.

## Tools Used

- **DuckDB and SQL:** Data loading, joins, feature engineering, outcome construction, and quality validation
- **Python 3.11:** Modeling, evaluation, uplift analysis, and strategy comparison
- **pandas and NumPy:** Data preparation, aggregation, validation, and reporting
- **scikit-learn:** Preprocessing pipelines, Logistic Regression, Gradient Boosting, and model-evaluation metrics
- **Matplotlib:** Cumulative gains visualization
- **Jupyter Notebook:** End-to-end development, documentation, and presentation
- **Ollama and Qwen:** Local language model for interpreting questions and explaining campaign comparisons

## Repository Structure

```text
offer-targeting-incrementality-analysis/
├── .gitignore
├── README.md
├── requirements.txt
├── offer_targeting_incrementality_analysis.ipynb
├── campaign_targeting_demo.ipynb
├── campaign_decision_assistant.ipynb
├── campaign_assistant_data/
│   ├── response_summary.csv
│   └── uplift_summary.csv
└── Raw Data/
    ├── assignments.csv
    ├── campaigns.csv
    ├── customers.csv
    ├── eligibility.csv
    ├── offer_events.csv
    └── transactions.csv
```

The six CSV files contain the synthetic source data. The analysis notebook contains the data preparation, modeling, and
strategy comparison. The assistant demo uses the exported summaries
in `campaign_assistant_data`. The development notebook documents the
assistant's revisions and evaluation. The DuckDB database is created
or updated when the analysis notebook is run.

## How to Run the Project

### Campaign analysis

1. Install Python 3.11 and Jupyter Notebook.
2. Install the required packages:

   ```bash
   pip install jupyter duckdb pandas numpy scikit-learn matplotlib
   ```

3. Download or clone the repository and confirm that the six source
   CSV files are in the `Raw Data` folder.
4. Open a terminal in the main project folder and start Jupyter:

   ```bash
   jupyter notebook
   ```

5. Open `offer_targeting_incrementality_analysis.ipynb`, restart the
   kernel, and select **Run All Cells**.

The notebook loads the source data, creates the DuckDB tables, trains
the models, and reproduces the strategy comparison.

### AI decision assistant

1. Install and start Ollama.
2. Download the model from a terminal:

   ```bash
   ollama pull qwen3:4b-instruct-2507-q4_K_M
   ```

3. Install the Python packages used by the assistant:

   ```bash
   pip install pandas ollama ipython jupyter
   ```

4. Confirm that `response_summary.csv` and `uplift_summary.csv` are
   in the `campaign_assistant_data` folder.
5. Open `campaign_targeting_demo.ipynb` and run the setup and function
   cells in order.
6. Run the live question cell and enter a question, such as:

   ```text
   Compare targeting at 30% and 60%. Recommend a strategy at each depth.
   ```

The assistant displays its answer and the comparison results supporting
it. Each question is handled independently and saved to
`assistant_demo_runs.jsonl`.