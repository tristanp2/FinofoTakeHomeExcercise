# Welcome

Hello Finofo,
Thanks for providing this take-home exercise! I have quite a bit of professional experience working with excel, having made a few excel add-ins in the last few years. This task differs quite a bit from my previous work with excel, so I've done my best based on the provided info. 

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
4. Parse table cell contents. For each cell in the table:
    1. Generate parse tree for formula in cell if it does not already exist
    2. Find or generate parse trees for all cell references in tree until only cell references to literal values exist
    3. Store parse tree for easy reference i.e in dictionaries with cell name as key, month as key, and each dependency as key
5. Simplify trees if desired, as some trees can be easily collapsed. For example, the tree generated for a month's projected cost is just a cascade of multiplications by 1.04. Therefore it can be simplified to (historical month cost) * (1.04)^n, where n is the tree height, or the months out.

#### Accessing data after parsing

After parsing the sheet using the above algorithm, we have parse trees generated for all occupied cells in the table. Storing these formulas in this way makes it easy to track base dependencies and calculate as needed. For example, calculating the projected product sales for a given month would be as simple as finding the parse tree by month and then traversing the parse tree. If a value like historical product sales is changed, we could look up all parse trees that depend on that value and recalculate.

### Comments

After giving this problem some more consideration, I've changed my approach to how I would parse and track formulas. I think using a general purpose Excel formula parser is the most robust approach, as it allows us to structure the sheet contents as parse trees, which makes it much easier to track dependencies as well as recalculate formulas. Using this approach also makes it easier to identify and simplify formulas that involve repeated operations. 
