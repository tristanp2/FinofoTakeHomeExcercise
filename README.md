# Welcome

Hello Finofo,
Thanks for providing this take-home exercise! I've done my best based on the provided info.

### Observations/Assumptions

From the description provided, it seems that the main goal for this tool is to parse the formulas used for the projection values into a structure that allows the formula parameters to be easily tweaked. 
You mention that the sheet format can be quite varied, though without seeing any other examples of input sheets, I'm going to assume that other sheets will follow the same overall structure, and it's mostly just the fields and their relationships that will vary. That is, I'm moving forward with the assumption that the provided sheet will contain a table with the fields defined on the left hand side, and where each column after that defines historical/projection data for a given month corresponding to the defined fields.
From the provided sheet, I can see that there the historical data is the base values for the projection calculations. A projection's values are calculated by formulas in which the parameters can be other values from the same projection, values from a previous projection/historical data, or constants. "Cost of Goods Sold" also contains a condition, so we should be able to parse those as well.
For example, product sales are calculated using this simple formula: (previous column's product sales value) * 1.04. We want to ensure that when parsing, we correctly identify this as a formula with a reference to the previous month's product sales value, multiplied by a constant. 

From this, I believe that a good structure in which to store this data is as follows:
- The parsed set of fields that can contain either historical data, or projection calculations
- The set of historical data corresponding to these fields. These should be easy to reference in calculations
- The set of projection calculations corresponding to these fields. How exactly to store these will depend on what exactly we can expect these calculations to look like. I've made some assumptions based on the provided sheet. For example, we can convert projections down to single formulas where possible. All of the projection formulas have the same parameters across all months, with the exception of marketing budget which doubles as of Feb 2025

### Algorithm Overview

1. Determine sheet boundaries (i.e limit where to search through the sheet)
2. Find fields
    1. Step down sheet starting from top-leftmost sheet boundary.
    2. Each non-empty cell defines a budget data field. Add each to list of defined fields, noting corresponding row number.
    3. Stop when bottom leftmost boundary is passed
3. Determine where historical data ends and projection data starts. This could be done by either of the following methods:
    - Based on month headers. Historical data will be any column with month previous to the current month. Projection data is any column from current month or after.
    - Based on labels in sheet. Depending on what we can expect for input sheet format, we could search for the Historical and Projection labels in the first occupied row of the sheet and note their locations
4. Gather historical data
    1. Start from first historical month header. Use month header as identifier for this set of historical data
    2. Gather each cell value from the current column, associating each value with the corresponding field row number.
    3. Store gathered values as data object for this month. Move to next month header
    4. Stop if month header is next month or greater
5. Parse and store projection formulas
    1. Start at first cell under first projection month header
    2. For each row corresponding to a field, categorize and store the row's calculation as one of the following:
        - Repeated formula dependent on previous month's values. These formulas can be converted into a form that allows calculation based only on historical values and months out. For example, since the product sale formula is the same for every projection month, it can be simplified to (historical product sales) * (1.04)^n, where n is the number of months out
        - Repeated formula dependent only on current month's values. For example, total sales is just the sum of product and service sales of the current month
        - Formulas that differ throughout the row. For example, projected marketing budget is equal to the July 2024 marketing budget until Feb 2025, where it changes to double the July 2024 marketing budget. These should be stored by projection month
        - Hard coded value? This could be stored per month if the value differs, or just for the field row if the value is repeated
    3. Determine tweakable parameters of calculations. This is essentially just finding the constants that are operated upon in each calculation, and storing them as variables that can be changed as desired. This would probably be done as calculation objects are created

#### Accessing data after parsing

After parsing the sheet using the above algorithm, retrieving the projection value for field F on month M would happen as follows:
    - If calculation for field F is dependent on historical data and months out, retrieve historical data, calculate months out to month M, and return calculated value
    - If calculation for field F is dependent only on other values for month M, calculate their values first, then return calculated value
    - If calculation for field F differs depending on month, look up the specific calculation used for month M, then return calculated value

### Comments

The algorithm above is what I came up with based on the relatively limited information provided. My approach would likely change with more information regarding the exact intended use of the data after parsing, the input formats that are expected to be parsed, and the nature of the formulas contained in the input sheets. For example, I made the assumption that projection calculations for things like "Product Sales" will use calculations solely based on historical data, as is the case in the provided sheet. If you wanted to allow users to change the calculation in any specific month, it would make more sense to store each projection calculation as it is presented in the sheet. I.e The "Product Sales" calculation for Sept 2024 would just be a calculation with a reference to the previous month's calculation .




