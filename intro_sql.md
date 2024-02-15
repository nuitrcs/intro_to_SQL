# **Introduction to SQL**

* [What is SQL](#what-is-sql)
* [Database for this workshop](#database-for-this-workshop)
* [Comments in SQL](#comments-in-sql)
* [`SELECT`](#select)
* [`LIMIT`](#limit)
* [`OFFSET`](#offset)
* [`WHERE`](#where)
* [`BETWEEN`](#between)
* [`IN`](#in)
* [`IS NULL`](#is-null)
* [`ORDER BY`](#order-by)
* [`DISTINCT`](#distinct)
* [Functions and arithmetic](#functions-and-arithmetic)
* [`GROUP BY`](#group-by)
* [`HAVING`](#having)
* [Aliasing](#aliasing)
* [`CASE WHEN`](#case-when)
* [Joins](#joins)
* [Table Names and Aliases](#table-names-and-aliases)
* [Subqueries](#subqueries)
* [Common table expressions](#common-table-expressions-cte)
* [Recap and resources to continue learning](#recap-and-resources-to-continue-learning)

## What is SQL?

SQL, which is an acronym for Structured Query Language, is a language widely used to interact with most relational databases. You can create, query, update, and delete databases using SQL. This workshop focuses on querying databases.

If you have experience with `dplyr` in R or `Pandas` in Python, you'll find that SQL is similar in many ways.

“Database” is not a synonym of “dataset.” Databases are collections of data typically controlled by database management systems (such as MySQL or SQLite). Databases are common for large quantities of data, often consisting of multiple tables of data, that multiple users need to access efficiently and securely at the same time.

It’s important to note that, while SQL is one language, there are multiple dialects. There are [standards for SQL](https://blog.ansi.org/sql-standard-iso-iec-9075-2023-ansi-x3-135/#gref), but different vendors of database management systems provide various [versions of SQL syntax](https://www.datacamp.com/blog/sql-server-postgresql-mysql-whats-the-difference-where-do-i-start), such as MySQL, PostgreSQL, and SQLite. However, most of the syntax, particularly basic syntax, is very similar across dialects.

## Database for this workshop

We're working with the `hospital` database from [sql-practice.com](https://www.sql-practice.com/). You can find the schema (the set of tables, their columns and types, and the relationships between them) in the left side bar > SQL Database > View Schema.

## Comments in SQL

This is how you can comment your code in SQL:

```sql
/* this is a comment; it can 
span multiple lines */

SELECT * FROM patients; /* this is also a comment */

-- this is a single line comment

SELECT * from patients; -- another single line comment
```

## `SELECT`

Select is the command we use most often in SQL.  It lets us select data (specified rows and columns) from one or more tables.  Columns are selected by name, rows are selected with conditional statements (values of a particular column meeting some criteria). 
The basic format of a `SELECT` command is this:

```sql
SELECT <columns>
FROM <table>;
```

For example:

```sql
SELECT patient_id, first_name, last_name
FROM patients;
```

If we want all of the columns, we can use `*` as shorthand:

```sql
SELECT *
FROM patients;
```

A couple of notes:

- SQL is case-insensitive, but many times you'll see the key terms in all caps. This is good practice for readability.
- Note that you use a semicolon `;` to end the statement. This is the standard way to separate SQL statements in systems that allow for more that one statement to be executed in the same call. (With [sql-practice.com](https://www.sql-practice.com/) we don't have to worry about that.)

### Exercise 1

Select the first\_name and last\_name columns from the patients table.

## `LIMIT`

Instead of getting all rows, we can specify a limit of the number of rows to retrieve.

```sql
SELECT *
FROM patients
LIMIT 5;
```

The order of the rows is not random, but it is not guaranteed to be in any particular order by default either. Be careful about this when "looking at the data."

### Exercise 2

Select 10 rows from the patients table selecting the gender and birth\_date columns.

## `OFFSET`

We can  skip rows that would normally be at the beginning of the result set with offset.  This is useful usually when we sort the result set in a particular order, something we'll get to later. 

```sql
SELECT *
FROM patients
LIMIT 5
OFFSET 2;
```

## `WHERE`

Instead of getting all rows or a specific number of rows, we can also specify which rows we want by specifying conditions on the values of particular columns (e.g., equals, greater than, less than).

```sql
SELECT *
FROM patients
WHERE gender = 'M';
```

Note that string (text) values in SQL are surrounded with single quotes. This may vary across databases, but typically it's safer to [use single quotes for strings in SQL](https://stackoverflow.com/questions/1992314/what-is-the-difference-between-single-and-double-quotes-in-sql).

You can combine conditions together with `AND` and `OR`:

```sql
SELECT *
FROM patients
WHERE gender = 'M' AND allergies IS NULL;
```

`WHERE` operators include:

| Operator | Description |
|:---:|:---|
| = | Equal |
| > | Greater than |
| < | Less than |
| >= | Greater than or equal |
| <= | Less than or equal |
| <> or != | Not equal |
| AND | Logical operator AND |
| OR | Logical operator OR |
| NOT | To negate boolean values |

### Exercise 3

a) Select rows from the patients table where the city is Toronto.

b) Select rows from the patients table where the city is Toronto and the province\_id is not ON.

c) Select rows from the patients table where the first\_name starts with a D. (Hint [here](https://www.w3schools.com/sql/sql_like.asp).)

## `BETWEEN`

`BETWEEN` is shorthand for `# <= x <= #`.  The endpoints are inclusive:

```sql
SELECT *
FROM patients
WHERE birth_date BETWEEN '1963-01-01' AND '2000-01-01';
```
## `IN`

`IN` lets you specify a lot of values that you would otherwise join together with an `OR` statement:

```sql
SELECT *
FROM patients
WHERE city IN ('Barrie', 'Dundas', 'Hamilton');
```

### Exercise 4

Select rows from the patients table where the first name is John, Jack, or Sam using `IN`.

## `IS NULL`

As hinted earlier, missing data in SQL is `NULL`.  `NULL` values occur where there are no data entered for a specific row and column.  You can test for `NULL` with `IS NULL`:

```sql
SELECT *
FROM patients
WHERE patient_id IS NULL;
```

Note that `NULL` is different than an empty string ('').

There is also the opposite: `IS NOT NULL`.

You cannot use `= NULL`.  The following returns no rows, even though there are rows with `NULL` allergies:

```sql
SELECT *
FROM patients
WHERE allergies = NULL;
```

Keep in mind that `NULL` values are omitted from the results of comparison tests. 

## `ORDER BY`

We can determine the order that our results are shown in:

```sql
SELECT *
FROM patients
ORDER BY height;
```

By default, sorting is done in ascending (`ASC`) order.  To get the reverse order of sorting, use `DESC` (descending):

```sql
SELECT *
FROM patients
ORDER BY height DESC;
```

This is often useful to combine with `LIMIT`:

```sql
SELECT *
FROM patients
ORDER BY height DESC
LIMIT 5;
```

And with `OFFSET` to view results in smaller chunks:

```sql
SELECT *
FROM patients
ORDER BY height DESC
LIMIT 5
OFFSET 5;
```

You can order by multiple columns:

```sql
SELECT *
FROM patients
ORDER BY height DESC, weight DESC;
```

### Exercise 5

Order the rows in patients by province\_id and then by city.

## `DISTINCT`

`DISTINCT` removes duplicate rows from the result.  It comes before the list of column names, and applies to the combination of all columns.

Notice the difference between this:

```sql
SELECT DISTINCT first_name
FROM patients
ORDER BY first_name;
```

And this:

```sql
SELECT DISTINCT first_name, last_name
FROM patients
ORDER BY first_name;
```

## Functions and Arithmetic

There are many common, built-in functions. You can find them in the documentation of whichever version of SQL you are using.

For example, we can get the minimum or maximum of a column from results:

```sql
SELECT MIN(height)
FROM patients;
SELECT MAX(height)
FROM patients;
```

These functions apply to the result set, not the full table: 

```sql
SELECT MAX(height)
FROM patients
WHERE height < 200;
```

You can also do arithmetic:

```sql
SELECT height, height + 1
FROM patients;
```

Functions need to be used in the part of the query where you specify the values (or other values) that you are selecting, not the `WHERE` clause.  The following doesn't work:

```sql
SELECT *
FROM patients
WHERE height = MAX(height);
```

To achieve this, you can use a subquery, which we may learn about later.

If we wanted to count the number of distinct values in a column, we can use the count function in combination with `distinct`:

```sql
SELECT COUNT(DISTINCT first_name)
FROM patients;
```

Count without distinct will count the number of rows *that aren't null*.

```sql
SELECT COUNT(allergies)
FROM patients;
```

Keep in mind that `NULL` is not counted by `count`, even though it is returned as a result by `DISTINCT`.

We can use `*` to count the total number of rows, including any with NULL in a column:

```sql
SELECT COUNT(*)
FROM patients;
```

### Exercise 6

a) What are the allergies present in the patients table?
b) How many people in the patients table have an allergy to penicillin?

## `GROUP BY`

`GROUP BY` is used to divide results into groups, where you then apply some summary function to each group.  You will generally select the column you're grouping by, and then a summary function.  The most common operation is counting.  We use `COUNT(*)` to count the number of rows in each group:

```sql
SELECT first_name, COUNT(*)
FROM patients
GROUP BY first_name
ORDER BY COUNT(*) DESC;
```

However, we can use other functions as well.

You can group by multiple columns:

```sql
SELECT first_name, last_name, COUNT(*)
FROM patients
GROUP BY first_name, last_name
ORDER BY COUNT(*) DESC;
```

### Exercise 7

Count the number of cities for each province\_id in the patients table.  Order the results by `COUNT(\*)`.

## `HAVING`

`HAVING` is similar to a `WHERE` clause but it applies to the result of a `GROUP BY` operation; `WHERE` applies before data are grouped by `GROUP BY`.

```sql
SELECT first_name, COUNT(*)
FROM patients
GROUP BY first_name
HAVING COUNT(*) < 40
ORDER BY COUNT(*) DESC;
```

### Exercise 8 

Select the province\_id from the patients table that have more than 20 cities associated with them.

## Aliasing

You can rename columns and tables in queries.  This will mostly be useful when we're joining tables together, but it can also be useful when you're working with functions. Or simply for convenience in looking at the output.

```sql
SELECT first_name, COUNT(*) AS patient_count
FROM patients
GROUP BY first_name
ORDER BY patient_count DESC;
```

## `CASE WHEN`

`CASE WHEN` is a conditional statement that can be used to create new columns based on the values of other columns.  It's similar to an `if` statement in other programming languages.

```sql
SELECT
	height,
    CASE
    	WHEN height < 160 THEN 'Small'
        WHEN height between 160 and 180 THEN 'Medium'
        ELSE 'Tall'
    END AS height_category
FROM patients;
```

## Joins

Looking at the database schema, we can see that information is split between four tables.  The lines between the tables show where there is a column in one table that is linked to a column in another table.  These are called foreign keys.  

In the table names, there are key icons next to some columns.  These columns are primary key columns.  A primary key can be a single column or a combination of multiple columns.  Primary keys have to have unique values.  They are frequently used to link tables to each other (although you could link tables with other columns too), and they are also used to index a table, which among other things makes lookups (where conditions) on those columns faster.

Now we'll learn how to join tables together. Diagrams such as [this one](https://images.app.goo.gl/U5RCPEhNaQuMn7y7A) can be useful to understand the different types of joins. Please note that you can join more than two tables together.

### `INNER JOIN`

The first and most common type of join is called an inner join.  You specify the tables to join, the conditions to use to match the tables up, and you get back the rows from both tables that meet the conditions.

```sql
SELECT *
FROM patients
INNER JOIN admissions ON patients.patient_id = admissions.patient_id;
```

Note that both tables have a column called patient\_id.  We add the table name to the front of the column name when referencing them. You can do this anytime, but typically only do it when you're joining and there's ambiguity. 

We can also group by, order by, and use other where clause conditions on the joined tables.  

### Exercise 10

a) Join the patients table to the province\_names table.
b) Join the admissions table to the doctors table.

### Table Names and Aliases

We can alias tables as well as columns.  If a column name appears in both tables, then we have to specify the table name when selecting it.

```sql
SELECT *
FROM patients AS p
INNER JOIN admissions AS a ON p.patient_id = a.patient_id;
```

and we often drop the `AS`:

```sql 
SELECT *
FROM patients p
INNER JOIN admissions a ON p.patient_id = a.patient_id;
```

### `LEFT JOIN`

With an inner join, we only get the results that are in both tables.  But there are other types of joins.

If we want to know which rows in a table don't have a match in the other table, we use a `LEFT JOIN` or `RIGHT JOIN` (depending on which table you want all of the results from).

Notice the difference between these:

```sql
SELECT *
FROM patients p
LEFT JOIN admissions a ON p.patient_id = a.patient_id;
```

```sql
SELECT *
FROM admissions a
RIGHT JOIN patients p ON p.patient_id = a.patient_id;
```

And this:

```sql
SELECT *
FROM patients p
RIGHT JOIN admissions a ON p.patient_id = a.patient_id;
```

### Exercise 11

a) Select the patient\_id of patients attended by a cardiologist.  Use a left join.
b) Select the patient\_id of patients attended by a cardiologist.  Now use a right join.

### `FULL OUTER JOIN`

A `FULL OUTER JOIN` is like doing a left and right join at the same time: you get rows that are in both tables, plus rows from both tables that don't match the other table. This join is less commonly used. The syntax is the same as the other joins.

# Bonus materials

## Subqueries

We can use the results of one query as values in another query. For example, we discussed above that this doesn't work: 

```sql
SELECT *
FROM patients
WHERE height = MAX(height);
```
You can use a subquery:

```sql
SELECT *
FROM patients
WHERE height = (SELECT MAX(HEIGHT) FROM patients);
```

The subquery is executed first, and then the result is used the broader query.

We can also use subqueries with `IN`:

```sql
SELECT first_name, last_name
FROM patients
WHERE patient_id IN (
  SELECT patient_id
  FROM admissions
  WHERE admission_date BETWEEN '2018-01-01' AND '2018-12-31'
);
```

But you can also do the above query by joining tables together, as we may learn later.  (`IN` is an expensive operation, meaning it can take a long time to run in large databases.)

### Exercise 9

Find the first\_name and last\_name of patients who have been attended by a cardiologist. (Hint: you may need to use more than two tables. The database schema is your friend!)

## Common Table Expressions (CTE)

CTE help simplify relatively complex queries. You can give a name to a result set and use that in a subsequent query. CTE can help make long subqueries more readable. They're particularly useful if you're using the same subquery multiple times.

```sql
WITH patients_2018 AS (
  SELECT patient_id
  FROM admissions
  WHERE admission_date BETWEEN '2018-01-01' AND '2018-12-31'
  )
  
SELECT first_name, last_name
FROM patients
WHERE patient_id IN patients_2018;
```

Please note that you can create several CTE by separating them with a comma.

## Recap and resources to continue learning

This tutorial provided an introduction to SQL. While this is a good starting place, there are more things to learn! This tutorial draws from a longer [Introduction to databases workshop](https://github.com/nuitrcs/databases_workshop) taught by [Research Computing and Data Services at Northwestern University](https://www.it.northwestern.edu/departments/it-services-support/research/), as well as by this [resource guide](https://sites.northwestern.edu/researchcomputing/resource-guides/resource-guide-sql/). You can check them out to continue learning! 🧠💪

One key aspect that this workshop didn't cover is how to connect to a database. We used [sql-practice.com](https://www.sql-practice.com/), which makes it very easy to run SQL in a web browser for an introductory workshop. But in research, you'll probably be connecting to a database from a programming language like R or Python. Here you can see [how to connect to a database using the `DBI` package in R](https://github.com/nuitrcs/databases_workshop/blob/master/r/r_databases.Rmd) and [how to connect to a database using the `psycopg2` package in Python](https://github.com/nuitrcs/databases_workshop/blob/master/python/postgresql_from_python.ipynb). If you are affiliated with Northwestern University and you run into a problem during your research, you can always [submit a consult request](https://services.northwestern.edu/TDClient/30/Portal/Requests/ServiceDet?ID=93).
