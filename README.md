# Loan Portfolio Risk Report

### Dashboard Link : *https://app.fabric.microsoft.com/links/fq0GmsRLEt?ctid=c2f90710-3741-48e7-b58c-40fb84092b24&pbi_source=linkShare&bookmarkGuid=78c5d627-7506-4f77-a9ae-ddf3a7470518*

## Problem Statement

Retail banks and non-banking financial companies (NBFCs) face growing pressure to monitor loan portfolio health in real time. Rising default rates, uneven credit score distribution, and inconsistent income profiles across employment types make it difficult for risk teams to identify vulnerable segments before they become non-performing assets (NPAs).

This report provides an end-to-end analytical view of a retail loan portfolio — tracking default rates by employment type and year, understanding how credit scores and demographics drive loan behaviour, and surfacing high-risk accounts that require immediate attention. The goal is to enable credit risk analysts and portfolio managers to move from reactive NPA management to proactive risk intervention.

Since the overall default rate sits above the 10% industry benchmark, the dashboard helps stakeholders identify which loan purposes, employment categories, and income brackets are driving defaults — and quantify the scale of high-risk exposure before it affects the balance sheet.

---

## Steps Followed

- **Step 1** : Loaded the `Loan_default` dataset (CSV format) into Power BI Desktop. The dataset contains applicant demographics, loan details, credit scores, employment information, and default status for a synthetic Indian retail banking portfolio.

- **Step 2** : Opened Power Query Editor. Under the View tab, enabled **Column Distribution**, **Column Quality**, and **Column Profile** to inspect data health across all columns.

- **Step 3** : Changed column profiling from the default 1,000-row sample to **entire dataset** to ensure accurate quality checks across all records.

- **Step 4** : Inspected key columns for data issues — standardized casing in `EmploymentType`, `LoanPurpose`, and `Education` fields using Power Query transformations. Verified that the `Default` column stores consistent binary values (1 = defaulted, 0 = healthy).

- **Step 5** : Created the following **calculated columns** directly in the `Loan_default` table:

  **Age groups** — segments applicants into lifecycle bands for demographic analysis:
  ```DAX
  Age groups =
  IF(Loan_default[Age] <= 25, "18-25",
  IF(Loan_default[Age] <= 35, "26-35",
  IF(Loan_default[Age] <= 50, "36-50",
  "50+")))
  ```

  **Income Bracket** — categorizes applicants by annual income:
  ```DAX
  Income Bracket =
  IF(Loan_default[Income] < 300000, "Low (<3L)",
  IF(Loan_default[Income] < 700000, "Mid (3L-7L)",
  IF(Loan_default[Income] < 1200000, "Upper-Mid (7L-12L)",
  "High (12L+)")))
  ```

  **Credit Score Bins** — groups credit scores into standard risk bands:
  ```DAX
  Credit Score Bins =
  IF(Loan_default[CreditScore] < 580, "Poor (300-579)",
  IF(Loan_default[CreditScore] < 670, "Fair (580-669)",
  IF(Loan_default[CreditScore] < 740, "Good (670-739)",
  IF(Loan_default[CreditScore] < 800, "Very Good (740-799)",
  "Exceptional (800+)"))))
  ```

  **Year** — extracted from loan issue date for time-series analysis:
  ```DAX
  year = YEAR(Loan_default[Date])
  ```

- **Step 6** : Created four dedicated **Measures Tables** to keep DAX measures organized by report page — `Measures table1`, `Measures table 2`, `Measures Table 3`, and `Measure Table 4`. This follows best practice for report maintainability and prevents measure clutter inside the data table.

- **Step 7** : Created the following DAX measures across the four measures tables —

  **Measures table1 — Loan Overview page:**

  ```DAX
  Loan amount by purpose =
  CALCULATE(SUM(Loan_default[LoanAmount]))

  Average income by employee type =
  AVERAGEX(
      FILTER(Loan_default, NOT(ISBLANK(Loan_default[Income]))),
      Loan_default[Income]
  )

  Default rate by employment type =
  DIVIDE(
      COUNTROWS(FILTER(Loan_default, Loan_default[Default] = 1)),
      COUNTROWS(Loan_default),
      0
  )

  Average Loan By Age groups =
  AVERAGEX(
      FILTER(Loan_default, NOT(ISBLANK(Loan_default[LoanAmount]))),
      Loan_default[LoanAmount]
  )

  Default rate by year =
  DIVIDE(
      COUNTROWS(FILTER(Loan_default, Loan_default[Default] = 1)),
      COUNTROWS(Loan_default),
      0
  )
  ```

  **Measures table 2 — Demographics page:**

  ```DAX
  Median By Credit Score Bins =
  MEDIANX(Loan_default, Loan_default[LoanAmount])

  Average Loan Amount(High Credit) =
  CALCULATE(
      AVERAGE(Loan_default[LoanAmount]),
      Loan_default[CreditScore] >= 740
  )

  Total loans(credit bins) =
  COUNTROWS(Loan_default)

  Loans By education type =
  COUNTROWS(Loan_default)
  ```

  **Measures Table 3 — Financial Risk page:**

  ```DAX
  YOY Loan Amount change =
  VAR CurrentYear = SUM(Loan_default[LoanAmount])
  VAR PreviousYear =
      CALCULATE(
          SUM(Loan_default[LoanAmount]),
          SAMEPERIODLASTYEAR(Loan_default[Date])
      )
  RETURN
  DIVIDE(CurrentYear - PreviousYear, PreviousYear, 0)
   YOY default loans change =
   DIVIDE(
  CALCULATE(COUNTROWS(FILTER('Loan_default','Loan_default'[Default]=true())),'Loan_default'[year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))) - 
  CALCULATE(COUNTROWS(FILTER('Loan_default','Loan_default'[Default]=true())),'Loan_default'[year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))-1)
   ,CALCULATE(COUNTROWS(FILTER('Loan_default','Loan_default'[Default]=true())),'Loan_default'[year]=YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))-1),0)*100 



  YTD Loan Amount =
  CALCULATE(
      SUM(Loan_default[LoanAmount]),
      DATESYTD(Loan_default[Date])
  )
  ```

  **Measure Table 4 — Executive Summary page:**

  ```DAX
  Total Loans Issued = COUNTROWS('Loan_default')

  Total Loan Amount = SUM(Loan_default[LoanAmount])

  Average Loan Amount =
  Average Loan Amount = AVERAGE('Loan_default'[LoanAmount])

  Average Credit Score =
  AVERAGEX(
      FILTER(Loan_default, NOT(ISBLANK(Loan_default[CreditScore]))),
      Loan_default[CreditScore]
  )

  Overall Default Rate =
  DIVIDE(
      COUNTROWS(FILTER(Loan_default, Loan_default[Default] = 1)),
      COUNTROWS(Loan_default),
      0
  )*100

  High Risk Accounts =
  COUNTROWS(
      FILTER(
          'Loan_default',
          'Loan_default'[CreditScore] < 580 &&
          'Loan_default'[Default] = true()
      )
  )

  Default Rate YoY Label =
  VAR CurrentRate = [Overall Default Rate]
  VAR LastYearRate =
      CALCULATE(
          [Overall Default Rate],
          SAMEPERIODLASTYEAR(Loan_default[Date])
      )
  VAR Change = CurrentRate - LastYearRate
  RETURN
      IF(
          Change > 0,
          "▲ " & FORMAT(ABS(Change), "0.0%") & " vs last year",
          "▼ " & FORMAT(ABS(Change), "0.0%") & " vs last year"
      )
  ```

- **Step 8** : Applied **conditional formatting** on the Overall Default Rate card — value turns red above 20%, amber between 10–20%, and green below 10% — using Power BI's `fx` rules-based color formatting on the callout value.

- **Step 9** : Built the **Executive Summary** page (Page 4) as the landing view with 6 KPI cards:
  - Total Loans Issued
  - Total Loan Amount
  - Average Loan Amount
  - Average Credit Score
  - Overall Default Rate
  - High Risk Accounts
  - Default Rate YoY Label (trend indicator)

  Card backgrounds set to transparent to sit cleanly against the dark canvas. Colored accent rectangles (4px wide) placed behind each card to indicate status — blue for volume metrics, red for risk metrics, green and amber for health indicators.

- **Step 10** : Built **Page 1 — Loan Default and Overview** with:
  - Loan Amount by Purpose (line chart)
  - Average Income by Employment Type (line chart)
  - Default Rate by Employment Type % (line chart)
  - Average Loan Amount by Age Group (line chart)
  - Default Rate by Year (line chart)

- **Step 11** : Built **Page 2 — Applicant Demographics and Financial Profile** with:
  - Median Loan Amount by Credit Score Category (line chart)
  - Average Loan Amount by Marital Status and Age Group (donut chart)
  - Total Loans by Credit Score Bins (line chart)
  - Loan — Middle-Age Adults by Mortgage/Dependence (clustered column chart)
  - Number of Loans by Education Type (line chart)

- **Step 12** : Built **Page 3 — Financial Risk Metrics** with:
  - YoY Loan Amount Change by Year (line chart)
  - YoY Default Loans Change by Year (line chart)
  - YTD Loan Amount by Credit Score Bins and Marital Status (ribbon chart)
  - Loan Amount by Income Bracket and Employment Type (decomposition tree)

- **Step 13** : Applied a consistent dark theme (`#1B2A3B` canvas background) across all 4 pages with a header rectangle shape and page title text boxes for uniform navigation context.

- **Step 14** : Report published to Power BI Service.

---

## Snapshot of Dashboard (Power BI Service)

<img width="1348" height="788" alt="Screenshot 2026-07-06 213639" src="https://github.com/user-attachments/assets/a2f59197-18ba-4687-ad74-cb87bbb9c6dc" />


---

## Report Snapshot (Power BI Desktop)



**Page 1 — Loan Default and Overview**

<img width="1304" height="719" alt="Screenshot 2026-07-06 213821" src="https://github.com/user-attachments/assets/23e138e3-848e-48f2-b35e-052fe8a36a7d" />


**Page 2 — Applicant Demographics and Financial Profile**

<img width="1296" height="730" alt="Screenshot 2026-07-06 213846" src="https://github.com/user-attachments/assets/503c7be5-4303-4852-9bd7-05e4e44f8c04" />


**Page 3 — Financial Risk Metrics**

<img width="1303" height="720" alt="Screenshot 2026-07-06 213910" src="https://github.com/user-attachments/assets/9287a253-930c-476a-afd1-85dee7f3ea91" />


**Page 4 — Executive Summary**

<img width="1300" height="716" alt="Screenshot 2026-07-06 213933" src="https://github.com/user-attachments/assets/522d2f1a-1800-48de-9972-9a282e6e9907" />


---

## Insights

A four-page interactive report was created on Power BI Desktop and published to Power BI Service. The following inferences can be drawn from the dashboard:

### [1] Portfolio Scale

- The portfolio spans thousands of loan accounts across multiple purposes — Home, Auto, Education, Business, and Personal — with total disbursed loan amount giving a clear picture of overall credit exposure.
- Average loan amount varies significantly by age group, with middle-age adults (36–50) typically carrying higher loan values, consistent with peak earning and asset acquisition years in India.

### [2] Default Rate Patterns

- Default rate by employment type reveals that unemployed and self-employed applicants carry disproportionately higher default rates compared to salaried employees — a direct signal for tighter underwriting criteria in these segments.
- Default rate by year shows whether the portfolio's risk profile is improving or deteriorating over time — a rising trend indicates systemic issues beyond individual borrower risk.
- YoY default loans change on the Risk Metrics page quantifies whether the absolute number of defaults is growing faster than the portfolio itself — a ratio that matters more than the raw default count alone.

### [3] Credit Score Distribution

- Median loan amount by credit score category shows a clear positive relationship — applicants with higher credit scores receive larger loans, validating the bank's credit underwriting process.
- Total loans by credit score bins reveals what proportion of the portfolio sits in the Poor (<580) and Fair (580–669) bands — a high concentration here signals elevated portfolio-level risk regardless of current default rates.
- The Average Credit Score KPI on the Executive Summary page, when below 580, flags the entire portfolio as operating in a high-risk credit band on average.

### [4] Demographic Risk Segments

- Middle-age adults with both a mortgage and dependents show a distinct loan behaviour — higher loan amounts combined with higher financial obligations, making this the segment most vulnerable to income disruption.
- Education type influences loan uptake significantly — graduate and postgraduate applicants tend to access higher loan amounts, likely reflecting higher income potential and better creditworthiness scores.
- Marital status combined with age group in the donut chart surfaces whether joint-income households behave differently from single applicants in terms of average loan size.

### [5] Income and Employment Risk

- The decomposition tree on the Financial Risk page breaks total loan amount by Income Bracket first, then by Employment Type — revealing which income-employment combinations drive the largest credit exposure and which are most associated with default.
- High-income self-employed applicants may show surprisingly elevated default rates compared to mid-income salaried employees — a counterintuitive finding that has direct implications for risk-based pricing.

### [6] High Risk Accounts

- High Risk Accounts (credit score below 580 and defaulted) represents the most immediately actionable segment — these accounts have already defaulted and had poor credit at origination, raising questions about the bank's approval criteria at the time of disbursement.
- Tracking this KPI over time on the Executive Summary page helps risk teams measure whether improved underwriting standards are reducing the flow of high-risk approvals into the portfolio.
