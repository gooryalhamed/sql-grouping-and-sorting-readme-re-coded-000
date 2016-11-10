# Grouping and Sorting Data 

## Objectives 

1. Explain the importance of grouping and sorting data stored in a database
2. Group and sort data with the `GROUP BY()` and `ORDER BY()` keywords
3. Craft advanced queries using aggregator functions along with sorting keywords and other conditional clauses 

## Grouping and Sorting Data

SQL isn't picky about how it returns data to you, based on your queries. It will simply return the relevant table rows in the order in which they exist in the table. This is often insufficient for the purposes of data analysis and organization. 

How common is it to order a list of items alphabetically? Or numerically from least to greatest? 

We can tell our SQL queries and aggregate functions to group and sort our data using a number of clauses:

* `ORDER BY()`
* `LIMIT`
* `GROUP BY()`
* `HAVING` and `WHERE`
* `ASC`/`DESC`


Let's take a closer look at how we use these keywords to narrow our search criteria as well as to order and group it.

## Setting up the Database

Some cats are very famous, and accordingly very wealthy. Our Pet's Database will have a Cats table in which each cat has a name, age, breed and net worth. Our database will also have an Owners table and `cats_owners` join table so that a cat can have many owners and an owner can have many cats.

**Creating the Database:**

Create the database in your terminal with the following: 

```bash
sqlite3 pets_database.db
```

**Creating the tables:**

In the `sqlite3>` prompt in your terminal:

**Cats table:**

```sql
CREATE TABLE cats (
id INTEGER PRIMARY KEY,
name TEXT,
age INTEGER,
breed TEXT, 
net_worth INTEGER
);
```

**Owners Table:**

```sql
CREATE TABLE owners (id INTEGER PRIMARY KEY, name TEXT);
```

**Cats Owners Table:**

```sql
CREATE TABLE cats_owners (
cat_id INTEGER,
owner_id INTEGER
);
```

**Inserting the values:**

**Cats:**

```sql
INSERT INTO cats (id, name, age, breed, net_worth) VALUES (1, "Maru", 3, "Scottish Fold", 1000000);
INSERT INTO cats (id, name, age, breed, net_worth) VALUES (2, "Hana", 1, "Tabby", 21800);
INSERT INTO cats (id, name, age, breed, net_worth) VALUES (3, "Grumpy Cat", 4, "Persian", 181600);
INSERT INTO cats (id, name, age, breed, net_worth) VALUES (4, "Lil' Bub", 2, "Tortoiseshell", 2000000);
```

**Owners:**

```sql
INSERT INTO owners (name) VALUES ("mugumogu");
INSERT INTO owners (name) VALUES ("Sophie");
INSERT INTO owners (name) VALUES ("Penny");
```

**Cats Owners:**

```sql
INSERT INTO cats_owners (cat_id, owner_id) VALUES (3, 2);
INSERT INTO cats_owners (cat_id, owner_id) VALUES (3, 3);
INSERT INTO cats_owners (cat_id, owner_id) VALUES (1, 2);
```

### Code Along I: `ORDER BY()`

#### Syntax

```sql
SELECT column_name, column_name
FROM table_name
ORDER BY column_name ASC|DESC, column_name ASC|DESC;
```

`ORDER BY()` will automatically sort the returned values in ascending order. Use the `DESC` keyword, as above, to sort in descending order. 

#### Exercise

Imagine you're working for an important investment firm in Manhattan. The investors are interested in investing in a lucrative and popular cat. They need your help to decide which cat that will be. They want a list of famous and wealthy cats. We can do that with a basic `SELECT` statement:

```sql
SELECT * FROM cats WHERE net_worth > 0;
```

This will return:

```
name             age         breed          net_worth 
---------------  ----------  -------------  ----------
Maru             3           Scottish Fold  1000000   
Hana             1           Tabby          21800     
Grumpy Cat       4           Persian        181600    
Lil' Bub         2           Tortoiseshell  2000000   
```   

Our investors are busy people though. They don't have time to manually sort through this list of cats for the best candidate. They want you to return the list to them with the cats sorted by net worth, from greatest to least.  

We can do so with the following lines:

```sql
SELECT * FROM cats ORDER BY(net_worth) DESC;
```

This will return:

```
name             age         breed          net_worth 
---------------  ----------  -------------  ----------
Lil' Bub         2           Tortoiseshell  2000000   
Maru             3           Scottish Fold  1000000   
Grumpy Cat       4           Persian        181600    
Hana             1           Tabby          21800     
```

### Code Along II: The `LIMIT` Keyword

Turns out our investors are very impatient. They don't want to review the list themselves, they just want you to return to them the wealthiest cat. We can accomplish this by using the `LIMIT` keyword with the above query:

```sql
SELECT * FROM cats ORDER BY(net_worth) DESC LIMIT 1;
name             age         breed          net_worth 
---------------  ----------  -------------  ----------
Lil' Bub         2           Tortoiseshell  2000000   
```

The LIMIT keyword specifies how many of records that result from the query you'd like to actually return. 

### Code Along III: `GROUP BY()`

The `GROUP BY()` keyword is very similar to `ORDER BY()`. The only difference is that `ORDER BY()` sorts the resulting data set of basic queries while `GROUP BY()` sorts the result sets of aggregate functions. 

#### Syntax

```sql
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name;
```

#### Exercise

Let's calculate the sum of the net worth of all of the cats that belong to our second owner:

```sql
SELECT SUM(Cats.net_worth) 
FROM cats
INNER JOIN Cats
ON cats_owners.cat_id = Cats.id 
WHERE cats_owners.owner_id = 2;
```

This should return:

```
SUM(Cats.net_worth) 
--------------------
1181600 
```

In the above query, we use the `SUM(Cats.net_worth)` aggregator. `SUM` looks at all of the values in the `net_worth` column of the Cats table (or whatever column you specify in parentheses) and takes the sum of the those values.  

### Code Along IV: `Having` vs `Where` clause<sup>1</sup>  
Suppose we have a table called employee_bonus as shown below. Note that the table has multiple entries for employees Abigail and Matthew. 

employee_bonus

Employee   | Bonus
-----------|-------
Matthew    |1000|
Abigail    |2000|
Matthew    |500|
Tom     	 |700|
Abigail 	 |1250|  

To calculate the total bonus that each employee received, we would write a SQL statement like this:  

```sql
SELECT employee, sum(bonus) from employee_bonus group by employee;
```  

This should return:  

Employee   | Bonus
-----------|-------
Matthew    |1500|
Abigail    |3250|
Tom        |700|

Now, suppose we wanted to find the employees who received more than $1,000 in bonuses for the year of 2007. You might think that we could write a query like this:  

```sql  
BAD SQL:
select employee, sum(bonus) from employee_bonus 
group by employee where sum(bonus) > 1000;
```  

Unfortunately the above will not work because the where clause doesn’t work with aggregates – like sum, avg, max, etc. What we need to use is the `HAVING` clause. The having clause was added to sql just so we could compare aggregates to other values – just how the ‘where’ clause can be used with non-aggregates. Now, the correct sql will look like this:

```sql
GOOD SQL:
select employee, sum(bonus) from employee_bonus 
group by employee having sum(bonus) > 1000;
```  

#### Difference between having and where clause
The difference between the having and where clause in sql is that the where clause can not be used with aggregates, but the having clause can. One way to think of it is that the having clause is an additional filter to the where clause.  


## Resources: 
1. [Having vs Where clause](http://www.programmerinterview.com/index.php/database-sql/having-vs-where-clause/)

 
<p data-visibility='hidden'>View <a href='https://learn.co/lessons/sql-grouping-and-sorting-readme' title='Grouping and Sorting Data'>Grouping and Sorting Data</a> on Learn.co and start learning to code for free.</p>

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/sql-grouping-and-sorting-readme'>Grouping and Sorting Data</a> on Learn.co and start learning to code for free.</p>
