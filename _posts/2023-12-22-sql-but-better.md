---
layout: post
title: SQL... but better
lead: I can't believe it's not SQL
categories: work
tags: Data, SQL, Malloy, PRQL, DuckDB
---

So, with my first actual technical blog - here's something that is really interesting, but probably very difficult to convince a team to take on board; but hey-ho it's interesting to me.

SQL, at least the DQL part is a language created with the idea to sound english - this allows the basic structure of SQL

```sql
SELECT field1, field2
FROM table1
WHERE field1 > 100;
```

... to be read like (with some grammar stuff added) ...

```
Select these fields from this table where this field 
is greater than 100
```

... pretty much like for like.

which you know, is great and all, but this does [mean a lot of shortcomings and leaves many areas for improvement](https://www.scattered-thoughts.net/writing/against-sql) - and does lead to the statement

![There's got to be a better way!](https://c.tenor.com/YygntU2fx3kAAAAC/tenor.gif)

And there are in the corners of the internet, the following are a couple of ones that I've played around with, or have heard good things about - out of pure simplicity sake, I'm going to split the run through into 2 categories - friendly SQL & SQL alternatives.

# Friendly SQL

This category I'm kind of using to cover a number of changes made by groups like the DuckDB developers - this amounts to small snippets and shortcuts that can be used within standard ANSI SQL.

A couple of examples include

- No need for  `SELECT *`
    ```sql
    /* Instead of SELECT * FROM tbl1 */
    FROM tbl1
    ```

- Not needed to expand the `*` with fields we don't want, or small calculations
    ```sql
    SELECT *
    EXCLUDE (field1, field2) -- this will show all apart from field1 & field 2
    REPLACE (field3 + 5 AS field3) -- this will replace field3 with field3 + 5
    FROM tbl1
    ```

- No need to specify every column when grouping and ordering
    ```sql
    SELECT field1,
        field2,
        field3, -- note the trailing column 🥰
    FROM tbl1
    GROUP BY ALL -- same as GROUP BY field1, field2, field3
    ORDER BY ALL -- same as ORDER BY field1, field2, field3
    ```

- Using aliased names in `WHERE` / `GROUP BY` / `HAVING`
    ```sql
    SELECT
        field1 AS some_bool,
        field2 AS some_data,
        SUM(field3) AS total_stuff
    FROM tbl1
    WHERE
        some_bool = 1
    GROUP BY
        some_bool,
        some_data
    HAVING
        total_stuff > 0
    ```

[This post on DuckDB](https://duckdb.org/2022/05/04/friendlier-sql.html) or [this MotherDuck LinkedIn stream](https://www.linkedin.com/events/friendliersqlwithduckdb7140691289125105664/?lipi=urn%3Ali%3Apage%3Ad_flagship3_company%3BKhCQvloEQ6iTA%2Fzt1jeiqA%3D%3D) with [this accompanying Colab](https://colab.research.google.com/drive/1abiNsdLTymLlgtKQ8FJgOVFQ3aTe2b_f) delves into this in more detail - a nice thing about DuckDB as well is that it's super easy to spin up or you can even use a demo up on the [DuckDB home page](https://shell.duckdb.org/).

Unfortunately, these changes are only applicable to DuckDB, much like with DuckDB though; there are other friendly SQL/quality of life changes in platforms such as Snowflake.

# SQL Alternatives

The other 2 things that I'll briefly run through are essentially new languages that compiles to SQL; we have 2 to showcase, PRQL (pronounced **prequel**) which is a general purpose query language, and Malloy which is more towards a semantic layer for analysis.

## PRQL

[Prequel](https://prql-lang.org) is one of my favourites; it has many of the quality of life improvements featured under the friendly SQL section, but contains solutions for many of the problems raised in [the Against SQL article](https://www.scattered-thoughts.net/writing/against-sql); it's more concise, includes many features of other languages that really helps it along, and is system agnostic (with switches for different engines).

The best introductory example is provided on the [PRQL playground](https://prql-lang.org/playground/)

```
from invoices                               # A PRQL query begins with a table
                                            # Subsequent lines "transform" (modify) it
derive {                                    # "derive" adds columns to the result
  transaction_fee = 0.8,                    # "=" sets a column name
  income = total - transaction_fee          # Calculations can use other column names
}

# starts a comment; commenting out a line leaves a valid query
filter income > 5                           # "filter" replaces both of SQL's WHERE & HAVING
filter invoice_date >= @2010-01-16          # Clear date syntax
group customer_id (                         # "group" performs the pipeline in (...) on each group
  aggregate {                               # "aggregate" reduces each group to a single row
    sum_income = sum income,                # ... using SQL SUM(), COUNT(), etc. functions
    ct = count customer_id,          
  }
)
join c=customers (==customer_id)            # join on "customer_id" from both tables
derive name = f"{c.last_name}, 
    {c.first_name}"                         # F-strings like Python
derive db_version = s"version()"            # S-string offers escape hatch to SQL
select {                                    # "select" passes along only the named columns
  c.customer_id, 
  name, 
  sum_income, 
  ct, 
  db_version,
}                                           # trailing commas always ignored
sort {-sum_income}                          # "sort" sorts the result; "-" is decreasing order
take 1..10                                  # Limit to a range - could also be "take 10"
```

which compiles to

```sql
WITH table_1 AS (
  SELECT
    customer_id,
    total - 0.8 AS _expr_0,
    invoice_date
  FROM
    invoices
),
table_0 AS (
  SELECT
    COALESCE(SUM(_expr_0), 0) AS sum_income,
    COUNT(*) AS ct,
    customer_id
  FROM
    table_1
  WHERE
    _expr_0 > 5
    AND invoice_date >= DATE '2010-01-16'
  GROUP BY
    customer_id
)
SELECT
  c.customer_id,
  CONCAT(c.last_name, ', ', c.first_name) AS name,
  table_0.sum_income,
  table_0.ct,
  version() AS db_version
FROM
  table_0
  JOIN customers AS c ON table_0.customer_id = c.customer_id
ORDER BY
  table_0.sum_income DESC
LIMIT
  10
```