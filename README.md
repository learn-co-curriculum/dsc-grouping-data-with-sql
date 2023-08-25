# Grouping Data with SQL

## Introduction

Sometimes you may wish to find the mean, median, min, or max of a column feature. For example, there could be a customer relational database that you've been working with and you may wonder if there are differences in overall sales across offices or regions. We can use aggregate functions in SQL to assist with performing these analyses.

## Objectives

You will be able to:

* Describe the relationship between aggregate functions and `GROUP BY` statements
* Use `GROUP BY` statements in SQL to apply aggregate functions like: `COUNT`, `MAX`, `MIN`, and `SUM`
* Create an alias in a SQL query
* Use the `HAVING` clause to compare different aggregates
* Compare the difference between the `WHERE` and `HAVING` clause

## Entity Relationship Diagram

Once again we will be using this database, with 8 tables relating to customers, orders, employees, etc.

<img src="https://curriculum-content.s3.amazonaws.com/data-science/images/Database-Schema.png">

## Connecting to the Database

As usual, start by creating a connection to the database. We will also import pandas in order to display the results in a convenient format.


```python
import sqlite3
import pandas as pd
```


```python
conn = sqlite3.Connection('data.sqlite')
```

## `GROUP BY` and Aggregate Functions

Let's start by looking at some `GROUP BY` statements to aggregate our data. The `GROUP BY` clause groups records into summary rows and returns one record for each group.

Typically, `GROUP BY`  also involves an aggregate function (`COUNT`, `AVG`, etc.).

Lastly, `GROUP BY` can group by one column or multiple columns.

### Count of Customers by Country

One of the most common uses of `GROUP BY` is to count the number of records in each group. To do that, we'll also use the `COUNT` aggregate function.


```python
q = """
SELECT country, COUNT(*)
FROM customers
GROUP BY country
;
"""
# Displaying just the first 10 countries for readability
pd.read_sql(q, conn).head(10)
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
      <th>country</th>
      <th>COUNT(*)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Australia</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Austria</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Belgium</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Canada</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Denmark</td>
      <td>2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Finland</td>
      <td>3</td>
    </tr>
    <tr>
      <th>6</th>
      <td>France</td>
      <td>12</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Germany</td>
      <td>13</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Hong Kong</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Ireland</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



Cool, we have the number of customers per country!

### Interpreting `COUNT(*)`

Why did we pass in `*` to `COUNT(*)`?

`COUNT` is a function that is being invoked, similar to a function in Python. When we say to count `*`, we mean count every row containing non-null column values.

You will also see examples using `COUNT(1)`, which counts every row regardless of whether it contains non-null column values, or something like `COUNT(customerNumber)`, which just counts whether some particular column is non-null.

Most of the time this does not make a significant difference in the results produced or the processing speed, since databases have optimizers designed for this purpose. But it is useful to be able to recognize the various forms.

### Alternative `GROUP BY` Syntax

Another thing to be aware of is that instead of specifying an actual column name to group by, we can group the data using the index of one of the columns already specified in the `SELECT` statement. These are 1-indexed (unlike Python, which is 0-indexed). So an alternative way to write the previous query would be:


```python
q = """
SELECT country, COUNT(*)
FROM customers
GROUP BY 1
;
"""
# Displaying just the first 10 countries for readability
pd.read_sql(q, conn).head(10)
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
      <th>country</th>
      <th>COUNT(*)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Australia</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Austria</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Belgium</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Canada</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Denmark</td>
      <td>2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Finland</td>
      <td>3</td>
    </tr>
    <tr>
      <th>6</th>
      <td>France</td>
      <td>12</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Germany</td>
      <td>13</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Hong Kong</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Ireland</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



## Aliasing

An alias is a shorthand for a table or column name. Aliases reduce the amount of typing required to enter a query, and can result in both queries and results that are easier to read.

Aliases are especially useful with `JOIN`, `GROUP BY`, and aggregates (`SUM`, `COUNT`, etc.). For example, we could rewrite the previous query like this, so that the count of customers is called `customer_count` instead of `COUNT(*)`:


```python
q = """
SELECT country, COUNT(*) AS customer_count
FROM customers
GROUP BY country
;
"""
# Displaying just the first 10 countries for readability
pd.read_sql(q, conn).head(10)
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
      <th>country</th>
      <th>customer_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Australia</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Austria</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Belgium</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Canada</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Denmark</td>
      <td>2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Finland</td>
      <td>3</td>
    </tr>
    <tr>
      <th>6</th>
      <td>France</td>
      <td>12</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Germany</td>
      <td>13</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Hong Kong</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Ireland</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



Other notes on aliases:

* An alias only exists for the duration of the query.
* The keyword `AS` is optional in SQLite. So, you could just say `COUNT(*) customer_count` with the same outcome. Historically some forms of SQL required `AS` and others would not work with `AS`, but most work either way now. In a professional setting you will likely have a style guide indicating whether or not to use it.

## Other Aggregations

Aside from `COUNT()` some other useful aggregations include:
- `MIN()`
- `MAX()`
- `SUM()`
- `AVG()`

These are mainly useful when working with numeric data.

### Payment Summary Statistics

In the cell below, we calculate various summary statistics about payments, grouped by customer.


```python
q = """
SELECT
    customerNumber,
    COUNT(*) AS number_payments,
    MIN(amount) AS min_purchase,
    MAX(amount) AS max_purchase,
    AVG(amount) AS avg_purchase,
    SUM(amount) AS total_spent
FROM payments
GROUP BY customerNumber
;
"""
pd.read_sql(q, conn)
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
      <th>customerNumber</th>
      <th>number_payments</th>
      <th>min_purchase</th>
      <th>max_purchase</th>
      <th>avg_purchase</th>
      <th>total_spent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>103</td>
      <td>3</td>
      <td>1676.14</td>
      <td>14571.44</td>
      <td>7438.120000</td>
      <td>22314.36</td>
    </tr>
    <tr>
      <th>1</th>
      <td>112</td>
      <td>3</td>
      <td>14191.12</td>
      <td>33347.88</td>
      <td>26726.993333</td>
      <td>80180.98</td>
    </tr>
    <tr>
      <th>2</th>
      <td>114</td>
      <td>4</td>
      <td>7565.08</td>
      <td>82261.22</td>
      <td>45146.267500</td>
      <td>180585.07</td>
    </tr>
    <tr>
      <th>3</th>
      <td>119</td>
      <td>3</td>
      <td>19501.82</td>
      <td>49523.67</td>
      <td>38983.226667</td>
      <td>116949.68</td>
    </tr>
    <tr>
      <th>4</th>
      <td>121</td>
      <td>4</td>
      <td>1491.38</td>
      <td>50218.95</td>
      <td>26056.197500</td>
      <td>104224.79</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>93</th>
      <td>486</td>
      <td>3</td>
      <td>5899.38</td>
      <td>45994.07</td>
      <td>25908.863333</td>
      <td>77726.59</td>
    </tr>
    <tr>
      <th>94</th>
      <td>487</td>
      <td>2</td>
      <td>12573.28</td>
      <td>29997.09</td>
      <td>21285.185000</td>
      <td>42570.37</td>
    </tr>
    <tr>
      <th>95</th>
      <td>489</td>
      <td>2</td>
      <td>7310.42</td>
      <td>22275.73</td>
      <td>14793.075000</td>
      <td>29586.15</td>
    </tr>
    <tr>
      <th>96</th>
      <td>495</td>
      <td>2</td>
      <td>6276.60</td>
      <td>59265.14</td>
      <td>32770.870000</td>
      <td>65541.74</td>
    </tr>
    <tr>
      <th>97</th>
      <td>496</td>
      <td>3</td>
      <td>30253.75</td>
      <td>52166.00</td>
      <td>38165.730000</td>
      <td>114497.19</td>
    </tr>
  </tbody>
</table>
<p>98 rows × 6 columns</p>
</div>



### Filtered Payment Summary Statistics with `WHERE`

Similar to before we used `GROUP BY` and aggregations, we can use `WHERE` to filter the data. For example, if we only wanted to include payments made in 2004:


```python
q = """
SELECT
    customerNumber,
    COUNT(*) AS number_payments,
    MIN(amount) AS min_purchase,
    MAX(amount) AS max_purchase,
    AVG(amount) AS avg_purchase,
    SUM(amount) AS total_spent
FROM payments
WHERE strftime('%Y', paymentDate) = '2004'
GROUP BY customerNumber
;
"""
pd.read_sql(q, conn)
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
      <th>customerNumber</th>
      <th>number_payments</th>
      <th>min_purchase</th>
      <th>max_purchase</th>
      <th>avg_purchase</th>
      <th>total_spent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>103</td>
      <td>2</td>
      <td>1676.14</td>
      <td>6066.78</td>
      <td>3871.460</td>
      <td>7742.92</td>
    </tr>
    <tr>
      <th>1</th>
      <td>112</td>
      <td>2</td>
      <td>14191.12</td>
      <td>33347.88</td>
      <td>23769.500</td>
      <td>47539.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>114</td>
      <td>2</td>
      <td>44894.74</td>
      <td>82261.22</td>
      <td>63577.980</td>
      <td>127155.96</td>
    </tr>
    <tr>
      <th>3</th>
      <td>119</td>
      <td>2</td>
      <td>19501.82</td>
      <td>47924.19</td>
      <td>33713.005</td>
      <td>67426.01</td>
    </tr>
    <tr>
      <th>4</th>
      <td>121</td>
      <td>2</td>
      <td>17876.32</td>
      <td>34638.14</td>
      <td>26257.230</td>
      <td>52514.46</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>83</th>
      <td>486</td>
      <td>2</td>
      <td>5899.38</td>
      <td>45994.07</td>
      <td>25946.725</td>
      <td>51893.45</td>
    </tr>
    <tr>
      <th>84</th>
      <td>487</td>
      <td>1</td>
      <td>12573.28</td>
      <td>12573.28</td>
      <td>12573.280</td>
      <td>12573.28</td>
    </tr>
    <tr>
      <th>85</th>
      <td>489</td>
      <td>1</td>
      <td>7310.42</td>
      <td>7310.42</td>
      <td>7310.420</td>
      <td>7310.42</td>
    </tr>
    <tr>
      <th>86</th>
      <td>495</td>
      <td>1</td>
      <td>6276.60</td>
      <td>6276.60</td>
      <td>6276.600</td>
      <td>6276.60</td>
    </tr>
    <tr>
      <th>87</th>
      <td>496</td>
      <td>1</td>
      <td>52166.00</td>
      <td>52166.00</td>
      <td>52166.000</td>
      <td>52166.00</td>
    </tr>
  </tbody>
</table>
<p>88 rows × 6 columns</p>
</div>



Some additional notes:

* Look at the difference in the first row values. It appears that customer 103 made 3 payments in the database overall, but only made 2 payments in 2004. So this row still represents the same customer as in the previous query, but it contains different aggregated information about that customer.
* This returned 88 rows rather than 98, because some of the customers are present in the overall database but did not make any purchases in 2004. 
* Recall that you can filter based on something in a `WHERE` clause even if you do not `SELECT` that column. We are not displaying the `paymentDate` values because this would not make much sense in aggregate, but we can still use that column for filtering.

## The `HAVING` Clause

Finally, we can also filter our aggregated views with the `HAVING` clause. The `HAVING` clause works similarly to the `WHERE` clause, except it is used to filter data selections on conditions **after** the `GROUP BY` clause.

For example, if we wanted to filter to only select aggregated payment information about customers with average payment amounts over 50,000:


```python
q = """
SELECT
    customerNumber,
    COUNT(*) AS number_payments,
    MIN(amount) AS min_purchase,
    MAX(amount) AS max_purchase,
    AVG(amount) AS avg_purchase,
    SUM(amount) AS total_spent
FROM payments
GROUP BY customerNumber
HAVING avg_purchase > 50000
;
"""
pd.read_sql(q, conn)
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
      <th>customerNumber</th>
      <th>number_payments</th>
      <th>min_purchase</th>
      <th>max_purchase</th>
      <th>avg_purchase</th>
      <th>total_spent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>124</td>
      <td>9</td>
      <td>11044.30</td>
      <td>111654.40</td>
      <td>64909.804444</td>
      <td>584188.24</td>
    </tr>
    <tr>
      <th>1</th>
      <td>141</td>
      <td>13</td>
      <td>20009.53</td>
      <td>120166.58</td>
      <td>55056.844615</td>
      <td>715738.98</td>
    </tr>
    <tr>
      <th>2</th>
      <td>239</td>
      <td>1</td>
      <td>80375.24</td>
      <td>80375.24</td>
      <td>80375.240000</td>
      <td>80375.24</td>
    </tr>
    <tr>
      <th>3</th>
      <td>298</td>
      <td>2</td>
      <td>47375.92</td>
      <td>61402.00</td>
      <td>54388.960000</td>
      <td>108777.92</td>
    </tr>
    <tr>
      <th>4</th>
      <td>321</td>
      <td>2</td>
      <td>46781.66</td>
      <td>85559.12</td>
      <td>66170.390000</td>
      <td>132340.78</td>
    </tr>
    <tr>
      <th>5</th>
      <td>450</td>
      <td>1</td>
      <td>59551.38</td>
      <td>59551.38</td>
      <td>59551.380000</td>
      <td>59551.38</td>
    </tr>
  </tbody>
</table>
</div>



Note that in most flavors of SQL we can't use an alias in the `HAVING` clause. This is due to the internal order of execution of the SQL commands. So in most cases outside of SQLite you would need to write that query like this, repeating the aggregation code in the `HAVING` clause:


```python
q = """
SELECT
    customerNumber,
    COUNT(*) AS number_payments,
    MIN(amount) AS min_purchase,
    MAX(amount) AS max_purchase,
    AVG(amount) AS avg_purchase,
    SUM(amount) AS total_spent
FROM payments
GROUP BY customerNumber
HAVING AVG(amount) > 50000
;
"""
pd.read_sql(q, conn)
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
      <th>customerNumber</th>
      <th>number_payments</th>
      <th>min_purchase</th>
      <th>max_purchase</th>
      <th>avg_purchase</th>
      <th>total_spent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>124</td>
      <td>9</td>
      <td>11044.30</td>
      <td>111654.40</td>
      <td>64909.804444</td>
      <td>584188.24</td>
    </tr>
    <tr>
      <th>1</th>
      <td>141</td>
      <td>13</td>
      <td>20009.53</td>
      <td>120166.58</td>
      <td>55056.844615</td>
      <td>715738.98</td>
    </tr>
    <tr>
      <th>2</th>
      <td>239</td>
      <td>1</td>
      <td>80375.24</td>
      <td>80375.24</td>
      <td>80375.240000</td>
      <td>80375.24</td>
    </tr>
    <tr>
      <th>3</th>
      <td>298</td>
      <td>2</td>
      <td>47375.92</td>
      <td>61402.00</td>
      <td>54388.960000</td>
      <td>108777.92</td>
    </tr>
    <tr>
      <th>4</th>
      <td>321</td>
      <td>2</td>
      <td>46781.66</td>
      <td>85559.12</td>
      <td>66170.390000</td>
      <td>132340.78</td>
    </tr>
    <tr>
      <th>5</th>
      <td>450</td>
      <td>1</td>
      <td>59551.38</td>
      <td>59551.38</td>
      <td>59551.380000</td>
      <td>59551.38</td>
    </tr>
  </tbody>
</table>
</div>



## Combining the `WHERE` and `HAVING` Clauses

We can also use the `WHERE` and `HAVING` clauses in conjunction with each other for more complex rules.

For example, let's say we want to filter based on customers who have made **at least 2 purchases of over 50000 each**.

To convert that into SQL logic, that means we first want to limit the records to purchases over 50000 (using `WHERE`), then after aggregating, limit to customers who have made at least 2 purchases fitting that previous requirement (using `HAVING`).


```python
q = """
SELECT
    customerNumber,
    COUNT(*) AS number_payments,
    MIN(amount) AS min_purchase,
    MAX(amount) AS max_purchase,
    AVG(amount) AS avg_purchase,
    SUM(amount) AS total_spent
FROM payments
WHERE amount > 50000
GROUP BY customerNumber
HAVING number_payments >= 2
;
"""
pd.read_sql(q, conn)
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
      <th>customerNumber</th>
      <th>number_payments</th>
      <th>min_purchase</th>
      <th>max_purchase</th>
      <th>avg_purchase</th>
      <th>total_spent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>124</td>
      <td>5</td>
      <td>55639.66</td>
      <td>111654.40</td>
      <td>87509.512</td>
      <td>437547.56</td>
    </tr>
    <tr>
      <th>1</th>
      <td>141</td>
      <td>5</td>
      <td>59830.55</td>
      <td>120166.58</td>
      <td>85024.068</td>
      <td>425120.34</td>
    </tr>
    <tr>
      <th>2</th>
      <td>151</td>
      <td>2</td>
      <td>58793.53</td>
      <td>58841.35</td>
      <td>58817.440</td>
      <td>117634.88</td>
    </tr>
    <tr>
      <th>3</th>
      <td>363</td>
      <td>2</td>
      <td>50799.69</td>
      <td>55425.77</td>
      <td>53112.730</td>
      <td>106225.46</td>
    </tr>
  </tbody>
</table>
</div>



We can also use the `ORDER BY` and `LIMIT` clauses in queries containing these complex rules. Say we want to find the customer with the lowest total amount spent, who nevertheless fits the criteria described above. That would be:


```python
q = """
SELECT
    customerNumber,
    COUNT(*) AS number_payments,
    MIN(amount) AS min_purchase,
    MAX(amount) AS max_purchase,
    AVG(amount) AS avg_purchase,
    SUM(amount) AS total_spent
FROM payments
WHERE amount > 50000
GROUP BY customerNumber
HAVING number_payments >= 2
ORDER BY total_spent
LIMIT 1
;
"""
pd.read_sql(q, conn)
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
      <th>customerNumber</th>
      <th>number_payments</th>
      <th>min_purchase</th>
      <th>max_purchase</th>
      <th>avg_purchase</th>
      <th>total_spent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>363</td>
      <td>2</td>
      <td>50799.69</td>
      <td>55425.77</td>
      <td>53112.73</td>
      <td>106225.46</td>
    </tr>
  </tbody>
</table>
</div>



## Summary

In this lesson, you learned how to use aggregate functions, aliases, and the `HAVING` clause to filter selections.
