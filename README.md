
# Grouping Data with SQL

## Introduction

Just as with Pandas, we can use aggregate functions in SQL to assist with data manipulation. Sometimes you may wish to find the mean, median, min, or max of a column feature. For example, there could be a customer relational database that you've been working with and you may wonder if there are differences in overall sales across offices or regions.

## Objectives

You will be able to:

* Describe the relationship between aggregate functions and `GROUP BY` statements
* Use `GROUP BY` statements in SQL to apply aggregate functions like: `COUNT`, `MAX`, `MIN`, and `SUM`
* Create an alias in a SQL query
* Use the `HAVING` clause to compare different aggregates
* Compare the difference between the `WHERE` and `HAVING` clause

## Database Schema
<img src="images/Database-Schema.png">


```python
import sqlite3
import pandas as pd
```

## Connecting to the Database

As usual, start by creating a connection to the database and instantiating a cursor object.


```python
conn = sqlite3.Connection('data.sqlite')
cur = conn.cursor()
```

## `GROUP BY` and Aggregate Functions

Let's start by looking at some `GROUP BY` statements to aggregate our data. The `GROUP BY` clause groups records into summary rows and returns one record for each group.
Typically, `GROUP BY`  also involves an aggregate function (`COUNT`, `AVG`, etc.). Lastly, `GROUP BY` can group by one or more columns.

In the cell below, we'll join the offices and employees tables in order to count the number of employees per city.


```python
cur.execute("""SELECT city, COUNT(employeeNumber)
                      FROM offices
                      JOIN employees
                      USING(officeCode)
                      GROUP BY city
                      ORDER BY count(employeeNumber) DESC;""")
df = pd.DataFrame(cur.fetchall())
df.columns = [x[0] for x in cur.description]
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>COUNT(employeeNumber)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>San Francisco</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Paris</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sydney</td>
      <td>4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Boston</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>London</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



## Aliasing

An Alias is a shorthand for a table or column name. Aliases reduce the amount of typing required to enter a query. Generally, complex queries with aliases are easier to read.
Aliases are useful with `JOIN`, `GROUP BY`, and aggregates (`SUM`, `COUNT`, etc.).
An Alias only exists for the duration of the query.

You can alias your `GROUP BY` by specifying the index of our selection order that we want to group by. This is simply written as `GROUP BY 1`, with the number "1" referring to the first column name that we are selecting.

Additionally, we can also rename our aggregate to a more descriptive name using the `AS` clause.


```python
cur.execute("""SELECT city, COUNT(employeeNumber) AS numEmployees
               FROM offices
               JOIN employees
               USING(officeCode)
               GROUP BY 1
               ORDER BY numEmployees DESC;""")
df = pd.DataFrame(cur.fetchall())
df.columns = [x[0] for x in cur.description]
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>numEmployees</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>San Francisco</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Paris</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sydney</td>
      <td>4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Boston</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>London</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



## Other Aggregations

Aside from `COUNT()` some other useful aggregations include:
- `MIN()`
- `MAX()`
- `SUM()`
- `AVG()`


```python
cur.execute("""SELECT customerName,
               COUNT(customerName) AS number_purchases,
               MIN(amount) AS min_purchase,
               MAX(amount) AS max_purchase,
               AVG(amount) AS avg_purchase,
               SUM(amount) AS total_spent
               FROM customers
               JOIN payments
               USING(customerNumber)
               GROUP BY customerName
               ORDER BY SUM(amount) DESC;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
print(len(df))
df.head()
```

    98





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>customerName</th>
      <th>number_purchases</th>
      <th>min_purchase</th>
      <th>max_purchase</th>
      <th>avg_purchase</th>
      <th>total_spent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Euro+ Shopping Channel</td>
      <td>13</td>
      <td>20009.53</td>
      <td>120166.58</td>
      <td>55056.844615</td>
      <td>715738.98</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mini Gifts Distributors Ltd.</td>
      <td>9</td>
      <td>11044.30</td>
      <td>111654.40</td>
      <td>64909.804444</td>
      <td>584188.24</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Australian Collectors, Co.</td>
      <td>4</td>
      <td>7565.08</td>
      <td>82261.22</td>
      <td>45146.267500</td>
      <td>180585.07</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Muscle Machine Inc</td>
      <td>4</td>
      <td>20314.44</td>
      <td>58841.35</td>
      <td>44478.487500</td>
      <td>177913.95</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Dragon Souveniers, Ltd.</td>
      <td>4</td>
      <td>2611.84</td>
      <td>105743.00</td>
      <td>39062.757500</td>
      <td>156251.03</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>customerName</th>
      <th>number_purchases</th>
      <th>min_purchase</th>
      <th>max_purchase</th>
      <th>avg_purchase</th>
      <th>total_spent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>93</th>
      <td>Royale Belge</td>
      <td>4</td>
      <td>1128.20</td>
      <td>14379.90</td>
      <td>7304.295000</td>
      <td>29217.18</td>
    </tr>
    <tr>
      <th>94</th>
      <td>Frau da Collezione</td>
      <td>2</td>
      <td>7612.06</td>
      <td>17746.26</td>
      <td>12679.160000</td>
      <td>25358.32</td>
    </tr>
    <tr>
      <th>95</th>
      <td>Atelier graphique</td>
      <td>3</td>
      <td>1676.14</td>
      <td>14571.44</td>
      <td>7438.120000</td>
      <td>22314.36</td>
    </tr>
    <tr>
      <th>96</th>
      <td>Auto-Moto Classics Inc.</td>
      <td>3</td>
      <td>5858.56</td>
      <td>9658.74</td>
      <td>7184.753333</td>
      <td>21554.26</td>
    </tr>
    <tr>
      <th>97</th>
      <td>Boards &amp; Toys Co.</td>
      <td>2</td>
      <td>3452.75</td>
      <td>4465.85</td>
      <td>3959.300000</td>
      <td>7918.60</td>
    </tr>
  </tbody>
</table>
</div>



## The `HAVING` clause

Finally, we can also filter our aggregated views with the `HAVING` clause. The `HAVING` clause works similarly to the `WHERE` clause, except it is used to filter data selections on conditions **after** the `GROUP BY` clause. For example, if we wanted to filter based on a customer's last name, we would use the `WHERE` clause. However, if we wanted to filter a list of cities with at least 5 customers, we would use the `HAVING` clause. First, we would `GROUP BY` city and then use the `HAVING` clause, which will allow us to pass conditions on the result of this aggregation.


```python
cur.execute("""SELECT city, COUNT(customerNumber) AS number_customers
               FROM customers
               GROUP BY 1
               HAVING COUNT(customerNumber)>=5;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
print(len(df))
df.head()
```

    2





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>number_customers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Madrid</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NYC</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>



## Combining the `WHERE` and `HAVING` clause
We can also use the `WHERE` and `HAVING` clauses in conjunction with each other for more complex rules.
For example, let's say we want a list of customers who have made at least 2 purchases of over 50K each.


```python
cur.execute("""SELECT customerName,
               COUNT(amount) AS number_purchases_over_50K
               FROM customers
               JOIN payments
               USING(customerNumber)
               WHERE amount >= 50000
               GROUP BY customerName
               HAVING count(amount) >= 2
               ORDER BY count(amount) DESC;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
print(len(df))
df.head()
```

    4





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>customerName</th>
      <th>number_purchases_over_50K</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Euro+ Shopping Channel</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mini Gifts Distributors Ltd.</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Muscle Machine Inc</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Online Diecast Creations Co.</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



## Summary

In this lesson, you learned how to use aggregate functions, aliases, and the `HAVING` clause to filter selections.
