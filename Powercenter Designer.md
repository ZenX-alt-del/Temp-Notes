# Business Components
- Business Components are higher-level abstractions of data, not just raw tables or files.
- They allow you to define metadata objects, like dimensions, hierarchies, cubes, and measures, that reflect how business users analyze data.
- These components are stored in the repository and can be reused across mappings.
- They are mainly used for OLAP (Online Analytical Processing), Data Warehousing, and Business Intelligence integration.
- They let you design ETL flows that directly supports analytical reporting, not just raw table loads.

## Types Of Business Components
**Dimensions**
- Represent business perspectives (e.g., Time, Geography, Product).
- Contain hierarchies (Year → Quarter → Month → Day).

**Cubes**
- Logical structures for multidimensional analysis.
- Combine dimensions and measures for OLAP reporting.

**Measures**
- Numberic values that can be aggregated (e.g., Sales, Premium, Claims).
- Used in cubes to provide analytical metrics.

**Hierarchies**
- Define drill-down paths within dimensions.
- *Example,* Conutry -> State -> City.

# Mappings
- It is a set of source definitions, target definitions, and transformation objects linked together.
- It defines ETL logic - how data is extracted from sources, transformed according to business rules, and loaded into targets.
- When the integration service runs a session, it uses the instructions defined in the mappings to read, transform, and write data.

**Why Mappings Matter**
- They are the heart of ETL in PowerCenter.
- Allow developers to visually design complex data flows without coding.
- Ensure reusability and consistency across workflows.
- Can be grouped into mapplets (reusable sets of transformations).

## Components Of A Mapping
Every mapping must contain these core elements:

### 1. Source Definition
- Describes the structure of the source (tables, files, XML, etc.).
- *Example,* A sales CSV file with columns like `Date`, `Product`, `Revenue`.

### 2. Transformations
Modify or process the data before loading.
Common transformations include:
- **Filter:** Select rows based on conditions.
- **Joiner:** Combine data from multiple sources.
- **Aggregator:** Summarize data (e.g., total sales).
- **Lookup:** Fetch reference data from another table.
- **Expression:** Apply formulas or conversions.

### 3. Target Definition
- Defines the structure of the destination (warehouse tables, files, applications).
- *Example,* Oracle data warehouse table `Sales_Analytics`.

### 4. Links
- Connect sources, transformations, and targets to define the data path.

## Example Flow
```shell
[ Source: Sales_CSV ]
        |
        v
[ Transformation: Filter (only 2025 data) ]
        |
        v
[ Transformation: Expression (convert USD → INR) ]
        |
        v
[ Transformation: Aggregator (sum revenue by region) ]
        |
        v
[ Target: Oracle DW Table 'Sales_Analytics' ]
```

# Transformations
- They are the core logic blocks that are used inside a mapping to define how data should be processed before it reaches the target.
- They define how data is modified, filtered, aggregated, or enriched before being loaded into targets.
- There are classified into 2 types - active (can change row counts) and passive (do not change row counts).

The main transformations include:

| **Transformations** | **Type** | **Purpose** | **Example** |
| -- | -- | -- | -- |
| **Expression** | Passive | Performs row-level calculations | Convert currency, concatenate strings, apply formulas |
| **Flter** | Active | Filters rows based on a condition | Only pass records where `Sales` is greater than a certain threshold |
| **Aggregator** | Active | Performs aggregate calculations like `SUM`, `AVG`, `COUNT`, `MAX`, `MIN` | calcultae total sales per region |
| **Lookup** | Active / Passive | Looks up values from a reference table or file | Fetch customer detials from a master table |
| **Joiner** | Active | Joins data from 2 heterogeneous data sources | Combine customer data from Oracle with sales data from flat files
| **Sorter** | Active | Sorts data based on specified field | Sort orders by date before loading |
| **Update Strategy** | Active | Defines how rows should be treated in the target (`insert`, `update`, `delete`, `reject`) | Update existing customer records, insert new ones |
| **Sequence Generator** | Passive | Generates unique numberic values (like surrogate keys) | Create incremental IDs for new records |
| **Router** | Active | Routes data into multiple groups based on conditions | Separate high-value customers from low-value customers |
| **Rank** | Active | Selects top or bottom `N` records based on a measure | Find top 10 highest selling products |
| **Union** | Active | Combines data from multiple pipelines into one | Merge sales data from different regions |
| **Stored Procedure** | Passive / Active | Calls a database stored procedure | Executes a PL/SQL procedure for complex business logic |
| **Normalizer** | Active | Converts column-based data into row-based data | Pivot monthly sales columns into rows |
| **Transaction Control** | Active | Defines commmit/rollback points in a workflow | Commit data every 100 rows |
| **XML Transformation** | Active / Passive | Handles XML parsing and generation | Read XML customer records and load into relational tables |

## Expression Transformation
- A Passive transformation (does not change the number of rows).
- Used to perform row-level calculations and manipulate data values.
- You can write expressions using built-in functions, operators, and constants.

**Data Manipulation**
- Perform string concatenation - `FIRST_NAME || ' ' || LAST_NAME`.
- Substring extraction - `SUBSTR(PHONE, 1, 3)`.

**Mathematical Operations**
- Arithmetic (`SALARY * 0.10` for bonus).
- Rounding, truncation, absolute values.

**Date/Time Functions**
- Add/subtract days (`ADD_TO_DATE(HIRE_DATE,'DD',30)`).
- Extract year/month/day (`TO_CHAR(DOB,'YYYY')`).

**Conditional Logic**
- Use `IIF` (if-then-else logic)
- Example: `IIF(GENDER='M','Male','Female')`, it will return `Male` if `GENDER` is `M`, otherwise it will return `F`.

**Type Conversions**
- Convert between datatypes (`TO_INTEGER`, `TO_CHAR`, `TO_DATE`).

**Default Values**
- Handle nulls (`NVL(COMMISSION,0)`).
- This replaces a null value with the specifeid default value.

## Filter Transformation
- Used to filter out rows that do not meet a specified condition.
- Only rows that satisfy the condition are passed to the next stage; others are dropped.

**How it works**
- You define a filter condition (an expression that evaluates to `TRUE` or `FALSE`).
- For each incoming row, if the condition evaluates to `TRUE`, the row is passed downstream and if the condition evaluates to `FALSE`, the row is discarded.

**Syntax Examples**
```shell
SALARY > 5000
IIF(GENDER='M', TRUE, FALSE) (passes only male records)
COUNTRY IN ('India','UK','US')
```

## Router Transformation
- It can be thought of as a more advanced version of the Filter transformation.
- Unlike Filter transformation, where rows that don't satisfy the condition are dropped, Router transformation includes multiple condition and can route the data into different groups.
- It is used to split data into multiple groups based on conditions.
- Each group is defined by a group filter condition.

**How it works**
- You define multiple output groups inside the Router.
- Each group has a condition (similar to a Filter).
- Rows are evaluated against each condition.
- If a row meets a condition, it is routed to that group’s output.
- If no condition is met, the row can go to the Default group (optional).

**Differences Between Filter and Router Transformations**

| **Filter** | **Router** |
| -- | -- | 
| Only one condition per transformation | Multiple conditions in one transformation |
| Rows either pass or drop | Rows can be routed into different groups |
| No default output | Has a defult group for unmatched rows |

## Aggregator Transformation
- Performs aggregate functions such as `SUM`, `AVG`, `COUNT`, `MAX`, `MIN`, `FIRST`, `LAST`.
- Groups data based on group-by ports.

**How it works**
- *Input Rows*: Multiple rows flow into the transformation.
- *Group By Ports*: You select one or more columns to group data (like SQL `GROUP BY`).
- *Aggregate Functions*: Apply functions on non-grouped columns.
- *Output Rows*: One row per group with aggregated values.

**Difference from Expression Transformation**

| **Expression** | **Aggregator** |
| -- | -- |
| Row level calculations | Group-level calculations |
| One output row per input row | One output row per group |
| Passive (doesn't change row count) | Active (may reduce row count) |

## Joiner Transformation
- Allows you to join data from two sources that don’t necessarily come from the same database.
- Works much like SQL joins, but inside the ETL flow.
- You can join based on conditions you define (like matching keys).

**Join Types**
- *Normal Join:* Only matching rows are returned (like SQL `INNER JOIN`).
- *Master Outer Join:* All rows from Master, plus matching rows from Detail.
- *Detail Outer Join:* All rows from Detail, plus matching rows from Master.
- *Full Outer Join:* All rows from both sources, with null where no match exists.

**How it works**
- Two pipelines feed into the Joiner: a Master and a Detail source.
- You choose which one is Master and which is Detail (this affects performance and caching).
- Define join condition.
- Define join type and output ports.

### Source Types And Performance Considerations
- *Master Source:* The pipeline you designate as "Master".
- *Detail Source:* The pipeline you designate as "Detail".
- The Joiner builds a cache of the Master source rows and then compares Detail rows against it.

**Caching Behavior**
- Informatica caches all rows from the Master source.
- Each row from the Detail source is compared against this cache.
- Larger Master = larger cache = more memory usage and slower performance.

**Row Volume**
- If the Master has millions of rows, the cache becomes heavy.
- If the Detail has millions of rows, performance is still manageable because only Detail rows are streamed and compared against the cache.

**Join Condition**
- Indexed join keys in the Master source improve lookup speed.
- Poorly chosen Master/Detail roles can lead to excessive memory usage.

## Lookup Transformation
- Can be Passive (returns at most one row per input row) or Active (returns multiple rows if configured).
- Used to fetch related data from a lookup source and add it to the pipeline.
- Works like a SQL join, but is more flexible and optimized for ETL.

**How it works**
- Define lookup source. It can be a table, flat file, or cached data.
- Specify lookup condition. It defines how input rows match rows in the lookup table (like a join condition).
- Select return ports, if using unconnected lookup. These are the columns from the lookup source that should be returned to the calling transformation (e.g., expression, or update strategy transformations).
- Handle no match. If no match is found, you can configure default values or nulls.

**Difference from Joiner Transformation**

| **Lookup** | **Joiner** |
| -- | -- |
| Fetches data from a reference table/file | Joins 2 pipelines in a mapping |
| Can be passive or active | Always active |
| Supports caching for performance | No caching |
| Often used for enrichment | Often used for combining sources |

### Types Of Lookup Transformation
There are 2 types of lookup transformation:

**Connected Lookup**
- Directly connected to the mapping pipeline.
- Input ports are linked to upstream transformations, and output ports are linked downstream.
- Returns multiple columns (not just one value).
- Used when you need to bring in additional attributes from a reference table/file for every row.

**Unconnected Lookup**
- Not connected directly in the mapping flow.
- Called from an expression using the `LKP` function.
- Returns only one column value per call.
- Useful when you need a single value lookup occasionally, not for every row.

## Sorter Transformation
- An active transformation that sorts data based on one or more columns.
- Works like an `ORDER BY` clause in SQL.
- Can also eliminate duplicates if configured.
- Multiple columns can be used for sorting.
- Enabling `Distinct` option in the properties tab removes duplicate rows.
- Can also be configured to make sorting case sensitive or case-insensitive.
- You can specify whether null values appear first or last.

## Update Strategy Transformation
- It is used to define how rows should be treated when they reach the target - whether they should be inserted, updated, deleted, or rejected.
- An active transformation that applies row-level DML logic.
- You write an expression that evaluates to a numeric code, which tells Informatica what to do with each row.
- It provides row-level control where the action is decided for each row individually.
- It is used for error handling or skipping unwanted rows.
- You must consfigure the session to allow updates/deletes/inserts.

**How it works**
Inside transformation, you set the Update Strategy expression using constants. These constants are:
- `DD_INSERT(0)`: Insert the row into the target.
- `DD_UPDATE(1)`: Update the row in the target.
- `DD_DELETE(2)`: Delete the row from the target.
- `DD_REJECT(3)`: → Reject the row (it won’t be loaded).

Each incoming row is evaluated, and based on the expression, Informatica applies the appropriate action.

**Example Scenario**
Suppose you are loading insurance policies into a warehouse table:
- If the policy is new, insert it.
- If the policy already exists, update it.
- If the policy is marked inactive, delete it.

```shell
IIF(NEW_FLAG = 'Y', DD_INSERT,
    IIF(INACTIVE_FLAG = 'Y', DD_DELETE,
        DD_UPDATE))
```

## Sequence Generator Transformation
- It is used to create unique numeric values, often for primary keys or surrogate keys in target tables. 
- It’s a passive transformation (does not change the number of rows) but adds new values to each row.
- It can generate numbers in incremental order or with a cycle.

**How it works**
The Sequence Generator has two important output ports:
- `NEXTVAL`: Generates the next sequence number for each row.
- `CURRVAL`: Holds the current sequence number (useful if you need the same value across multiple columns in one row).

You configure:
- *Start Value*: The first number in the sequence.
- *Increment By*: The step size (e.g., 1, 10).
- *End Value*: The maximum number before cycling or stopping.
- *Cycle Option*: Whether the sequence restarts after reaching the end value.

## Rank Transformation
-An active transformation that assigns ranks to rows within a group.
- Lets you filter out rows based on Top N or Bottom N logic.
- Often used for reporting scenarios like “Top 10 Customers by Sales” or “Lowest 5 Policies by Premium.”

**How it works**
*Group By Ports*
- Define grouping keys (e.g., REGION, DEPARTMENT).
- Ranking is applied within each group.

*Rank Port*
- Select a numeric column (e.g., PREMIUM, SALES).
- Rows are ranked based on this port.

*Top/Bottom Option*
- Choose whether to return the Top N or Bottom N rows.

*Number of Ranks*
- Specify how many rows to return (e.g., Top 5, Bottom 3).

## Union Transformation
- Works like a SQL UNION operator.
- Combines data from multiple sources or branches of a mapping (data from multiple pipelines are merged into a single pipeline).
- Ensures that the output has a consistent structure (same number of columns and compatible datatypes).
- It is an active transformation.

**How it Works**
*Input Groups*
- You can connect multiple input pipelines to the Union transformation.
- Each input group must have the same number of ports and compatible datatypes.

*Output Group*
- The Union transformation merges all input groups into a single output group.
- Rows from all inputs flow into the output sequentially.

*Active Behavior*
- The number of rows changes because multiple inputs are combined.
- No deduplication is performed (unlike SQL UNION which removes duplicates).
- If you need duplicates removed, you must use a Sorter with Distinct after Union.

**Differences From Joiner**

| **Union** | **Joiner** |
| -- | -- |
| Combines rows vertically (stacking) | Combines rows horizontally (joining columns) |
| Requires same structure | Can join different structures |
| No join condition | Requires join condition |

## Stored Procedure Transformation
- It  allows you to call and execute database stored procedures directly from within a mapping. 
- It’s a way to leverage existing business logic written in the database while integrating it into your ETL flow.
- It can be either passive or active based on the procedures.
- IF the procedure only returns values, then the transformation will be passive.
- If the procedure affects row count, then the transformaiton will become active.

**How it works**
*Define Strored Procedure Transformation*
- In Designer, you import or define the stored procedure from the database.
- The transformation creates ports corresponding to the procedure’s input/output parameters.

*Input Ports*
- Values passed from the mapping into the stored procedure.

*Output Ports*
- Values returned by the stored procedure back into the mapping.

*Execution*
- During session run, Informatica calls the stored procedure for each row (or once, depending on configuration).

## Normalizer Transformation
- It is used to convert column data into multiple rows.
- It’s particularly useful when dealing with denormalized or repeating groups of data in sources like COBOL files, flat files, or legacy systems.
- It is an active transformation.

**How it works**
*Input*
- Source data has repeating groups (e.g., SALES_JAN, SALES_FEB, SALES_MAR…).

*Normalizer Setup*
- Define an occurs clause (number of repeating fields).
- Map repeating columns to a single output port.

*Output*
- Normalizer generates multiple rows for each input row, one per occurrence.
- Adds a Generated Key column to identify the occurrence number.

**Example Scenario**
Suppose you have a source file with monthly sales data:

```shell
CUST_ID | SALES_JAN | SALES_FEB | SALES_MAR
101     | 5000      | 6000      | 5500
102     | 7000      | 8000      | 7500
```

*Normalizer Setup*
- Occurs = 3 (for Jan, Feb, Mar).
- Output ports: CUST_ID, SALES, MONTH_KEY.

*Output Data*
```shell
CUST_ID | SALES | MONTH_KEY
101     | 5000  | 1   (Jan)
101     | 6000  | 2   (Feb)
101     | 5500  | 3   (Mar)
102     | 7000  | 1   (Jan)
102     | 8000  | 2   (Feb)
102     | 7500  | 3   (Mar)
```

## Transaction Control Transformation
- It is used to control commit and rollback operations during a session run.
- It is an active transformation.
- Allows you to define transaction boundaries dynamically, based on conditions in the data.
- Supports commit, rollback, or continue operations.

**How it works**
The transformation uses the `TC_CONTINUE_TRANSACTION` variable, which can be set to one of the following constants:
- `TC_COMMIT_BEFORE (1)` → Commits all rows in the pipeline before processing the current row.
- `TC_COMMIT_AFTER (2)` → Commits all rows in the pipeline after processing the current row.
- `TC_ROLLBACK_BEFORE (3)` → Rolls back all rows in the pipeline before processing the current row.
- `TC_ROLLBACK_AFTER (4)` → Rolls back all rows in the pipeline after processing the current row.

You write expressions inside the transformation to decide which action to take for each row.

**Example Scenario**
Suppose you are loading sales transactions into a warehouse. You want to:
- Commit after every 100 rows.
- Rollback if a transaction amount is negative.

*Transaction Control Expression*
```shell
IIF(ROWCOUNT % 100 = 0, TC_COMMIT_AFTER,
    IIF(AMOUNT < 0, TC_ROLLBACK_BEFORE,
        TC_CONTINUE_TRANSACTION))
```

**Differences From Session Control Interval**

| **Session Control Interval** | **Transaction Control Transformation** |
| -- | -- |
| Fixed (e.g., every 10,000 rows) | Dynamic, based on conditions |
| Defined at session level | Defined at row level in mapping |
| No rollback logic | Supports rollback before/after |

## XML Transformation
- It is designed to handle XML data parsing, generating, and processing XML documents within the ETL workflow.
- It allows you to read XML data into rows and columns or create XML output from relational data.
- It works with XML schemas (XSD) to define structure.

### Types Of XML Transsformations
**XML Parser Transformation**
- Reads XML data from a source (file, message, etc.).
- Converts XML elements/attributes into rows and columns.
- Requires an XML schema (XSD) to understand structure.

**XML Generator Transformation**
- Takes relational data (rows/columns) and generates XML output.
- Useful for creating XML files/messages for downstream systems.
- Uses XSD to format the output correctly.

**XML Source Qualifier**
- Reads XML files directly as a source in a mapping.
- Works with XML Parser to break down the data.

# Mapplets
- The Mapplet in Informatica PowerCenter Designer is a reusable object that contains a set of transformations, much like a mini‑mapping. 
- It’s designed to encapsulate logic you want to use across multiple mappings, so you don’t have to rebuild the same transformation sequence repeatedly.
- It functions like a "subroutine" in programming - you build once and reuse multiple times.

**How it works**
- In Mapplet Designer, drag and drop transformations.
- Define input ports (mapplet receives data) and output ports (mapplet sends data).
- Insert the mapplet into a mapping just like a transformation.
- Connect source data to the mapplet’s input ports, and mapplet’s output ports to targets or other transformations.
- During session run, the mapplet executes its internal transformations as part of the overall mapping flow.

**Difference From Reusable Transformations**

| **Mapplet** | **Reusable Transformation** |
| -- | -- |
| Contains multiple transformations | Contains only one transformation |
| Acts like a mini-mapping | Acts like a single reusable block |
| More flexible and powerful | Simpler, but less versatile |

# User Defined Functions (UDF)
- A reusable function created by the developer in the Designer.
- Encapsulates logic (like string manipulation, mathematical calculations, or conditional rules).
- Can be used in Expression transformations, Filter conditions, Update Strategy expressions, etc.
- Helps maintain consistency and reduces repetitive coding.

## Defining A UDF
1. **Open Informatica Designer**
- Go to Tools → User-Defined Functions.

2. **Create New Function**
- Click New and provide a name (e.g., CalculateDiscount).
- Choose the return type (string, integer, decimal, etc.).

3. **Write the Expression**
- Define the logic using Informatica’s expression syntax.

*Example*
```shell
IIF(PREMIUM > 5000, PREMIUM * 0.10, PREMIUM * 0.05)
```

4. **Save and Validate**
- Save the UDF and validate the syntax.

5. **Use in Transformations**
- In an Expression transformation, instead of rewriting the formula, simply call:

```shell
CalculateDiscount(PREMIUM)
```
