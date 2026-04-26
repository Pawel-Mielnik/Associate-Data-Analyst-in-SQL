
# Intro to Databases and SQL

## What is a database?

A **database** is an organized system for storing and managing data so that it can be stored reliably, retrieved efficiently, and manipulated systematically.
Databases power web and mobile applications, business systems, analytics dashboards, and reporting tools you use every day.

## How data is organized: tables, rows, columns

In a relational database, data is stored in **tables** made of rows and columns, similar to a spreadsheet.
Each table usually represents one type of entity, such as customers, orders, or products.

- **Rows** represent individual records, entities, or events (for example, one customer or one order).
- **Columns** represent specific attributes or properties of those records (for example, `name`, `email`, `order_date`).

This clear separation between structure (columns) and entries (rows) makes it possible to query and analyze data efficiently with SQL.

## SQL basics

**SQL (Structured Query Language)** is the standard language for working with data in relational databases.
You write **SQL queries** to request data, and the database returns the results you need.

SQL focuses on a single purpose—**working with data**—so it is simpler than general-purpose programming languages like Python or Java.
Core operations include reading data, filtering it, sorting it, limiting results, and aggregating values.

***

## Selecting columns with SELECT

The **SELECT** statement tells the database **which columns** to retrieve, and the **FROM** clause specifies **which table** to read from.

Basic example:

```sql
SELECT column_1,
       column_2
FROM table_1;
```

Common patterns:

- `SELECT column_1` – selects a **single** column.
- `SELECT column_1, column_2` – selects **multiple** columns, separated by commas.
- `SELECT *` – selects **all** columns from the table.

Using `*` is handy for quick exploration, but in real projects it is usually better to list only the columns you actually need.

***

## Sorting results with ORDER BY

The **ORDER BY** clause sorts the rows returned by a query according to one or more columns.

Example:

```sql
SELECT column_1,
       column_2
FROM table_1
ORDER BY column_1 DESC;
```

Sorting behavior:

- `ORDER BY column_1` – sorts in **ascending** order (lowest to highest: A–Z, 0–9).
- `ORDER BY column_1 DESC` – sorts in **descending** order (highest to lowest: Z–A, 9–0).

Sorting makes it easier to answer questions such as “show the top-rated products” or “list the newest records first.”

***

## Limiting the number of rows with LIMIT

The **LIMIT** clause restricts how many rows are returned by a query.

Example:

```sql
SELECT *
FROM table_1
LIMIT 10;
```

Key points:

- `LIMIT` is placed at the **end** of the query.
- It returns only the **first N rows** produced by the query.
- Limiting results helps avoid slow, expensive queries and prevents accidentally returning huge datasets.

Some SQL flavors have slightly different syntax for limiting rows, but the idea is the same: **control how much data you get back**.

***

## Temporary nature of SELECT results

Running a **SELECT** query does **not** modify the original database.
The result is a **temporary snapshot** of the data at the moment the query runs.

If you want to keep the results for later use, you must:

- Export them to a file (for example, CSV) on your computer or cloud storage, or
- Save them into another table using commands like `CREATE TABLE ... AS SELECT ...` (depending on the database system).

This makes SELECT safe for exploration and analysis because it only **reads** data.

***

## Handling errors in SQL

SQL is strict about **syntax**, and small mistakes (missing commas, typos, wrong keywords) can stop a query from running.
Learning to read and understand error messages is essential for debugging queries.

Good practices:

- Use a **good SQL editor** with syntax highlighting and auto-completion.
- Double-check the spelling of table and column names.
- Test queries with `LIMIT` to avoid accidentally pulling too much data.
- Use documentation and AI assistance when you are unsure about syntax or functions.

The more you practice SQL, the **fewer errors** you will make, and the faster you will recognize and fix common problems.

***

## Finding unique values with DISTINCT

The **DISTINCT** keyword returns only **unique** values, removing duplicates from the result set.

Example:

```sql
SELECT DISTINCT column_name
FROM table_name;
```

Important details:

- `DISTINCT` is placed **immediately after** `SELECT`.
- When used with a single column, it returns each unique value in that column once.
- When used with multiple columns, it returns unique **combinations** of those columns.

DISTINCT is useful when you need, for example, a list of all unique countries, categories, or user IDs in a table.

***

## Column aliasing with AS

A **column alias** is a temporary name you assign to a column or expression in the query results using the `AS` keyword.

Example:

```sql
SELECT column_1 AS alias_name,
       column_2
FROM table_1;
```

Key points:

- The alias appears in the **result column header**, not in the underlying table.
- Aliases improve **readability**, especially for calculated columns or long column names.
- Good aliases are descriptive, consistent, and often use `snake_case` (for example, `total_price`, `average_rating`).

Aliases are especially helpful when preparing data for reports, dashboards, or downstream analysis.

***

## SQL style conventions

Following consistent **style conventions** makes SQL code easier to read and maintain.

Common conventions:

- Write SQL **keywords in uppercase** (for example, `SELECT`, `FROM`, `ORDER BY`, `LIMIT`, `DISTINCT`, `AS`).
- Use lowercase (or snake_case) for table and column names, such as `customer_orders` or `order_date`.
- Format queries with line breaks and indentation to clearly separate clauses.

Good style does not change how the query runs, but it makes code clearer for both you and others.

***

## SQL flavors

Different database systems implement their own **SQL flavors**, such as MySQL, PostgreSQL, SQL Server, and Oracle.

They all share the same **core syntax** (SELECT, FROM, WHERE, ORDER BY, etc.), but differ in:

- Specific functions and data types.
- Details of features like limiting rows (`LIMIT` vs `TOP` vs `FETCH`).

The positive side is that once you learn standard SQL, your **skills transfer** easily between platforms.

***

## Client–server architecture

Most production databases use a **client–server architecture**.

- The **database server** stores and manages the databases and processes SQL queries.
- **Clients** (for example, web apps, desktop tools, or SQL editors) connect to the server over a network and send SQL commands.

Multiple users and applications can connect to the same database server at the same time, each sending queries and receiving results independently.
This architecture allows databases to centralize data, enforce security, and handle many requests efficiently.

***