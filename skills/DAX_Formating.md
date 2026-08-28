---
name: DAX Formating
description: Format calculated fields in DAX language following sqlbi community rules.
---

At the begining of each session, and every time your receive instructions like "create a measure", "create a metric", "create a calculated column" in DAX, apply the following rules to standarize implementation:

Reference: https://www.sqlbi.com/articles/rules-for-dax-code-formatting/

Below is an example of a properly formatted DAX expression:

```DAX
CALCULATE (
    SUMX (
        Orders,
        Orders[Amount]
    ),
    FILTER (
        ALL ( Customers ),
        CALCULATE (
            COUNTROWS ( Sales ),
            ALL ( Calendar[Date] )
        ) > 42 + 8 – 25 * ( 3 - 1 )
            + 2 – 1 + 2 – 1
            + CALCULATE (
                  2 + 2 – 2
                  + 2 - 2
              )
            – CALCULATE ( 4 )
    )
)
```

The following is the set of rules:

* Never use shortened CALCULATE syntax
	* It means don’t use [measure](filter) but CALCULATE( [measure], filter ) instead – sorry Marius)
* Always put a space before parenthesis ‘(‘ and ‘)’
* Always put a space before any operand and operator in an expression
* If an expression has to be split in more rows, the operator is the first character in a new line
* A function call in an expression splitted in more rows has to be always in a new row, preceded by an operator
*Never put a space between table name and column name
* Only use single quotes for table name if it is required
	* So omit single quotes if table name has no spaces
* Never use table names for measures
* Always use table names for column reference
	* Even when you define a calculated column within a table
* Always put a space before an argument, if it is in the same line
* Write a function inline only if it has a single argument that is not a function call
* Always put arguments on a new line if the function call has 2 or more arguments
* If the function is written on more lines:
	* The opening parenthesis ‘(‘ is on the same line of the function call
	* The arguments are in new lines, indented 4 spaces from the beginning of the function call
	* The closing parenthesis is aligned with the beginning of the function call
	* The comma separating two arguments is on the same line of the previous argument (no spaces before)
* Definition of calculated column/measure is in the row before, including the assignment
	* Use ‘=’ to define a calculated column
	* Use ‘:=’ to define a measure
Based on the last rule, calculated columns and measures are defined as follows:

```DAX
calculated_column =
CALCULATE ( … )
  
measure :=
CALCULATE ( … )
```

# Naming conventions

Always use "snake_case" for measures and calculated fields.

Where possible, also include the measure's definition directly here as a comment (`///`, `/* */`, or `---`, depending on the file/language extension), not only in the field description.

> Warning
> Keep in mind that, as a best practice, calculated columns should ideally be created in Power Query (M language) instead.
> So avoid creating them in DAX before evaluating the performance impact and confirming which language is the best fit for that column.
