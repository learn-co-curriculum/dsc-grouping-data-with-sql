
# Grouping Data with SQL

## Introduction

In this section, you'll learn how to use aggregate functions in SQL.

## Objectives

You will be able to:

* Write queries with aggregate functions like `COUNT`, `MAX`, `MIN`, and `SUM`
* Create an alias for the return value of an aggregate function
* Use `GROUP BY` to sort the data sets returned by aggregate functions
* Compare aggregates using the `HAVING` clause


```python
import sqlite3
import pandas as pd
```

## Database Schema
<img src="database-schema.png">

## Connecting to the Database


```python
conn = sqlite3.Connection('data.sqlite')
cur = conn.cursor()
```

## Groupby and Aggregate Functions

Lets start by looking at some groupby statements to aggregate our data.


```python
#Here we join the offices and employees tables in order to count the number of employees per city.
cur.execute("""select city,
                      count(employeeNumber)
                      from offices
                      join employees
                      using(officeCode)
                      group by city;""")
pd.DataFrame(cur.fetchall())
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
      <th>0</th>
      <th>1</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Boston</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>London</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NYC</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Paris</td>
      <td>5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>San Francisco</td>
      <td>6</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Sydney</td>
      <td>4</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Tokyo</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>



## Ordering and Aliasing
We can also alias our groupby by specifying the number of our selection order that we want to group by. Additionally, we can also order or limit our selection with the order by and limit clauses.


```python
cur.execute("""select city,
                      count(employeeNumber)
                      from offices
                      join employees
                      using(officeCode)
                      group by 1
                      order by count(employeeNumber) desc
                      limit 5;""")
pd.DataFrame(cur.fetchall())
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
      <th>0</th>
      <th>1</th>
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



## Retrieving Column Names
Recall that we can also retrieve our column names when using sqlite3 (note that this will be the default behavior in other environments such as sql workbench)


```python
cur.execute("""select city,
                      count(employeeNumber)
                      from offices
                      join employees
                      using(officeCode)
                      group by 1
                      order by count(employeeNumber) desc
                      limit 5;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
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
      <th>count(employeeNumber)</th>
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



## Aliasing our Aggregate Function Name
Now that we can view our column names, we can also practice using alias's to name our aggregations.


```python
cur.execute("""select city,
                      count(employeeNumber) as employeeCount
                      from offices
                      join employees
                      using(officeCode)
                      group by 1
                      order by count(employeeNumber) desc
                      limit 5;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
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
      <th>employeeCount</th>
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

Aside from count() some other useful aggregations include:
    * min()
    * max()
    * sum()
    * avg()


```python
cur.execute("""select customerName,
                      count(*) as number_purchases,
                      min(amount) as min_purchase,
                      max(amount) as max_purchase,
                      avg(amount) as avg_purchase,
                      sum(amount) as total_spent
                      from customers
                      join payments
                      using(customerNumber)
                      group by 1
                      order by sum(amount) desc;""")
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
      <td>116208.40</td>
      <td>65071.26</td>
      <td>55056.844615</td>
      <td>715738.98</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mini Gifts Distributors Ltd.</td>
      <td>9</td>
      <td>101244.59</td>
      <td>85410.87</td>
      <td>64909.804444</td>
      <td>584188.24</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Australian Collectors, Co.</td>
      <td>4</td>
      <td>44894.74</td>
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
      <td>105743.00</td>
      <td>44380.15</td>
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
      <td>1627.56</td>
      <td>7304.295000</td>
      <td>29217.18</td>
    </tr>
    <tr>
      <th>94</th>
      <td>Frau da Collezione</td>
      <td>2</td>
      <td>17746.26</td>
      <td>7612.06</td>
      <td>12679.160000</td>
      <td>25358.32</td>
    </tr>
    <tr>
      <th>95</th>
      <td>Atelier graphique</td>
      <td>3</td>
      <td>14571.44</td>
      <td>6066.78</td>
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



## The having clause

Finally, we can also filter our aggregated views with the having clause. The having clause works like the where clause but is used to filter data selections on conditions post the group by. For example, if we wanted to filter based on a customer's last name, we would use the where clause. However, if we wanted to filter a list of city's with at least 5 customers, we would using the having clause; we would first groupby city and count the number of customers, and the having clause allows us to pass conditions on the result of this aggregation.


```python
cur.execute("""select city,
                      count(customerNumber) as number_customers
                      from customers
                      group by 1
                      having count(customerNumber)>=5;""")
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



## Combining the where and having clause
We can also use jthe where and having clause in conjunction with each other for more complex rules.
For example, let's say we want a list of customers who have made at least 3 purchases of over 50K each.


```python
cur.execute("""select customerName,
                      count(amount) as number_purchases_over_50K
                      from customers
                      join payments
                      using(customerNumber)
                      where amount >= 50000
                      group by 1
                      having count(amount) >= 3
                      order by count(amount) desc;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
print(len(df))
df.head()
```

    53





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
      <td>13</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mini Gifts Distributors Ltd.</td>
      <td>9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Anna's Decorations, Ltd</td>
      <td>4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Australian Collectors, Co.</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Baane Mini Imports</td>
      <td>4</td>
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
      <th>number_purchases_over_50K</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>48</th>
      <td>Stylish Desk Decors, Co.</td>
      <td>3</td>
    </tr>
    <tr>
      <th>49</th>
      <td>Suominen Souveniers</td>
      <td>3</td>
    </tr>
    <tr>
      <th>50</th>
      <td>Toys of Finland, Co.</td>
      <td>3</td>
    </tr>
    <tr>
      <th>51</th>
      <td>Toys4GrownUps.com</td>
      <td>3</td>
    </tr>
    <tr>
      <th>52</th>
      <td>Vitachrome Inc.</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>



## Summary

After this section, you should have a good idea of how to use aggregate functions, aliases and the having clause to filter selections.
