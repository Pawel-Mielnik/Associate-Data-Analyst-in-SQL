
***
# Aggregate Functions in SQL

In this chapter we learn how to **summarize data** using **aggregate functions**, combine them with filters, round numerical results, use arithmetic in queries, and apply **aliasing** to keep results readable.

***

## 1. Aggregate Functions Basics

### 1.1 What is an aggregate function?

- When analyzing data, we often want to understand the **dataset as a whole**, not just individual rows.
- An **aggregate function** performs a **calculation over many values** and returns **one single value** (for example, the average budget of all films).

**Key idea:** Aggregate functions “collapse” many rows into **one summary value**.

***

### 1.2 Core aggregate functions

We already know one aggregate function: **`COUNT()`**. Now we add four more:

- **`AVG()`** – average value of a column.
- **`SUM()`** – sum of all values in a column.
- **`MIN()`** – smallest value.
- **`MAX()`** – largest value.
- **`COUNT()`** – number of non-NULL values.

They are written in the **`SELECT`** clause, just like `COUNT()`.

Examples:

```sql
-- Average budget of all films
SELECT AVG(budget) AS avg_budget
FROM films;

-- Total budget of all films
SELECT SUM(budget) AS total_budget
FROM films;

-- Minimum and maximum budget
SELECT
  MIN(budget) AS min_budget,
  MAX(budget) AS max_budget
FROM films;
```

**Note:** The functions operate on a **column** (vertical), not on individual rows horizontally.

***

### 1.3 Numerical vs non‑numerical data

- **`AVG()`** and **`SUM()`** only make sense on **numeric** columns, because they require arithmetic.
- **`COUNT()`**, **`MIN()`**, and **`MAX()`** can also be used on **non-numeric** fields:
    - strings (e.g. country names),
    - dates,
    - other comparable types.

Examples:

```sql
-- Count non-NULL countries
SELECT COUNT(country) AS count_countries
FROM films;

-- Alphabetically first and last country
SELECT
  MIN(country) AS first_country,
  MAX(country) AS last_country
FROM films;
```

Here, `MIN(country)` returns the **alphabetically first** country (e.g. Afghanistan), and `MAX(country)` returns the **alphabetically last** (e.g. West Germany, in this dataset).

***

### 1.4 Aliasing aggregated results

By default, the result column name becomes the **function name** (for example, `min`), which is not very descriptive.

Best practice is to **alias** aggregated results using `AS`:

```sql
SELECT MIN(country) AS first_country
FROM films;
```

**Key idea:** Always give summary values a **clear alias** so readers instantly know what they represent.

***

## 2. Summarizing Subsets with WHERE

### 2.1 Combining WHERE with aggregate functions

We can combine aggregate functions with **`WHERE`** to summarize **only a subset** of the data.

Because **`WHERE` executes before `SELECT`**, the filter is applied **first**, and then the aggregate function runs on the filtered rows.

Example: average budget for films released in **2010 or later**:

```sql
SELECT AVG(budget) AS avg_budget_2010_plus
FROM films
WHERE release_year >= 2010;
```


***

### 2.2 More examples with WHERE

Example: total budget of films released in **2010**:

```sql
SELECT SUM(budget) AS total_budget_2010
FROM films
WHERE release_year = 2010;
```

Example: smallest and largest budgets for 2010 films:

```sql
SELECT
  MIN(budget) AS min_budget_2010,
  MAX(budget) AS max_budget_2010
FROM films
WHERE release_year = 2010;
```

Example: how many non-missing budgets we have for 2010:

```sql
SELECT COUNT(budget) AS count_budgets_2010
FROM films
WHERE release_year = 2010;
```

Here `COUNT(budget)` gives the number of **non-NULL** budgets, e.g. 194 recorded budgets.

***

## 3. Rounding Numeric Results with ROUND()

### 3.1 ROUND() basics

When we compute averages or other numeric summaries, we may get **long decimal numbers**.

SQL’s **`ROUND()`** function helps clean them up.

- `ROUND(number, decimal_places)`:
    - `number` – expression to round (e.g. `AVG(budget)`),
    - `decimal_places` – how many digits after the decimal point.

Example: average budget rounded to **2 decimal places**:

```sql
SELECT ROUND(AVG(budget), 2) AS avg_budget_rounded
FROM films;
```

This is useful for **currency** or other values where many decimal places are unnecessary.

***

### 3.2 Rounding to whole numbers

The second parameter is **optional**:

- `ROUND(number)` or `ROUND(number, 0)` both round to a **whole number**.

Example:

```sql
SELECT ROUND(AVG(budget)) AS avg_budget_whole
FROM films;
```

This returns an integer (no decimal part).

***

### 3.3 Rounding with negative decimal places

`ROUND()` also accepts **negative** decimal positions, which rounds to the **left** of the decimal point.

Example (conceptual):

```sql
-- Round to the nearest 100,000
SELECT ROUND(AVG(budget), -5) AS avg_budget_rounded_100k
FROM films;
```

- `-5` means “round to 5 places on the **left** side of the decimal”, e.g. to the nearest **hundred thousand**.

**Key idea:** `ROUND()` works only on **numeric** expressions.

***

## 4. Arithmetic in SQL

### 4.1 Basic arithmetic operators

We can perform basic arithmetic in SQL:

- `+` addition
- `-` subtraction
- `*` multiplication
- `/` division

Examples:

```sql
SELECT
  2 + 3   AS sum_example,
  7 - 4   AS diff_example,
  5 * 6   AS product_example,
  4 / 2   AS division_example;
```

We can use **parentheses** to clarify the calculation order, especially when combining multiple operations.

***

### 4.2 Integer vs decimal division

SQL behaves like many programming languages for division:

- If you divide an **integer by an integer**, the result may be treated as an **integer** (e.g. `4 / 3` → `1`).
- To get a **decimal**, use decimal numbers:

```sql
SELECT 4 / 3        AS int_division,   -- often 1
       4.0 / 3.0    AS decimal_division;  -- 1.3333...
```

**Key idea:** Add decimal points (e.g. `4.0`) to get **fractional** results.

***

### 4.3 Aggregate functions vs arithmetic

Difference in perspective:

- **Aggregate functions** (e.g. `SUM(budget)`) work **vertically** on a **column**, combining all rows into one number.
- **Arithmetic expressions** (e.g. `gross - budget`) work **horizontally** on **each row**, combining columns in that row.

Example: per-film profit (horizontal):

```sql
SELECT
  title,
  gross - budget AS profit
FROM films;
```

Example: total profit across all films (vertical):

```sql
SELECT SUM(gross - budget) AS total_profit
FROM films;
```


***

## 5. Aliasing with Arithmetic and Functions

### 5.1 Aliasing arithmetic expressions

When we write an expression like `gross - budget`, SQL will display the result column with an **unnamed or auto-generated** field name.

We should always **alias** these expressions for clarity:

```sql
SELECT
  title,
  gross - budget AS profit
FROM films;
```

Here **`profit`** clearly explains what the expression represents.

***

### 5.2 Aliasing multiple functions

If we use the same function multiple times, we would otherwise end up with repeated, unhelpful names:

```sql
-- Bad: both results might be shown as "max"
SELECT
  MAX(budget),
  MAX(gross)
FROM films;
```

Better:

```sql
SELECT
  MAX(budget) AS max_budget,
  MAX(gross)  AS max_gross
FROM films;
```

**Key idea:** Aliasing is essential when using **multiple functions** or complex expressions so that each output column has a **meaningful name**.

***

## 6. Aliases and Order of Execution

We need to understand how **aliases** fit into SQL’s **execution order**.

### 6.1 Execution order recap

So far, the simplified logical order is:

1. **FROM** – choose the table(s).
2. **WHERE** – filter rows.
3. **SELECT** – compute expressions and apply **aliases**.
4. **LIMIT** – restrict number of returned rows.

### 6.2 Why aliases cannot be used in WHERE

Aliases are created during the **`SELECT`** step.

But **`WHERE`** is processed **before** `SELECT`. So this query will **not work**:

```sql
-- INVALID: alias used in WHERE
SELECT gross - budget AS profit
FROM films
WHERE profit > 1000000;
```

At the time `WHERE` is evaluated, `profit` **does not exist yet**, so SQL throws an error.

Instead, we must use the original expression in `WHERE`:

```sql
SELECT gross - budget AS profit
FROM films
WHERE gross - budget > 1000000;
```

**Key idea:** You **cannot** use a field alias in `WHERE` because `WHERE` runs **before** the alias is created in `SELECT`.

***
