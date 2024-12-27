# Data Preparation and Cleaning

## Add Category Column
This Power Query code snippet categorizes Revolut transactions based on their descriptions. It uses nested `if` and `else if` statements to check for specific keywords or transaction types within the "Description" column.

For example:
- If the "Type" is `"ATM"`, the category is `"ATM"`.
- If the "Type" is `"TRANSFER"` and the description contains `"From"`, it is categorized as `"Transfer From Friends"`.
- Other conditions categorize transactions into groups like `"Food & Dining"`, `"Retail"`, `"Transport"`, and more.

```powerquery
= Table.AddColumn(#"Changed Type", "Category", each 
    if [Type] = "ATM" then "ATM" 
    else if [Type] = "TRANSFER" and Text.Contains([Description], "From") then "Transfer From Friends"
    else if [Type] = "TRANSFER" and Text.Contains([Description], "To") then "Money To Friends"
    else if [Type] = "TOPUP" or Text.Contains([Description], "Top-Up") or Text.Contains([Description], "Vodafone") 
        or Text.Contains([Description], "Balance migration") or Text.Contains([Description], "GoMo") or Text.Contains([Description], "Rebtel") 
        then "Transfers & Top-Ups"
    else if [Type] = "FEE" or Text.Contains([Description], "Irish Stamp Duty") or Text.Contains([Description], "FX") 
        or Text.Contains([Description], "Exchanged") then "Finance & Banking"
    else if [Type] = "REFUND" or [Type] = "TEMP_BLOCK" then "Refund and Temp Block"
    else if [Type] = "CARD_PAYMENT" and (Text.Contains([Description], "gym") or Text.Contains([Description], "protein")) then "Health and Fitness"
    else if [Type] = "CARD_PAYMENT" and (Text.Contains([Description], "SuperValu") or Text.Contains([Description], "Daybreak") or Text.Contains([Description], "Spar")
        or Text.Contains([Description], "Centra") or Text.Contains([Description], "Dealz")) then "Retail"
    else if [Type] = "CARD_PAYMENT" and (Text.Contains([Description], "food") or Text.Contains([Description], "Burger King") 
        or Text.Contains([Description], "McDonalds")) then "Food & Dining"
    else "Other")
```


## Fill Down Missing Balance Columns

This Power Query code snippet uses the `Table.FillDown` function to handle missing values in the `"Balance"` column of my dataset.

If there are any gaps in the `"Balance"` column where values are missing, this code will fill those gaps by copying the last available balance value down the column until it encounters another non-blank balance value.

```powerquery
= Table.FillDown(#"Added Custom",{"Balance"})
```

## Add Income/Expense Column

This Power Query code snippet creates a new column named `"Income/Expense"` to classify each transaction as either `"Income"` or `"Expense"`.

It checks the value in the `"Amount"` column of each row:

- If the `"Amount"` is greater than 0 (positive), it assigns the value `"Income"` to the new `"Income/Expense"` column for that row.
- If the `"Amount"` is less than 0 (negative), it assigns the value `"Expense"` to the `"Income/Expense"` column for that row.

```powerquery
= Table.AddColumn(#"Filled Down", "Income/Expense", each if([Amount]) > 0 then "Income" else "Expense")
```

## Data Modeling

Once the data was satisfactorily cleaned, the datasets were designed using the principles of fact and dimension data modeling concepts.

### Dimension Tables:
1. **Dim Date:**
   - Contains date-related attributes such as Date, Day of Week, Month, Quarter, and Year.
   - Enables time-series analysis and granular date-level filtering.

2. **Dim Type:**
   - Stores information about transaction types and allows for analysis based on transaction categories.
   - Columns include:
     - **Type:** Details whether the transaction was an ATM, Transfer, or Card Payment.
     - **Category:** Adds more details, such as Food & Dining, Health & Wellness, Retail, and Entertainment.
     - **Description:** Provides merchant-specific details, e.g., Lidl, Tesco, ATM Used, or Transfer Recipient.

3. **Dim Currency:**
   - Contains information about the currencies involved in transactions.
   - Enables the analysis of transactions in different currencies.

4. **Dim Product:**
   - Holds information about the Revolut products used, e.g., Current Account.
   - Enables the analysis of transactions across different Revolut products.

5. **Dim State:**
   - Represents the state of the transaction (e.g., Completed, Reverted).
   - Enables the analysis of the state of different transactions.

---

### Fact Table:
**Fact RevTX:**
   - The central table holding core transaction data. Key columns include:
     - **DateID:** Foreign key referencing the Dim Date table.
     - **TypeID:** Foreign key referencing the Dim Type table.
     - **CurrencyID:** Foreign key referencing the Dim Currency table.
     - **ProductID:** Foreign key referencing the Dim Product table.
     - **StateID:** Foreign key referencing the Dim State table.
     - **Amount:** Transaction amount.
     - **Balance:** Account balance after the transaction.
     - **Income/Expense:** Indicates whether the transaction is income or expense.

---

### Relationships:
- The **Fact RevTX** table has many-to-one relationships with all the dimension tables.
- These relationships enable flexible analysis by combining data from different dimensions.

![Data Model](images/Data%20Model.png)

## Data Dimension

### Creating the Date Dimension Table

A foundational element of the data model is the **Date Dimension** table. This table is a crucial reference for date-level analysis.

---

### **Implementation**

1. **Custom Date Table Creation:**
   - Leveraged Power Query to dynamically generate a comprehensive Date Dimension table.
   - Created a sequence of dates and calculated attributes such as **Day of Week**, **Month**, **Quarter**, and **Year**.

2. **Incorporating Best Practices:**
   - To ensure the Date Dimension table adheres to industry standards and facilitates robust analysis, I referred to the following resources:
     - **Gina M. Gonek's "Power BI Date/Time Dimension Toolkit":**
       - [Link](https://ginameronek.com/2014/10/01/its-just-a-matter-of-time-power-bi-date-time-dimension-toolkit)
       - This resource provided valuable guidance on best practices for creating and utilizing Date Dimension tables within the Power BI environment.
     - **Devin Knight's "Creating a Date Dimension with Power Query":**
       - [Link](https://devinknightsql.com/2015/06/16/creating-a-date-dimension-with-power-query/)
       - This guide offered practical insights and code examples for implementing a Date Dimension table effectively within the Power Query editor.

---

### **Key Features of the Date Dimension Table**

1. **Dynamic Date Range:**
   - The table is designed to dynamically adjust to accommodate evolving analysis needs.
   - Ensures comprehensive coverage across the desired time period.

2. **Hierarchical Attributes:**
   - Includes attributes like **Year**, **Quarter**, and **Month**.
   - Enables flexible hierarchical drilldown within visualizations.
   - Provides a nuanced understanding of time-based trends.
