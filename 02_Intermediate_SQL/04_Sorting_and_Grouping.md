
***
# Sorting and Grouping in SQL

In this chapter we learn how to **sort** results with `ORDER BY`, **group** data with `GROUP BY`, and **filter grouped results** with `HAVING`, using the full SQL execution order to understand what happens when.

***

## 1. Sorting Results with ORDER BY

### 1.1 Why we sort

- **Sorting** means arranging rows in a **specific order** so patterns and extremes are easier to see.
- Analogy: to find your **three longest coats**, it’s much easier if your closet is sorted by **garment type and length** instead of being a mess.

***

### 1.2 ORDER BY basics

- Use **`ORDER BY`** to sort result rows by one or more fields.
- When used alone, it comes **after `FROM`** (and after `WHERE` if present).
- Default sorting is **ascending**:
    - numbers: smallest → largest,
    - text: A → Z,
    - also note that **symbols and numbers** usually come **before** the letter A.

Examples:

```sql
-- Budgets from smallest to largest
SELECT title, budget
FROM films
ORDER BY budget;

-- Titles alphabetically
SELECT title
FROM films
ORDER BY title;
```


***

### 1.3 ASC and DESC

- `ORDER BY` is **ascending by default**, but we can make it explicit with **`ASC`**:

```sql
ORDER BY budget ASC;
```

- To sort in **descending** order (largest → smallest, Z → A), use **`DESC`**:

```sql
-- Highest budgets first
SELECT title, budget
FROM films
ORDER BY budget DESC;
```

If the column contains many **NULL** values, we often add a **`WHERE`** filter first:

```sql
SELECT title, budget
FROM films
WHERE budget IS NOT NULL
ORDER BY budget DESC;
```

This prevents NULLs from dominating the top or bottom of the ordered list.

***

### 1.4 Sorting by fields not selected

- We can sort by a field **even if it’s not in the `SELECT` list**:

```sql
SELECT title
FROM films
ORDER BY release_year;
```

This is valid, but for clarity it’s usually better to **include the sorting field** in the result:

```sql
SELECT title, release_year
FROM films
ORDER BY release_year;
```


***

### 1.5 ORDER BY multiple fields

`ORDER BY` can sort by **several fields**. We separate fields with commas; the **first** field decides the primary order, additional fields act as **tie-breakers**.

Example: sort films by **Oscar wins**, then by **IMDb score**:

```sql
SELECT title, oscars, imdb_score
FROM films
ORDER BY oscars DESC, imdb_score DESC;
```

- First, rows are ordered by `oscars` (more wins first).
- When films have the same `oscars` value, those films are sorted among themselves by `imdb_score`.

We can also mix sort directions:

```sql
-- Oldest birthdates first, but names Z → A within the same date
SELECT name, birthdate
FROM people
ORDER BY birthdate ASC, name DESC;
```


***

### 1.6 ORDER BY in the execution order

`ORDER BY` happens **late** in the execution flow, right before `LIMIT`.

Simplified logical order:

1. **FROM**
2. **WHERE**
3. **SELECT**
4. **ORDER BY**
5. **LIMIT**

This means SQL first chooses the table, filters rows, computes columns, **then** sorts them, and only at the end applies any row limit.

***

## 2. Grouping Data with GROUP BY

### 2.1 Why group data?

- In real analyses we often summarise data **per group**, e.g.:
    - average duration **per certification**,
    - number of films **per language**,
    - total revenue **per year**.

We use **`GROUP BY`** to create these groups and then apply **aggregate functions** to each group.

***

### 2.2 GROUP BY with a single field

- **`GROUP BY`** usually appears **after `FROM` and `WHERE`**.
- It is commonly used with **aggregate functions** such as `COUNT`, `AVG`, `SUM`, etc.

Example: number of films per certification:

```sql
SELECT
  certification,
  COUNT(title) AS film_count
FROM films
GROUP BY certification;
```

**Important rule:**
If you use `GROUP BY`, every column in the `SELECT` list must be either:

- in the `GROUP BY` clause, **or**
- wrapped in an **aggregate function**.

Otherwise SQL doesn’t know which single value to show for that column and will return an error.

***

### 2.3 Error handling with GROUP BY

This query is **invalid**:

```sql
-- INVALID: title not grouped and not aggregated
SELECT certification, title
FROM films
GROUP BY certification;
```

SQL will complain because `title` isn’t in `GROUP BY` and isn’t aggregated.

Valid version:

```sql
SELECT
  certification,
  COUNT(title) AS film_count
FROM films
GROUP BY certification;
```

or, if we want something like the **shortest title per certification**:

```sql
SELECT
  certification,
  MIN(title) AS first_title
FROM films
GROUP BY certification;
```


***

### 2.4 GROUP BY multiple fields

We can group by **multiple fields**, similar to sorting:

```sql
SELECT
  certification,
  language,
  COUNT(title) AS film_count
FROM films
GROUP BY certification, language;
```

Here each **(certification, language)** pair is one group. Example interpretations:

- 5 films with **missing** `certification` and `language`.
- 2 films that are **unrated** and in **Japanese**.
- 2 films that are rated **R** and in **Norwegian**.

The **order** of fields in `GROUP BY` affects how groups are logically structured, but the set of groups is defined by **all listed columns together**.

***

### 2.5 GROUP BY with ORDER BY

We often:

1. **Group** the data,
2. **Aggregate** per group,
3. **Sort** groups by the aggregated value.

Example: number of films per certification, sorted from most common to least:

```sql
SELECT
  certification,
  COUNT(title) AS film_count
FROM films
GROUP BY certification
ORDER BY film_count DESC;
```

Here we are allowed to use the **alias** `film_count` in `ORDER BY` because of the execution order.

`GROUP BY` and `ORDER BY` order:

1. `FROM`
2. `WHERE`
3. **`GROUP BY`**
4. `SELECT` (aggregations and aliases)
5. **`ORDER BY`**
6. `LIMIT`

***

## 3. Filtering Grouped Data with HAVING

### 3.1 Why we need HAVING

- **`WHERE` filters individual rows** **before** grouping and aggregation.
- We **cannot** use `WHERE` to filter based on the result of an **aggregate function**.

Example (invalid):

```sql
-- INVALID: WHERE cannot see COUNT(title) after grouping
SELECT
  release_year,
  COUNT(title) AS film_count
FROM films
GROUP BY release_year
WHERE film_count > 10;  -- not allowed
```

To filter **groups** by their aggregated values, we use **`HAVING`**:

```sql
SELECT
  release_year,
  COUNT(title) AS film_count
FROM films
GROUP BY release_year
HAVING COUNT(title) > 10;
```

This returns only years in which **more than 10 films** were released.

***

### 3.2 Full execution order with HAVING

Consider this written query:

```sql
SELECT
  certification,
  COUNT(title) AS title_count
FROM films
WHERE certification IN ('G', 'PG', 'PG-13')
GROUP BY certification
HAVING COUNT(title) > 500
ORDER BY title_count DESC
LIMIT 3;
```

Written order:

1. `SELECT`
2. `FROM films`
3. `WHERE certification IN (...)`
4. `GROUP BY certification`
5. `HAVING COUNT(title) > 500`
6. `ORDER BY title_count`
7. `LIMIT 3`

**Execution order:**

1. **FROM** – get rows from `films`.
2. **WHERE** – filter rows with `certification` G, PG, or PG-13.
3. **GROUP BY** – group remaining rows by `certification`.
4. **HAVING** – filter groups based on `COUNT(title)`.
5. **SELECT** – compute `COUNT(title)` and alias `title_count`.
6. **ORDER BY** – sort groups by `title_count`.
7. **LIMIT** – keep only the top 3 groups.

Because **`HAVING`** runs **before** `SELECT`, we **cannot** use the `title_count` alias in `HAVING`, but we **can** use it in `ORDER BY`.

***

### 3.3 HAVING vs WHERE

- **`WHERE`** filters **rows** *before* grouping.
- **`HAVING`** filters **groups** *after* grouping and aggregation.

Two example questions:

1. **“What films were released in the year 2000?”**
    - This is about **individual films**.
    - No grouping implied.
    - Use **`WHERE`**:

```sql
SELECT title
FROM films
WHERE release_year = 2000;
```

2. **“In what years was the average film duration over two hours?”**
    - This is about **years**, with an **average duration** per year.
    - Needs grouping by `release_year` + aggregation `AVG(duration)`.
    - Use **`HAVING`** to filter groups by the average.

Step by step:

```sql
SELECT
  release_year,
  AVG(duration) AS avg_duration
FROM films
GROUP BY release_year
HAVING AVG(duration) > 120;
```

    - `GROUP BY release_year` creates one group per year.
    - `AVG(duration)` computes the average duration per year.
    - `HAVING AVG(duration) > 120` keeps only years where the average is **over 120 minutes**.

**Key idea:**
Use **`WHERE`** when the condition talks about **individual rows**.
Use **`HAVING`** when the condition talks about **aggregated, grouped data**.

***

