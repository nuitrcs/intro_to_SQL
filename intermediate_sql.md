# **Intermediate SQL**

* [Database for this workshop](#database-for-this-workshop)
* [Joins](#joins)
* [Table Names and Aliases](#table-names-and-aliases)
* [`UNION`, `EXCEPT`, and `INTERSECT`](#union-except-and-intersect)
* [Subqueries](#subqueries)
* [Common table expressions](#common-table-expressions-cte)
* [Window functions](#window-functions)
* [Recap and resources to continue learning](#recap-and-resources-to-continue-learning)
* [Answers to the exercises](#answers-to-the-exercises)

This tutorial assumes previous knowledge at the level of the introduction to SQL tutorial available in [this GitHub repository](https://github.com/nuitrcs/SQL_workshops).

## Database for this workshop

We're working with the `hospital` database from [sql-practice.com](https://www.sql-practice.com/). You can find the schema (the set of tables, their columns and types, and the relationships between them) in the left side bar > SQL Database > View Schema.

## Joins

Looking at the database schema, we can see that information is split between four tables.  The lines between the tables show where there is a column in one table that is linked to a column in another table.  These are called foreign keys. Foreign keys are used to link records in different tables.

In the table names, there are key icons next to some columns.  These columns are primary key columns. Primary keys uniquely identify records in the table they belong to.  A primary key can be a single column or a combination of multiple columns.  Primary keys have to have unique values.  They are frequently used to link tables to each other (although you could link tables with other columns too).

Now we'll learn how to join tables together. Diagrams such as [this one](https://images.app.goo.gl/U5RCPEhNaQuMn7y7A) can be useful to understand the different types of joins. Please note that you can join more than two tables together.

### Exercise 1

Look at the database schema and write down the column(s) that you can use to join these tables:

a) patients and admissions
b) admissions and doctors
c) patients and province\_names
d) patients and doctors

### `INNER JOIN`

The first and most common type of join is called an inner join.  You specify the tables to join, the conditions to use to match the tables up, and you get back the rows from both tables that meet the conditions.

Inner joins only return rows that have matching values in both tables.

```sql
SELECT *
FROM patients
INNER JOIN admissions ON patients.patient_id = admissions.patient_id;
```

Note that both tables have a column called patient\_id.  We add the table name to the front of the column name when referencing them. You can do this anytime, but typically only do it when you're joining and there's ambiguity. 

We can also group by, order by, and use other where clause conditions on the joined tables.  

### Exercise 2

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

A left join returns all records from the left table and only the matching records from the right table.

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

### Exercise 3 

a) Select the patient\_id of patients attended by a cardiologist.
b) Select the first and last name of patients who have been admitted, but don't have a diagnosis record.

### `FULL OUTER JOIN`

A `FULL OUTER JOIN` is like doing a left and right join at the same time: you get rows that are in both tables, plus rows from both tables that don't match the other table. 

This join is less commonly used.

The syntax is the same as the other joins.

### Exercise 4

Select all patients and their admission details, ensuring that all patients appear in the result even if they don’t have any admission record, and all admissions appear even if they don’t have corresponding patient details.

## `UNION`, `EXCEPT`, and `INTERSECT`

When working with SQL, there are times you may want to combine or compare the results of multiple queries. These operators – `UNION`, `EXCEPT`, and `INTERSECT` – allow you to do that. Each of these operators returns a new result set based on how the sets overlap or differ. All three operators require that the queries being combined are in the same order and have the same number of columns and compatible data types.

### `UNION`

The `UNION` operator is used to combine the result sets of two or more `SELECT` statements. It removes duplicate rows from the result set.

If you want to include duplicates, you can use `UNION ALL` instead. `UNION ALL` is typically faster since it doesn't check for duplicates.

For example, suppose we want a list of all patient\_ids that appear in either the patients table or the admissions table:

```sql
SELECT patient_id
FROM patients
UNION
SELECT patient_id
FROM admissions;
```

This query will return each patient\_id only once, even if the same ID appears in both tables. To keep duplicates, use `UNION ALL`:

```sql
SELECT patient_id
FROM patients
UNION ALL
SELECT patient_id
FROM admissions;
```

#### Exercise 5

a) Get a list of all first\_name values from both the patients and doctors tables. Ensure that duplicate names are included in the results if they appear in both tables.
b) List all province\_id values that are present in either patients or province\_names tables, excluding duplicates.

### `EXCEPT`

The `EXCEPT` operator returns rows from the first query that are not present in the second query’s results. It’s helpful if you want to find rows in one table but not in another.

For example, if we want a list of patient\_ids that are in the patients table but have not been admitted, we can write:

```sql
SELECT patient_id
FROM patients
EXCEPT
SELECT patient_id
FROM admissions;
```

This will return the patient\_ids of patients who have never been admitted (i.e., those in patients but not in admissions).

#### Exercise 6

Are there patient\_ids that appear in the admissions table but do not exist in the patients table?

### `INTERSECT`

The `INTERSECT` operator returns only the rows that appear in both result sets. It’s useful when you want to find overlapping records between two tables or queries.

For example, if we want a list of patient\_ids that appear in both the patients and admissions tables, we can use:

```sql
SELECT patient_id
FROM patients
INTERSECT
SELECT patient_id
FROM admissions;
```

This will return only the patient\_ids that are in both tables.

#### Exercise 7

a) Get a list of first\_name values that are common between the patients and doctors tables.
b) Get the province\_ids that appear in both the patients and province\_names tables, ensuring the provinces are referenced in both tables.

## Subqueries

We can use the results of one query as values in another query.

For example, this doesn't work: 

```sql
SELECT *
FROM patients
WHERE height = MAX(height);
```
Instead, you can use a subquery:

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

But you can also do the above query by joining tables together.  (`IN` is an expensive operation, meaning it can take a long time to run in large databases.)

### Exercise 8

Using subqueries, find the first\_name and last\_name of patients who have been attended by a cardiologist. (Hint: you may need to use more than two tables. The database schema is your friend!)

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
WHERE patient_id IN (SELECT patient_id FROM patients_2018);
```

Please note that you can create several CTE by separating them with a comma.

### Exercise 9

a) Create a CTE to list the patient\_ids of patients admitted in 2019, and then use it to retrieve first\_name, last\_name, and city of these patients from the patients table.
b) Using a CTE, find the number of admissions each doctor has attended. Then, list each doctor\_id, first\_name, last\_name, and the admission count. Only show doctors who have attended more than 5 admissions.

## Window Functions

Window functions allow you to perform calculations across a set of rows that are related to the current row, without collapsing rows into a single result (in contrast to aggregate functions). They’re helpful when you need calculations like running totals, rank, or moving averages.

In SQL, a window function is typically paired with an `OVER` clause, which defines the "window" of rows to include in the calculation.

### Basic Syntax

The basic syntax of a window function is:

```sql
SELECT column_name,
       WINDOW_FUNCTION() OVER (PARTITION BY column ORDER BY column) AS alias
FROM table_name;
```

Some common window functions include:

- `ROW_NUMBER()`: Assigns a unique number to each row, starting from 1.
- `RANK()`: Assigns a ranking number to each row, with ties receiving the same number, and the next number being skipped.
- `DENSE_RANK()`: Similar to `RANK()`, but without skipping numbers for ties.
- `SUM()`, `AVG()`, etc.: Performs aggregate calculations over a specified window.

#### Example: `ROW_NUMBER()`

If we want to assign a unique row number to each patient based on their admission_date, we could write:

```sql
SELECT patient_id,
       admission_date,
       ROW_NUMBER() OVER (ORDER BY admission_date) AS row_num
FROM admissions;
```

This assigns a unique row number to each record, ordered by admission\_date.

If there are ties (i.e., rows with identical values in admission\_date), `ROW_NUMBER()` does not treat them the same way. Instead, it assigns unique numbers in the order they appear, effectively "breaking" ties arbitrarily (often based on internal row ordering in the database).

#### Example: Partitioning with `PARTITION BY`

The `PARTITION BY` clause breaks the data into groups, and the window function is applied to each group independently.

For example, if we want to rank patients by their admission date within each province\_id, we can use:

```sql
SELECT a.patient_id,
       p.province_id,
       a.admission_date,
       RANK() OVER (PARTITION BY p.province_id ORDER BY a.admission_date) AS admission_rank
FROM admissions AS a
JOIN patients AS p ON a.patient_id = p.patient_id;
```

This query assigns a rank to each patient within their province based on admission\_date.

#### Example: Moving Average

Window functions also support calculations across a “moving” window of rows.

To calculate a moving average, you can use `ROWS` or `RANGE` within the `OVER` clause. `ROWS` and `RANGE` differ in how they define the window over which calculations are performed. `ROWS` defines the window strictly based on the physical order and number of rows relative to the current row. This is particularly useful when you want an exact number of adjacent rows (e.g., the previous two rows), regardless of their values in relation to each other. `RANGE` defines the window based on the values of the ordering column rather than the exact number of rows. This approach is useful when working with time-series data, as it allows you to include rows within a specific range of values, not just the closest rows.

Calculations across a "moving" window of rows can be useful for calculating trends, such as average admissions over time. For example, to calculate a moving average of 3 admissions, including the current row and the previous two rows:

```sql
WITH daily_admissions AS (
  SELECT admission_date,
         COUNT(patient_id) AS admissions_count
  FROM admissions
  GROUP BY admission_date
)

SELECT admission_date,
       admissions_count,
       AVG(admissions_count) OVER (ORDER BY admission_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_average
FROM daily_admissions;
```

#### Exercise 10

a) List each patient’s patient\_id, admission\_date, and their rank in terms of admission\_date across all admissions. Use the `ROW_NUMBER()` window function for ranking.
b) Using `PARTITION BY`, calculate the average weight for patients from each province. Display province\_id, patient\_id, and the calculated average weight per province.
c) Get the top three oldest patients in each province by birth\_date. Show province\_id, patient\_id, first\_name, last\_name, and birth\_date for each top patient.

## Recap and resources to continue learning

This tutorial provided an overview of intermediate SQL. While this is a good place to start using SQL in real life, there are more things to learn! This tutorial draws from a longer [Introduction to databases workshop](https://github.com/nuitrcs/databases_workshop) taught by [Research Computing and Data Services at Northwestern University](https://www.it.northwestern.edu/departments/it-services-support/research/), as well as by this [resource guide](https://sites.northwestern.edu/researchcomputing/resource-guides/resource-guide-sql/) and ChatGPT. You can check them out to continue learning! 🧠💪

One key aspect that this workshop didn't cover is how to connect to a database. We used [sql-practice.com](https://www.sql-practice.com/), which makes it very easy to run SQL in a web browser for an introductory workshop. But in research, you'll probably be connecting to a database from a programming language like R or Python. Here you can see [how to connect to a database using the `DBI` package in R](https://github.com/nuitrcs/databases_workshop/blob/master/r/r_databases.Rmd) and [how to connect to a database using the `psycopg2` package in Python](https://github.com/nuitrcs/databases_workshop/blob/master/python/postgresql_from_python.ipynb). If you are affiliated with Northwestern University and you run into a problem during your research, you can always [submit a consult request](https://services.northwestern.edu/TDClient/30/Portal/Requests/ServiceDet?ID=93).

## Answers to the exercises

### Exercise 1

a) patients and admissions: patient\_id
b) admissions and doctors: attending\_doctor\_id / doctor\_id
c) patients and province\_names: province\_id
d) patients and doctors: patient\_id to join with admissions and then attending\_doctor\_id / doctor\_id

### Exercise 2

a)

```sql
SELECT *
FROM patients 
INNER JOIN province_names ON patients.province_id = province_names.province_id;
```

b)

```sql
SELECT *
FROM admissions 
INNER JOIN doctors ON admissions.attending_doctor_id = doctors.doctor_id;
```

### Exercise 3

a)

```sql
SELECT a.patient_id
FROM doctors d
LEFT JOIN admissions a ON d.doctor_id = a.attending_doctor_id
WHERE d.specialty = 'Cardiologist';
```

b)

```sql
SELECT p.first_name, p.last_name
FROM admissions a
RIGHT JOIN patients p ON a.patient_id = p.patient_id
WHERE a.diagnosis IS NULL;
```

### Exercise 4

```sql
SELECT p.*, a.*
FROM patients p
FULL OUTER JOIN admissions a ON p.patient_id = a.patient_id;
```

### Exercise 5

a)

```sql
SELECT first_name
FROM patients
UNION ALL
SELECT first_name
FROM doctors;
```

b)

```sql
SELECT province_id
FROM patients
UNION
SELECT province_id
FROM province_names;
```

### Exercise 6

```sql
SELECT patient_id
FROM admissions
EXCEPT
SELECT patient_id
FROM patients;
```

### Exercise 7

a)

```sql
SELECT first_name
FROM patients
INTERSECT
SELECT first_name
FROM doctors;
```

b)

```sql
SELECT province_id
FROM patients
INTERSECT
SELECT province_id
FROM province_names;
```

### Exercise 8

```sql
SELECT first_name, last_name
FROM patients
WHERE patient_id IN (
  SELECT patient_id
  FROM admissions
  WHERE attending_doctor_id IN (
    SELECT doctor_id
    FROM doctors
    WHERE specialty = 'Cardiologist'
    )
  );
```

### Exercise 9

a) 

```sql
WITH admissions_2019 AS (
  SELECT patient_id
  FROM admissions
  WHERE admission_date BETWEEN '2019-01-01' AND '2019-12-31'
)
SELECT first_name, last_name, city
FROM patients
WHERE patient_id IN (SELECT patient_id FROM admissions_2019);
```

b) 

```sql
WITH doctor_admission_counts AS (
  SELECT attending_doctor_id, COUNT(*) AS admission_count
  FROM admissions
  GROUP BY attending_doctor_id
)
SELECT d.doctor_id, d.first_name, d.last_name, dac.admission_count
FROM doctors d
JOIN doctor_admission_counts dac ON d.doctor_id = dac.attending_doctor_id
WHERE dac.admission_count > 5;
```

### Exercise 10

a)

```sql
SELECT patient_id, admission_date,
       ROW_NUMBER() OVER (ORDER BY admission_date) AS admission_rank
FROM admissions;
```

b)

```sql
SELECT p.province_id, p.patient_id,
       AVG(p.weight) OVER (PARTITION BY p.province_id) AS avg_weight_province
FROM patients p;
```

c)

```sql
SELECT p.province_id, p.patient_id, p.first_name, p.last_name, p.birth_date
FROM (
  SELECT province_id, patient_id, first_name, last_name, birth_date,
         ROW_NUMBER() OVER (PARTITION BY province_id ORDER BY birth_date) AS rank
  FROM patients
) p
WHERE rank <= 3;
```
