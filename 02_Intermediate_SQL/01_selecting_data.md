***

# Selecting Data in SQL

These notes cover **COUNT**, **DISTINCT**, SQL **execution order**, **debugging**, and **formatting/style** using a simple films database with four tables: `films`, `reviews`, `people`, and `roles`.

***

## 1. The Films Database

We are working with a database that contains four tables: **`films`**, **`reviews`**, **`people`**, and **`roles`**. Each table has specific fields (columns) with defined data types, represented together in a **schema** diagram.

***

## 2. COUNT()

### 2.1 What COUNT() does

- **`COUNT()`** returns the **number of records** (rows) that have a **non‑NULL value** in a given field.
- Basic form when counting a single column:

```sql
SELECT COUNT(birthdate) AS count_birthdates
FROM people;
```

- This query returns the number of people who have a birthdate stored in `people.birthdate` (e.g., `6152` birthdates).

**Key idea:** `COUNT(column_name)` only counts rows where that column is **not NULL**.

***

### 2.2 COUNT() with multiple fields

If you want to count more than one field, you must **call `COUNT()` separately for each field**:

```sql
SELECT
  COUNT(name)       AS count_names,
  COUNT(birthdate)  AS count_birthdates
FROM people;
```

Each `COUNT()` is independent and counts non-NULL values for its specific column.

***

### 2.3 Using `*` with COUNT()

- `COUNT(*)` counts the **number of rows in the entire table**, regardless of which fields are NULL.
- Example:

```sql
SELECT COUNT(*) AS total_people
FROM people;
```

- Here, the `*` means “all columns” and is used as a **shortcut** to count all records in the table.

**Key idea:**

- `COUNT(column)` → counts **non‑NULL values** in that column.
- `COUNT(*)` → counts **all rows** in the table.

***

## 3. DISTINCT

### 3.1 What DISTINCT does

- **`DISTINCT`** returns only **unique values** from a column, removing duplicates in the result set.
- Example: list all unique languages in the `films` table:

```sql
SELECT DISTINCT language
FROM films;
```

This returns each language **once**, even if many films share that language.

***

### 3.2 COUNT() with DISTINCT

You can combine `COUNT()` with `DISTINCT` to count **unique values**:

```sql
SELECT COUNT(DISTINCT birthdate) AS unique_birthdates
FROM people;
```

- `COUNT(birthdate)` counts **all non‑NULL birthdates**, including duplicates (e.g. many people with the same date).
- `COUNT(DISTINCT birthdate)` counts **how many different dates** appear, ignoring how many people share each date.

**Key idea:**

- `COUNT(birthdate)` → “How many birthdates are stored?”
- `COUNT(DISTINCT birthdate)` → “How many different birthdates exist?”

***

## 4. Order of Execution in SQL

In SQL, code is **not executed in the order you write it**. The logical processing order is different from the written order.

Typical simple query:

```sql
SELECT name
FROM people
LIMIT 10;
```


### 4.1 Simplified execution order

1. **FROM** – SQL first decides **which table** (or tables) it reads from (`people` here).
2. **SELECT** – then it **selects the columns** to return (`name`).
3. **LIMIT** – finally, it **limits** how many rows are returned (e.g. first 10 names).

**Analogy:**
Like grabbing a coat:

- First, decide **which closet** has coats (`FROM`).
- Then select **which coat** you want (`SELECT`).
- Then maybe decide to take **only one coat** (`LIMIT`).


### 4.2 Why order matters

- Knowing the execution order helps with **debugging** and **aliasing**.
- If you create an **alias** in `SELECT`, you can only use it in parts of the query that are processed **after** `SELECT`.
If you refer to an alias **too early** (e.g. in `FROM`), the database doesn’t “know” it yet.

***

## 5. Debugging SQL

Debugging = **finding and fixing errors** in your SQL queries.

### 5.1 Error messages

- Some error messages are **very helpful** and point directly to the problem.
    - Example: misspelling a column name like `nmae` instead of `name`.
- Others are **less specific**, and you must inspect your code carefully.

***

### 5.2 Common comma errors

A very common error is **forgetting a comma** between selected fields.

Example (incorrect):

```sql
SELECT title, country duration
FROM films;
```

- The database might show an error with a **caret** (`^`) pointing at `country` or `duration`.
- The real issue is a **missing comma** between `country` and `duration`.

Correct version:

```sql
SELECT
  title,
  country,
  duration
FROM films;
```


***

### 5.3 Keyword errors

- If you **misspell a keyword**, SQL will also show an error with a caret pointing at the incorrect word.
- Example: writing `FRM` instead of `FROM` will trigger an error exactly at the bad keyword.

***

### 5.4 Mindset for errors

- There are many possible SQL errors, but **comma mistakes**, **misspelled field names**, and **misspelled keywords** are among the most common.
- **Debugging is a core skill**: you improve by making mistakes, reading error messages, and fixing your queries.

***

## 6. SQL Formatting

### 6.1 SQL is flexible, but…

- SQL does **not require** specific formatting:
    - New lines are optional.
    - Capitalization is optional.
    - Indentation is optional.
- Example: you can put everything on **one line** and it will still run:

```sql
SELECT title, release_year, country FROM films LIMIT 3;
```

But while this works, it is **harder to read**, especially for long or complex queries.

***

### 6.2 Best practices for formatting

Over time, SQL users have developed **common style conventions**:

- Use **uppercase** for SQL keywords: `SELECT`, `FROM`, `WHERE`, `LIMIT`, etc.
- Put the main clauses on **separate lines**.
- Optionally, put each selected column on its own line.

Example (more readable):

```sql
SELECT
  title,
  release_year,
  country
FROM films
LIMIT 3;
```

This version returns the same result as the one-line query, but is **much easier to read**.

***

### 6.3 Style guides

- Many teams or companies adopt a **SQL style guide** (for example, Holywell’s style guide).
- A style guide typically defines:
    - How to **indent**.
    - Which words to **capitalize**.
    - How to **name** tables, columns, and aliases.

**Important:** There is **no single universal standard**, but the main goal is **clear and readable** SQL.

***

## 7. Semicolons

- In PostgreSQL, a semicolon at the end of a single query is **not strictly required**; the query will often run without it.
- However, adding a **semicolon (`;`)** is considered **best practice**:

```sql
SELECT
  title,
  release_year,
  country
FROM films
LIMIT 3;
```

**Why the semicolon is useful:**

- Some SQL dialects **require** it, so using it makes your code more portable.
- It clearly marks the **end of a query**, which matters when you have **multiple queries in one file**.
- Think of it like a **period at the end of a sentence**.

***

## 8. Non‑standard Field Names

### 8.1 Fields with spaces

- Good practice: column names should **not contain spaces**.
- Sometimes, other people create tables with **spaces in column names**, like `"release year"`.

To query these fields, you must use **double quotes** around the field name:

```sql
SELECT
  title,
  "release year",
  country
FROM films;
```

Without the quotes, SQL would treat `"release year"` as two separate tokens and fail.

***

## 9. Why We Format SQL

- Following a **style guide** and writing **clean, readable queries** makes collaboration easier.
- Other people (and your future self) can more easily:
    - Understand what your query does.
    - **Debug** problems.
    - Modify or extend the code.

Clean SQL is a **professional standard** and shows respect for anyone who will read or maintain your queries.

***
