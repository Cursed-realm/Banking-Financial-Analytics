# Banking & Financial Analytics — Loan Default Risk Analysis (2007-2011)

An end-to-end data analytics project on LendingClub's historical loan data, using Excel, SQL, exploratory data analysis, statistical hypothesis testing, and an interactive Power BI dashboard.

## Objective

To analyze historical loan-level data from a real peer-to-peer lender (2007-2011) and answer:

- Which customer and loan characteristics are associated with higher default risk?
- Is there statistically significant evidence that credit history predicts default?
- Which customer segments should receive additional scrutiny, and which are relatively low-risk?

## Dataset

- **Source:** LendingClub historical loan data, via Kaggle
- **Size:** 39,717 rows, trimmed to 26 relevant columns; 38,577 rows after removing unresolved ("Current") loans
- **Key columns:** loan_amnt, term_months, int_rate, grade, annual_inc, emp_length_years, dti, purpose, issue_d, loan_status

## Tools & Libraries

Excel · Python · Pandas · NumPy · Matplotlib · Seaborn · SQL (SQLite) · Power BI · Power Query · DAX

## Methodology

**1. Data Cleaning**
- Formatted raw fields in Excel — converted loan term, employment length, and interest rate from inconsistent text formats into clean numeric columns
- Fixed two real date-parsing bugs in Python: `earliest_cr_line` had 11,599 rows misread with a 2026 year instead of 1926-2000 due to 2-digit-year misinterpretation; `issue_d` had its year value silently shifted into the day field by the same root cause, corrected by reconstructing the true date from the underlying components and verifying the result against LendingClub's known 2007-2011 growth trend
- Removed 1,140 unresolved ("Current") loans, keeping only Fully Paid and Charged Off loans for default analysis
- Handled missing values across five columns using context-appropriate methods: median imputation for skewed numeric fields (emp_length_years, revol_util), zero-fill for a count field (pub_rec_bankruptcies, justified since ~96% of known values were already zero), and explicit labels for text fields (emp_title, title)

**2. SQL Analysis**
- Loaded the cleaned dataset into a SQLite database and used CASE statements, CTEs, and window functions to compute default rate across credit grade, loan purpose, income bracket, employment length, loan amount, and loan term
- Found default rate rises monotonically from 5.99% (Grade A) to 33.78% (Grade G), and used RANK() to rank loan purposes by risk
- Built a monthly loan-volume trend with a running total using a CTE and window function, spanning all 55 months from 2007-06 to 2011-12

**3. Statistical Analysis**
- Computed mean, median, and standard deviation for loan_amnt, int_rate, annual_inc, and dti — found loan_amnt and annual_inc are right-skewed, while int_rate and dti are approximately symmetric
- Identified annual income outliers using the IQR method (1,762 borrowers, 4.57% of the dataset) and confirmed visually with a distribution plot — kept in the dataset as genuine high earners rather than data errors
- Calculated the correlation between credit grade and default (r = 0.20) and ran a manual two-sample hypothesis test (NumPy, no SciPy) comparing good-credit (Grade A-C) and poor-credit (Grade D-G) borrowers: **default rate 11.42% vs 24.97%, t = -27.56** — a highly significant result

**4. Dashboard**
An interactive Power BI dashboard with four visuals: Default Rate by Credit Grade, Default Rate by Loan Term, Default Rate by Income Bracket, and Default Rate by Loan Purpose.

## Key Findings

1. Credit grade is a strong, statistically significant predictor of default — risk rises steadily from Grade A (5.99%) to Grade G (33.78%), confirmed by a two-sample hypothesis test (t = -27.56)
2. Loan term matters more than loan amount alone — 60-month loans default more than twice as often as 36-month loans (25.31% vs 11.09%)
3. Lower-income borrowers default nearly twice as often as very-high-income borrowers (18.04% vs 11.11%)
4. Small business loans are the highest-risk loan category (27.08% default), compounded by also carrying a higher average loan amount than most other purposes
5. Employment length is a weak standalone predictor of default, varying only ~1.4 percentage points across brackets — worth flagging as a candidate for further investigation rather than treating as a reliable risk signal
