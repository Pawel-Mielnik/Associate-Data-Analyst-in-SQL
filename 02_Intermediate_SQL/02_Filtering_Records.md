***

# Filtering Records in SQL

In this chapter we extend our toolkit from just **selecting and counting** data to **filtering** records using `WHERE`, comparison operators, logical operators (`AND`, `OR`, `BETWEEN`), text filters (`LIKE`, `IN`), and handling **NULL** values.

***

## 1. Filtering Numbers with WHERE

### 1.1 WHERE clause basics

- The **`WHERE`** clause lets us **filter rows** so we only see data relevant to our question.
- Analogy: if the table is a **closet** full of coats, `WHERE` is how we say “only coats where the **color is green**”.

Example (films released after 1960):

```sql
SELECT title, release_year
FROM films
WHERE release_year > 1960;
```

Here `WHERE release_year > 1960` keeps only rows that satisfy that condition.

***

### 1.2 Comparison operators for numbers

With `WHERE` we use **comparison operators** to filter numeric values:

- `>`  = **greater than** (e.g. after a given year)
- `<`  = **less than** (e.g. before a given year)
- `=`  = **equal to** (exact match)
- `>=` = **greater than or equal to**
- `<=` = **less than or equal to**
- `<>` = **not equal to** (SQL standard)

Examples:

```sql
-- Films released after 1960
WHERE release_year > 1960;

-- Films released before 1960
WHERE release_year < 1960;

-- Films released in 1960
WHERE release_year = 1960;

-- Films released during or before 1960
WHERE release_year <= 1960;

-- All films except those from 1960
WHERE release_year <> 1960;
```

**Key idea:** `<>` means “**not equal to**”, so it returns all rows **except** the specified value.

***

### 1.3 WHERE with strings

- We can also use `WHERE` and `=` with **text (strings)**.
- For strings we must use **single quotes** around the value.

Example:

```sql
SELECT title, country
FROM films
WHERE country = 'Japan';
```

This returns only films where the `country` exactly equals `'Japan'`.

***

### 1.4 Order of execution with WHERE

When we use `WHERE` and `LIMIT` together, the **written** order and **execution** order differ.

Written order:

```sql
SELECT name
FROM people
WHERE birthdate > '1960-01-01'
LIMIT 5;
```

Execution order:

1. **FROM** – choose the table (`people`).
2. **WHERE** – filter rows (e.g. birthdate after 1960-01-01).
3. **SELECT** – pick the columns to show (`name`).
4. **LIMIT** – restrict how many rows to return (5 rows).

Analogy with coats:

- Go to the **closet** (`FROM`),
- find where the **green coats** are (`WHERE`),
- pick the **coats** you want (`SELECT`),
- take only **five** of them (`LIMIT`).

***

## 2. Filtering with Multiple Criteria

Sometimes we need more than one condition, for example **color is yellow** **and** **length is short**. SQL gives us **`OR`**, **`AND`**, and **`BETWEEN`** to build richer filters.

***

### 2.1 OR operator

- Use **`OR`** when **at least one** of the conditions should be true.
- Example: films released in **1994 or 2000**:

```sql
SELECT title, release_year
FROM films
WHERE release_year = 1994
   OR release_year = 2000;
```

**Important:** You must repeat the **field name and operator** for each `OR` condition.

This is **invalid**:

```sql
-- INVALID
WHERE release_year = 1994 OR 2000;
```

SQL needs to know **what** `2000` refers to and what operator to use.

***

### 2.2 AND operator

- Use **`AND`** when **all conditions** must be true.
- Example: films released between **1994 and 2000** using explicit comparisons:

```sql
SELECT title, release_year
FROM films
WHERE release_year >= 1994
  AND release_year <= 2000;
```

As with `OR`, you must name the **field** separately in every `AND` condition.

***

### 2.3 Combining AND and OR (with parentheses)

We can mix `AND` and `OR` in the same query, but we must be careful about **execution order**. Parentheses group conditions and make the logic clear.

Example: films released in **1994 or 1995** **and** with certification **PG or R**:

```sql
SELECT title, release_year, certification
FROM films
WHERE (release_year = 1994 OR release_year = 1995)
  AND (certification = 'PG' OR certification = 'R');
```

- Parentheses ensure the database first decides **which years** we accept, then which **certifications**.
- Without parentheses, SQL may evaluate conditions in a different order and give **unexpected results**.

***

### 2.4 BETWEEN for numeric ranges

Checking numeric **ranges** is so common that SQL provides the **`BETWEEN`** shorthand.

These two queries are equivalent:

```sql
-- Using >= and <=
SELECT title, release_year
FROM films
WHERE release_year >= 1994
  AND release_year <= 2000;

-- Using BETWEEN (inclusive)
SELECT title, release_year
FROM films
WHERE release_year BETWEEN 1994 AND 2000;
```

**Key idea:** `BETWEEN` is **inclusive** – results include both **start and end** values (1994 and 2000).

We can combine `BETWEEN` with `AND`/`OR`, e.g. films released between 1994 and 2000 **from the UK**:

```sql
SELECT title, release_year, country
FROM films
WHERE release_year BETWEEN 1994 AND 2000
  AND country = 'United Kingdom';
```


***

## 3. Filtering Text

Now we move from numeric filters to **text filters**, using **patterns** instead of only exact matches.

***

### 3.1 LIKE and wildcards

- **`LIKE`** lets us match **patterns** in text fields.
- We use **wildcards** as placeholders:
    - `%` (percent) = **zero, one, or many** characters.
    - `_` (underscore) = **exactly one** character.

Examples:

```sql
-- Names starting with 'Ad' (Adel, Adelaide, Aden, etc.)
WHERE name LIKE 'Ad%';

-- Names with exactly 3 characters (Eve, Eva, etc.)
WHERE name LIKE '___';
```

- `%` after `Ad` means “anything can follow”.
- `___` (three underscores) matches exactly **three-character** names.

Another example: three-letter names starting with `E`:

```sql
WHERE name LIKE 'E__';
```

Note: `Eva Mendes` would not match this pattern, because it has more than three characters.

***

### 3.2 NOT LIKE

- **`NOT LIKE`** finds rows that **do not** match a pattern.

Example: people whose names do **not** contain `A.`:

```sql
SELECT name
FROM people
WHERE name NOT LIKE '%A.%';
```

**Note:** Pattern matching is often **case-sensitive**, so `'A'` and `'a'` may be treated differently depending on the database.

***

### 3.3 Wildcard positions and combinations

We can place `%` and `_` **anywhere** in the pattern and combine them:

- Names that **end with** `r`:

```sql
WHERE name LIKE '%r';
```

- Names where the **third character** is `t`:

```sql
WHERE name LIKE '__t%';
```

Here:

- `__` = any two characters,
- `t` = third character,
- `%` = any trailing characters.

***

### 3.4 IN for cleaner OR conditions

Sometimes we want to filter on **several specific values**, e.g. many years or countries.

Without `IN` we might write:

```sql
WHERE release_year = 1920
   OR release_year = 1930
   OR release_year = 1940;
```

With **`IN`** this becomes shorter and cleaner:

```sql
WHERE release_year IN (1920, 1930, 1940);
```

Another example with text:

```sql
SELECT title, country
FROM films
WHERE country IN ('Germany', 'France');
```

**Key idea:** `IN (a, b, c)` behaves like **multiple `OR` conditions** on the same field.

***

## 4. Filtering NULL Values

### 4.1 What is NULL?

- **`NULL`** represents a **missing** or **unknown** value in SQL.
- Real databases often contain `NULL`s because:
    - data is incomplete or unknown,
    - someone forgot to enter a value,
    - the information does not apply.

**Important:** `NULL` is **not** the same as `0`, an empty string `''`, or `false`. It means “we **don’t know** the value”.

***

### 4.2 How NULL affects analysis

Example: we have a `deathdate` column in `people`.

If we do:

```sql
SELECT COUNT(*)
FROM people;
```

we count **all people**, regardless of their `deathdate`.

But if **half of the `deathdate` values are NULL**, and we ignore that, we might wrongly assume we know the death date for everyone. That would lead to **incorrect conclusions** (for example, about posthumous success).

***

### 4.3 IS NULL and IS NOT NULL

To check which rows have missing values, we use **`IS NULL`**:

```sql
-- People without a recorded birthdate
SELECT name
FROM people
WHERE birthdate IS NULL;
```

To get only rows with **non-missing** values, we use **`IS NOT NULL`**:

```sql
-- People with a recorded birthdate
SELECT name
FROM people
WHERE birthdate IS NOT NULL;
```

We can also use these with `COUNT()`:

```sql
-- Count missing birthdates
SELECT COUNT(*)
FROM people
WHERE birthdate IS NULL;      -- e.g. 2245

-- Count non-missing birthdates
SELECT COUNT(*)
FROM people
WHERE birthdate IS NOT NULL;  -- e.g. 6152
```


***

### 4.4 COUNT(column) vs WHERE IS NOT NULL

Two equivalent ways to count **non-NULL** values in a column:

```sql
-- 1) Using COUNT on the column
SELECT COUNT(birthdate)
FROM people;

-- 2) Using COUNT(*) with IS NOT NULL
SELECT COUNT(*)
FROM people
WHERE birthdate IS NOT NULL;
```

Both queries count **only rows where `birthdate` is NOT NULL**. The result is the **same**.

***

### 4.5 Summary of NULL handling

- **NULL = missing/unknown** value.
- NULLs are **very common** in real data.
- Good practice:
    - Find how many NULL values you have (`IS NULL`).
    - Decide whether to **include or exclude** them (`IS NOT NULL`).
- Using `IS NULL` and `IS NOT NULL` will quickly become **second nature**, because dealing with missing data is a constant part of real-world SQL work.

***


