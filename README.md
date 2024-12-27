# My Revolut Data Story

## Executive Summary
This project analyzed my Revolut transaction data to uncover spending habits and financial patterns. Using Power BI, I transformed raw account statement data into a structured data model and built interactive visualizations to explore key metrics.

### Key Findings:
- **Income and Expenses Alignment**: My expenses align with my income, as I typically use my Revolut account for shopping transactions to avoid losing my bank ATM card.
- **2021 Spending Spike**: Expenses spiked in 2021, coinciding with my transition to full-time work after graduation.
- **Weekend Spending Trends**: Weekend spending is notably higher, particularly on groceries, with Lidl, Tesco, and SPAR among my top vendors.

This analysis highlights opportunities for improved financial planning and better budgeting decisions.

---

## Dataset Used
This project uses my Revolut account statement from March 2020 to December 2024.

## Technologies
- **Power BI**: Used for data cleaning, modeling, and visualization.

---

## Data Cleaning & Preparation
The dataset was cleaned and transformed in Power BI before being organized into fact and dimension tables.

### Specific Cleaning and Transformation Tasks:
1. **Missing Balance Values**:
   - Populated empty cells with the last known balance using the column's Fill function.
2. **Income or Expense Classification**:
   - Created a new column to classify transactions:
     - **Expense**: If `Amount < 0`
     - **Income**: If `Amount > 0`
3. **Transaction Categorization**:
   - Developed custom logic to create a "Category" column by analyzing the “Type” and “Description” columns. For example:
     - If the description contained "Gym," it was categorized under **Health and Fitness**.

More details on the data preparation, cleaning, and modeling can be found [here](DataPreparationCleaning.md).

---

## Data Modeling
Once cleaned, the dataset was structured using the principles of fact and dimension data modeling.

### Dimension Tables:
- **Dim Date**: Contains date-related attributes (e.g., Date, Day of Week, Month, Quarter, Year) for time-series analysis.
- **Dim Type**: Details transaction types and categories:
  - Columns: `Type` (e.g., ATM, Transfer, Card Payment), `Category` (e.g., Food & Dining, Health & Wellness), `Description` (e.g., merchant names like Lidl, Tesco, etc.).
- **Dim Currency**: Stores information about transaction currencies.
- **Dim Product**: Contains details about Revolut products used (e.g., Current Account).
- **Dim State**: Indicates the state of a transaction (e.g., completed, reverted).

### Fact Table:
- **Fact RevTX**: The central transaction table containing:
  - `DateID`, `TypeID`, `CurrencyID`, `ProductID`, `StateID`: Foreign keys referencing dimension tables.
  - `Amount`: Transaction amount.
  - `Balance`: Account balance after the transaction.
  - `Income/Expense`: Indicates transaction type.

### Relationships:
- The `Fact RevTX` table has many-to-one relationships with all dimension tables, enabling flexible data analysis.

![Data Model](images/MyRevolutDataModel.png)

---

## Analysis and Visualization
I created several DAX measures to calculate key performance indicators (KPIs), including:

### DAX Measures:
**Total Income**:
```DAX
   Total Income =
   CALCULATE(
       SUM('Fact RevTX'[Amount]),
       'Fact RevTX'[Income/Expense] = "Income"
   )
```

**Total Expenses**:
```DAX
	Total Expenses =
	CALCULATE(
	    SUM('Fact RevTX'[Amount]),
	    'Fact RevTX'[Income/Expense] = "Expense"
	) * -1
```

**Net Balance**:
```DAX
	Net Balance = [Total Income] - [Total Expenses]
```

**Savings Rate**:
```DAX
	Savings Rate =
	DIVIDE(
	    [Net Balance],
	    [Total Income],
	    0
	)
```

**YoY Expense Growth**:
```DAX
	YoY Expense Growth =
	VAR CurrentYearExpense =
	    CALCULATE (
	        [Total Expenses],
	        FILTER (ALL ('Dim Date'), 'Dim Date'[Year] = MAX ('Dim Date'[Year]))
	    )
	VAR PreviousYearExpense =
	    CALCULATE (
	        [Total Expenses],
	        FILTER (ALL('Dim Date'), 'Dim Date'[Year] = MAX ('Dim Date'[Year]) - 1)
	    )
	RETURN
	    DIVIDE (CurrentYearExpense - PreviousYearExpense, PreviousYearExpense, 0)
```

### Insights from the Report
![Revolut Report](images/RevolutReport.png)

#### Income and Expenses Alignment:
- My total expenses align closely with my income, as Revolut is primarily used for daily transactions and purchases, reflected in a low savings rate of **0.16%**.

#### YoY Expense Growth in 2021:
- A sharp rise in expenses in 2021 coincides with my transition to full-time work and the easing of COVID-19 restrictions, reflecting a lifestyle shift.

#### Weekend Spending Patterns:
- Weekend spending dominates, with the **'Expenses by Day of the Week'** chart highlighting peak activity on **Saturdays and Thursdays**, driven largely by grocery shopping.

#### Top Expense Categories and Vendors:
- The **'Top Expenses'** chart highlights grocery spending at **Lidl, Tesco, and SPAR**, aligning with weekend trends, while **MYPROTEIN** reflects fitness-related spending in 2023.

---

### Conclusion and Next Steps
This report provides a clear snapshot of my spending habits, highlighting how my Revolut account is primarily used for everyday transactions and purchases. 

The analysis illustrates trends such as:
- Consistent alignment of income and expenses.
- Increased spending in 2021 due to life changes.
- Notable weekend spending patterns driven by grocery shopping and leisure activities.

While this account is not intended for saving, the project demonstrates how data analysis can offer meaningful insights into financial behavior.

Although I cannot share the interactive Power BI dashboard, this document encapsulates the key findings and methodologies applied. In the future, I may explore recreating a similar report with sample data or alternative tools.
