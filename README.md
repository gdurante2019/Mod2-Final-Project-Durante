
# Final Project Submission, Module 2

- Student name: Gina Durante
- Student pace: full time (4/15/19 cohort)
- Instructor name: Rafael Carrasco
- Blog post URL: https://gdurante2019.github.io (when completed)

# Project Description

## Goals

The goal of the project is to use statistical analysis of data in the Northwind SQL database to answer at least four questions of business relevance to Northwind, an importer of food and beverage products.  To answer these questions using statistical analysis, it will be necessary to perform hypothesis testing.  

The first of these four questions is a required question:  **_Do discounts have a statistically significant effect on the number of products customers order? If so, at what level(s) of discount?_**

## Deliverables

The following deliverables are required:

1. A **_Jupyter Notebook_** containing any code written for this project. 
2. A **_Blog Post_** explaining process, methodology, and findings.  
3. An **_"Executive Summary" PowerPoint Presentation_** that explains the hypothesis tests run, findings, and relevance to company stakeholders.  
4.  A **_Video Walkthrough_** of the “Executive Summary” presentation. 


## Questions Addressed
1.  (Required) Do discounts have a statistically significant effect on the average **quantity of products per order**? 
2.   Do discounts have a statistically significant effect on the average **revenue per order**?
3.   Does the _level_ of discount (e.g., 5%, 10%, 15%...) have a statistically significant effect on the average **quantity of products per order**?
4.  Does the _level_ of discount (e.g., 5%, 10%, 15%...) have a statistically significant effect on the average **revenue per order**?
5.  Do the revenue distributions differ in a statistically significant manner from one product category to the next?  Put another way, do some product categories have statistically significantly higher revenues per order than others?

## Importing data

#### Import libraries


```python
import pandas as pd
from sqlite3 import *
from sqlalchemy import *
import pprint as pp
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline

```

#### Create connection object


```python
engine = create_engine('sqlite:///Northwind_small.sqlite', echo=False)
con = engine.connect()
```

#### Inspect the database schema (table names and column names)


```python
# Get names of tables in database

inspector = inspect(engine)
print("Table names")
table_names = inspector.get_table_names()
pp.pprint(inspector.get_table_names())
print()

```

    Table names
    ['Category',
     'Customer',
     'CustomerCustomerDemo',
     'CustomerDemographic',
     'Employee',
     'EmployeeTerritory',
     'Order',
     'OrderDetail',
     'Product',
     'Region',
     'Shipper',
     'Supplier',
     'Territory',
     'placeholder']
    



```python
# Get column names for a specific table
    
def print_col_names(var_table):
    var_table = var_table
    print(f"Column names for table {var_table}:")
    for i in inspector.get_columns(var_table):
        print(i['name'])
    print()

```


```python
print_col_names('Customer')
```

    Column names for table Customer:
    Id
    CompanyName
    ContactName
    ContactTitle
    Address
    City
    Region
    PostalCode
    Country
    Phone
    Fax
    



```python
print_col_names('Order')
```

    Column names for table Order:
    Id
    CustomerId
    EmployeeId
    OrderDate
    RequiredDate
    ShippedDate
    ShipVia
    Freight
    ShipName
    ShipAddress
    ShipCity
    ShipRegion
    ShipPostalCode
    ShipCountry
    



```python
print_col_names('OrderDetail')
```

    Column names for table OrderDetail:
    Id
    OrderId
    ProductId
    UnitPrice
    Quantity
    Discount
    



```python
print_col_names('Product')
```

    Column names for table Product:
    Id
    ProductName
    SupplierId
    CategoryId
    QuantityPerUnit
    UnitPrice
    UnitsInStock
    UnitsOnOrder
    ReorderLevel
    Discontinued
    



```python
print_col_names('Category')
```

    Column names for table Category:
    Id
    CategoryName
    Description
    


#### Importing data and setting up dataframes


```python
# Create a connection object and a cursor object to execute SQL commands
engine = create_engine('sqlite:///Northwind_small.sqlite', echo=False)
con = engine.connect()

# Set up query
query = """
        SELECT o.id, o.CustomerID, o.OrderDate, 
        od.ProductID, od.UnitPrice as [OrderUnitPrice], 
        od.Quantity as [OrderQty], od.Discount, 
        p.ProductName, p.CategoryId, 
        cat.CategoryName, cat.Description as [CatDescription]
        FROM [Order] as o 
        JOIN OrderDetail as oD
        ON o.ID = od.OrderID   
        JOIN Product as p
        ON od.ProductID = p.ID
        JOIN Category as cat
        ON cat.id = p.CategoryId
        ORDER BY ProductID
        """

# Execute query and create df from results
res = con.execute(query)
df = pd.read_sql(query, con)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10285</td>
      <td>QUICK</td>
      <td>2012-08-20</td>
      <td>1</td>
      <td>14.4</td>
      <td>45</td>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10294</td>
      <td>RATTC</td>
      <td>2012-08-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>18</td>
      <td>0.00</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10317</td>
      <td>LONEP</td>
      <td>2012-09-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>20</td>
      <td>0.00</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10348</td>
      <td>WANDK</td>
      <td>2012-11-07</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10354</td>
      <td>PERIC</td>
      <td>2012-11-14</td>
      <td>1</td>
      <td>14.4</td>
      <td>12</td>
      <td>0.00</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
  </tbody>
</table>
</div>



#### Function to pull original data from sqlite db and load into pandas dataframe


```python
# Function load_df() to pull original data from sqlite db and load into pandas 
# dataframe.  This function is used as a subfunction later in the notebook 
# to enable reconstruction of the database if needed.


def load_df(query):
    
    # Create a connection object and a cursor object to execute SQL commands
    engine = create_engine('sqlite:///Northwind_small.sqlite', echo=False)
    con = engine.connect()
    
    # Set up query
    query = """
            SELECT o.id, o.CustomerID, o.OrderDate, 
            od.ProductID, od.UnitPrice as [OrderUnitPrice], 
            od.Quantity as [OrderQty], od.Discount, 
            p.ProductName, p.CategoryId, 
            cat.CategoryName, cat.Description as [CatDescription]
            FROM [Order] as o 
            JOIN OrderDetail as oD
            ON o.ID = od.OrderID   
            JOIN Product as p
            ON od.ProductID = p.ID
            JOIN Category as cat
            ON p.CategoryId = cat.Id
            ORDER BY ProductID
            """
    
    # Execute query and create df from results
    res = con.execute(query)
    df = pd.read_sql(query, con)
    return df

```


```python
df = load_df(query)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10285</td>
      <td>QUICK</td>
      <td>2012-08-20</td>
      <td>1</td>
      <td>14.4</td>
      <td>45</td>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10294</td>
      <td>RATTC</td>
      <td>2012-08-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>18</td>
      <td>0.00</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10317</td>
      <td>LONEP</td>
      <td>2012-09-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>20</td>
      <td>0.00</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10348</td>
      <td>WANDK</td>
      <td>2012-11-07</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10354</td>
      <td>PERIC</td>
      <td>2012-11-14</td>
      <td>1</td>
      <td>14.4</td>
      <td>12</td>
      <td>0.00</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
  </tbody>
</table>
</div>



# First question:  Quantity and discount/no discount

Do discounts result in a statistically significant effect on **_quantity of products per order_**?


## Null and alternative hypotheses

-  **Null hypothesis:** No statistically significant difference between the means of quantities of discounted products purchased per order versus quantities of non-discounted products purchased per order
    -  Ho:  $\mu_1 = \mu_2$

-  **Alternative hypothesis:**  Statistically significant differences between the means of quantities of discounted products purchased per order versus quantities of non-discounted products purchased per order are unlikely to be due to chance
    -  Ha:  $\mu_1 \neq \mu_2$

## EDA

###  Create dataframes looking at subsets of data

Create dataframes that contain products ordered without discounts and products ordered that were discounted:


```python
df_no_disc = df.loc[df['Discount'] == 0]
df_no_disc.head()
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>10294</td>
      <td>RATTC</td>
      <td>2012-08-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>18</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10317</td>
      <td>LONEP</td>
      <td>2012-09-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>20</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10354</td>
      <td>PERIC</td>
      <td>2012-11-14</td>
      <td>1</td>
      <td>14.4</td>
      <td>12</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>6</th>
      <td>10406</td>
      <td>QUEE</td>
      <td>2013-01-07</td>
      <td>1</td>
      <td>14.4</td>
      <td>10</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>7</th>
      <td>10413</td>
      <td>LAMAI</td>
      <td>2013-01-14</td>
      <td>1</td>
      <td>14.4</td>
      <td>24</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_disc = df.loc[df['Discount'] > 0]
df_disc.head()
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10285</td>
      <td>QUICK</td>
      <td>2012-08-20</td>
      <td>1</td>
      <td>14.4</td>
      <td>45</td>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10348</td>
      <td>WANDK</td>
      <td>2012-11-07</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10370</td>
      <td>CHOPS</td>
      <td>2012-12-03</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10522</td>
      <td>LEHMS</td>
      <td>2013-04-30</td>
      <td>1</td>
      <td>18.0</td>
      <td>40</td>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10526</td>
      <td>WARTH</td>
      <td>2013-05-05</td>
      <td>1</td>
      <td>18.0</td>
      <td>8</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
    </tr>
  </tbody>
</table>
</div>



### Create plotting functions for visualizations


```python
# Functions for plotting average order quantities

def hist_plot1_qty(series1, label=None, color='darkcyan', alpha=0.7, bins=30, xlim=[0, 150]):
    plt.hist(series1, label=label, color=color, alpha=alpha, bins=bins)
    plt.title(f"Order quantity by occurrence")
    plt.xlabel("Order Quantity")
    plt.ylabel("Number")
    plt.legend(loc='best')
    plt.xlim(xlim)
    plt.show()
    
# function for plotting two series    
def hist_plot2_qty(series1, series2, label=None, label1=None, color1='b', color2='r', alpha=0.4, bins=30, xlim=[0, 150]):
    plt.hist(series1, label=label, color=color1, alpha=alpha, bins=bins)
    plt.hist(series2, label=label1, color=color2, alpha=alpha, bins=bins)
    plt.title(f"Order quantity by occurrence")
    plt.xlabel("Order Quantity")
    plt.ylabel("Number")
    plt.legend(loc='best')
    plt.xlim(xlim)  
    plt.show()
    
# function for plotting three different series 
def hist_plot3_qty(var1, var2, var3, label=None, label1=None, label2=None, color='b', alpha=0.5, bins=30, xlim=[0, 150]):
    var1 = var1
    var2 = var2
    var3 = var3
    alpha = alpha
    plt.hist(var1, label=label, color='r', alpha=0.5, bins=bins)
    plt.hist(var2, label=label1, color='b', alpha=0.5, bins=bins)
    plt.hist(var3, label=label2, color='g', alpha=0.5, bins=bins)
    plt.title(f"Order quantity by occurrence")
    plt.xlabel("Order Quantity")
    plt.ylabel("Number")
    plt.legend(loc='best')
    plt.xlim(xlim)
    plt.show()

```

### Histogram of total average order quantity by average order quantity for no discount vs. some discount


```python
# plt.figure(figsize=(8, 6))
hist_plot2_qty(df_no_disc.OrderQty, df_disc.OrderQty, label='No discounts (X = 0% discount)', label1='All discounts (X = 1%-25%+ discount)')
```


![png](student_files/student_33_0.png)


The histogram shows the distribution of order quantities values (x-axis).  The distribution of both order quantities without discounts and total order quantities (whether discounted or not) appear very similar, with the smallest order amounts appearing at the far left hand side of graph and the highest order order amounts appearing at the far right.  

If discounts have an effect on the quantity of products sold, we would expect to see the distribution of quantities sold from discounts look different from the distribution of quantities sold that didn't involve discounts.  If the shape of the distribution changes, such that a greater proportion of quantities appear in the middle or far right-hand-side of the graph, then that difference in distribution suggests that discounts increase the quantity of products in a given order.  

## Testing for significance:  sampling, test setup and results 

To test whether or not the presence of a discount likely has an effect on the quantity of products sold, we can use a two-sided t-test.  Two prerequisites for performing this test are that 1) both datasets have approximately normal distributions and 2) the variances of the two datasets are equal.  

### Test chosen:  t-test (2-sided, 2-sample) 
We can see that the above distributions are not normal.  Before proceding with the 2-sides, 2-sample t-test, we must first address this issue.

Fortunately, we can obtain a normal distribution (per the Central Limit Theorem) by sampling each dataset multiple times and computing each sample's mean.  With a large enough number of samples (n > 30) we can expect that the distribution of the means will be normal or very nearly normal.  We can take these normal distributions and run statistical tests on them.  

Each time the cells below are run, 100 samples are taken 300 times for each of the two populations (orders with no discounts and orders with some level of discount).  The mean and standard deviation are computed for each of the 300 sampling events, and each of the resulting means and standard deviations is appended to the respective relevant list.  

Multiple sampling results in different results each time; however, because there are well over 30 means and standard deviations, we can assume that the distributions of these means and standard deviations are normal, per the Central Limit Theorem.  We can now perform hypothesis tests requiring normal distributions of statistics on these means and standard deviations.

### Sampling function to obtain normal distribution of means


```python
def sampling_mean(var, sampling_events=300, size=100):   # var is the dataframe and column of interest
    import numpy as np
    var = var
    samp_mean = []
    samp_std = []
    for i in range(sampling_events):
        sample = np.random.choice(var, size=size, replace=False)
        samp_mean.append(sample.mean())
        samp_std.append(sample.std())
    return samp_mean, samp_std

```


```python
sampling_no_disc_mean, sampling_no_disc_std = sampling_mean(df_no_disc.OrderQty)
print(np.mean(sampling_no_disc_mean))
print(np.std(sampling_no_disc_std))
```

    21.6004
    2.3409277308454226


### Histograms of sampling distributions


```python
hist_plot1_qty(sampling_no_disc_mean, label="Order qty, no discounts", xlim=None)

```


![png](student_files/student_42_0.png)



```python
sampling_w_disc_mean, sampling_w_disc_std = sampling_mean(df_disc.OrderQty)
print(np.mean(sampling_w_disc_mean))
print(np.std(sampling_w_disc_std))

```

    27.1053
    2.24426228479168



```python
hist_plot1_qty(sampling_w_disc_mean, label="Order qty, w/discounts", xlim=None)

```


![png](student_files/student_44_0.png)


### Barlett's Test for variance

The t-test performed above requires not only that the samples come from approximately normally distributed populations, but also that they come from populations that have approximately equal variances.  We use Bartlett’s test on normal populations to test the null hypothesis that the samples are from populations with equal variances. 


```python
import scipy.stats

scipy.stats.bartlett(sampling_no_disc_mean, sampling_w_disc_mean)
```




    BartlettResult(statistic=2.564061882357949, pvalue=0.10931738849981974)



When the p-value is > 0.05, we **_cannot_** reject the null hypothesis that the samples come from populations with similar or equal variances.  

One interesting point is that there appears to be more fluctuation in terms of means and distributions for order quantities than for order revenues (presumably because the revenue numbers are much larger than the quantity numbers).  Sometimes the distributions that result from resampling pass the Bartlett test, and sometimes they don't.  One thing that seems fairly consistent, though, is that the distributions are reasonably normal and the shapes for each are similar--meaning that it the distributions might not always pass Bartlett's test, but when they don't, they aren't so very far off the mark.  

Because of this, I would still like to run the t-test to find out what results we get.

### T-test for two independent samples


```python
import scipy.stats as scs

scs.ttest_ind(sampling_no_disc_mean, sampling_w_disc_mean)
```




    Ttest_indResult(statistic=-35.273313157553034, pvalue=3.1316578958420464e-148)



The p-value is <<< 0.05, so if the other conditions for this test (normality, similar variances) are met, we **should reject the null hypothesis** that the samples come from populations with the same means.  Now, depending on the particular samples that are pulled when the model is run, the distributions may or may not pass Bartlett's test.  If we do not pass Bartlett's test on a particular running of the model, we can take a look at the distributions to see how similar or dissimilar they are to each other and figure out if an alterative test would be better.

If we decide to perform this test and we get a p-value of << 0.05, then it's likely that there is an actual statistically significant difference between the populations that is not due to chance.  But what is the effect size of that difference?  Enter Cohen's d.

### Effect size:  Cohen's d

Cohen's d allows us to view the magnitude of the effect of a statistically significant difference.  From the "Trending Sideways:  Practical Engineering and Statistics Updates" blog page, Cohen’s d is "simply a measure of the distance between two means, measured in standard deviations."  So, we could expect that the majority of analyses will yield Cohen's d results in a range from 0 to 3 or 4 (representing the differences between two means in terms of standard deviations).  


```python
# From learn.co Section 20 Effect Size lesson, with conditional print statements added

def Cohen_d(group1, group2, print_stmt=True):

    # Compute Cohen's d.

    # group1: Series or NumPy array  ----> in this case, the array is the list of sample means from resampling
    # group2: Series or NumPy array  ----> in this case, the array is the list of sample means from resampling

    # returns a floating point number 

    diff = group1.mean() - group2.mean()

    n1, n2 = len(group1), len(group2)
    var1 = group1.var()
    var2 = group2.var()

    # Calculate the pooled threshold as shown earlier
    pooled_var = (n1 * var1 + n2 * var2) / (n1 + n2)
    
    # Calculate Cohen's d statistic
    d = diff / np.sqrt(pooled_var)
    
    if print_stmt:           # changing print_stmt argument in the function call to False will turn off these print statements
        if abs(d) >= 0 and abs(d) < .4:
            print(f"This Cohen's d result ({round(d, 8)}) represents a difference of {round(d, 2)} pooled standard deviations between the two groups, and suggests a **small** effect size.")
        elif abs(d) >= .4 and abs(d) < .7:
            print(f"This Cohen's d result ({round(d, 8)}) represents a difference of {round(d, 2)} pooled standard deviations between the two groups, and suggests a **medium** effect size.")
        else:
            print(f"This Cohen's d result ({round(d, 8)}) represents a difference of {round(d, 2)} pooled standard deviations between the two groups, and suggests a **large** effect size.")
    
    return d
```


```python
group1 = np.array(sampling_w_disc_mean)
group2 = np.array(sampling_no_disc_mean)

d = Cohen_d(group1, group2)
d

```

    This Cohen's d result (2.88486608) represents a difference of 2.88 pooled standard deviations between the two groups, and suggests a **large** effect size.





    2.884866082701662



The functions below allow the creation of plots showing the overlap of two randomly generated normal distributions look like at a given Cohen's d.  Following each pair's Cohen's d result is a visualization of the amount of overlap the given Cohen's d provides.  The functions below are adapted from the Effect Sizes lesson in Section 20 of Module 2 of the Flatiron Full-Time Online Data Science Bootcamp.


```python
def evaluate_PDF(var, x=4):
    '''Input: a random variable object, standard deviation
       output : x and y values for the normal distribution
       '''
    
    # Identify the mean and standard deviation of random variable 
    mean = var.mean()
    std = var.std()

    # Use numpy to calculate evenly spaced numbers over the specified interval (4 sd) and generate 100 samples.
    xs = np.linspace(mean - x*std, mean + x*std, 100)
    
    # Calculate the peak of normal distribution i.e. probability density. 
    ys = var.pdf(xs)

    return xs, ys # Return calculated values

def plot_pdfs(cohen_d=2):
    """Plot randomly generated, normally distributed PDFs for 
    distributions that differ by some number of stds.
    
    cohen_d: number of standard deviations between the means
    """
    group1 = scipy.stats.norm(cohen_d,  1)
    group2 = scipy.stats.norm(0, 1)
    xs, ys = evaluate_PDF(group1)
    plt.fill_between(xs, ys, label='Group1', color='#ff2289', alpha=0.7)

    xs, ys = evaluate_PDF(group2)
    plt.fill_between(xs, ys, label='Group2', color='#376cb0', alpha=0.7)


```


```python
plot_pdfs(d)
```


![png](student_files/student_58_0.png)


Red = group1 = all discounted products;  
Blue = group2 = all non-discounted products

This graph provides a way to visualize the difference of the means, as provided by the Cohen's d test result.  Cohen's d is a measure of the distance between the means, depicted in units of pooled standard deviation.  In this case, 

## Findings and interpretation of test results for Question 1

### Finding
There is a statistically significant difference in the **_quantity of products per order for non-discounted products vs. discounted products._**

### Guidelines for interpreting Cohen's d

Cohen offered basic guidelines for interpreting the result of his test (along with a caution not to simply mindlessly apply the guidance, but to evaluate it in the larger context of the field of analysis).  

In general, d = 0.2 would be considered a small effect, 0.5 a medium effect, and 0.8 or above a large effect.  In the function above, I have included conditional print statments to accompany the result of running the function.

#### Sign matters when interpreting Cohen's d

Note that Cohen's d can be positive or negative, depending on which dataset is assigned to group1 and which to group2.  In the Cohen's d calculation in the previous cell:
- 'group1' contained the distribution of means from resampling of the dataset containing only revenues from discounted products, and 
- 'group2' contained the distribution of means from resampling of the dataset containing revenues only from non-discounted products.  

### In conclusion
A positive Cohen's d tells us that the means in the **order quantities from discounted products** are d pooled standard deviations higher than the means from the **order quantities from non-discounted products.**

# Second question:  Revenues and discount / no discount

Do discounts result in a statistically significant effect on **_per-order revenue_**?


## Null and alternative hypotheses

-  **Null hypothesis:** No statistically significant difference between the means of _**per-order revenues**_ resulting from discounts and per-order revenues from non-discounted products
    -  Ho:  $\mu_1 = \mu_2$

-  **Alternative hypothesis:**  There are statistically significant differences between the means of revenues resulting from discounts and revenues from non-discounted products that are unlikely to be due to chance
    -  Ha:  $\mu_1 \neq \mu_2$

## EDA

Before proceding, we need to add a "Revenue" column

### Add Revenue, Revenue Percentage, and Revenue Fraction (revenue per row divided by total revenues)



```python
# Add Revenue, Revenue Percentage, and Revenue Fraction (actual revenue per row divided by total revenues)

def add_rev_col():
    
    # add a revenue column
    df["Revenue"] = df["OrderUnitPrice"] * (1 - df["Discount"]) * df["OrderQty"]
        
    # add columns showing percentage / fraction of total revenue 
    df["RevPercentTotal"] = (df["Revenue"] / df["Revenue"].sum())*100  
    df["RevFractionTotal"] = (df["Revenue"] / df["Revenue"].sum())  
    
    return df

```


```python
df = add_rev_col()
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10285</td>
      <td>QUICK</td>
      <td>2012-08-20</td>
      <td>1</td>
      <td>14.4</td>
      <td>45</td>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>518.4</td>
      <td>0.040955</td>
      <td>0.000410</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10294</td>
      <td>RATTC</td>
      <td>2012-08-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>18</td>
      <td>0.00</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>259.2</td>
      <td>0.020477</td>
      <td>0.000205</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10317</td>
      <td>LONEP</td>
      <td>2012-09-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>20</td>
      <td>0.00</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>288.0</td>
      <td>0.022753</td>
      <td>0.000228</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10348</td>
      <td>WANDK</td>
      <td>2012-11-07</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>183.6</td>
      <td>0.014505</td>
      <td>0.000145</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10354</td>
      <td>PERIC</td>
      <td>2012-11-14</td>
      <td>1</td>
      <td>14.4</td>
      <td>12</td>
      <td>0.00</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>172.8</td>
      <td>0.013652</td>
      <td>0.000137</td>
    </tr>
  </tbody>
</table>
</div>



### Create sub-dataframes -- all sales; non-discounted product sales; discounted products sales


```python
total_revenue = df.loc[df.Discount >= 0.00]
revenue_no_discount = df.loc[df.Discount == 0.00]
revenue_all_discounts = df.loc[df.Discount >=0.01]
```


```python
total_revenue.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10285</td>
      <td>QUICK</td>
      <td>2012-08-20</td>
      <td>1</td>
      <td>14.4</td>
      <td>45</td>
      <td>0.2</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>518.4</td>
      <td>0.040955</td>
      <td>0.000410</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10294</td>
      <td>RATTC</td>
      <td>2012-08-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>18</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>259.2</td>
      <td>0.020477</td>
      <td>0.000205</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10317</td>
      <td>LONEP</td>
      <td>2012-09-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>20</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>288.0</td>
      <td>0.022753</td>
      <td>0.000228</td>
    </tr>
  </tbody>
</table>
</div>




```python
revenue_no_discount.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>10294</td>
      <td>RATTC</td>
      <td>2012-08-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>18</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>259.2</td>
      <td>0.020477</td>
      <td>0.000205</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10317</td>
      <td>LONEP</td>
      <td>2012-09-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>20</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>288.0</td>
      <td>0.022753</td>
      <td>0.000228</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10354</td>
      <td>PERIC</td>
      <td>2012-11-14</td>
      <td>1</td>
      <td>14.4</td>
      <td>12</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>172.8</td>
      <td>0.013652</td>
      <td>0.000137</td>
    </tr>
  </tbody>
</table>
</div>




```python
revenue_all_discounts.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10285</td>
      <td>QUICK</td>
      <td>2012-08-20</td>
      <td>1</td>
      <td>14.4</td>
      <td>45</td>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>518.4</td>
      <td>0.040955</td>
      <td>0.000410</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10348</td>
      <td>WANDK</td>
      <td>2012-11-07</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>183.6</td>
      <td>0.014505</td>
      <td>0.000145</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10370</td>
      <td>CHOPS</td>
      <td>2012-12-03</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>183.6</td>
      <td>0.014505</td>
      <td>0.000145</td>
    </tr>
  </tbody>
</table>
</div>



###  Create visualization functions


```python
# Functions for plotting

def hist_plot1(series1, label=None, color='darkcyan', alpha=0.7, bins=50, xlim=[0, 3500], print_stmt=True):
    plt.hist(series1, label=label, color=color, alpha=alpha, bins=bins)
    plt.title(f"Revenue amounts by occurrence (xlim = {xlim})")
    plt.xlabel("Revenue")
    plt.ylabel("Number")
    plt.legend(loc="best")
    plt.xlim(xlim)
    plt.show()
    if print_stmt:
        print(f"Total revenues (discount and no discount):  {int(total_revenue.Revenue.sum())}")
        print(f"Total revenues from all discounted products:  {int(revenue_all_discounts.Revenue.sum())}")
        print(f"Percentage of total revenue from discounts:  {round((revenue_all_discounts.Revenue.sum()/total_revenue.Revenue.sum())*100, 3)}%")

def hist_plot2(series1, series2, label=None, label1=None, color1='b', color2='r', alpha=0.4, bins=100, xlim=[0, 3500], ylim=None):
    plt.hist(series1, label=label, color=color1, alpha=alpha, bins=bins)
    plt.hist(series2, label=label1, color=color2, alpha=alpha, bins=bins)
    plt.title(f"Revenue amounts by occurrence (xlim = {xlim})")
    plt.xlabel("Revenue")
    plt.ylabel("Number")
    plt.legend(loc='best')
    plt.xlim(xlim)  
    plt.ylim(ylim)
    plt.show()
```


```python
hist_plot2(revenue_no_discount.Revenue, revenue_all_discounts.Revenue, label="Avg order revenue, no discount", label1="Avg order revenue, all discounts", color1='g', color2='b', xlim=[0, 3500])

```


![png](student_files/student_78_0.png)



```python
hist_plot2(revenue_no_discount.Revenue, revenue_all_discounts.Revenue, label="Avg order revenue, no discount", label1="Avg order revenue, all discounts", color1='g', color2='b', xlim=[1500, 10000], ylim=[0,20])

```


![png](student_files/student_79_0.png)


The histogram shows the distribution of revenue values (x-axis).  The distribution of both revenues without discounts and all revenues coming from discounts appear very similar, with the smallest order amounts appearing most frequently (far left hand side of graph) and the highest revenue order amounts appearing at the far right.  

If discounts have an effect on sales revenues, we would expect to see the distribution of revenues from discounts look different from the distribution of revenues due to sales that didn't involve discounts.  If the shape of the distribution changes, such that a greater proportion of revenues appear in the middle or far right-hand-side of the graph, then that difference in distribution suggests that discounts actually increase revenues (since higher revenues would occur more frequently in that distribution than in the no-discount distribution).  

## Testing for significance:  sampling, test setup and results 

As for question one, we'd like to perform a two-sided two-sample test.  We need to check for normality and check variances between the two distributions.  

### Test chosen:  t-test (2-sided, 2-sample) 
We can see that the above distributions are not normal.  As we did for Question 1, we will perform resampling for each dataset to obtain a reasonaly normal distribution of means.  


Each time the cells below are run, 100 samples are taken 300 times for each of the two populations (revenues from no discounts, revenues from all discounts).  The mean and standard deviation are computed for each of the 300 sampling events, and each of the resulting means and standard deviations is appended to the respective relevant list.  


### Sampling to obtain normal distribution of means


```python
# using sampling_mean(var, sampling_events=300, size=100) function from Question 1

sampling_no_disc_mean, sampling_no_disc_std = sampling_mean(revenue_no_discount.Revenue)
print(np.mean(sampling_no_disc_mean))
print(np.std(sampling_no_disc_std))
```

    570.5213759999999
    381.14048065742645



```python
hist_plot1(sampling_no_disc_mean, label="Avg order revenue, no discounts", xlim=None)

```


![png](student_files/student_86_0.png)


    Total revenues (discount and no discount):  1265793
    Total revenues from all discounted products:  515094
    Percentage of total revenue from discounts:  40.693%



```python
sampling_w_disc_mean, sampling_w_disc_std = sampling_mean(revenue_all_discounts.Revenue)
print(np.mean(sampling_w_disc_mean))
print(np.std(sampling_w_disc_std))

```

    618.2533294666667
    331.2482261270112



```python
hist_plot1(sampling_w_disc_mean, label="Avg order revenue, all discounts", xlim=None)

```


![png](student_files/student_88_0.png)


    Total revenues (discount and no discount):  1265793
    Total revenues from all discounted products:  515094
    Percentage of total revenue from discounts:  40.693%


### Barlett's Test for variance

The t-test performed above requires not only that the samples come from approximately normally distributed populations, but also that they come from populations that have approximately equal variances.  We use Bartlett’s test on normal populations to test the null hypothesis that the samples are from populations with equal variances. 


```python
import scipy.stats

scipy.stats.bartlett(sampling_no_disc_mean, sampling_w_disc_mean)
```




    BartlettResult(statistic=0.04396538104808006, pvalue=0.8339180343436614)



The p-value is > 0.05, so we **_cannot reject the null hypothesis_** that the samples come from populations with similar or equal variances.  Therefore, the null hypothesis that the variances of the two data set populations are approximately equal.

### T-test for two independent samples


```python
import scipy.stats as scs

scs.ttest_ind(sampling_no_disc_mean, sampling_w_disc_mean)
```




    Ttest_indResult(statistic=-6.189709371451119, pvalue=1.1197350179704822e-09)



The p-value is << 0.05, so we **should reject the null hypothesis** that the samples come from populations with the same means.  Therefore, we can say that it's unlikely that the differences in sample means from one dataset to another are due to random chance.  

So, it's likely that there is an actual statistically significant difference between the two populations.  But what is the effect size of that difference?  Enter Cohen's d.

### Effect size:  Cohen's d

Using function from Cohen's d section in Question 1:


```python
group1 = np.array(sampling_w_disc_mean)
group2 = np.array(sampling_no_disc_mean)

Cohen_d(group1, group2)
```

    This Cohen's d result (0.50623208) represents a difference of 0.51 pooled standard deviations between the two groups, and suggests a **medium** effect size.





    0.506232078277466



Using evaluate_PDF and plot_pdfs functions defined in Question 1:  


```python
plot_pdfs(d)
```


![png](student_files/student_100_0.png)


Red = all discounted products; 
Blue = all non-discounted products

This graph provides a way to visualize the difference of the means, as provided by the Cohen's d test result.  Cohen's d is a measure of the distance between the means, depicted in units of pooled standard deviation.  In this case, 

## Findings and interpretation of test results for Question 2

### Findings
There is a statistically significant difference in the _**average revenues per order**_ for non-discounted products vs. discounted products.  The _average revenue per order was **higher** when discounts were present_ than when discounts were not present. 

### Guidelines for interpreting Cohen's d

As described in the Cohen's d section in Question 1 above, Cohen suggested rules of thumb for how to interpret Cohen's d.  In general, d = 0.2 would be considered a small effect, 0.5 a medium effect, and 0.8 or above a large effect.  

#### Sign matters when interpreting Cohen's d

Note that Cohen's d can be positive or negative, depending on which dataset is assigned to group1 and which to group2.  In the Cohen's d calculation in the previous cell:
- 'group1' contained the distribution of means from resampling of the dataset containing only revenues from discounted products, and 
- 'group2' contained the distribution of means from resampling of the dataset containing revenues only from non-discounted products.  

### In Conclusion

A positive Cohen's d from this ordering tells us that the means in the **revenues from discounted products are d pooled standard deviations higher** than the means from the revenues from non-discounted products.  

# Third question:  Per-order quantity vs. level of discount

Does the discount **_level_** (e.g., 5%, 10%, 15%...) affect the **_average order quantity_**?

While we were able to reject the null hypothesis in the first question (that order quantities are the same whether products are discounted or not), we should dig a bit deeper to find out whether some discount levels result in a greater quantities per order than others. 

## Null and alternative hypotheses for third question 

-  **Null hypothesis:**  No significant difference among the means of **product order quantities** from the various discount values 

    -  Ho:  $\mu_1 = \mu_2 = \mu_3 ... = \mu_i$)


-  **Alternative hypothesis:** Differences among the means of product order quantities resulting from various discount values are unlikely to be due to chance  

    -  Ha:  $\mu_1 \neq \mu_2 \neq \mu_3 \neq ... \neq \mu_i$

## EDA

###  Create sub-dataframes for discount levels


```python
# Create dataframes with different discount levels (1-5%, 5-10%, 10-15%, 15-20%, and 20% and up)
    
def create_disc_dfs():
    '''
    This function creates dataframes for each category of discount in 5% increments, up to 
    20% (which is represented as 20% and up).  The function creates the dataframes 
    for future use.  
    '''
    
    df_disc_01to05 = df.loc[(df["Discount"] >= .01) & (df["Discount"] < .05)]
    df_disc_05to10 = df.loc[(df["Discount"] >= .05) & (df["Discount"] < .10)]
    df_disc_10to15 = df.loc[(df["Discount"] >= .10) & (df["Discount"] < .15)]
    df_disc_15to20 = df.loc[(df["Discount"] >= .15) & (df["Discount"] < .2)]
    df_disc_20andup = df.loc[df["Discount"] >= .20]
    return df_disc_01to05, df_disc_05to10, df_disc_10to15, df_disc_15to20, df_disc_20andup

# when running this function, run the following line to extract all of the dataframes:
# df_disc_01to05, df_disc_05to10, df_disc_10to15, df_disc_15to20, df_disc_20andup = create_disc_dfs()

```


```python
df_disc_01to05, df_disc_05to10, df_disc_10to15, df_disc_15to20, df_disc_20andup = create_disc_dfs()

```

_Uncomment the lines in each of the cells below to see the top 5 rows of the dataframe and
info on the dataframe columns and datatypes_


```python
df_disc_01to05.head(3)
# rev_disc_01to05.info()
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>135</th>
      <td>11077</td>
      <td>RATTC</td>
      <td>2014-05-06</td>
      <td>6</td>
      <td>25.00</td>
      <td>1</td>
      <td>0.02</td>
      <td>Grandma's Boysenberry Spread</td>
      <td>2</td>
      <td>Condiments</td>
      <td>Sweet and savory sauces, relishes, spreads, an...</td>
      <td>24.5000</td>
      <td>0.001936</td>
      <td>0.000019</td>
    </tr>
    <tr>
      <th>329</th>
      <td>11077</td>
      <td>RATTC</td>
      <td>2014-05-06</td>
      <td>14</td>
      <td>23.25</td>
      <td>1</td>
      <td>0.03</td>
      <td>Tofu</td>
      <td>7</td>
      <td>Produce</td>
      <td>Dried fruit and bean curd</td>
      <td>22.5525</td>
      <td>0.001782</td>
      <td>0.000018</td>
    </tr>
    <tr>
      <th>378</th>
      <td>11077</td>
      <td>RATTC</td>
      <td>2014-05-06</td>
      <td>16</td>
      <td>17.45</td>
      <td>2</td>
      <td>0.03</td>
      <td>Pavlova</td>
      <td>3</td>
      <td>Confections</td>
      <td>Desserts, candies, and sweet breads</td>
      <td>33.8530</td>
      <td>0.002674</td>
      <td>0.000027</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_disc_05to10.head(3)
# rev_disc_05to10.info()

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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>26</th>
      <td>10905</td>
      <td>WELLI</td>
      <td>2014-02-24</td>
      <td>1</td>
      <td>18.0</td>
      <td>20</td>
      <td>0.05</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>342.00</td>
      <td>0.027019</td>
      <td>0.000270</td>
    </tr>
    <tr>
      <th>54</th>
      <td>10632</td>
      <td>WANDK</td>
      <td>2013-08-14</td>
      <td>2</td>
      <td>19.0</td>
      <td>30</td>
      <td>0.05</td>
      <td>Chang</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>541.50</td>
      <td>0.042780</td>
      <td>0.000428</td>
    </tr>
    <tr>
      <th>61</th>
      <td>10787</td>
      <td>LAMAI</td>
      <td>2013-12-19</td>
      <td>2</td>
      <td>19.0</td>
      <td>15</td>
      <td>0.05</td>
      <td>Chang</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>270.75</td>
      <td>0.021390</td>
      <td>0.000214</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_disc_10to15.head(3)
# rev_disc_10to15.info()
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>33</th>
      <td>11025</td>
      <td>WARTH</td>
      <td>2014-04-15</td>
      <td>1</td>
      <td>18.0</td>
      <td>10</td>
      <td>0.1</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>162.0</td>
      <td>0.012798</td>
      <td>0.000128</td>
    </tr>
    <tr>
      <th>50</th>
      <td>10485</td>
      <td>LINOD</td>
      <td>2013-03-25</td>
      <td>2</td>
      <td>15.2</td>
      <td>20</td>
      <td>0.1</td>
      <td>Chang</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>273.6</td>
      <td>0.021615</td>
      <td>0.000216</td>
    </tr>
    <tr>
      <th>84</th>
      <td>10485</td>
      <td>LINOD</td>
      <td>2013-03-25</td>
      <td>3</td>
      <td>8.0</td>
      <td>20</td>
      <td>0.1</td>
      <td>Aniseed Syrup</td>
      <td>2</td>
      <td>Condiments</td>
      <td>Sweet and savory sauces, relishes, spreads, an...</td>
      <td>144.0</td>
      <td>0.011376</td>
      <td>0.000114</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_disc_15to20.head(3)
# rev_disc_15to20.info()
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>10348</td>
      <td>WANDK</td>
      <td>2012-11-07</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>183.6</td>
      <td>0.014505</td>
      <td>0.000145</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10370</td>
      <td>CHOPS</td>
      <td>2012-12-03</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>183.6</td>
      <td>0.014505</td>
      <td>0.000145</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10526</td>
      <td>WARTH</td>
      <td>2013-05-05</td>
      <td>1</td>
      <td>18.0</td>
      <td>8</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>122.4</td>
      <td>0.009670</td>
      <td>0.000097</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_disc_20andup.head(3)
# rev_disc_20andup.info()
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10285</td>
      <td>QUICK</td>
      <td>2012-08-20</td>
      <td>1</td>
      <td>14.4</td>
      <td>45</td>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>518.4</td>
      <td>0.040955</td>
      <td>0.000410</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10522</td>
      <td>LEHMS</td>
      <td>2013-04-30</td>
      <td>1</td>
      <td>18.0</td>
      <td>40</td>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>576.0</td>
      <td>0.045505</td>
      <td>0.000455</td>
    </tr>
    <tr>
      <th>16</th>
      <td>10646</td>
      <td>HUNGO</td>
      <td>2013-08-27</td>
      <td>1</td>
      <td>18.0</td>
      <td>15</td>
      <td>0.25</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>202.5</td>
      <td>0.015998</td>
      <td>0.000160</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_disc_10to15.head(3)
# rev_disc_10to15.info()
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>33</th>
      <td>11025</td>
      <td>WARTH</td>
      <td>2014-04-15</td>
      <td>1</td>
      <td>18.0</td>
      <td>10</td>
      <td>0.1</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>162.0</td>
      <td>0.012798</td>
      <td>0.000128</td>
    </tr>
    <tr>
      <th>50</th>
      <td>10485</td>
      <td>LINOD</td>
      <td>2013-03-25</td>
      <td>2</td>
      <td>15.2</td>
      <td>20</td>
      <td>0.1</td>
      <td>Chang</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>273.6</td>
      <td>0.021615</td>
      <td>0.000216</td>
    </tr>
    <tr>
      <th>84</th>
      <td>10485</td>
      <td>LINOD</td>
      <td>2013-03-25</td>
      <td>3</td>
      <td>8.0</td>
      <td>20</td>
      <td>0.1</td>
      <td>Aniseed Syrup</td>
      <td>2</td>
      <td>Condiments</td>
      <td>Sweet and savory sauces, relishes, spreads, an...</td>
      <td>144.0</td>
      <td>0.011376</td>
      <td>0.000114</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_disc_15to20.head(3)
# rev_disc_15to20.info()
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>10348</td>
      <td>WANDK</td>
      <td>2012-11-07</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>183.6</td>
      <td>0.014505</td>
      <td>0.000145</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10370</td>
      <td>CHOPS</td>
      <td>2012-12-03</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>183.6</td>
      <td>0.014505</td>
      <td>0.000145</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10526</td>
      <td>WARTH</td>
      <td>2013-05-05</td>
      <td>1</td>
      <td>18.0</td>
      <td>8</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>122.4</td>
      <td>0.009670</td>
      <td>0.000097</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_disc_20andup.head(3)
# rev_disc_20andup.info()
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10285</td>
      <td>QUICK</td>
      <td>2012-08-20</td>
      <td>1</td>
      <td>14.4</td>
      <td>45</td>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>518.4</td>
      <td>0.040955</td>
      <td>0.000410</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10522</td>
      <td>LEHMS</td>
      <td>2013-04-30</td>
      <td>1</td>
      <td>18.0</td>
      <td>40</td>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>576.0</td>
      <td>0.045505</td>
      <td>0.000455</td>
    </tr>
    <tr>
      <th>16</th>
      <td>10646</td>
      <td>HUNGO</td>
      <td>2013-08-27</td>
      <td>1</td>
      <td>18.0</td>
      <td>15</td>
      <td>0.25</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>202.5</td>
      <td>0.015998</td>
      <td>0.000160</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_disc_05to10.groupby(['CategoryName']).count()  # shows sample size for each category

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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
    <tr>
      <th>CategoryName</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Beverages</th>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
    </tr>
    <tr>
      <th>Condiments</th>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
    </tr>
    <tr>
      <th>Confections</th>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
    </tr>
    <tr>
      <th>Dairy Products</th>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
      <td>41</td>
    </tr>
    <tr>
      <th>Grains/Cereals</th>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
    </tr>
    <tr>
      <th>Meat/Poultry</th>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
    </tr>
    <tr>
      <th>Produce</th>
      <td>11</td>
      <td>11</td>
      <td>11</td>
      <td>11</td>
      <td>11</td>
      <td>11</td>
      <td>11</td>
      <td>11</td>
      <td>11</td>
      <td>11</td>
      <td>11</td>
      <td>11</td>
      <td>11</td>
    </tr>
    <tr>
      <th>Seafood</th>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
    </tr>
  </tbody>
</table>
</div>



### Visualizations

#### Summary of quantities by level of discount


```python
df.groupby('Discount')['OrderQty'].sum().plot(kind='bar', 
            title ="Total Number of Items Sold by Discount Level", figsize=(8, 5), fontsize=12)
plt.show()

```


![png](student_files/student_124_0.png)


Around 28,500 product units sold (approximately 55%) were not discounted, whereas approximately 23,000 (around 45%) product units were sold at a discount.  Of these ~23,000 product units sold at a discount, product units sold at a 5% discount comprised the largest subset at just over 5000 units sold at this discount level.  The remaining four discount levels--10%, 15%, 20%, and 25%--represent almost identical unit sales of approximately 4500 units each.  

#### Visualizations of order quantities versus levels of discounts
As in the first case, we can create some initial visualizations to see if different discount levels result in differences in average order quantities.  


```python
hist_plot2_qty(df_no_disc.OrderQty, df_disc.OrderQty, label="Avg order qty, no discounts",  label1="Avg order qty, all discounts")

```


![png](student_files/student_127_0.png)



```python
hist_plot2_qty(df_no_disc.OrderQty, df_disc_01to05.OrderQty, label="Avg order qty, no discounts",  label1="Avg order qty, Discounts = 1-4.9%", xlim=([0,150]))

# there are very few data points in the df_disc_01to05 category; going forward, I will delete this one.
```


![png](student_files/student_128_0.png)



```python
hist_plot2_qty(df_no_disc.OrderQty, df_disc_05to10.OrderQty, label="Avg order qty, no discounts",  label1="Avg order qty, Discounts = 5-9.9%", xlim=([0,150]))

```


![png](student_files/student_129_0.png)



```python
hist_plot2_qty(df_no_disc.OrderQty, df_disc_10to15.OrderQty, label="Avg order qty, no discounts",  label1="Avg order qty, Discounts = 10-14.9%", xlim=([0,150]))

```


![png](student_files/student_130_0.png)



```python
hist_plot2_qty(df_no_disc.OrderQty, df_disc_15to20.OrderQty,  label="Avg order qty, no discounts",  label1="Avg order qty, Discounts = 15-19.9%", xlim=([0,150]))

```


![png](student_files/student_131_0.png)



```python
hist_plot2_qty(df_no_disc.OrderQty, df_disc_20andup.OrderQty,  label="Avg order qty, no discounts",  label1="Avg order qty, Discounts = 20% and up", xlim=([0,150]))

```


![png](student_files/student_132_0.png)


## Testing for significance:  sampling, test setup and results 
A review of the histogram plots for order quantities deriving from various discount levels shows that, much like the overall order quantity frequency histograms, the histograms of these revenue-from-discount subsets reveal non-normally distributed data.  To run the statistical tests needed to test the hypotheses in the second question, I will perform multiple sampling of the datasets, then performing statistical tests on the distribution of the means.  Per the Central Limit Theorem, these distributions should be approximately normal, and so can be used for other tests (specifically Tukey's test).

### First test chosen:  ANOVA

This test will tell us that at least one of the datasets is significantly different than the others.  (It doesn't tell us which one, though.)

As with the t-test above, I will first perform resampling and create distributions of the means of these resampling events to form approximately normal distributions to perform the ANOVA test.

### Multiple sampling of datasets to obtain normal distribution of means



```python
no_discounts = sampling_mean(revenue_no_discount.OrderQty)
print(f"The mean of the distribution is {round(np.mean(no_discounts[0]), 2)}")
print(f"The std deviation of the distribution is {round(np.std(no_discounts[1]), 2)}")
hist_plot1_qty(no_discounts[0],  label="Avg order qty, no discount", xlim=([0, 40]))

```

    The mean of the distribution is 21.76
    The std deviation of the distribution is 2.29



![png](student_files/student_136_1.png)



```python
all_discounts = sampling_mean(revenue_all_discounts.OrderQty)
print(f"The mean of the distribution is {round(np.mean(all_discounts[0]), 2)}")
print(f"The std deviation of the distribution is {round(np.std(all_discounts[1]), 2)}")
hist_plot1_qty(all_discounts[0], label="Avg order qty, all discounts", xlim=([0, 50]))

```

    The mean of the distribution is 27.12
    The std deviation of the distribution is 2.25



![png](student_files/student_137_1.png)



```python
# testing that function results can be plugged into ttest_ind

scs.ttest_ind(no_discounts[0], all_discounts[0])
```




    Ttest_indResult(statistic=-36.465964150081504, pvalue=3.945476681113779e-154)



##### Sub-dataframes based on revenues from a given discount level

As a reminder, here are the dataframes I've built so far:
-  total_revenue -- Total revenues, with **and** without discounts
-  revenue_no_discount -- All revenues resulting from sales of **non**-discounted products
-  revenue_all_discounts -- All revenues resulting from sales of products at **all** discount levels
-  df_disc_01to05.Revenue -- Products discounted 1-4.9%
-  df_disc_05to10.Revenue -- Products discounted 5-9.9%
-  df_disc_10to15.Revenue -- Products discounted 10-14.9%
-  df_disc_15to20.Revenue -- Products discounted 15-19.9%
-  df_disc_20andup.Revenue -- Products discounted 20% or more


```python
total_rev_means = sampling_mean(total_revenue.OrderQty)
np.mean(total_rev_means[0]), np.std(total_rev_means[1])

```




    (23.843966666666667, 2.5112563991838885)




```python
revs_no_disc_means = sampling_mean(revenue_no_discount.OrderQty)
np.mean(revs_no_disc_means[0]), np.std(revs_no_disc_means[1])
```




    (21.763033333333333, 2.3470803659571597)




```python
revs_all_disc_means = sampling_mean(revenue_all_discounts.OrderQty)
np.mean(revs_all_disc_means[0]), np.std(revs_all_disc_means[1])
```




    (27.08096666666667, 2.2441359474713334)



**Note**--I'm not including revenues from 1-4.9% discounts because there are only 7 data points--not enough revenue points in this sample to compute these statistics


```python
df_5to10_disc_means = sampling_mean(df_disc_05to10.OrderQty)
np.mean(df_5to10_disc_means[0]), np.std(df_5to10_disc_means[1])
```




    (27.908199999999997, 1.5019613711301814)




```python
df_10to15_disc_means = sampling_mean(df_disc_10to15.OrderQty)
np.mean(df_10to15_disc_means[0]), np.std(df_10to15_disc_means[1])
```




    (25.406566666666663, 1.999842545129951)




```python
df_15to20_disc_means = sampling_mean(df_disc_15to20.OrderQty)
np.mean(df_15to20_disc_means[0]), np.std(df_15to20_disc_means[1])
```




    (28.39096666666667, 1.170205827015813)




```python
df_20andup_disc_means = sampling_mean(df_disc_20andup.OrderQty)
np.mean(df_20andup_disc_means[0]), np.std(df_20andup_disc_means[1])
```




    (27.48933333333333, 1.7520488624991175)



### One-way ANOVA test using scipy.stats:
 
scipy.stats.f_oneway(*args)

A p-value of < $\alpha$ (0.05) signifies that there is a < 5% chance that we would see the differences we would observe among the sample means distributions.  A p-value of >= 0.05 signifies that there is a greater than 5% chance (p-value probability) that the differences observed among the distributions is due to chance.  


```python
scs.f_oneway(revs_no_disc_means[0], df_5to10_disc_means[0], df_10to15_disc_means[0], df_15to20_disc_means[0], df_20andup_disc_means[0])

```




    F_onewayResult(statistic=1017.8604514479314, pvalue=0.0)



The extremely low p-value of the ANOVA test tell us that we should reject the null hypothesis (that there is no statistically significant difference between the distributions of means) and strongly suggests that at least one of the distributions of means differs signficantly from at least one other.  

Because ANOVA doesn't tell us which variables differ from others, I'll run Tukey's test to find out.

### Second test chosen:  Tukey's Range Test

This test can tell us which variables are significantly different from one another.  

Per the codeacademy Hypothesis Testing presentation by Hillary Green-Lerman, three things are required for pairwise_tukeyhsd:
1. A vector of all data (concatenated using np.concatenate)
2. A vector of labels for the data
3. A level of significance (usually 0.05)



```python
from statsmodels.stats.multicomp import pairwise_tukeyhsd
```


```python
no_disc = np.array(revs_no_disc_means[0])           # the number of means in these lists 
disc_05to10 = np.array(df_5to10_disc_means[0])    # equals the sampling events input to 
disc_10to15 = np.array(df_10to15_disc_means[0])  # the sampling function that was run to 
disc_15to20 = np.array(df_15to20_disc_means[0])   # obtain this data
disc_20andup = np.array(df_20andup_disc_means[0])

v = np.concatenate([no_disc, disc_05to10, disc_10to15, disc_15to20, disc_20andup])
labels = ['no_disc'] * len(no_disc) + ['disc_05to10'] * len(disc_05to10) + ['disc_10to15'] * len(disc_10to15) + ['disc_15to20']*len(disc_15to20) + ['disc_20andup']*len(disc_20andup)
tukey_results = pairwise_tukeyhsd(v, labels, 0.05)
print(tukey_results)

type(tukey_results)

```

        Multiple Comparison of Means - Tukey HSD,FWER=0.05   
    =========================================================
       group1       group2    meandiff  lower   upper  reject
    ---------------------------------------------------------
    disc_05to10  disc_10to15  -2.5016  -2.8315 -2.1718  True 
    disc_05to10  disc_15to20   0.4828   0.1529  0.8126  True 
    disc_05to10  disc_20andup -0.4189  -0.7487  -0.089  True 
    disc_05to10    no_disc    -6.1452   -6.475 -5.8153  True 
    disc_10to15  disc_15to20   2.9844   2.6545  3.3143  True 
    disc_10to15  disc_20andup  2.0828   1.7529  2.4126  True 
    disc_10to15    no_disc    -3.6435  -3.9734 -3.3137  True 
    disc_15to20  disc_20andup -0.9016  -1.2315 -0.5718  True 
    disc_15to20    no_disc    -6.6279  -6.9578 -6.2981  True 
    disc_20andup   no_disc    -5.7263  -6.0562 -5.3964  True 
    ---------------------------------------------------------





    statsmodels.sandbox.stats.multicomp.TukeyHSDResults



Once again, for average order quantity (much more so than for average revenues per order), the relatively small number of potential point samples represented by quantity (as opposed to revenues) can result in differences in running these tests from one set of sampling runs to the next.  The first time I ran the sampling function and then ran Tukey's test, every single pairing was found to be significant (True for "reject null hypothesis" for all pairings).  The second time I ran the sampling functions and the tests, the result was to reject the null for all pairs except for disc_05to10 / disc_20andup.  

In any case, what I find is that the average order quantity is statistically significantly different for most, if not all, pairs.  What this means is that each of these distributions of average order quantity by discount level (except for the disc_15ot20 / no_disc pair) is statistically significant different from each other at the 0.05 level.  


### Effect size:  Cohen's d pairwise comparisons and visualization

To find out the effect sizes of the various discount levels, we can run Cohen's d to see which ones are most important.  Except for one pairing, all were significant, so will need to do Cohen's d on each of the other pairs.


```python
d = Cohen_d(disc_05to10, disc_10to15)
plot_pdfs(d)
```

    This Cohen's d result (1.7861853) represents a difference of 1.79 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_157_1.png)


Magenta = group1 = disc_05to10 ;   blue = group2 = disc_10to15

The average order quantities from discounts of 5-9.9% are **significantly higher** than the means of revenues from discounts of 10-14.9%.

That is a rather strange outcome; whereas one might expect such a result for revenues, where two somewhat opposing forces are at work (higher discounts might drive customers to purchase more, but at the same time reduce margins for the seller), it is not clear why we would see higher _quantities_ sold for a discount of 5% as opposed to a discount of 10%.  

It could be that Northwind is only offering the larger discounts on more expensive products:  the larger discount might prompt a customer to buy the product (as opposed to not buying it), but might not be enough in and of itself to prompt the customer to buy more units than planned.


```python
d = Cohen_d(disc_05to10, disc_15to20)
plot_pdfs(d)
```

    This Cohen's d result (-0.36344668) represents a difference of -0.36 pooled standard deviations between the two groups, and suggests a **small** effect size.



![png](student_files/student_159_1.png)


Magenta = group1 = disc_05to10 ;   blue = group2 = disc_15to20

The average order quantity on products discounted 5-9.9% is **slightly lower** than is the average order quantity on products discounted 15-19.9%.

Directionally, this result makes sense (the greater the discount, the larger the quantity sold) but some of the same comments I made in the previous pairwise comparison might apply here also.  While in the previous case, perhaps the 10% discount wasn't enough to spur additional quantity buying (but might have induced a customer to buy at least one unit).  But as the discount gets bigger, it might start driving increased quantity purchasing, even on more expensive products.  This might represent a continuum, as seen in the next graph...


```python
d = Cohen_d(disc_05to10, disc_20andup)
plot_pdfs(d)

# note that this pairing was not found to be statistically significant in Tukey's test.
```

    This Cohen's d result (0.26794938) represents a difference of 0.27 pooled standard deviations between the two groups, and suggests a **small** effect size.



![png](student_files/student_161_1.png)


Magenta = group1 = disc_05to10 ; blue = group2 = disc_20andup

The average order quantity on products discounted 5-9.9% is **slightly higher** than the average order quantity on products discounted 20% and up.  This is somewhat counterintuitive; all things being equal, one would expect to see more product units purchased at a 20%+ discount than at a 5% discount.  However, it is likely that a lot fewer products get the 20%+ discount treatment, so there just isn't as much volume available at that discount level.  It could also be that the larger discounts are relatively more available on more expensive products, and so one might not see as much of an increase in quantity purchased as with lower-cost items.



```python
d = Cohen_d(disc_05to10, no_disc)
plot_pdfs(d)
```

    This Cohen's d result (4.14557783) represents a difference of 4.15 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_163_1.png)


Magenta = group1 = disc_05to10 ;   blue = group2 = no_disc

The the average order quantity on products discounted 5-9.9% are **substantially higher** than the average order quantity from non-discounted products, which is not at all surprising.


```python
d = Cohen_d(disc_10to15, disc_15to20)
plot_pdfs(d)
```

    This Cohen's d result (-2.23519295) represents a difference of -2.24 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_165_1.png)


Magenta = group1 = disc_10to15 ;   blue = group2 = disc_15to20

The average order quantity on products discounted 10-14.9% is **significantly lower** than the average order quantity on products discounted 15-20%.  This is unsurprising.


```python
d = Cohen_d(disc_10to15, disc_20andup)
plot_pdfs(d)
```

    This Cohen's d result (-1.32737613) represents a difference of -1.33 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_167_1.png)


Magenta = group1 = disc_10to15 ;   blue = group2 = disc_20andup

The average order quantity on products discounted 10-14.9% is **significantly lower** than the average order quantity on products discounted 20% and up.  Again, unsurprising, and consistent with the previous two Cohen's d results and graphs.


```python
d = Cohen_d(disc_10to15, no_disc)
plot_pdfs(d)
```

    This Cohen's d result (2.44776268) represents a difference of 2.45 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_169_1.png)


Magenta = group1 = disc_10to15 ;   blue = group2 = no_disc

The average order quantity on products discounted 10-14.9% is **significantly higher** than the average order quantity on non-discounted products.  Again, consistent with the previous findings described above.


```python
d = Cohen_d(disc_15to20, disc_20andup)  
plot_pdfs(d)
```

    This Cohen's d result (0.59911024) represents a difference of 0.6 pooled standard deviations between the two groups, and suggests a **medium** effect size.



![png](student_files/student_171_1.png)


Magenta = group1 = disc_15to20 ;   blue = group2 = disc_20andup

The means of revenues from discounts of 15-19.9% are **slightly lower** than the means of revenues from discounts of 20% and up.

The average order quantity on products discounted 15-19.9% is **somewhat lower** than the average order quantity on products discounted 20% and up.  Again, consistent with the previous findings described above.


```python
d = Cohen_d(disc_15to20, no_disc)
plot_pdfs(d)
```

    This Cohen's d result (4.66508814) represents a difference of 4.67 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_173_1.png)


Magenta = group1 = disc_15to20 ;   blue = group2 = no_disc

**Big difference** between average order quantity for substantially discounted products (magenta) and non-discounted products (blue).


```python
d = Cohen_d(disc_20andup, no_disc)
plot_pdfs(d)
```

    This Cohen's d result (3.48630871) represents a difference of 3.49 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_175_1.png)


Magenta = group1 = disc_20andup ;   blue = group2 = no_disc

And once again, an **even bigger difference** between average order quantity for substantially discounted products (magenta) and non-discounted products (blue).

## Findings and interpretation of test results for Question 3

Results regarding average order quantity relative to discount offered are, with a few exceptions, intuitive.  
-  Not surprising:  Larger discounts tend to correlate in a statistically significant way to greater order unit quantities.
-  Somewhat surprising:  Products in the 5-9.9% discount band (which is comprised almost entirely of 5% discounts) tend to have higher effect sizes (measured by Cohen's d) than those at 10-15%.  They even have a slightly higher effect size than discounts at 20% and above (though Cohen's d for this pairing is small).  
-  Future exploration could focus on why these patterns appear.  A few possibilities:
   -  Smaller discounts are likely to be applied to a broader range of products than deeper discounts, so there are just a lot more products that can be purchased at that level
   -  It could be that larger discounts are applied relatively more often to more expensive products than less expensive ones.  Rather than driving additional unit purchases, these larger discounts might be applied to more expensive products to induce a purchase that might not otherwise have happened, and/or to hasten a sale in the case of perishable products. 

These findings, combined with those on average order revenue by discount levels, provide some intriguing insights into how pricing occurs and sales are made.  Some focused exploration on discounts by product categories and specific products will likely lead to targeted strategies for improving sales in certain categories and/or geographies.


I've concluded the following after running Tukey's Test and Cohen's d for each pairwise comparison: 
-  Discounts at the 5-10% level appear to drive the highest revenues per order, relative to all other discount levels (or no discount at all)
    -  While higher discounts are likely to drive more purchases, the per-unit revenue will decrease with larger discounts.  
    -  Perhaps this relatively small discount level is enough to get customers to buy more while preserving higher revenue per unit 

# Fourth question

Does the **_level of discount_** affect average revenues per order?


## Null and alternative hypotheses for fourth question 

-  **Null hypothesis:**  No significant difference among the means of per-order revenues from the various discount values 

    -  Ho:  $\mu_1 = \mu_2 = \mu_3 ... = \mu_i$)


-  **Alternative hypothesis:** Differences among the means of per-order revenues resulting from various discount values are unlikely to be due to chance  

    -  Ha:  $\mu_1 \neq \mu_2 \neq \mu_3 \neq ... \neq \mu_i$


## EDA 

### Summary of revenues by level of discount


```python
df.groupby('Discount')['Revenue'].sum().plot(kind='bar', 
            title ="Revenues by Discount Level", figsize=(8, 5), fontsize=12)
plt.show()

```


![png](student_files/student_183_0.png)


Almost 60% of product revenues are from non-discounted products--which means that over 40% of product revenues are the result of discounted products.  Of discounted products, those products discounted by 5% bring in the most revenue of all discounted products, with 25% representing the next highest revenue amount (~\$100000).  The remaining discount amounts with significant revenues are 10%, 15% and 20%.  All three are virtually equal, but 10% appears to represent slightly greater revenues than those from 15% or 20% discounts.

### Visualizations of discount levels vs. revenues from no discounts
As in the first case, we can create some initial visualizations to see if the magnitude of the discount results in differences in the sales revenues distributions.  


```python
# function for plotting three different series 

def hist_plot3(var1, var2, var3, label=None, label1=None, label2=None, color='b', alpha=0.5, bins=100, xlim=[0, 3500]):
    var1 = var1
    var2 = var2
    var3 = var3
    alpha = alpha
    plt.hist(var1, label=label, color='r', alpha=0.5, bins=100)
    plt.hist(var2, label=label1, color='b', alpha=0.5, bins=100)
    plt.hist(var3, label=label2, color='b', alpha=0.5, bins=100)
    plt.title(f"Revenue amounts by occurrence (xlim = {xlim})")
    plt.xlabel("Revenue")
    plt.ylabel("Number")
    plt.legend(loc='best')
    plt.xlim(xlim)
    plt.show()

```


```python
hist_plot3(total_revenue.Revenue, revenue_no_discount.Revenue, revenue_all_discounts.Revenue, label='Avg order revenue, discounts and no discounts', label1='Avg order revenue, no discounts', label2='Avg order revenue, all discounts')

print(f"Total revenues (discount and no discount):  {int(total_revenue.Revenue.sum())}")
print(f"Total revenues, no discount:  {int(revenue_no_discount.Revenue.sum())}")
print(f"Total revenues from all discounted products:  {int(revenue_all_discounts.Revenue.sum())}")

```


![png](student_files/student_187_0.png)


    Total revenues (discount and no discount):  1265793
    Total revenues, no discount:  750698
    Total revenues from all discounted products:  515094



```python
hist_plot2(revenue_no_discount.Revenue, revenue_all_discounts.Revenue, label='Avg order revenue, no discounts', label1='Avg order revenue, ALL discounts', alpha=0.5, bins=100)

```


![png](student_files/student_188_0.png)


##### Discounts = 1% to 4.99%:  Effect on total revenues

Because the revenues from this group are so small, I will be ignoring them for the purposes of this analysis.

##### Discounts = 5% to 9.99%:  Effect on total revenues


```python
hist_plot2(revenue_no_discount.Revenue, df_disc_05to10.Revenue, label='Avg order revenue, no discounts', label1='Avg order revenue, discounts 5-9.9%', alpha=0.5)

print(f"Revenues from discount level of 5-9.99%:  {int(df_disc_05to10.Revenue.sum())}")
print(f"Discount 5-9.99% as a percentage of total revenue:  {round((df_disc_05to10.Revenue.sum()/total_revenue.Revenue.sum())*100, 3)}%")
print(f"Discount 5-9.99% as a percentage of all discount revenue:  {round((df_disc_05to10.Revenue.sum()/revenue_all_discounts.Revenue.sum())*100, 3)}%")
```


![png](student_files/student_192_0.png)


    Revenues from discount level of 5-9.99%:  147681
    Discount 5-9.99% as a percentage of total revenue:  11.667%
    Discount 5-9.99% as a percentage of all discount revenue:  28.671%


##### Discounts = 10% to 14.99%:   Effect on total revenues


```python
hist_plot2(revenue_no_discount.Revenue, df_disc_10to15.Revenue, label='Avg order revenue, no discounts', label1='Avg order revenue, discounts 10-14.9%', alpha=.5)

print(f"Revenues from discount level of 10-14.99%:  {int(df_disc_10to15.Revenue.sum())}")
print(f"Discount 10-14.99% as a percentage of total revenue:  {round((df_disc_10to15.Revenue.sum()/total_revenue.Revenue.sum())*100, 3)}%")
print(f"Discount 10-14.99% as a percentage of all discount revenue:  {round((df_disc_10to15.Revenue.sum()/revenue_all_discounts.Revenue.sum())*100, 3)}%")
```


![png](student_files/student_194_0.png)


    Revenues from discount level of 10-14.99%:  91499
    Discount 10-14.99% as a percentage of total revenue:  7.229%
    Discount 10-14.99% as a percentage of all discount revenue:  17.764%


#####  Discounts = 15% to 19.99%:  Effect on total revenues


```python
hist_plot2(total_revenue.Revenue, df_disc_15to20.Revenue,  label='Avg order revenue, no discounts', label1='Avg order revenue, discounts 15-19.9%',alpha=0.5, bins=100)

print(f"Revenues from discount level of 15-19.99%:  {int(df_disc_15to20.Revenue.sum())}")
print(f"Discount 15-19.99% as a percentage of total revenue:  {round((df_disc_15to20.Revenue.sum()/total_revenue.Revenue.sum())*100, 3)}%")
print(f"Discount 15-19.99% as a percentage of all discount revenue:  {round((df_disc_15to20.Revenue.sum()/revenue_all_discounts.Revenue.sum())*100, 3)}%")
```


![png](student_files/student_196_0.png)


    Revenues from discount level of 15-19.99%:  87506
    Discount 15-19.99% as a percentage of total revenue:  6.913%
    Discount 15-19.99% as a percentage of all discount revenue:  16.988%


#####  Discounts = 20% or more:  Effect on total revenues


```python
hist_plot2(total_revenue.Revenue, df_disc_20andup.Revenue,  label='Avg order revenue, no discounts', label1='Avg order revenue, discounts 20% and up', alpha=0.5, bins=100)

print(f"Revenues from discount level of 20% and up:  {int(df_disc_20andup.Revenue.sum())}")
print(f"Discount 20% and up as a percentage of total revenue:  {round((df_disc_20andup.Revenue.sum()/total_revenue.Revenue.sum())*100, 3)}%")
print(f"Discount 20% and up as a percentage of all discount revenue:  {round((df_disc_20andup.Revenue.sum()/revenue_all_discounts.Revenue.sum())*100, 3)}%")
```


![png](student_files/student_198_0.png)


    Revenues from discount level of 20% and up:  188119
    Discount 20% and up as a percentage of total revenue:  14.862%
    Discount 20% and up as a percentage of all discount revenue:  36.521%


##### Totals and percentages of discount revenue to all revenues


```python
df.groupby(['Discount']).sum().sort_values(['Discount'], ascending=False)
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
      <th>Id</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>CategoryId</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
    <tr>
      <th>Discount</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0.25</th>
      <td>1648801</td>
      <td>5820</td>
      <td>4345.02</td>
      <td>4349</td>
      <td>643</td>
      <td>98938.5675</td>
      <td>7.816331</td>
      <td>0.078163</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>1711705</td>
      <td>6445</td>
      <td>3787.81</td>
      <td>4351</td>
      <td>661</td>
      <td>89181.1040</td>
      <td>7.045473</td>
      <td>0.070455</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>1672996</td>
      <td>6351</td>
      <td>3607.22</td>
      <td>4456</td>
      <td>595</td>
      <td>87506.1740</td>
      <td>6.913150</td>
      <td>0.069132</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>1833523</td>
      <td>7164</td>
      <td>4354.68</td>
      <td>4366</td>
      <td>742</td>
      <td>91499.1390</td>
      <td>7.228602</td>
      <td>0.072286</td>
    </tr>
    <tr>
      <th>0.06</th>
      <td>11077</td>
      <td>60</td>
      <td>34.00</td>
      <td>2</td>
      <td>4</td>
      <td>63.9200</td>
      <td>0.005050</td>
      <td>0.000050</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>1972417</td>
      <td>7311</td>
      <td>5697.32</td>
      <td>5182</td>
      <td>783</td>
      <td>147617.3745</td>
      <td>11.662047</td>
      <td>0.116620</td>
    </tr>
    <tr>
      <th>0.04</th>
      <td>11077</td>
      <td>20</td>
      <td>81.00</td>
      <td>1</td>
      <td>3</td>
      <td>77.7600</td>
      <td>0.006143</td>
      <td>0.000061</td>
    </tr>
    <tr>
      <th>0.03</th>
      <td>33231</td>
      <td>94</td>
      <td>73.95</td>
      <td>5</td>
      <td>15</td>
      <td>120.9105</td>
      <td>0.009552</td>
      <td>0.000096</td>
    </tr>
    <tr>
      <th>0.02</th>
      <td>22154</td>
      <td>52</td>
      <td>37.00</td>
      <td>4</td>
      <td>10</td>
      <td>59.7800</td>
      <td>0.004723</td>
      <td>0.000047</td>
    </tr>
    <tr>
      <th>0.01</th>
      <td>11077</td>
      <td>73</td>
      <td>15.00</td>
      <td>2</td>
      <td>8</td>
      <td>29.7000</td>
      <td>0.002346</td>
      <td>0.000023</td>
    </tr>
    <tr>
      <th>0.00</th>
      <td>14042897</td>
      <td>54519</td>
      <td>34467.91</td>
      <td>28599</td>
      <td>5448</td>
      <td>750698.6100</td>
      <td>59.306584</td>
      <td>0.593066</td>
    </tr>
  </tbody>
</table>
</div>




```python
print(f"The mean of the distribution is {round(revenue_no_discount.Revenue.mean(), 2)}")
hist_plot1(revenue_no_discount.Revenue, label="Average order revenues, no discounts") # plotting with xlim in place

```

    The mean of the distribution is 570.01



![png](student_files/student_201_1.png)


    Total revenues (discount and no discount):  1265793
    Total revenues from all discounted products:  515094
    Percentage of total revenue from discounts:  40.693%



```python
# hist_plot1(df_disc_05to10.Revenue, xlim=None)  # plotting without xlim to see outliers
print(f"The mean of the distribution is {round(df_disc_05to10.Revenue.mean(), 2)}")
hist_plot1(df_disc_05to10.Revenue, label="Average order revenues, discounts of 5-9.9%", print_stmt=False) # plotting with xlim in place

```

    The mean of the distribution is 793.99



![png](student_files/student_202_1.png)



```python
# hist_plot1(df_disc_10to15.Revenue, xlim=None)
print(f"The mean of the distribution is {round(df_disc_10to15.Revenue.mean(), 2)}")
hist_plot1(df_disc_10to15.Revenue, label="Average order revenues, discounts of 10-14.9%", print_stmt=False)


```

    The mean of the distribution is 528.9



![png](student_files/student_203_1.png)



```python
# hist_plot1(df_disc_15to20.Revenue, xlim=None)
print(f"The mean of the distribution is {round(df_disc_15to20.Revenue.mean(), 2)}")
hist_plot1(df_disc_15to20.Revenue, label="Average order revenues, discounts of 15-19.9%", print_stmt=False)

```

    The mean of the distribution is 557.36



![png](student_files/student_204_1.png)



```python
# hist_plot1(df_disc_20andup.Revenue, xlim=None)
print(f"The mean of the distribution is {round(df_disc_20andup.Revenue.mean(), 2)}")
hist_plot1(df_disc_20andup.Revenue, label="Average order revenues, discounts of 20% and up", xlim=[0, 3000], print_stmt=False)

```

    The mean of the distribution is 597.21



![png](student_files/student_205_1.png)


## Testing for significance:  sampling, test setup and results 
A review of the histogram plots for revenues deriving from various discount levels shows that, much like the overall revenue frequency histograms, the histograms of these revenue-from-discount subsets reveal non-normally distributed data.  To run the statistical tests needed to test the hypotheses in the second question, I will perform multiple sampling of the datasets, then performing statistical tests on the distribution of the means.  Per the Central Limit Theorem, these distributions should be approximately normal, and so can be used for other tests (specifically Tukey's test).

### First test chosen:  ANOVA

This test will tell us that at least one of the datasets is significantly different than the others.  (It doesn't tell us which one, though.)

As with the t-test above, I will first perform resampling and create distributions of the means of these resampling events to form approximately normal distributions to perform the ANOVA test.

### Multiple sampling of datasets to obtain normal distribution of means



```python
no_discounts = sampling_mean(revenue_no_discount.Revenue)
print(f"The mean of the distribution is {round(np.mean(no_discounts[0]), 2)}")
print(f"The std deviation of the distribution is {round(np.std(no_discounts[1]), 2)}")
hist_plot1(no_discounts[0], label="Average order revenues, no discounts", xlim=([0, 1200]))

```

    The mean of the distribution is 581.46
    The std deviation of the distribution is 384.4



![png](student_files/student_209_1.png)


    Total revenues (discount and no discount):  1265793
    Total revenues from all discounted products:  515094
    Percentage of total revenue from discounts:  40.693%



```python
all_discounts = sampling_mean(revenue_all_discounts.Revenue)
print(f"The mean of the distribution is {round(np.mean(all_discounts[0]), 2)}")
print(f"The std deviation of the distribution is {round(np.std(all_discounts[1]), 2)}")
hist_plot1(all_discounts[0], label="Average order revenues, all discounts", xlim=([0, 1200]))

```

    The mean of the distribution is 612.94
    The std deviation of the distribution is 335.39



![png](student_files/student_210_1.png)


    Total revenues (discount and no discount):  1265793
    Total revenues from all discounted products:  515094
    Percentage of total revenue from discounts:  40.693%



```python
# testing that function results can be plugged into ttest_ind

scs.ttest_ind(no_discounts[0], all_discounts[0])
```




    Ttest_indResult(statistic=-4.003612486005798, pvalue=7.022917511073499e-05)



##### Sub-dataframes based on revenues from a given discount level

As a reminder, here are the dataframes I've built so far:
-  total_revenue -- Total revenues, with **and** without discounts
-  revenue_no_discount -- All revenues resulting from sales of **non**-discounted products
-  revenue_all_discounts -- All revenues resulting from sales of products at **all** discount levels
-  df_disc_01to05.Revenue -- Products discounted 1-4.9%
-  df_disc_05to10.Revenue -- Products discounted 5-9.9%
-  df_disc_10to15.Revenue -- Products discounted 10-14.9%
-  df_disc_15to20.Revenue -- Products discounted 15-19.9%
-  df_disc_20andup.Revenue -- Products discounted 20% or more


```python
total_rev_means = sampling_mean(total_revenue.Revenue)
np.mean(total_rev_means[0]), np.std(total_rev_means[1])

```




    (583.0617516000001, 351.03901350261066)




```python
revs_no_disc_means = sampling_mean(revenue_no_discount.Revenue)
np.mean(revs_no_disc_means[0]), np.std(revs_no_disc_means[1])
```




    (564.6655116666666, 345.1591629191503)




```python
revs_all_disc_means = sampling_mean(revenue_all_discounts.Revenue)
np.mean(revs_all_disc_means[0]), np.std(revs_all_disc_means[1])
```




    (622.9363593333333, 385.2859280957691)



**Note**--I'm not including revenues from 1-4.9% discounts because there are only 7 data points--not enough revenue points in this sample to compute these statistics


```python
df_5to10_disc_means = sampling_mean(df_disc_05to10.Revenue)
np.mean(df_5to10_disc_means[0]), np.std(df_5to10_disc_means[1])
```




    (785.3587769166667, 388.22407082181945)




```python
df_10to15_disc_means = sampling_mean(df_disc_10to15.Revenue)
np.mean(df_10to15_disc_means[0]), np.std(df_10to15_disc_means[1])
```




    (529.1909292, 51.08710103154266)




```python
df_15to20_disc_means = sampling_mean(df_disc_15to20.Revenue)
np.mean(df_15to20_disc_means[0]), np.std(df_15to20_disc_means[1])
```




    (555.7451172499999, 51.47884666219702)




```python
df_20andup_disc_means = sampling_mean(df_disc_20andup.Revenue)
np.mean(df_20andup_disc_means[0]), np.std(df_20andup_disc_means[1])
```




    (595.8233765, 239.43192635432803)



### One-way ANOVA test using scipy.stats:
 
scipy.stats.f_oneway(*args)

A p-value of < $\alpha$ (0.05) signifies that there is a < 5% chance that we would see the differences we would observe among the sample means distributions.  A p-value of >= 0.05 signifies that there is a greater than 5% chance (p-value probability) that the differences observed among the distributions is due to chance.  


```python
scs.f_oneway(revs_no_disc_means[0], df_5to10_disc_means[0], df_10to15_disc_means[0], df_15to20_disc_means[0], df_20andup_disc_means[0])

```




    F_onewayResult(statistic=628.196879270906, pvalue=3.506107e-318)



The extremely low p-value of the ANOVA test tell us that we should reject the null hypothesis (that there is no statistically significant difference between the distributions of means) and strongly suggests that at least one of the distributions of means differs signficantly from at least one other.  

Because ANOVA doesn't tell us which variables differ from others, I'll run Tukey's test to find out.

### Second test chosen:  Tukey's Range Test

This test can tell us which variables are significantly different from one another.  

Per the codeacademy Hypothesis Testing presentation by Hillary Green-Lerman, three things are required for pairwise_tukeyhsd:
1. A vector of all data (concatenated using np.concatenate)
2. A vector of labels for the data
3. A level of significance (usually 0.05)



```python
from statsmodels.stats.multicomp import pairwise_tukeyhsd
```


```python
no_disc = np.array(revs_no_disc_means[0])           # the number of means in these lists 
disc_05to10 = np.array(df_5to10_disc_means[0])    # equals the sampling events input to 
disc_10to15 = np.array(df_10to15_disc_means[0])  # the sampling function that was run to 
disc_15to20 = np.array(df_15to20_disc_means[0])   # obtain this data
disc_20andup = np.array(df_20andup_disc_means[0])

v = np.concatenate([no_disc, disc_05to10, disc_10to15, disc_15to20, disc_20andup])
labels = ['no_disc'] * len(no_disc) + ['disc_05to10'] * len(disc_05to10) + ['disc_10to15'] * len(disc_10to15) + ['disc_15to20']*len(disc_15to20) + ['disc_20andup']*len(disc_20andup)
tukey_results = pairwise_tukeyhsd(v, labels, 0.05)
print(tukey_results)

type(tukey_results)

```

          Multiple Comparison of Means - Tukey HSD,FWER=0.05      
    ==============================================================
       group1       group2     meandiff   lower     upper   reject
    --------------------------------------------------------------
    disc_05to10  disc_10to15  -256.1678 -272.0348 -240.3009  True 
    disc_05to10  disc_15to20  -229.6137 -245.4806 -213.7468  True 
    disc_05to10  disc_20andup -189.5354 -205.4023 -173.6685  True 
    disc_05to10    no_disc    -220.6933 -236.5602 -204.8264  True 
    disc_10to15  disc_15to20   26.5542   10.6873   42.4211   True 
    disc_10to15  disc_20andup  66.6324   50.7655   82.4993   True 
    disc_10to15    no_disc     35.4746   19.6077   51.3415   True 
    disc_15to20  disc_20andup  40.0783   24.2114   55.9452   True 
    disc_15to20    no_disc      8.9204   -6.9465   24.7873  False 
    disc_20andup   no_disc     -31.1579  -47.0248  -15.291   True 
    --------------------------------------------------------------





    statsmodels.sandbox.stats.multicomp.TukeyHSDResults



On most occasions of running the sampling function to get these results, the result I get is that we can **reject the null hypothesis for all pairs** _except_ the disc_15to20 / no_disc pair.  What this means is that each of these distributions of means (except for the disc_15ot20 / no_disc pair) are statistically significantly different from each other at the 0.05 level.  

I have noticed that if I reduce the number of sampling events in the function, I will occasionally see slightly different results.  Sometimes, I will get a "reject the null hypothesis" result for every single pair.  Other times, I will get "False" for a few of the pairs (usually for the 15-20% / no discount pair, but have also seen it for one or two others).  Increasing the number of sampling events from 200 to 300 in the function seems to eliminate that variability.  

##### A few comments on Tukey results

Two really interesting things that emerge from running the Tukey test are:
1.  The differences between means of the disc_15to20 distribution and the means of the no_disc (no discounts group) are NOT statistically significant.  15-19% seems to be a fairly large discount, so I am a bit surprised to see that this pair was not significantly different.


2.  ALL other pair comparisons result in statistically significant differences.
    -  While perhaps this is not surprising, it does make me wonder if there is a linear relationship between discount levels, or if it's just the fact that there is a discount.  
    -  It's important to remind ourselves that sales from discounts of 1-4.9% were so low (only \\$288 dollars of over \\$1.2 million total revenues) that they were not included.  There were only 7 data points in that category.  It suggests that either such small discounts don't work, or that the company (or its salespeople) just don't offer such discounts.

### Effect size:  Cohen's d pairwise comparisons and visualization

To find out the effect sizes of the various discount levels, we can run Cohen's d to see which ones are most important.  Except for one pairing, all were significant, so will need to do Cohen's d on each of the other pairs.


```python
d = Cohen_d(disc_05to10, disc_10to15)
plot_pdfs(d)
```

    This Cohen's d result (3.51500644) represents a difference of 3.52 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_231_1.png)


Magenta = group1 = disc_05to10 ;   blue = group2 = disc_10to15

The means of revenues from discounts of 5-9.9% are **significantly higher** than the means of revenues from discounts of 10-14.9%.

A possible mechanism with two opposing forces might explain why we see this outcome:  
1.  A greater discount may drive customers to purchase more... BUT...
2.  The larger discount may ultimately put more downward pressure on the total revenue per sale...the customer may or may not be buying more stuff, but what they are buying, they are getting for a lower price --> lower revenue per unit


```python
d = Cohen_d(disc_05to10, disc_15to20)
plot_pdfs(d)
```

    This Cohen's d result (3.15453316) represents a difference of 3.15 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_233_1.png)


Magenta = group1 = disc_05to10 ;   blue = group2 = disc_15to20

The means of revenues from discounts of 5-9.9% are **significantly higher** than the means of revenues from discounts of 15-19.9%.


```python
d = Cohen_d(disc_05to10, disc_20andup)
plot_pdfs(d)
```

    This Cohen's d result (2.18613461) represents a difference of 2.19 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_235_1.png)


Magenta = group1 = disc_05to10 ; blue = group2 = disc_20andup

The means of revenues from discounts of 5-9.9% are **significantly higher** than the means of revenues from discounts of 20% and up.



```python
d = Cohen_d(disc_05to10, no_disc)
plot_pdfs(d)
```

    This Cohen's d result (2.40795871) represents a difference of 2.41 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_237_1.png)


Magenta = group1 = disc_05to10 ;   blue = group2 = no_disc

The means of revenues from discounts of 5-9.9% are **significantly higher** than the means of revenues from non-discounted products.


```python
d = Cohen_d(disc_10to15, disc_15to20)
plot_pdfs(d)
```

    This Cohen's d result (-0.72665591) represents a difference of -0.73 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_239_1.png)


Magenta = group1 = disc_10to15 ;   blue = group2 = disc_15to20

The means of revenues from discounts of 10-14.9% are **somewhat lower** than the means of revenues from discounts of 15-20%.  Another slightly counter-intuitive result.


```python
d = Cohen_d(disc_10to15, disc_20andup)
plot_pdfs(d)
```

    This Cohen's d result (-1.11771677) represents a difference of -1.12 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_241_1.png)


Magenta = group1 = disc_10to15 ;   blue = group2 = disc_20andup

The means of revenues from discounts of 10-14.9% are **lower** than the means of revenues from discounts of 20% and up.


```python
d = Cohen_d(disc_10to15, no_disc)
plot_pdfs(d)
```

    This Cohen's d result (-0.53255028) represents a difference of -0.53 pooled standard deviations between the two groups, and suggests a **medium** effect size.



![png](student_files/student_243_1.png)


Magenta = group1 = disc_10to15 ;   blue = group2 = no_disc

The means of revenues from discounts of 10-14.9% are **slightly lower** than the means of revenues from non-discounted products.  This is a somewhat counterintuitive result that warrants further investigation.


```python
d = Cohen_d(disc_15to20, disc_20andup)  
plot_pdfs(d)
```

    This Cohen's d result (-0.67352883) represents a difference of -0.67 pooled standard deviations between the two groups, and suggests a **medium** effect size.



![png](student_files/student_245_1.png)


Magenta = group1 = disc_15to20 ;   blue = group2 = disc_20andup

The means of revenues from discounts of 15-19.9% are **slightly lower** than the means of revenues from discounts of 20% and up.


```python
d = Cohen_d(disc_15to20, no_disc)
plot_pdfs(d)

# note that this pairing was not found to be statistically significant in Tukey's test.
```

    This Cohen's d result (-0.13411241) represents a difference of -0.13 pooled standard deviations between the two groups, and suggests a **small** effect size.



![png](student_files/student_247_1.png)


Magenta = group1 = disc_15to20 ;   blue = group2 = no_disc

The means of revenues from discounts of 15-19.9% are very similar to the means of revenues from non-discounted products.


```python
d = Cohen_d(disc_20andup, no_disc)
plot_pdfs(d)
```

    This Cohen's d result (0.38229256) represents a difference of 0.38 pooled standard deviations between the two groups, and suggests a **small** effect size.



![png](student_files/student_249_1.png)


Magenta = group1 = disc_20andup ;   blue = group2 = no_disc

The means of revenues from discounts of 20% and up are **slightly higher** than the means of revenues from non-discounted products.

## Findings and interpretation of test results for Question 4

I've concluded the following after running Tukey's Test and Cohen's d for each pairwise comparison: 
-  Discounts at the 5-10% level appear to drive the highest revenues per order, relative to all other discount levels (or no discount at all)
    -  While higher discounts are likely to drive more purchases, the per-unit revenue will decrease with larger discounts.  
    -  Perhaps this relatively small discount level is enough to get customers to buy more while preserving higher revenue per unit 
    
    
-  Generally, as the discount increases, these differences shrink (though they are still quite significant) 
    -  One notable exception is that in the "5-10% discount vs. no discount" pair, the difference between the means is not as great as it is for the "5-10% discount vs. 10-15% discount" pair
    -  It is possible that, in the cases where the difference between a given discount level and no discount is less than one would expect, there are some high-volume and/or high-priced products that are never discounted that drive the no-discount revenue means upward.
    
    
-  More investigation is needed to derive insights and make recommendations; questions to pursue include:
    -  Does the pattern of revenue by discounts hold across all categories, or is there a subset of products that create the outcomes we see?
    -  Similarly, is there a relationship between geography/sales region and the discounts that drive more revenues?
    -  Do we see certain discount levels drive more revenues at certain times of the year?
    -  Is there a linear or non-linear relationship between discount level and revenues?
    -  Does removal of outliers dramatically affect the relationships observed?

# Fifth question

Does the distribution of revenues per order vary in a statistically significant way across product categories?

This analysis can show us whether the distribution of revenues per order varies in a significant way across product categories.  This can help us understand if orders in certain product categories generate higher revenues on average. 

## Null and alternative hypotheses for third question 

-  **Null hypothesis:**  No significant difference among the means of order revenues distributions by product category

    -  Ho:  $\mu_1 = \mu_2 = \mu_3 ... = \mu_i$


-  **Alternative hypothesis:** Differences among the means of order revenues distributions by product category are unlikely to be due to chance  

    -  Ha:  $\mu_1 \neq \mu_2 \neq \mu_3 \neq ... \neq \mu_i$



```python
# df.groupby(['CategoryId', 'CategoryName', 'CatDescription']).sum().sort_values(['CategoryId'])

df.groupby(['CategoryId', 'CategoryName', 'Discount']).count().head(10)
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
      <th></th>
      <th></th>
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>ProductName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
    <tr>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>Discount</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="6" valign="top">1</th>
      <th rowspan="6" valign="top">Beverages</th>
      <th>0.00</th>
      <td>246</td>
      <td>246</td>
      <td>246</td>
      <td>246</td>
      <td>246</td>
      <td>246</td>
      <td>246</td>
      <td>246</td>
      <td>246</td>
      <td>246</td>
      <td>246</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>32</td>
      <td>32</td>
      <td>32</td>
      <td>32</td>
      <td>32</td>
      <td>32</td>
      <td>32</td>
      <td>32</td>
      <td>32</td>
      <td>32</td>
      <td>32</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">2</th>
      <th rowspan="4" valign="top">Condiments</th>
      <th>0.00</th>
      <td>131</td>
      <td>131</td>
      <td>131</td>
      <td>131</td>
      <td>131</td>
      <td>131</td>
      <td>131</td>
      <td>131</td>
      <td>131</td>
      <td>131</td>
      <td>131</td>
    </tr>
    <tr>
      <th>0.02</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>23</td>
      <td>23</td>
      <td>23</td>
      <td>23</td>
      <td>23</td>
      <td>23</td>
      <td>23</td>
      <td>23</td>
      <td>23</td>
      <td>23</td>
      <td>23</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.groupby(['CategoryId', 'CategoryName', 'Discount']).sum().head(20)
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
      <th></th>
      <th></th>
      <th>Id</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
    <tr>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>Discount</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="6" valign="top">1</th>
      <th rowspan="6" valign="top">Beverages</th>
      <th>0.00</th>
      <td>2623997</td>
      <td>10381</td>
      <td>7161.95</td>
      <td>5116</td>
      <td>153060.2500</td>
      <td>12.092044</td>
      <td>0.120920</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>277313</td>
      <td>1038</td>
      <td>1507.75</td>
      <td>898</td>
      <td>43567.3800</td>
      <td>3.441904</td>
      <td>0.034419</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>264816</td>
      <td>1219</td>
      <td>812.20</td>
      <td>728</td>
      <td>13027.6350</td>
      <td>1.029207</td>
      <td>0.010292</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>384608</td>
      <td>1348</td>
      <td>609.65</td>
      <td>885</td>
      <td>12889.9950</td>
      <td>1.018334</td>
      <td>0.010183</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>418197</td>
      <td>1353</td>
      <td>1005.40</td>
      <td>1140</td>
      <td>29569.3200</td>
      <td>2.336031</td>
      <td>0.023360</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>343213</td>
      <td>1017</td>
      <td>714.70</td>
      <td>765</td>
      <td>15753.6000</td>
      <td>1.244564</td>
      <td>0.012446</td>
    </tr>
    <tr>
      <th rowspan="7" valign="top">2</th>
      <th rowspan="7" valign="top">Condiments</th>
      <th>0.00</th>
      <td>1399149</td>
      <td>5724</td>
      <td>2834.55</td>
      <td>2793</td>
      <td>60577.8000</td>
      <td>4.785759</td>
      <td>0.047858</td>
    </tr>
    <tr>
      <th>0.02</th>
      <td>11077</td>
      <td>6</td>
      <td>25.00</td>
      <td>1</td>
      <td>24.5000</td>
      <td>0.001936</td>
      <td>0.000019</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>202371</td>
      <td>953</td>
      <td>337.90</td>
      <td>719</td>
      <td>11891.5300</td>
      <td>0.939453</td>
      <td>0.009395</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>243158</td>
      <td>995</td>
      <td>449.30</td>
      <td>536</td>
      <td>8934.7050</td>
      <td>0.705858</td>
      <td>0.007059</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>180730</td>
      <td>890</td>
      <td>373.65</td>
      <td>519</td>
      <td>10639.0250</td>
      <td>0.840503</td>
      <td>0.008405</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>148734</td>
      <td>563</td>
      <td>340.50</td>
      <td>312</td>
      <td>6105.2000</td>
      <td>0.482322</td>
      <td>0.004823</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>118286</td>
      <td>426</td>
      <td>244.40</td>
      <td>418</td>
      <td>7874.3250</td>
      <td>0.622086</td>
      <td>0.006221</td>
    </tr>
    <tr>
      <th rowspan="7" valign="top">3</th>
      <th rowspan="7" valign="top">Confections</th>
      <th>0.00</th>
      <td>2131272</td>
      <td>7572</td>
      <td>4636.56</td>
      <td>4618</td>
      <td>107128.4700</td>
      <td>8.463348</td>
      <td>0.084633</td>
    </tr>
    <tr>
      <th>0.03</th>
      <td>11077</td>
      <td>16</td>
      <td>17.45</td>
      <td>2</td>
      <td>33.8530</td>
      <td>0.002674</td>
      <td>0.000027</td>
    </tr>
    <tr>
      <th>0.04</th>
      <td>11077</td>
      <td>20</td>
      <td>81.00</td>
      <td>1</td>
      <td>77.7600</td>
      <td>0.006143</td>
      <td>0.000061</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>309325</td>
      <td>843</td>
      <td>705.63</td>
      <td>795</td>
      <td>18598.5775</td>
      <td>1.469322</td>
      <td>0.014693</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>318611</td>
      <td>1062</td>
      <td>584.23</td>
      <td>545</td>
      <td>10200.7440</td>
      <td>0.805878</td>
      <td>0.008059</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>308602</td>
      <td>951</td>
      <td>617.37</td>
      <td>818</td>
      <td>15433.5690</td>
      <td>1.219281</td>
      <td>0.012193</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>180335</td>
      <td>660</td>
      <td>314.23</td>
      <td>338</td>
      <td>4716.2640</td>
      <td>0.372594</td>
      <td>0.003726</td>
    </tr>
  </tbody>
</table>
</div>



## EDA

### Bar graphs

#### Product discount by product category


```python
df.groupby('CategoryName')['Discount'].sum().plot(kind='bar', 
            title ="Discount Level in Percentages by Product Category", figsize=(8, 5), fontsize=12)
plt.show()

```


![png](student_files/student_261_0.png)


The category with the greatest discounts is Beverages, with Seafood, Dairy Products, and Confections nearly tied in 2nd place with approximately 20% discounts in each category.

Let's also look at revenues by product category and see how that compares to percentage discount:

#### Total revenues by product category


```python
df.groupby('CategoryName')['Revenue'].sum().plot(kind='bar', 
            title ="Revenues by Category Name", figsize=(8, 5), fontsize=12)
plt.show()

```


![png](student_files/student_264_0.png)


#### Number of orders by product category


```python
df.groupby('CategoryName')['Discount'].count().plot(kind='bar', 
            title ="Orders by Product Category", figsize=(8, 5), fontsize=12)
plt.show()

```


![png](student_files/student_266_0.png)


### Comments on initial EDA

Interestingly, Beverages has the highest revenues at around \\$260,000 of any category, followed closely by Dairy Products at around \\$230,000, then Confections at around \\$170,000, and Meat/Poultry at around \\$160,000.  Although Seafood tends to have fairly significant discounts at around 20%, it is 5th of 8 in terms of revenu, at around \\$140,000. 

Some observations and comments:
  -  For a category such as seafood, which is likely to include both highly perishable items and shelf-stable items (e.g., canned tuna, salmon, clams), it may not be surprising to see that discounts make up a higher proportion of revenues than is the case for less perishable items.  
  -  On the other hand, Produce has fairly low discounts and relatively low revenues.  I can think of a couple of possible questions that might warrant further exploration:
    -  Although perishable, produce quantities and discounts might have been optimized for its highly perishable nature; in other words, maybe Northwind only has access to a certain amount of produce from suppliers, and this amount tends to sell out (rather than having to be discarded).  
    -  While there may be some truth to this potential explanation, as a category, Produce experiences relatively high rates of loss due to spoilage compared to other categories, so there might be some  opportunities to get more revenue by offering greater discounts on selected items (thereby selling produce that might otherwise have gone to waste).  
  -  Condiments and Cereals/Grains are almost carbon copies of each other, in terms of both discount level and revenues.  It is likely that the relatively low cost and shelf-stable nature of these products leads to relatively modest discounts (since spoilage rates are lower, so not as much urgency to induce purchases before items spoil) and relatively modest revenues, due to their general affordability.  

### Setting  up product category dataframes


```python
df_bev = df.loc[df["CategoryName"] == "Beverages"]
df_bev.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10285</td>
      <td>QUICK</td>
      <td>2012-08-20</td>
      <td>1</td>
      <td>14.4</td>
      <td>45</td>
      <td>0.2</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>518.4</td>
      <td>0.040955</td>
      <td>0.000410</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10294</td>
      <td>RATTC</td>
      <td>2012-08-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>18</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>259.2</td>
      <td>0.020477</td>
      <td>0.000205</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10317</td>
      <td>LONEP</td>
      <td>2012-09-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>20</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>288.0</td>
      <td>0.022753</td>
      <td>0.000228</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_cond = df.loc[df["CategoryName"] == "Condiments"]
df_cond.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>82</th>
      <td>10289</td>
      <td>BSBEV</td>
      <td>2012-08-26</td>
      <td>3</td>
      <td>8.0</td>
      <td>30</td>
      <td>0.0</td>
      <td>Aniseed Syrup</td>
      <td>2</td>
      <td>Condiments</td>
      <td>Sweet and savory sauces, relishes, spreads, an...</td>
      <td>240.0</td>
      <td>0.018960</td>
      <td>0.000190</td>
    </tr>
    <tr>
      <th>83</th>
      <td>10405</td>
      <td>LINOD</td>
      <td>2013-01-06</td>
      <td>3</td>
      <td>8.0</td>
      <td>50</td>
      <td>0.0</td>
      <td>Aniseed Syrup</td>
      <td>2</td>
      <td>Condiments</td>
      <td>Sweet and savory sauces, relishes, spreads, an...</td>
      <td>400.0</td>
      <td>0.031601</td>
      <td>0.000316</td>
    </tr>
    <tr>
      <th>84</th>
      <td>10485</td>
      <td>LINOD</td>
      <td>2013-03-25</td>
      <td>3</td>
      <td>8.0</td>
      <td>20</td>
      <td>0.1</td>
      <td>Aniseed Syrup</td>
      <td>2</td>
      <td>Condiments</td>
      <td>Sweet and savory sauces, relishes, spreads, an...</td>
      <td>144.0</td>
      <td>0.011376</td>
      <td>0.000114</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_confect = df.loc[df["CategoryName"] == "Confections"]
df_confect.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>336</th>
      <td>10255</td>
      <td>RICSU</td>
      <td>2012-07-12</td>
      <td>16</td>
      <td>13.9</td>
      <td>35</td>
      <td>0.00</td>
      <td>Pavlova</td>
      <td>3</td>
      <td>Confections</td>
      <td>Desserts, candies, and sweet breads</td>
      <td>486.5</td>
      <td>0.038434</td>
      <td>0.000384</td>
    </tr>
    <tr>
      <th>337</th>
      <td>10263</td>
      <td>ERNSH</td>
      <td>2012-07-23</td>
      <td>16</td>
      <td>13.9</td>
      <td>60</td>
      <td>0.25</td>
      <td>Pavlova</td>
      <td>3</td>
      <td>Confections</td>
      <td>Desserts, candies, and sweet breads</td>
      <td>625.5</td>
      <td>0.049416</td>
      <td>0.000494</td>
    </tr>
    <tr>
      <th>338</th>
      <td>10287</td>
      <td>RICAR</td>
      <td>2012-08-22</td>
      <td>16</td>
      <td>13.9</td>
      <td>40</td>
      <td>0.15</td>
      <td>Pavlova</td>
      <td>3</td>
      <td>Confections</td>
      <td>Desserts, candies, and sweet breads</td>
      <td>472.6</td>
      <td>0.037336</td>
      <td>0.000373</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_dairy = df.loc[df["CategoryName"] == "Dairy Products"]
df_dairy.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>216</th>
      <td>10248</td>
      <td>VINET</td>
      <td>2012-07-04</td>
      <td>11</td>
      <td>14.0</td>
      <td>12</td>
      <td>0.0</td>
      <td>Queso Cabrales</td>
      <td>4</td>
      <td>Dairy Products</td>
      <td>Cheeses</td>
      <td>168.0</td>
      <td>0.013272</td>
      <td>0.000133</td>
    </tr>
    <tr>
      <th>217</th>
      <td>10296</td>
      <td>LILAS</td>
      <td>2012-09-03</td>
      <td>11</td>
      <td>16.8</td>
      <td>12</td>
      <td>0.0</td>
      <td>Queso Cabrales</td>
      <td>4</td>
      <td>Dairy Products</td>
      <td>Cheeses</td>
      <td>201.6</td>
      <td>0.015927</td>
      <td>0.000159</td>
    </tr>
    <tr>
      <th>218</th>
      <td>10327</td>
      <td>FOLKO</td>
      <td>2012-10-11</td>
      <td>11</td>
      <td>16.8</td>
      <td>50</td>
      <td>0.2</td>
      <td>Queso Cabrales</td>
      <td>4</td>
      <td>Dairy Products</td>
      <td>Cheeses</td>
      <td>672.0</td>
      <td>0.053089</td>
      <td>0.000531</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_grains = df.loc[df["CategoryName"] == "Grains/Cereals"]
df_grains.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>535</th>
      <td>10251</td>
      <td>VICTE</td>
      <td>2012-07-08</td>
      <td>22</td>
      <td>16.8</td>
      <td>6</td>
      <td>0.05</td>
      <td>Gustaf's Knäckebröd</td>
      <td>5</td>
      <td>Grains/Cereals</td>
      <td>Breads, crackers, pasta, and cereal</td>
      <td>95.76</td>
      <td>0.007565</td>
      <td>0.000076</td>
    </tr>
    <tr>
      <th>536</th>
      <td>10435</td>
      <td>CONSH</td>
      <td>2013-02-04</td>
      <td>22</td>
      <td>16.8</td>
      <td>12</td>
      <td>0.00</td>
      <td>Gustaf's Knäckebröd</td>
      <td>5</td>
      <td>Grains/Cereals</td>
      <td>Breads, crackers, pasta, and cereal</td>
      <td>201.60</td>
      <td>0.015927</td>
      <td>0.000159</td>
    </tr>
    <tr>
      <th>537</th>
      <td>10553</td>
      <td>WARTH</td>
      <td>2013-05-30</td>
      <td>22</td>
      <td>21.0</td>
      <td>24</td>
      <td>0.00</td>
      <td>Gustaf's Knäckebröd</td>
      <td>5</td>
      <td>Grains/Cereals</td>
      <td>Breads, crackers, pasta, and cereal</td>
      <td>504.00</td>
      <td>0.039817</td>
      <td>0.000398</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_meat = df.loc[df["CategoryName"] == "Meat/Poultry"]
df_meat.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>178</th>
      <td>10420</td>
      <td>WELLI</td>
      <td>2013-01-21</td>
      <td>9</td>
      <td>77.6</td>
      <td>20</td>
      <td>0.10</td>
      <td>Mishi Kobe Niku</td>
      <td>6</td>
      <td>Meat/Poultry</td>
      <td>Prepared meats</td>
      <td>1396.8</td>
      <td>0.110350</td>
      <td>0.001103</td>
    </tr>
    <tr>
      <th>179</th>
      <td>10515</td>
      <td>QUICK</td>
      <td>2013-04-23</td>
      <td>9</td>
      <td>97.0</td>
      <td>16</td>
      <td>0.15</td>
      <td>Mishi Kobe Niku</td>
      <td>6</td>
      <td>Meat/Poultry</td>
      <td>Prepared meats</td>
      <td>1319.2</td>
      <td>0.104219</td>
      <td>0.001042</td>
    </tr>
    <tr>
      <th>180</th>
      <td>10687</td>
      <td>HUNGO</td>
      <td>2013-09-30</td>
      <td>9</td>
      <td>97.0</td>
      <td>50</td>
      <td>0.25</td>
      <td>Mishi Kobe Niku</td>
      <td>6</td>
      <td>Meat/Poultry</td>
      <td>Prepared meats</td>
      <td>3637.5</td>
      <td>0.287369</td>
      <td>0.002874</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_produce = df.loc[df["CategoryName"] == "Produce"]
df_produce.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>136</th>
      <td>10262</td>
      <td>RATTC</td>
      <td>2012-07-22</td>
      <td>7</td>
      <td>24.0</td>
      <td>15</td>
      <td>0.00</td>
      <td>Uncle Bob's Organic Dried Pears</td>
      <td>7</td>
      <td>Produce</td>
      <td>Dried fruit and bean curd</td>
      <td>360.0</td>
      <td>0.028441</td>
      <td>0.000284</td>
    </tr>
    <tr>
      <th>137</th>
      <td>10385</td>
      <td>SPLIR</td>
      <td>2012-12-17</td>
      <td>7</td>
      <td>24.0</td>
      <td>10</td>
      <td>0.20</td>
      <td>Uncle Bob's Organic Dried Pears</td>
      <td>7</td>
      <td>Produce</td>
      <td>Dried fruit and bean curd</td>
      <td>192.0</td>
      <td>0.015168</td>
      <td>0.000152</td>
    </tr>
    <tr>
      <th>138</th>
      <td>10459</td>
      <td>VICTE</td>
      <td>2013-02-27</td>
      <td>7</td>
      <td>24.0</td>
      <td>16</td>
      <td>0.05</td>
      <td>Uncle Bob's Organic Dried Pears</td>
      <td>7</td>
      <td>Produce</td>
      <td>Dried fruit and bean curd</td>
      <td>364.8</td>
      <td>0.028820</td>
      <td>0.000288</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_seafood = df.loc[df["CategoryName"] == "Seafood"]
df_seafood.head(3)
# df_seafood.info()
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>183</th>
      <td>10273</td>
      <td>QUICK</td>
      <td>2012-08-05</td>
      <td>10</td>
      <td>24.8</td>
      <td>24</td>
      <td>0.05</td>
      <td>Ikura</td>
      <td>8</td>
      <td>Seafood</td>
      <td>Seaweed and fish</td>
      <td>565.44</td>
      <td>0.044671</td>
      <td>0.000447</td>
    </tr>
    <tr>
      <th>184</th>
      <td>10276</td>
      <td>TORTU</td>
      <td>2012-08-08</td>
      <td>10</td>
      <td>24.8</td>
      <td>15</td>
      <td>0.00</td>
      <td>Ikura</td>
      <td>8</td>
      <td>Seafood</td>
      <td>Seaweed and fish</td>
      <td>372.00</td>
      <td>0.029389</td>
      <td>0.000294</td>
    </tr>
    <tr>
      <th>185</th>
      <td>10357</td>
      <td>LILAS</td>
      <td>2012-11-19</td>
      <td>10</td>
      <td>24.8</td>
      <td>30</td>
      <td>0.20</td>
      <td>Ikura</td>
      <td>8</td>
      <td>Seafood</td>
      <td>Seaweed and fish</td>
      <td>595.20</td>
      <td>0.047022</td>
      <td>0.000470</td>
    </tr>
  </tbody>
</table>
</div>



### Histograms of product categories


```python
print()
print("Category:  Beverages")
hist_plot1(df_bev.Revenue, label="Avg order revenue, discount and no discount, Beverages", color='tab:blue', print_stmt=False)
print(f"The mean of the distribution is {round(df_bev.Revenue.mean(), 2)}")
hist_plot1(df_bev.Revenue, label="Avg order revenue, discount and no discount, Beverages", color='tab:blue', xlim=None, print_stmt=False)
print(f"The mean of the distribution is {round(df_bev.Revenue.mean(), 2)}")
print()
print()

# note the large outliers
```

    
    Category:  Beverages



![png](student_files/student_279_1.png)


    The mean of the distribution is 663.04



![png](student_files/student_279_3.png)


    The mean of the distribution is 663.04
    
    



```python
print()
print()
print("Category:  Condiments")
# hist_plot1(df_cond.Revenue, color='tab:blue')
# print(f"The mean of the distribution is {round(df_cond.Revenue.mean(), 2)}")
hist_plot1(df_cond.Revenue, label="Avg order revenue, discount and no discount, Condiments", color='tab:blue', xlim=None, print_stmt=False)
print(f"The mean of the distribution is {round(df_cond.Revenue.mean(), 2)}")
print()
print()

```

    
    
    Category:  Condiments



![png](student_files/student_280_1.png)


    The mean of the distribution is 490.96
    
    



```python
print()
print()
print("Category:  Confections")
hist_plot1(df_confect.Revenue, label="Avg order revenue, discount and no discount, Confections", color='tab:blue', print_stmt=False)
print(f"The mean of the distribution is {round(df_confect.Revenue.mean(), 2)}")
# hist_plot1(df_confect.Revenue, color='tab:blue', xlim=None)
# print(f"The mean of the distribution is {round(df_confect.Revenue.mean(), 2)}")
print()
print()

```

    
    
    Category:  Confections



![png](student_files/student_281_1.png)


    The mean of the distribution is 501.07
    
    



```python
print()
print()
print("Category:  Dairy Products")
hist_plot1(df_dairy.Revenue, label="Avg order revenue, all, Dairy Products", color='tab:blue', print_stmt=False)
print(f"The mean of the distribution is {round(df_dairy.Revenue.mean(), 2)}")
# hist_plot1(df_dairy.Revenue, color='tab:blue', xlim=None)
# print(f"The mean of the distribution is {round(df_dairy.Revenue.mean(), 2)}")
print()
print()

```

    
    
    Category:  Dairy Products



![png](student_files/student_282_1.png)


    The mean of the distribution is 640.73
    
    



```python
print()
print()
print("Category:  Meat/Poultry")
hist_plot1(df_meat.Revenue, label="Avg order revenue, discount/no discount, Meat/Poultry",color='tab:blue', print_stmt=False)
print(f"The mean of the distribution is {round(df_meat.Revenue.mean(), 2)}")
# hist_plot1(df_meat.Revenue, color='tab:blue', xlim=None)
# print(f"The mean of the distribution is {round(df_meat.Revenue.mean(), 2)}")
print()
print()

```

    
    
    Category:  Meat/Poultry



![png](student_files/student_283_1.png)


    The mean of the distribution is 942.33
    
    



```python
print()
print()
print("Category:  Grains/Cereals")
hist_plot1(df_grains.Revenue, label="Avg order revenue, discount/no discount, Grains/Cereals", color='tab:blue', print_stmt=False)
print(f"The mean of the distribution is {round(df_grains.Revenue.mean(), 2)}")
# hist_plot1(df_grains.Revenue, color='tab:blue', xlim=None)
# print(f"The mean of the distribution is {round(df_grains.Revenue.mean(), 2)}")
print()
print()

```

    
    
    Category:  Grains/Cereals



![png](student_files/student_284_1.png)


    The mean of the distribution is 488.49
    
    



```python
print()
print()
print("Category:  Produce")
hist_plot1(df_produce.Revenue, label="Avg order revenue, discount/no discount, Produce", color='tab:blue', print_stmt=False)
print(f"The mean of the distribution is {round(df_produce.Revenue.mean(), 2)}")
# hist_plot1(df_produce.Revenue, color='tab:blue', xlim=None)
# print(f"The mean of the distribution is {round(df_produce.Revenue.mean(), 2)}")
print()
print()

```

    
    
    Category:  Produce



![png](student_files/student_285_1.png)


    The mean of the distribution is 735.18
    
    



```python
print()
print()
print("Category:  Seafood")
# hist_plot1(df_seafood.Revenue, color='tab:blue')
# print(f"The mean of the distribution is {round(df_seafood.Revenue.mean(), 2)}")
hist_plot1(df_seafood.Revenue, label="Avg order revenue, discount/no discount, Seafood", color='tab:blue', xlim=None, print_stmt=False)
print(f"The mean of the distribution is {round(df_seafood.Revenue.mean(), 2)}")
print()
print()

```

    
    
    Category:  Seafood



![png](student_files/student_286_1.png)


    The mean of the distribution is 397.76
    
    


## Testing for significance:  sampling, test setup, and results

### Pulling samples for each product category and taking means

As with question 2, to ensure reasonably normal distributions for hypothesis testing, I will perform resampling for each product category to get a distribution of means. 


```python
bev_means = sampling_mean(df_bev.Revenue)
mean_bev_means = np.mean(bev_means[0])
std_bev_means = np.std(bev_means[1])
print(f"The mean of the means of beverage order revenues resampling is {round(mean_bev_means,2)}")
print(f"The std deviation of the means of beverage order revenues resampling is {round(std_bev_means,2)}")
hist_plot1(bev_means[0], label="Resampling means, order revenues, Beverages", xlim=([0, 1400]), print_stmt=False)

```

    The mean of the means of beverage order revenues resampling is 663.44
    The std deviation of the means of beverage order revenues resampling is 486.78



![png](student_files/student_289_1.png)



```python
cond_means = sampling_mean(df_cond.Revenue)
mean_cond_means = np.mean(cond_means[0])
std_cond_means = np.std(cond_means[1])
print(f"The mean of the means of Condiments order revenues resampling is {round(mean_cond_means,2)}")
print(f"The std deviation of the means of Condiments order revenues resampling is {round(std_cond_means,2)}")
hist_plot1(cond_means[0], label="Resampling means, order revenues, Condiments",  xlim=([0, 1000]), print_stmt=False)
```

    The mean of the means of Condiments order revenues resampling is 491.51
    The std deviation of the means of Condiments order revenues resampling is 40.06



![png](student_files/student_290_1.png)



```python
confect_means = sampling_mean(df_confect.Revenue)
mean_confect_means = np.mean(confect_means[0])
std_confect_means = np.std(confect_means[1])
print(f"The mean of the means of Confections order revenues resampling is {round(mean_confect_means,2)}")
print(f"The std deviation of the means of Confections order revenues resampling is {round(std_confect_means,2)}")
hist_plot1(confect_means[0], label="Resampling means, order revenues, Confections", xlim=([0, 1000]), print_stmt=False)
```

    The mean of the means of Confections order revenues resampling is 499.85
    The std deviation of the means of Confections order revenues resampling is 100.95



![png](student_files/student_291_1.png)



```python
meat_means = sampling_mean(df_meat.Revenue)
mean_meat_means = np.mean(meat_means[0])
std_meat_means = np.std(meat_means[1])
round(mean_meat_means,2)
print(f"The mean of the means of Meat/Poultry order revenues resampling is {round(mean_meat_means,2)}")
print(f"The std deviation of the means of Meat/Poultry order revenues resampling is {round(std_meat_means,2)}")
hist_plot1(meat_means[0], label="Resampling means, order revenues, Meat/Poultry", xlim=([0, 2000]), print_stmt=False)

```

    The mean of the means of Meat/Poultry order revenues resampling is 938.78
    The std deviation of the means of Meat/Poultry order revenues resampling is 184.78



![png](student_files/student_292_1.png)



```python
dairy_means = sampling_mean(df_dairy.Revenue)
mean_dairy_means = np.mean(dairy_means[0])
std_dairy_means = np.std(dairy_means[1])
round(mean_dairy_means,2)
print(f"The mean of the means of Dairy Products order revenues resampling is {round(mean_dairy_means,2)}")
print(f"The std deviation of the means of Dairy Products order revenues resampling is {round(std_dairy_means,2)}")
hist_plot1(dairy_means[0], label="Resampling means, order revenues, Dairy Products", xlim=([0, 1200]), print_stmt=False)

```

    The mean of the means of Dairy Products order revenues resampling is 645.36
    The std deviation of the means of Dairy Products order revenues resampling is 119.31



![png](student_files/student_293_1.png)



```python
grains_means = sampling_mean(df_grains.Revenue)
mean_grains_means = np.mean(grains_means[0])
std_grains_means = np.std(grains_means[1])
round(mean_grains_means,2)
print(f"The mean of the means of Grains/Cereals Products order revenues resampling is {round(mean_grains_means,2)}")
print(f"The std deviation of the means of Grains/Cereals Products order revenues resampling is {round(std_grains_means,2)}")
hist_plot1(grains_means[0], label="Resampling means, order revenues, Grains/Cereals", xlim=([0, 1000]), print_stmt=False)

```

    The mean of the means of Grains/Cereals Products order revenues resampling is 487.07
    The std deviation of the means of Grains/Cereals Products order revenues resampling is 80.64



![png](student_files/student_294_1.png)



```python
produce_means = sampling_mean(df_produce.Revenue)
mean_produce_means = np.mean(produce_means[0])
std_produce_means = np.std(produce_means[1])
round(mean_produce_means,2)
print(f"The mean of the means of Produce order revenues resampling is {round(mean_produce_means,2)}")
print(f"The std deviation of the means of Produce order revenues resampling is {round(std_produce_means,2)}")
hist_plot1(produce_means[0], label="Resampling means, order revenues, Produce", xlim=([0, 1400]), print_stmt=False)
```

    The mean of the means of Produce order revenues resampling is 734.53
    The std deviation of the means of Produce order revenues resampling is 92.73



![png](student_files/student_295_1.png)



```python
seafood_means = sampling_mean(df_seafood.Revenue)
mean_seafood_means = np.mean(seafood_means[0])
std_seafood_means = np.std(seafood_means[1])
round(mean_seafood_means,2)
print(f"The mean of the means of Seafood order revenues resampling is {round(mean_seafood_means,2)}")
print(f"The std deviation of the means of Seafood order revenues resampling is {round(std_seafood_means,2)}")
hist_plot1(seafood_means[0], label="Resampling means, order revenues, Seafood", xlim=([0, 800]), print_stmt=False)

```

    The mean of the means of Seafood order revenues resampling is 398.49
    The std deviation of the means of Seafood order revenues resampling is 61.55



![png](student_files/student_296_1.png)


### Average order revenue by product category


```python
means = [mean_bev_means, mean_cond_means, mean_confect_means, mean_meat_means, 
         mean_dairy_means, mean_grains_means, mean_produce_means, mean_seafood_means]

names = ['Beverages', 'Condiments', 'Confections', 
         'Meat/Poultry', 'Dairy Products', 'Grains/Cereals',
        'Produce','Seafood']

df_means = pd.DataFrame(means, names)

df_means.plot(kind='bar', color='tab:blue', alpha=.9, legend=None, title ="Average Order Revenue by Product Category", figsize=(8, 5), fontsize=12)
plt.show()
```


![png](student_files/student_298_0.png)


### ANOVA test on means distributions


```python
scs.f_oneway(bev_means[0], cond_means[0], confect_means[0], meat_means[0], dairy_means[0], grains_means[0], produce_means[0], seafood_means[0])

```




    F_onewayResult(statistic=1776.047633256583, pvalue=0.0)



The ANOVA test on the sample means shows an infinitesimally small p-value, meaning that we should reject the null hypothesis that there is no statistically significant difference among the revenue distributions of the different product categories.

To find out which distributions are different enough from one another to be statistically significant, I will run Tukey's test.

### Tukey's test on means distributions


```python
bev_means_np = np.array(bev_means[0])
cond_means_np = np.array(cond_means[0]) 
confect_means_np = np.array(confect_means[0])
dairy_means_np = np.array(dairy_means[0])
meat_means_np = np.array(meat_means[0]) 
grains_means_np = np.array(grains_means[0])
produce_means_np = np.array(produce_means[0]) 
seafood_means_np = np.array(seafood_means[0])

v = np.concatenate([bev_means_np, cond_means_np, confect_means_np,
                    dairy_means_np, meat_means_np, grains_means_np, 
                    produce_means_np, seafood_means_np])
labels = ['Beverages'] * len(bev_means_np) + ['Condiments'] * len(cond_means_np) + ['Confections'] * len(confect_means_np) + ['Dairy Products']*len(dairy_means_np) + ['Meat/Poultry']*len(meat_means_np) + ['Cereals/Grains']*len(grains_means_np) + ['Produce']*len(produce_means_np) + ['Seafood']*len(seafood_means_np)
tukey_results = pairwise_tukeyhsd(v, labels, 0.05)
print(tukey_results)

type(tukey_results)
```

            Multiple Comparison of Means - Tukey HSD,FWER=0.05        
    ==================================================================
        group1         group2      meandiff   lower     upper   reject
    ------------------------------------------------------------------
      Beverages    Cereals/Grains -176.3666 -194.1524 -158.5807  True 
      Beverages      Condiments   -171.9365 -189.7223 -154.1507  True 
      Beverages     Confections    -163.593 -181.3788 -145.8072  True 
      Beverages    Dairy Products  -18.0862  -35.872   -0.3004   True 
      Beverages     Meat/Poultry   275.3426  257.5568  293.1284  True 
      Beverages       Produce      71.0916   53.3058   88.8774   True 
      Beverages       Seafood     -264.9544 -282.7402 -247.1686  True 
    Cereals/Grains   Condiments     4.4301   -13.3557  22.2159  False 
    Cereals/Grains  Confections    12.7736   -5.0122   30.5594  False 
    Cereals/Grains Dairy Products  158.2803  140.4945  176.0662  True 
    Cereals/Grains  Meat/Poultry   451.7091  433.9233  469.4949  True 
    Cereals/Grains    Produce      247.4582  229.6724  265.244   True 
    Cereals/Grains    Seafood      -88.5879 -106.3737  -70.8021  True 
      Condiments    Confections     8.3435   -9.4423   26.1293  False 
      Condiments   Dairy Products  153.8503  136.0644  171.6361  True 
      Condiments    Meat/Poultry   447.279   429.4932  465.0649  True 
      Condiments      Produce      243.0281  225.2423  260.8139  True 
      Condiments      Seafood      -93.018  -110.8038  -75.2321  True 
     Confections   Dairy Products  145.5068  127.7209  163.2926  True 
     Confections    Meat/Poultry   438.9355  421.1497  456.7214  True 
     Confections      Produce      234.6846  216.8988  252.4704  True 
     Confections      Seafood     -101.3615 -119.1473  -83.5756  True 
    Dairy Products  Meat/Poultry   293.4288  275.643   311.2146  True 
    Dairy Products    Produce      89.1778    71.392   106.9637  True 
    Dairy Products    Seafood     -246.8682  -264.654 -229.0824  True 
     Meat/Poultry     Produce     -204.2509 -222.0368 -186.4651  True 
     Meat/Poultry     Seafood      -540.297 -558.0828 -522.5112  True 
       Produce        Seafood      -336.046 -353.8319 -318.2602  True 
    ------------------------------------------------------------------





    statsmodels.sandbox.stats.multicomp.TukeyHSDResults



### Effect size:  Cohen's D

We can get a sense of the magnitude of the statistically significant differences among the product categories by computing and plotting Cohen's d for each pair.


```python
d_bev_cond = Cohen_d(bev_means_np, cond_means_np)
plot_pdfs(d_bev_cond)
```

    This Cohen's d result (1.62585469) represents a difference of 1.63 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_306_1.png)



```python
d_bev_dairy = Cohen_d(bev_means_np, dairy_means_np)
plot_pdfs(d_bev_dairy)
```

    This Cohen's d result (0.16055547) represents a difference of 0.16 pooled standard deviations between the two groups, and suggests a **small** effect size.



![png](student_files/student_307_1.png)



```python
d_bev_meat = Cohen_d(bev_means_np, meat_means_np)
plot_pdfs(d_bev_meat)
```

    This Cohen's d result (-2.27595947) represents a difference of -2.28 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_308_1.png)



```python
d_bev_grains = Cohen_d(bev_means_np, grains_means_np)
plot_pdfs(d_bev_grains)
```

    This Cohen's d result (1.65508548) represents a difference of 1.66 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_309_1.png)



```python
d_bev_produce = Cohen_d(bev_means_np, produce_means_np)
plot_pdfs(d_bev_produce)
```

    This Cohen's d result (-0.66677525) represents a difference of -0.67 pooled standard deviations between the two groups, and suggests a **medium** effect size.



![png](student_files/student_310_1.png)



```python
d_bev_seafood = Cohen_d(bev_means_np, seafood_means_np)
plot_pdfs(d_bev_seafood)
```

    This Cohen's d result (2.49154765) represents a difference of 2.49 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_311_1.png)



```python
d_cond_confect = Cohen_d(cond_means_np, confect_means_np)
plot_pdfs(d_cond_confect)
```

    This Cohen's d result (-0.19889735) represents a difference of -0.2 pooled standard deviations between the two groups, and suggests a **small** effect size.



![png](student_files/student_312_1.png)



```python
d_cond_dairy = Cohen_d(cond_means_np, dairy_means_np)
plot_pdfs(d_cond_dairy)
```

    This Cohen's d result (-3.02144627) represents a difference of -3.02 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_313_1.png)



```python
d_cond_meat = Cohen_d(cond_means_np, meat_means_np)
plot_pdfs(d_cond_meat)
```

    This Cohen's d result (-6.63885587) represents a difference of -6.64 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_314_1.png)



```python
d_cond_grains = Cohen_d(cond_means_np, grains_means_np)
plot_pdfs(d_cond_grains)
```

    This Cohen's d result (0.12488345) represents a difference of 0.12 pooled standard deviations between the two groups, and suggests a **small** effect size.



![png](student_files/student_315_1.png)



```python
d_cond_produce = Cohen_d(cond_means_np, produce_means_np)
plot_pdfs(d_cond_produce)
```

    This Cohen's d result (-6.81656087) represents a difference of -6.82 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_316_1.png)



```python
d_cond_seafood = Cohen_d(cond_means_np, seafood_means_np)
plot_pdfs(d_cond_seafood)
```

    This Cohen's d result (2.67215594) represents a difference of 2.67 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_317_1.png)



```python
d_confect_dairy = Cohen_d(confect_means_np, dairy_means_np)
plot_pdfs(d_confect_dairy)
```

    This Cohen's d result (-2.54615497) represents a difference of -2.55 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_318_1.png)



```python
d_confect_meat = Cohen_d(confect_means_np, meat_means_np)
plot_pdfs(d_confect_meat)
```

    This Cohen's d result (-6.07982689) represents a difference of -6.08 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_319_1.png)



```python
d_confect_grains = Cohen_d(confect_means_np, grains_means_np)
plot_pdfs(d_confect_grains)
```

    This Cohen's d result (0.29065039) represents a difference of 0.29 pooled standard deviations between the two groups, and suggests a **small** effect size.



![png](student_files/student_320_1.png)



```python
d_confect_produce = Cohen_d(confect_means_np, produce_means_np)
plot_pdfs(d_confect_produce)
```

    This Cohen's d result (-5.32251585) represents a difference of -5.32 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_321_1.png)



```python
d_confect_seafood = Cohen_d(confect_means_np, seafood_means_np)
plot_pdfs(d_confect_seafood)
```

    This Cohen's d result (2.33474402) represents a difference of 2.33 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_322_1.png)



```python
d_dairy_meat = Cohen_d(dairy_means_np, meat_means_np)
plot_pdfs(d_dairy_meat)
```

    This Cohen's d result (-3.77394204) represents a difference of -3.77 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_323_1.png)



```python
d_dairy_grains = Cohen_d(dairy_means_np, grains_means_np)
plot_pdfs(d_dairy_grains)
```

    This Cohen's d result (3.01034195) represents a difference of 3.01 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_324_1.png)



```python
d_dairy_produce = Cohen_d(dairy_means_np, produce_means_np)
plot_pdfs(d_dairy_produce)
```

    This Cohen's d result (-1.69218869) represents a difference of -1.69 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_325_1.png)



```python
d_dairy_seafood = Cohen_d(dairy_means_np, seafood_means_np)
plot_pdfs(d_dairy_seafood)
```

    This Cohen's d result (4.73531703) represents a difference of 4.74 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_326_1.png)



```python
d_meat_grains = Cohen_d(meat_means_np, grains_means_np)
plot_pdfs(d_meat_grains)
```

    This Cohen's d result (6.58125633) represents a difference of 6.58 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_327_1.png)



```python
d_meat_produce = Cohen_d(meat_means_np, produce_means_np)
plot_pdfs(d_meat_produce)
```

    This Cohen's d result (2.97186024) represents a difference of 2.97 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_328_1.png)



```python
d_meat_seafood = Cohen_d(meat_means_np, seafood_means_np)
plot_pdfs(d_meat_seafood)
```

    This Cohen's d result (7.91121656) represents a difference of 7.91 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_329_1.png)



```python
d_grains_produce = Cohen_d(grains_means_np, produce_means_np)
plot_pdfs(d_grains_produce)
```

    This Cohen's d result (-6.51463463) represents a difference of -6.51 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_330_1.png)



```python
d_grains_seafood = Cohen_d(grains_means_np, seafood_means_np)
plot_pdfs(d_grains_seafood)
```

    This Cohen's d result (2.38169432) represents a difference of 2.38 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_331_1.png)



```python
d_produce_seafood = Cohen_d(produce_means_np, seafood_means_np)
plot_pdfs(d_produce_seafood)
```

    This Cohen's d result (8.99337848) represents a difference of 8.99 pooled standard deviations between the two groups, and suggests a **large** effect size.



![png](student_files/student_332_1.png)


## Findings and interpretation

The results of Tukey's test suggest that the magnitude of the differences of the distribution of means for each product category is statistically significant for all but a few category pairings:  Cereals/Grains and Condiments; Cereals/Grains and Confections; Condiments and Confections; and possibly one or two other pairings (have seen Beverages and Dairy Products not being statistically significant on occasion).  This result is interesting--but not really surprising--in light of what is shown in the bar graph of revenues by category:  the three categories represented by the first three pairing above are the most shelf-stable, and relatively inexpensive, of the 8 product categories.  They also are discounted relatively less frequently than most other categories (with the possible exception of produce), probably due to lower rates of spoilage, therefore requiring fewer discounts to move products before they spoil.  

In addition, we see some very large differences in average order value among certain product categories.  The bar chart showing the average order revenue by category shows that in an intuitive manner; Tukey's test and Cohen's d tell us which differences are statistically significant (all but 4) and to what degree.  

This last point on the value of Cohen's d is important:  while you can see differences in the original bar chart across product categories, it's interesting to note that the difference shown by Cohen's d are sometimes more or less dramatic than the bar chart implies.  

As a reminder, here is the bar chart of average order revenue by product category:


```python
df_means.plot(kind='bar', legend=None, title ="Average Order Revenue by Product Category", figsize=(8, 5), fontsize=12)
plt.show()
```


![png](student_files/student_336_0.png)


While there is a notable difference on the bar chart between, say, Produce and Condiments, the Cohen's d value for this pair is 6.65!  That's close to 7 pooled standard deviations separating the means of these two categories.  That suggests that the difference between these two is even more significant than this bar chart would suggest.  This is instructive because it can help us identify under- or over-performing categories on which to focus specific incentives.

# Future potential work:  Additional questions

I have a number of general questions that I would like to explore.  Below are a few that I had started to investigate, but did not have time to complete.

## Sixth possible question

We have seen that there are substantial differences in the revenues and discounts offered across the 8 categories.  But what about within categories?  Do the discounts within categories vary by product?

### Null and alternative hypotheses for fourth question 

-  **Null hypothesis:**  No significant difference among the means of discount values by product within a category

    -  Ho:  $\mu_1 = \mu_2 = \mu_3 ... = \mu_i$)


-  **Alternative hypothesis:** Differences among the means of discount values by product within a category are unlikely to be due to chance  

    -  Ha:  $\mu_1 \neq \mu_2 \neq \mu_3 \neq ... \neq \mu_i$


### EDA

###  High-level overview:  Beverages

Looking at the distribution of revenues in this category by discount...


```python
df_bev = df.loc[df["CategoryName"] == "Beverages"]

```


```python
df_bev.groupby(['Discount']).sum().sort_values(['Discount'])

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
      <th>Id</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>CategoryId</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
    <tr>
      <th>Discount</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0.00</th>
      <td>2623997</td>
      <td>10381</td>
      <td>7161.95</td>
      <td>5116</td>
      <td>246</td>
      <td>153060.250</td>
      <td>12.092044</td>
      <td>0.120920</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>277313</td>
      <td>1038</td>
      <td>1507.75</td>
      <td>898</td>
      <td>26</td>
      <td>43567.380</td>
      <td>3.441904</td>
      <td>0.034419</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>264816</td>
      <td>1219</td>
      <td>812.20</td>
      <td>728</td>
      <td>25</td>
      <td>13027.635</td>
      <td>1.029207</td>
      <td>0.010292</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>384608</td>
      <td>1348</td>
      <td>609.65</td>
      <td>885</td>
      <td>36</td>
      <td>12889.995</td>
      <td>1.018334</td>
      <td>0.010183</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>418197</td>
      <td>1353</td>
      <td>1005.40</td>
      <td>1140</td>
      <td>39</td>
      <td>29569.320</td>
      <td>2.336031</td>
      <td>0.023360</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>343213</td>
      <td>1017</td>
      <td>714.70</td>
      <td>765</td>
      <td>32</td>
      <td>15753.600</td>
      <td>1.244564</td>
      <td>0.012446</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_bev_dsc = df_bev.drop(['Id', 'ProductId', 'OrderUnitPrice', 'OrderQty', 'RevPercentTotal', 'RevFractionTotal', 'CustomerId', 'OrderDate', 'CategoryName', 'CatDescription'], axis=1)
df_bev_dsc.head()
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
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>Revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>518.4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.00</td>
      <td>Chai</td>
      <td>1</td>
      <td>259.2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.00</td>
      <td>Chai</td>
      <td>1</td>
      <td>288.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>183.6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.00</td>
      <td>Chai</td>
      <td>1</td>
      <td>172.8</td>
    </tr>
  </tbody>
</table>
</div>



We can see that the largest revenue category by far, with over \\$150,000 is 0% discount, followed by approximately \\$40,000 for the 5% discount grouping (almost all of the discounts in the 5-9.9% discount group are 5% discounts).  

###  Detailed breakdown:  Beverages

Let's take a look at the details of discounts in Beverages, sorting by discount level and product name:


```python
df_bev = df.loc[df['CategoryName']=="Beverages"]

df_bev.groupby(['Discount', 'ProductName']).sum().sort_values(['Discount']).head(20)
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
      <th></th>
      <th>Id</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>CategoryId</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
    <tr>
      <th>Discount</th>
      <th>ProductName</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="12" valign="top">0.00</th>
      <th>Chai</th>
      <td>235242</td>
      <td>22</td>
      <td>374.40</td>
      <td>391</td>
      <td>22</td>
      <td>6681.600</td>
      <td>0.527859</td>
      <td>0.005279</td>
    </tr>
    <tr>
      <th>Steeleye Stout</th>
      <td>266316</td>
      <td>875</td>
      <td>424.80</td>
      <td>531</td>
      <td>25</td>
      <td>8812.800</td>
      <td>0.696228</td>
      <td>0.006962</td>
    </tr>
    <tr>
      <th>Sasquatch Ale</th>
      <td>128134</td>
      <td>408</td>
      <td>156.80</td>
      <td>269</td>
      <td>12</td>
      <td>3542.000</td>
      <td>0.279825</td>
      <td>0.002798</td>
    </tr>
    <tr>
      <th>Rhönbräu Klosterbier</th>
      <td>308816</td>
      <td>2175</td>
      <td>212.35</td>
      <td>631</td>
      <td>29</td>
      <td>4642.250</td>
      <td>0.366746</td>
      <td>0.003667</td>
    </tr>
    <tr>
      <th>Laughing Lumberjack Lager</th>
      <td>75320</td>
      <td>469</td>
      <td>98.00</td>
      <td>115</td>
      <td>7</td>
      <td>1610.000</td>
      <td>0.127193</td>
      <td>0.001272</td>
    </tr>
    <tr>
      <th>Lakkalikööri</th>
      <td>277308</td>
      <td>1976</td>
      <td>446.40</td>
      <td>591</td>
      <td>26</td>
      <td>10234.800</td>
      <td>0.808568</td>
      <td>0.008086</td>
    </tr>
    <tr>
      <th>Outback Lager</th>
      <td>245338</td>
      <td>1610</td>
      <td>324.00</td>
      <td>438</td>
      <td>23</td>
      <td>6129.000</td>
      <td>0.484202</td>
      <td>0.004842</td>
    </tr>
    <tr>
      <th>Guaraná Fantástica</th>
      <td>353001</td>
      <td>792</td>
      <td>139.50</td>
      <td>704</td>
      <td>33</td>
      <td>3011.400</td>
      <td>0.237906</td>
      <td>0.002379</td>
    </tr>
    <tr>
      <th>Côte de Blaye</th>
      <td>150244</td>
      <td>532</td>
      <td>3530.90</td>
      <td>320</td>
      <td>14</td>
      <td>79577.000</td>
      <td>6.286731</td>
      <td>0.062867</td>
    </tr>
    <tr>
      <th>Chartreuse verte</th>
      <td>169736</td>
      <td>624</td>
      <td>266.40</td>
      <td>343</td>
      <td>16</td>
      <td>5677.200</td>
      <td>0.448509</td>
      <td>0.004485</td>
    </tr>
    <tr>
      <th>Chang</th>
      <td>202413</td>
      <td>38</td>
      <td>342.00</td>
      <td>408</td>
      <td>19</td>
      <td>7125.000</td>
      <td>0.562888</td>
      <td>0.005629</td>
    </tr>
    <tr>
      <th>Ipoh Coffee</th>
      <td>212129</td>
      <td>860</td>
      <td>846.40</td>
      <td>375</td>
      <td>20</td>
      <td>16017.200</td>
      <td>1.265389</td>
      <td>0.012654</td>
    </tr>
    <tr>
      <th rowspan="8" valign="top">0.05</th>
      <th>Outback Lager</th>
      <td>31973</td>
      <td>210</td>
      <td>45.00</td>
      <td>63</td>
      <td>3</td>
      <td>897.750</td>
      <td>0.070924</td>
      <td>0.000709</td>
    </tr>
    <tr>
      <th>Steeleye Stout</th>
      <td>11046</td>
      <td>35</td>
      <td>18.00</td>
      <td>18</td>
      <td>1</td>
      <td>307.800</td>
      <td>0.024317</td>
      <td>0.000243</td>
    </tr>
    <tr>
      <th>Sasquatch Ale</th>
      <td>31849</td>
      <td>102</td>
      <td>36.40</td>
      <td>135</td>
      <td>3</td>
      <td>1675.800</td>
      <td>0.132391</td>
      <td>0.001324</td>
    </tr>
    <tr>
      <th>Rhönbräu Klosterbier</th>
      <td>32633</td>
      <td>225</td>
      <td>23.25</td>
      <td>210</td>
      <td>3</td>
      <td>1546.125</td>
      <td>0.122147</td>
      <td>0.001221</td>
    </tr>
    <tr>
      <th>Ipoh Coffee</th>
      <td>10340</td>
      <td>43</td>
      <td>36.80</td>
      <td>40</td>
      <td>1</td>
      <td>1398.400</td>
      <td>0.110476</td>
      <td>0.001105</td>
    </tr>
    <tr>
      <th>Lakkalikööri</th>
      <td>10273</td>
      <td>76</td>
      <td>14.40</td>
      <td>33</td>
      <td>1</td>
      <td>451.440</td>
      <td>0.035665</td>
      <td>0.000357</td>
    </tr>
    <tr>
      <th>Côte de Blaye</th>
      <td>52977</td>
      <td>190</td>
      <td>1212.10</td>
      <td>145</td>
      <td>5</td>
      <td>34294.525</td>
      <td>2.709331</td>
      <td>0.027093</td>
    </tr>
    <tr>
      <th>Chartreuse verte</th>
      <td>21942</td>
      <td>78</td>
      <td>36.00</td>
      <td>82</td>
      <td>2</td>
      <td>1402.200</td>
      <td>0.110776</td>
      <td>0.001108</td>
    </tr>
  </tbody>
</table>
</div>



Now let's sort by product name and revenue, to see which products bring in the most revenue.  

Note:  because CategoryId for all beverages is 1, the number in the CategoryId column represents the total number of orders of that product.  For example, there were 24 orders for Cote de Blaye, 28 orders for Ipoh Coffee, etc.

We see Cote de Blaye is at the top of the list for revenues by a large margin, followed by Ipoh Coffee, Chang, Lakkalikööri, and Steeleye Stout.  


```python
df_bev.groupby(['ProductName']).sum().sort_values(['Revenue'], ascending=False)
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
      <th>Id</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>CategoryId</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
    <tr>
      <th>ProductName</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Côte de Blaye</th>
      <td>255583</td>
      <td>912</td>
      <td>5902.40</td>
      <td>623</td>
      <td>1.10</td>
      <td>24</td>
      <td>141396.735</td>
      <td>11.170605</td>
      <td>0.111706</td>
    </tr>
    <tr>
      <th>Ipoh Coffee</th>
      <td>298531</td>
      <td>1204</td>
      <td>1205.20</td>
      <td>580</td>
      <td>1.40</td>
      <td>28</td>
      <td>23526.700</td>
      <td>1.858653</td>
      <td>0.018587</td>
    </tr>
    <tr>
      <th>Chang</th>
      <td>470965</td>
      <td>88</td>
      <td>786.60</td>
      <td>1057</td>
      <td>4.50</td>
      <td>44</td>
      <td>16355.960</td>
      <td>1.292151</td>
      <td>0.012922</td>
    </tr>
    <tr>
      <th>Lakkalikööri</th>
      <td>415841</td>
      <td>2964</td>
      <td>662.40</td>
      <td>981</td>
      <td>2.05</td>
      <td>39</td>
      <td>15760.440</td>
      <td>1.245104</td>
      <td>0.012451</td>
    </tr>
    <tr>
      <th>Steeleye Stout</th>
      <td>383553</td>
      <td>1260</td>
      <td>612.00</td>
      <td>883</td>
      <td>1.70</td>
      <td>36</td>
      <td>13644.000</td>
      <td>1.077901</td>
      <td>0.010779</td>
    </tr>
    <tr>
      <th>Chai</th>
      <td>406841</td>
      <td>38</td>
      <td>651.60</td>
      <td>828</td>
      <td>2.95</td>
      <td>38</td>
      <td>12788.100</td>
      <td>1.010284</td>
      <td>0.010103</td>
    </tr>
    <tr>
      <th>Chartreuse verte</th>
      <td>318790</td>
      <td>1170</td>
      <td>500.40</td>
      <td>793</td>
      <td>2.00</td>
      <td>30</td>
      <td>12294.540</td>
      <td>0.971291</td>
      <td>0.009713</td>
    </tr>
    <tr>
      <th>Outback Lager</th>
      <td>416162</td>
      <td>2730</td>
      <td>552.00</td>
      <td>817</td>
      <td>2.45</td>
      <td>39</td>
      <td>10672.650</td>
      <td>0.843159</td>
      <td>0.008432</td>
    </tr>
    <tr>
      <th>Rhönbräu Klosterbier</th>
      <td>490967</td>
      <td>3450</td>
      <td>339.45</td>
      <td>1155</td>
      <td>2.40</td>
      <td>46</td>
      <td>8177.490</td>
      <td>0.646037</td>
      <td>0.006460</td>
    </tr>
    <tr>
      <th>Sasquatch Ale</th>
      <td>202659</td>
      <td>646</td>
      <td>246.40</td>
      <td>506</td>
      <td>0.95</td>
      <td>19</td>
      <td>6350.400</td>
      <td>0.501693</td>
      <td>0.005017</td>
    </tr>
    <tr>
      <th>Guaraná Fantástica</th>
      <td>544786</td>
      <td>1224</td>
      <td>216.00</td>
      <td>1125</td>
      <td>2.90</td>
      <td>51</td>
      <td>4504.365</td>
      <td>0.355853</td>
      <td>0.003559</td>
    </tr>
    <tr>
      <th>Laughing Lumberjack Lager</th>
      <td>107466</td>
      <td>670</td>
      <td>137.20</td>
      <td>184</td>
      <td>0.60</td>
      <td>10</td>
      <td>2396.800</td>
      <td>0.189352</td>
      <td>0.001894</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_bev.groupby(['ProductName', 'Discount']).sum().sort_values(['ProductName'], ascending=True)
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
      <th></th>
      <th>Id</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>CategoryId</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
    <tr>
      <th>ProductName</th>
      <th>Discount</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="6" valign="top">Chai</th>
      <th>0.00</th>
      <td>235242</td>
      <td>22</td>
      <td>374.40</td>
      <td>391</td>
      <td>22</td>
      <td>6681.600</td>
      <td>0.527859</td>
      <td>0.005279</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>10905</td>
      <td>1</td>
      <td>18.00</td>
      <td>20</td>
      <td>1</td>
      <td>342.000</td>
      <td>0.027019</td>
      <td>0.000270</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>11025</td>
      <td>1</td>
      <td>18.00</td>
      <td>10</td>
      <td>1</td>
      <td>162.000</td>
      <td>0.012798</td>
      <td>0.000128</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>53177</td>
      <td>5</td>
      <td>82.80</td>
      <td>98</td>
      <td>5</td>
      <td>1407.600</td>
      <td>0.111203</td>
      <td>0.001112</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>42354</td>
      <td>4</td>
      <td>68.40</td>
      <td>170</td>
      <td>4</td>
      <td>2318.400</td>
      <td>0.183158</td>
      <td>0.001832</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>54138</td>
      <td>5</td>
      <td>90.00</td>
      <td>139</td>
      <td>5</td>
      <td>1876.500</td>
      <td>0.148247</td>
      <td>0.001482</td>
    </tr>
    <tr>
      <th rowspan="6" valign="top">Chang</th>
      <th>0.00</th>
      <td>202413</td>
      <td>38</td>
      <td>342.00</td>
      <td>408</td>
      <td>19</td>
      <td>7125.000</td>
      <td>0.562888</td>
      <td>0.005629</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>32270</td>
      <td>6</td>
      <td>57.00</td>
      <td>50</td>
      <td>3</td>
      <td>902.500</td>
      <td>0.071299</td>
      <td>0.000713</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>10485</td>
      <td>2</td>
      <td>15.20</td>
      <td>20</td>
      <td>1</td>
      <td>273.600</td>
      <td>0.021615</td>
      <td>0.000216</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>53993</td>
      <td>10</td>
      <td>87.40</td>
      <td>125</td>
      <td>5</td>
      <td>1744.200</td>
      <td>0.137795</td>
      <td>0.001378</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>106974</td>
      <td>20</td>
      <td>174.80</td>
      <td>247</td>
      <td>10</td>
      <td>3432.160</td>
      <td>0.271147</td>
      <td>0.002711</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>64830</td>
      <td>12</td>
      <td>110.20</td>
      <td>207</td>
      <td>6</td>
      <td>2878.500</td>
      <td>0.227407</td>
      <td>0.002274</td>
    </tr>
    <tr>
      <th rowspan="6" valign="top">Chartreuse verte</th>
      <th>0.25</th>
      <td>21120</td>
      <td>78</td>
      <td>32.40</td>
      <td>41</td>
      <td>2</td>
      <td>499.500</td>
      <td>0.039461</td>
      <td>0.000395</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>21501</td>
      <td>78</td>
      <td>36.00</td>
      <td>13</td>
      <td>2</td>
      <td>187.200</td>
      <td>0.014789</td>
      <td>0.000148</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>42407</td>
      <td>156</td>
      <td>64.80</td>
      <td>80</td>
      <td>4</td>
      <td>1009.800</td>
      <td>0.079776</td>
      <td>0.000798</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>21942</td>
      <td>78</td>
      <td>36.00</td>
      <td>82</td>
      <td>2</td>
      <td>1402.200</td>
      <td>0.110776</td>
      <td>0.001108</td>
    </tr>
    <tr>
      <th>0.00</th>
      <td>169736</td>
      <td>624</td>
      <td>266.40</td>
      <td>343</td>
      <td>16</td>
      <td>5677.200</td>
      <td>0.448509</td>
      <td>0.004485</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>42084</td>
      <td>156</td>
      <td>64.80</td>
      <td>234</td>
      <td>4</td>
      <td>3518.640</td>
      <td>0.277979</td>
      <td>0.002780</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">Côte de Blaye</th>
      <th>0.00</th>
      <td>150244</td>
      <td>532</td>
      <td>3530.90</td>
      <td>320</td>
      <td>14</td>
      <td>79577.000</td>
      <td>6.286731</td>
      <td>0.062867</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>52977</td>
      <td>190</td>
      <td>1212.10</td>
      <td>145</td>
      <td>5</td>
      <td>34294.525</td>
      <td>2.709331</td>
      <td>0.027093</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>21213</td>
      <td>76</td>
      <td>527.00</td>
      <td>19</td>
      <td>2</td>
      <td>4505.850</td>
      <td>0.355971</td>
      <td>0.003560</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>20777</td>
      <td>76</td>
      <td>421.60</td>
      <td>99</td>
      <td>2</td>
      <td>16695.360</td>
      <td>1.318964</td>
      <td>0.013190</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>10372</td>
      <td>38</td>
      <td>210.80</td>
      <td>40</td>
      <td>1</td>
      <td>6324.000</td>
      <td>0.499608</td>
      <td>0.004996</td>
    </tr>
    <tr>
      <th rowspan="6" valign="top">Guaraná Fantástica</th>
      <th>0.00</th>
      <td>353001</td>
      <td>792</td>
      <td>139.50</td>
      <td>704</td>
      <td>33</td>
      <td>3011.400</td>
      <td>0.237906</td>
      <td>0.002379</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>31105</td>
      <td>72</td>
      <td>10.80</td>
      <td>102</td>
      <td>3</td>
      <td>348.840</td>
      <td>0.027559</td>
      <td>0.000276</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>31552</td>
      <td>72</td>
      <td>12.60</td>
      <td>80</td>
      <td>3</td>
      <td>307.800</td>
      <td>0.024317</td>
      <td>0.000243</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>42213</td>
      <td>96</td>
      <td>17.10</td>
      <td>60</td>
      <td>4</td>
      <td>218.025</td>
      <td>0.017224</td>
      <td>0.000172</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>32203</td>
      <td>72</td>
      <td>13.50</td>
      <td>63</td>
      <td>3</td>
      <td>226.800</td>
      <td>0.017918</td>
      <td>0.000179</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>54712</td>
      <td>120</td>
      <td>22.50</td>
      <td>116</td>
      <td>5</td>
      <td>391.500</td>
      <td>0.030929</td>
      <td>0.000309</td>
    </tr>
    <tr>
      <th>Ipoh Coffee</th>
      <th>0.20</th>
      <td>21975</td>
      <td>86</td>
      <td>92.00</td>
      <td>40</td>
      <td>2</td>
      <td>1472.000</td>
      <td>0.116291</td>
      <td>0.001163</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th rowspan="3" valign="top">Lakkalikööri</th>
      <th>0.15</th>
      <td>63760</td>
      <td>456</td>
      <td>97.20</td>
      <td>162</td>
      <td>6</td>
      <td>2249.100</td>
      <td>0.177683</td>
      <td>0.001777</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>21584</td>
      <td>152</td>
      <td>36.00</td>
      <td>94</td>
      <td>2</td>
      <td>1353.600</td>
      <td>0.106937</td>
      <td>0.001069</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>21262</td>
      <td>152</td>
      <td>32.40</td>
      <td>41</td>
      <td>2</td>
      <td>499.500</td>
      <td>0.039461</td>
      <td>0.000395</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Laughing Lumberjack Lager</th>
      <th>0.00</th>
      <td>75320</td>
      <td>469</td>
      <td>98.00</td>
      <td>115</td>
      <td>7</td>
      <td>1610.000</td>
      <td>0.127193</td>
      <td>0.001272</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>10939</td>
      <td>67</td>
      <td>14.00</td>
      <td>40</td>
      <td>1</td>
      <td>476.000</td>
      <td>0.037605</td>
      <td>0.000376</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>10923</td>
      <td>67</td>
      <td>14.00</td>
      <td>24</td>
      <td>1</td>
      <td>268.800</td>
      <td>0.021236</td>
      <td>0.000212</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>10284</td>
      <td>67</td>
      <td>11.20</td>
      <td>5</td>
      <td>1</td>
      <td>42.000</td>
      <td>0.003318</td>
      <td>0.000033</td>
    </tr>
    <tr>
      <th rowspan="6" valign="top">Outback Lager</th>
      <th>0.20</th>
      <td>32933</td>
      <td>210</td>
      <td>45.00</td>
      <td>82</td>
      <td>3</td>
      <td>984.000</td>
      <td>0.077738</td>
      <td>0.000777</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>21557</td>
      <td>140</td>
      <td>30.00</td>
      <td>58</td>
      <td>2</td>
      <td>739.500</td>
      <td>0.058422</td>
      <td>0.000584</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>42497</td>
      <td>280</td>
      <td>57.00</td>
      <td>69</td>
      <td>4</td>
      <td>729.000</td>
      <td>0.057592</td>
      <td>0.000576</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>31973</td>
      <td>210</td>
      <td>45.00</td>
      <td>63</td>
      <td>3</td>
      <td>897.750</td>
      <td>0.070924</td>
      <td>0.000709</td>
    </tr>
    <tr>
      <th>0.00</th>
      <td>245338</td>
      <td>1610</td>
      <td>324.00</td>
      <td>438</td>
      <td>23</td>
      <td>6129.000</td>
      <td>0.484202</td>
      <td>0.004842</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>41864</td>
      <td>280</td>
      <td>51.00</td>
      <td>107</td>
      <td>4</td>
      <td>1193.400</td>
      <td>0.094281</td>
      <td>0.000943</td>
    </tr>
    <tr>
      <th rowspan="6" valign="top">Rhönbräu Klosterbier</th>
      <th>0.00</th>
      <td>308816</td>
      <td>2175</td>
      <td>212.35</td>
      <td>631</td>
      <td>29</td>
      <td>4642.250</td>
      <td>0.366746</td>
      <td>0.003667</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>32633</td>
      <td>225</td>
      <td>23.25</td>
      <td>210</td>
      <td>3</td>
      <td>1546.125</td>
      <td>0.122147</td>
      <td>0.001221</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>53081</td>
      <td>375</td>
      <td>37.20</td>
      <td>103</td>
      <td>5</td>
      <td>684.945</td>
      <td>0.054112</td>
      <td>0.000541</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>21306</td>
      <td>150</td>
      <td>13.95</td>
      <td>26</td>
      <td>2</td>
      <td>163.370</td>
      <td>0.012907</td>
      <td>0.000129</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>64671</td>
      <td>450</td>
      <td>46.50</td>
      <td>181</td>
      <td>6</td>
      <td>1122.200</td>
      <td>0.088656</td>
      <td>0.000887</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>10460</td>
      <td>75</td>
      <td>6.20</td>
      <td>4</td>
      <td>1</td>
      <td>18.600</td>
      <td>0.001469</td>
      <td>0.000015</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">Sasquatch Ale</th>
      <th>0.25</th>
      <td>10548</td>
      <td>34</td>
      <td>14.00</td>
      <td>10</td>
      <td>1</td>
      <td>105.000</td>
      <td>0.008295</td>
      <td>0.000083</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>21138</td>
      <td>68</td>
      <td>25.20</td>
      <td>32</td>
      <td>2</td>
      <td>313.600</td>
      <td>0.024775</td>
      <td>0.000248</td>
    </tr>
    <tr>
      <th>0.00</th>
      <td>128134</td>
      <td>408</td>
      <td>156.80</td>
      <td>269</td>
      <td>12</td>
      <td>3542.000</td>
      <td>0.279825</td>
      <td>0.002798</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>31849</td>
      <td>102</td>
      <td>36.40</td>
      <td>135</td>
      <td>3</td>
      <td>1675.800</td>
      <td>0.132391</td>
      <td>0.001324</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>10990</td>
      <td>34</td>
      <td>14.00</td>
      <td>60</td>
      <td>1</td>
      <td>714.000</td>
      <td>0.056407</td>
      <td>0.000564</td>
    </tr>
    <tr>
      <th rowspan="6" valign="top">Steeleye Stout</th>
      <th>0.20</th>
      <td>21164</td>
      <td>70</td>
      <td>32.40</td>
      <td>95</td>
      <td>2</td>
      <td>1195.200</td>
      <td>0.094423</td>
      <td>0.000944</td>
    </tr>
    <tr>
      <th>0.00</th>
      <td>266316</td>
      <td>875</td>
      <td>424.80</td>
      <td>531</td>
      <td>25</td>
      <td>8812.800</td>
      <td>0.696228</td>
      <td>0.006962</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>11046</td>
      <td>35</td>
      <td>18.00</td>
      <td>18</td>
      <td>1</td>
      <td>307.800</td>
      <td>0.024317</td>
      <td>0.000243</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>31858</td>
      <td>105</td>
      <td>50.40</td>
      <td>95</td>
      <td>3</td>
      <td>1409.400</td>
      <td>0.111345</td>
      <td>0.001113</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>31877</td>
      <td>105</td>
      <td>50.40</td>
      <td>105</td>
      <td>3</td>
      <td>1392.300</td>
      <td>0.109994</td>
      <td>0.001100</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>21292</td>
      <td>70</td>
      <td>36.00</td>
      <td>39</td>
      <td>2</td>
      <td>526.500</td>
      <td>0.041594</td>
      <td>0.000416</td>
    </tr>
  </tbody>
</table>
<p>67 rows × 8 columns</p>
</div>




```python
# df_bev.groupby(['ProductName', 'Discount']).sum().sort_values(['ProductName'], ascending=True)

df_bev_dsc.groupby(['Discount']).sum().sort_values(['Discount']).plot(kind='bar', 
            title ="Beverages--Revenues by Discount", legend=False, figsize=(8, 5), fontsize=12)

plt.show()

```


![png](student_files/student_354_0.png)


### Observations and comments on revenues by discount within the Beverages category

We can see that virtually every product in this category is sold at almost all discount levels (including 0%).  With more time, I would like to explore the distributions of revenues by discount level for each named product, and run tests (Tukey's test, among others) to figure out whether those distributions differ significantly across product name.  

For example, within the Beverage category, I noticed an interesting distribution of revenues by discount level for Cote de Blaye (which is an expensive wine product and is the largest single product by revenue in the Beverages category).  


```python
df.loc[df.ProductName == "Côte de Blaye"].head(4)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>966</th>
      <td>10329</td>
      <td>SPLIR</td>
      <td>2012-10-15</td>
      <td>38</td>
      <td>210.8</td>
      <td>20</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>4005.2</td>
      <td>0.316418</td>
      <td>0.003164</td>
    </tr>
    <tr>
      <th>967</th>
      <td>10351</td>
      <td>ERNSH</td>
      <td>2012-11-11</td>
      <td>38</td>
      <td>210.8</td>
      <td>20</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>4005.2</td>
      <td>0.316418</td>
      <td>0.003164</td>
    </tr>
    <tr>
      <th>968</th>
      <td>10353</td>
      <td>PICCO</td>
      <td>2012-11-13</td>
      <td>38</td>
      <td>210.8</td>
      <td>50</td>
      <td>0.20</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>8432.0</td>
      <td>0.666144</td>
      <td>0.006661</td>
    </tr>
    <tr>
      <th>969</th>
      <td>10360</td>
      <td>BLONP</td>
      <td>2012-11-22</td>
      <td>38</td>
      <td>210.8</td>
      <td>10</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>2108.0</td>
      <td>0.166536</td>
      <td>0.001665</td>
    </tr>
  </tbody>
</table>
</div>



_Grouping by discount level and sorting on revenues:_


```python
df_bev.groupby(['ProductName']).sum()
df_bev.loc[df['ProductName'] == 'Côte de Blaye']
df_bev_cote = df_bev.loc[df['ProductName'] == 'Côte de Blaye']
# df_bev_cote.groupby(['Discount', 'Revenue']).sum()

df_bev_cote.groupby(['Discount']).sum()


df_bev_cote_disc_rev = df_bev_cote.drop(columns=['Id', 'ProductId', 'OrderUnitPrice', 'RevPercentTotal', 'RevFractionTotal'])
df_bev_cote_disc_rev.rename(columns={'CategoryId': 'Number of Orders'})
print("Revenue totals sorted by discount, Cote de Blaye")
df_bev_cote_disc_rev.groupby(['Discount']).sum().sort_values(['Revenue'], ascending=False)

# Note that, since the CategoryId for Beverages is '1' and each row
# represents a summation of each item, the number showing in the 
# CategoryId column represents the number of orders placed at that 
# discount level.  OrderQty is the total number of units purchased
# across all orders at that discount level.
```

    Revenue totals sorted by discount, Cote de Blaye





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
      <th>OrderQty</th>
      <th>CategoryId</th>
      <th>Revenue</th>
    </tr>
    <tr>
      <th>Discount</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0.00</th>
      <td>320</td>
      <td>14</td>
      <td>79577.000</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>145</td>
      <td>5</td>
      <td>34294.525</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>99</td>
      <td>2</td>
      <td>16695.360</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>40</td>
      <td>1</td>
      <td>6324.000</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>19</td>
      <td>2</td>
      <td>4505.850</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_bev_dsc.groupby(['Discount']).sum().sort_values(['Revenue'], ascending=False).plot(kind='bar', 
            title ="Beverages--Discount, sorted by Revenue", figsize=(8, 5), fontsize=12)
plt.show()
```


![png](student_files/student_359_0.png)


### Cote de Blaye:  observations and comments on revenues by discount

Sorting revenues by discount level, we can see that the greatest order revenues occurred at 0% and 5% discount, with the next highest order revenues occurring at 20% and 25% and relatively low order revenues at the 10% or 15% level.  
-  Almost \\$80,000 of total sales of the product occurred without any discount.  
-  The next highest revenues for the product (almost \\$40,000) were made at the 5% discount level.  
-  The third highest revenues of over \\$16,000 were made with a 20\% discount applied.  
-  Over \\$6300 was made at the 25% discount level.  
-  Finally, just over \\$4000 in revenues was realized on product at the 10\% level.  (No product was sold at the 15\% discount level.)

I was curious to know if the date of the sale had any bearing on these sales figures.  I thought that perhaps seasonality might be at play.  Below is the dataset for this product, sorted by discount.


```python
df.loc[df.ProductName == "Côte de Blaye"].sort_values(['Discount'], ascending=False)

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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>970</th>
      <td>10372</td>
      <td>QUEE</td>
      <td>2012-12-04</td>
      <td>38</td>
      <td>210.8</td>
      <td>40</td>
      <td>0.25</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>6324.000</td>
      <td>0.499608</td>
      <td>0.004996</td>
    </tr>
    <tr>
      <th>968</th>
      <td>10353</td>
      <td>PICCO</td>
      <td>2012-11-13</td>
      <td>38</td>
      <td>210.8</td>
      <td>50</td>
      <td>0.20</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>8432.000</td>
      <td>0.666144</td>
      <td>0.006661</td>
    </tr>
    <tr>
      <th>972</th>
      <td>10424</td>
      <td>MEREP</td>
      <td>2013-01-23</td>
      <td>38</td>
      <td>210.8</td>
      <td>49</td>
      <td>0.20</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>8263.360</td>
      <td>0.652821</td>
      <td>0.006528</td>
    </tr>
    <tr>
      <th>978</th>
      <td>10672</td>
      <td>BERGS</td>
      <td>2013-09-17</td>
      <td>38</td>
      <td>263.5</td>
      <td>15</td>
      <td>0.10</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>3557.250</td>
      <td>0.281029</td>
      <td>0.002810</td>
    </tr>
    <tr>
      <th>976</th>
      <td>10541</td>
      <td>HANAR</td>
      <td>2013-05-19</td>
      <td>38</td>
      <td>263.5</td>
      <td>4</td>
      <td>0.10</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>948.600</td>
      <td>0.074941</td>
      <td>0.000749</td>
    </tr>
    <tr>
      <th>985</th>
      <td>10865</td>
      <td>QUICK</td>
      <td>2014-02-02</td>
      <td>38</td>
      <td>263.5</td>
      <td>60</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>15019.500</td>
      <td>1.186568</td>
      <td>0.011866</td>
    </tr>
    <tr>
      <th>981</th>
      <td>10816</td>
      <td>GREAL</td>
      <td>2014-01-06</td>
      <td>38</td>
      <td>263.5</td>
      <td>30</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>7509.750</td>
      <td>0.593284</td>
      <td>0.005933</td>
    </tr>
    <tr>
      <th>967</th>
      <td>10351</td>
      <td>ERNSH</td>
      <td>2012-11-11</td>
      <td>38</td>
      <td>210.8</td>
      <td>20</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>4005.200</td>
      <td>0.316418</td>
      <td>0.003164</td>
    </tr>
    <tr>
      <th>977</th>
      <td>10616</td>
      <td>GREAL</td>
      <td>2013-07-31</td>
      <td>38</td>
      <td>263.5</td>
      <td>15</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>3754.875</td>
      <td>0.296642</td>
      <td>0.002966</td>
    </tr>
    <tr>
      <th>966</th>
      <td>10329</td>
      <td>SPLIR</td>
      <td>2012-10-15</td>
      <td>38</td>
      <td>210.8</td>
      <td>20</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>4005.200</td>
      <td>0.316418</td>
      <td>0.003164</td>
    </tr>
    <tr>
      <th>975</th>
      <td>10540</td>
      <td>QUICK</td>
      <td>2013-05-19</td>
      <td>38</td>
      <td>263.5</td>
      <td>30</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>7905.000</td>
      <td>0.624510</td>
      <td>0.006245</td>
    </tr>
    <tr>
      <th>974</th>
      <td>10518</td>
      <td>TORTU</td>
      <td>2013-04-25</td>
      <td>38</td>
      <td>263.5</td>
      <td>15</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>3952.500</td>
      <td>0.312255</td>
      <td>0.003123</td>
    </tr>
    <tr>
      <th>973</th>
      <td>10479</td>
      <td>RATTC</td>
      <td>2013-03-19</td>
      <td>38</td>
      <td>210.8</td>
      <td>30</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>6324.000</td>
      <td>0.499608</td>
      <td>0.004996</td>
    </tr>
    <tr>
      <th>979</th>
      <td>10783</td>
      <td>HANAR</td>
      <td>2013-12-18</td>
      <td>38</td>
      <td>263.5</td>
      <td>5</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>1317.500</td>
      <td>0.104085</td>
      <td>0.001041</td>
    </tr>
    <tr>
      <th>980</th>
      <td>10805</td>
      <td>THEBI</td>
      <td>2013-12-30</td>
      <td>38</td>
      <td>263.5</td>
      <td>10</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>2635.000</td>
      <td>0.208170</td>
      <td>0.002082</td>
    </tr>
    <tr>
      <th>971</th>
      <td>10417</td>
      <td>SIMOB</td>
      <td>2013-01-16</td>
      <td>38</td>
      <td>210.8</td>
      <td>50</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>10540.000</td>
      <td>0.832680</td>
      <td>0.008327</td>
    </tr>
    <tr>
      <th>982</th>
      <td>10817</td>
      <td>KOENE</td>
      <td>2014-01-06</td>
      <td>38</td>
      <td>263.5</td>
      <td>30</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>7905.000</td>
      <td>0.624510</td>
      <td>0.006245</td>
    </tr>
    <tr>
      <th>983</th>
      <td>10828</td>
      <td>RANCH</td>
      <td>2014-01-13</td>
      <td>38</td>
      <td>263.5</td>
      <td>2</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>527.000</td>
      <td>0.041634</td>
      <td>0.000416</td>
    </tr>
    <tr>
      <th>984</th>
      <td>10831</td>
      <td>SANTG</td>
      <td>2014-01-14</td>
      <td>38</td>
      <td>263.5</td>
      <td>8</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>2108.000</td>
      <td>0.166536</td>
      <td>0.001665</td>
    </tr>
    <tr>
      <th>969</th>
      <td>10360</td>
      <td>BLONP</td>
      <td>2012-11-22</td>
      <td>38</td>
      <td>210.8</td>
      <td>10</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>2108.000</td>
      <td>0.166536</td>
      <td>0.001665</td>
    </tr>
    <tr>
      <th>986</th>
      <td>10889</td>
      <td>RATTC</td>
      <td>2014-02-16</td>
      <td>38</td>
      <td>263.5</td>
      <td>40</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>10540.000</td>
      <td>0.832680</td>
      <td>0.008327</td>
    </tr>
    <tr>
      <th>987</th>
      <td>10964</td>
      <td>SPECD</td>
      <td>2014-03-20</td>
      <td>38</td>
      <td>263.5</td>
      <td>5</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>1317.500</td>
      <td>0.104085</td>
      <td>0.001041</td>
    </tr>
    <tr>
      <th>988</th>
      <td>10981</td>
      <td>HANAR</td>
      <td>2014-03-27</td>
      <td>38</td>
      <td>263.5</td>
      <td>60</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>15810.000</td>
      <td>1.249019</td>
      <td>0.012490</td>
    </tr>
    <tr>
      <th>989</th>
      <td>11032</td>
      <td>WHITC</td>
      <td>2014-04-17</td>
      <td>38</td>
      <td>263.5</td>
      <td>25</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>6587.500</td>
      <td>0.520425</td>
      <td>0.005204</td>
    </tr>
  </tbody>
</table>
</div>



The three largest discounts were given on sales between November 1, 2012 and January 31, 2013.  I also noticed that a few other discounted sales were made at the end of the year, but more were made at the beginning of the year.  There were also a number of non-discounted sales that happened close to the of 2014, but there were just as many--if not more--non-discounted orders at other times of the year. 

The number of datapoints in this subset of product data is quite small, which is why a robust statistical test would be challenging to run.  With more time, I would research which tests would be most likely to yield meaningful results, given the small sample size for this product.  Given the high revenues per order, it would be helpful to understand more about the sales patterns of this product--seasonality and geography in particular.  Perhaps there is unmet customer demand that could be tapped by identifying revenue potential in different geographies or at different times of the year.




```python
df.loc[df.ProductName == "Côte de Blaye"].sort_values(['Revenue'], ascending=False)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>988</th>
      <td>10981</td>
      <td>HANAR</td>
      <td>2014-03-27</td>
      <td>38</td>
      <td>263.5</td>
      <td>60</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>15810.000</td>
      <td>1.249019</td>
      <td>0.012490</td>
    </tr>
    <tr>
      <th>985</th>
      <td>10865</td>
      <td>QUICK</td>
      <td>2014-02-02</td>
      <td>38</td>
      <td>263.5</td>
      <td>60</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>15019.500</td>
      <td>1.186568</td>
      <td>0.011866</td>
    </tr>
    <tr>
      <th>971</th>
      <td>10417</td>
      <td>SIMOB</td>
      <td>2013-01-16</td>
      <td>38</td>
      <td>210.8</td>
      <td>50</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>10540.000</td>
      <td>0.832680</td>
      <td>0.008327</td>
    </tr>
    <tr>
      <th>986</th>
      <td>10889</td>
      <td>RATTC</td>
      <td>2014-02-16</td>
      <td>38</td>
      <td>263.5</td>
      <td>40</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>10540.000</td>
      <td>0.832680</td>
      <td>0.008327</td>
    </tr>
    <tr>
      <th>968</th>
      <td>10353</td>
      <td>PICCO</td>
      <td>2012-11-13</td>
      <td>38</td>
      <td>210.8</td>
      <td>50</td>
      <td>0.20</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>8432.000</td>
      <td>0.666144</td>
      <td>0.006661</td>
    </tr>
    <tr>
      <th>972</th>
      <td>10424</td>
      <td>MEREP</td>
      <td>2013-01-23</td>
      <td>38</td>
      <td>210.8</td>
      <td>49</td>
      <td>0.20</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>8263.360</td>
      <td>0.652821</td>
      <td>0.006528</td>
    </tr>
    <tr>
      <th>975</th>
      <td>10540</td>
      <td>QUICK</td>
      <td>2013-05-19</td>
      <td>38</td>
      <td>263.5</td>
      <td>30</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>7905.000</td>
      <td>0.624510</td>
      <td>0.006245</td>
    </tr>
    <tr>
      <th>982</th>
      <td>10817</td>
      <td>KOENE</td>
      <td>2014-01-06</td>
      <td>38</td>
      <td>263.5</td>
      <td>30</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>7905.000</td>
      <td>0.624510</td>
      <td>0.006245</td>
    </tr>
    <tr>
      <th>981</th>
      <td>10816</td>
      <td>GREAL</td>
      <td>2014-01-06</td>
      <td>38</td>
      <td>263.5</td>
      <td>30</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>7509.750</td>
      <td>0.593284</td>
      <td>0.005933</td>
    </tr>
    <tr>
      <th>989</th>
      <td>11032</td>
      <td>WHITC</td>
      <td>2014-04-17</td>
      <td>38</td>
      <td>263.5</td>
      <td>25</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>6587.500</td>
      <td>0.520425</td>
      <td>0.005204</td>
    </tr>
    <tr>
      <th>970</th>
      <td>10372</td>
      <td>QUEE</td>
      <td>2012-12-04</td>
      <td>38</td>
      <td>210.8</td>
      <td>40</td>
      <td>0.25</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>6324.000</td>
      <td>0.499608</td>
      <td>0.004996</td>
    </tr>
    <tr>
      <th>973</th>
      <td>10479</td>
      <td>RATTC</td>
      <td>2013-03-19</td>
      <td>38</td>
      <td>210.8</td>
      <td>30</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>6324.000</td>
      <td>0.499608</td>
      <td>0.004996</td>
    </tr>
    <tr>
      <th>966</th>
      <td>10329</td>
      <td>SPLIR</td>
      <td>2012-10-15</td>
      <td>38</td>
      <td>210.8</td>
      <td>20</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>4005.200</td>
      <td>0.316418</td>
      <td>0.003164</td>
    </tr>
    <tr>
      <th>967</th>
      <td>10351</td>
      <td>ERNSH</td>
      <td>2012-11-11</td>
      <td>38</td>
      <td>210.8</td>
      <td>20</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>4005.200</td>
      <td>0.316418</td>
      <td>0.003164</td>
    </tr>
    <tr>
      <th>974</th>
      <td>10518</td>
      <td>TORTU</td>
      <td>2013-04-25</td>
      <td>38</td>
      <td>263.5</td>
      <td>15</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>3952.500</td>
      <td>0.312255</td>
      <td>0.003123</td>
    </tr>
    <tr>
      <th>977</th>
      <td>10616</td>
      <td>GREAL</td>
      <td>2013-07-31</td>
      <td>38</td>
      <td>263.5</td>
      <td>15</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>3754.875</td>
      <td>0.296642</td>
      <td>0.002966</td>
    </tr>
    <tr>
      <th>978</th>
      <td>10672</td>
      <td>BERGS</td>
      <td>2013-09-17</td>
      <td>38</td>
      <td>263.5</td>
      <td>15</td>
      <td>0.10</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>3557.250</td>
      <td>0.281029</td>
      <td>0.002810</td>
    </tr>
    <tr>
      <th>980</th>
      <td>10805</td>
      <td>THEBI</td>
      <td>2013-12-30</td>
      <td>38</td>
      <td>263.5</td>
      <td>10</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>2635.000</td>
      <td>0.208170</td>
      <td>0.002082</td>
    </tr>
    <tr>
      <th>984</th>
      <td>10831</td>
      <td>SANTG</td>
      <td>2014-01-14</td>
      <td>38</td>
      <td>263.5</td>
      <td>8</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>2108.000</td>
      <td>0.166536</td>
      <td>0.001665</td>
    </tr>
    <tr>
      <th>969</th>
      <td>10360</td>
      <td>BLONP</td>
      <td>2012-11-22</td>
      <td>38</td>
      <td>210.8</td>
      <td>10</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>2108.000</td>
      <td>0.166536</td>
      <td>0.001665</td>
    </tr>
    <tr>
      <th>979</th>
      <td>10783</td>
      <td>HANAR</td>
      <td>2013-12-18</td>
      <td>38</td>
      <td>263.5</td>
      <td>5</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>1317.500</td>
      <td>0.104085</td>
      <td>0.001041</td>
    </tr>
    <tr>
      <th>987</th>
      <td>10964</td>
      <td>SPECD</td>
      <td>2014-03-20</td>
      <td>38</td>
      <td>263.5</td>
      <td>5</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>1317.500</td>
      <td>0.104085</td>
      <td>0.001041</td>
    </tr>
    <tr>
      <th>976</th>
      <td>10541</td>
      <td>HANAR</td>
      <td>2013-05-19</td>
      <td>38</td>
      <td>263.5</td>
      <td>4</td>
      <td>0.10</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>948.600</td>
      <td>0.074941</td>
      <td>0.000749</td>
    </tr>
    <tr>
      <th>983</th>
      <td>10828</td>
      <td>RANCH</td>
      <td>2014-01-13</td>
      <td>38</td>
      <td>263.5</td>
      <td>2</td>
      <td>0.00</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>527.000</td>
      <td>0.041634</td>
      <td>0.000416</td>
    </tr>
  </tbody>
</table>
</div>



## Seventh possible question

Do revenues from discounts vary in a statistically significant way from revenues on non-discounted products in each product category?


### Null and alternative hypotheses 

-  **Null hypothesis:**  No significant difference between revenues from non-discounted products in a category versus revenues from discounted products in a category

    -  Ho:  $\mu_1 = \mu_2 = \mu_3 ... = \mu_i$)


-  **Alternative hypothesis:** Differences between revenues from non-discounted products in a category versus revenues from discounted products in a category are unlikely to be due to chance  

    -  Ha:  $\mu_1 \neq \mu_2 \neq \mu_3 \neq ... \neq \mu_i$



```python
# dataframes:

# df_bev
# df_cond
# df_confect
# df_dairy
# df_meat
# df_grains
# df_produce
# df_seafood
# df.loc[df["CategoryName"] == "Condiments"]
```

### Create dataframes (discounts/no discounts), EDA, Sampling, T-tests

### Beverages:  Discounts vs. no discounts

##### Create dataframes Revenues from no discounts and Revenues from all discounts


```python
df_bev_no_disc = df_bev.loc[df["Discount"] == 0]
df_bev_no_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>10294</td>
      <td>RATTC</td>
      <td>2012-08-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>18</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>259.2</td>
      <td>0.020477</td>
      <td>0.000205</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10317</td>
      <td>LONEP</td>
      <td>2012-09-30</td>
      <td>1</td>
      <td>14.4</td>
      <td>20</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>288.0</td>
      <td>0.022753</td>
      <td>0.000228</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10354</td>
      <td>PERIC</td>
      <td>2012-11-14</td>
      <td>1</td>
      <td>14.4</td>
      <td>12</td>
      <td>0.0</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>172.8</td>
      <td>0.013652</td>
      <td>0.000137</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_bev_disc = df_bev.loc[df["Discount"] > 0]
df_bev_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10285</td>
      <td>QUICK</td>
      <td>2012-08-20</td>
      <td>1</td>
      <td>14.4</td>
      <td>45</td>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>518.4</td>
      <td>0.040955</td>
      <td>0.000410</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10348</td>
      <td>WANDK</td>
      <td>2012-11-07</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>183.6</td>
      <td>0.014505</td>
      <td>0.000145</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10370</td>
      <td>CHOPS</td>
      <td>2012-12-03</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>183.6</td>
      <td>0.014505</td>
      <td>0.000145</td>
    </tr>
  </tbody>
</table>
</div>



##### EDA / Visualization


```python
hist_plot2(df_bev_no_disc.Revenue, df_bev_disc.Revenue, label="Beverage revenues per order, no discounts", label1="Beverage revenues per order, ALL discounts", xlim=([0, 2500]))

```


![png](student_files/student_375_0.png)


##### Sampling to create normal distribution of sample means


```python
bev_no_disc_mean, bev_no_disc_std = sampling_mean(df_bev_no_disc.Revenue)
print(f"The mean of the distribution is {round(np.mean(bev_no_disc_mean), 2)}")
print(f"The std deviation of the distribution is {round(np.std(bev_no_disc_std), 2)}")
hist_plot1(bev_no_disc_mean, label="Resampling means, order revs, NO discounts,  Beverages", xlim=([0, 1200]), print_stmt=False)
```

    The mean of the distribution is 618.11
    The std deviation of the distribution is 460.16



![png](student_files/student_377_1.png)



```python
bev_disc_mean, bev_disc_std = sampling_mean(df_bev_disc.Revenue)
print(f"The mean of the distribution is {round(np.mean(bev_disc_mean), 2)}")
print(f"The std deviation of the distribution is {round(np.std(bev_disc_std), 2)}")
hist_plot1(bev_disc_mean, label="Resampling means, order revs, ALL discounts,  Beverages", xlim=([0, 1200]), print_stmt=False)
```

    The mean of the distribution is 731.42
    The std deviation of the distribution is 339.08



![png](student_files/student_378_1.png)



```python
df_bev_disc.Discount.value_counts()
```




    0.20    39
    0.15    36
    0.25    32
    0.05    26
    0.10    25
    Name: Discount, dtype: int64




```python
# Divide df_bev_disc into two dataframes:  5-19% and 20+%

df_bev_disc_5_15 = df_bev_disc.loc[(df['Discount'] >= 0.05) & (df['Discount'] < 0.20)]
df_bev_disc_5_15.head(3)


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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>10348</td>
      <td>WANDK</td>
      <td>2012-11-07</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>183.6</td>
      <td>0.014505</td>
      <td>0.000145</td>
    </tr>
    <tr>
      <th>5</th>
      <td>10370</td>
      <td>CHOPS</td>
      <td>2012-12-03</td>
      <td>1</td>
      <td>14.4</td>
      <td>15</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>183.6</td>
      <td>0.014505</td>
      <td>0.000145</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10526</td>
      <td>WARTH</td>
      <td>2013-05-05</td>
      <td>1</td>
      <td>18.0</td>
      <td>8</td>
      <td>0.15</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>122.4</td>
      <td>0.009670</td>
      <td>0.000097</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_bev_disc_20plus = df_bev_disc.loc[(df['Discount'] >= 0.2)]
df_bev_disc_20plus.head(3)

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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10285</td>
      <td>QUICK</td>
      <td>2012-08-20</td>
      <td>1</td>
      <td>14.4</td>
      <td>45</td>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>518.4</td>
      <td>0.040955</td>
      <td>0.000410</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10522</td>
      <td>LEHMS</td>
      <td>2013-04-30</td>
      <td>1</td>
      <td>18.0</td>
      <td>40</td>
      <td>0.20</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>576.0</td>
      <td>0.045505</td>
      <td>0.000455</td>
    </tr>
    <tr>
      <th>16</th>
      <td>10646</td>
      <td>HUNGO</td>
      <td>2013-08-27</td>
      <td>1</td>
      <td>18.0</td>
      <td>15</td>
      <td>0.25</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>202.5</td>
      <td>0.015998</td>
      <td>0.000160</td>
    </tr>
  </tbody>
</table>
</div>



#### Test chosen:  t-test (2-sided, 2-sample) 

Have gotten relatively normal distributions by taking the means of multiple samples.  Need to check to make sure that variances are similar before performing the t-test.  We can use Bartlett's test in this case to check for similar variances.


```python
import scipy.stats

scipy.stats.bartlett(bev_disc_mean, bev_no_disc_mean)
```




    BartlettResult(statistic=15.45656010891127, pvalue=8.442339866062341e-05)



The p-value is < 0.05, so we **_can reject the null hypothesis_** that the samples come from populations with similar or equal variances.  


```python
scipy.stats.levene(bev_disc_mean, bev_no_disc_mean)
```




    LeveneResult(statistic=12.009820337335572, pvalue=0.0005671280164836042)



Like the Bartlett test, the Levene test evaluates two different distributions to determine whether or not they have equal variances.  The Levene test is less sensitive to normality than Bartlett's, so it is an alternative to the latter when the distributions deviate from normal.

Both tests yield p-values that are < 0.05 (although Levene's test is close to 0.05).  We can reject the null hypothesis that the distributions have the same variances.  This is useful information:  while the distributions are reasonably normal, they don't appear to be the same shape.  The distribution of discounted product revenue sales seems to have a stronger left skew than the non-discounted product sales.  You can also look at the mean and standard deviations to see that the means are substantially different.  

Although the distributions do not have the same variances, they are quite similar.  I think that it could be useful to run the 2-sided 2-sample t-test on these two distributions.   


```python
scs.ttest_ind(bev_disc_mean, bev_no_disc_mean)
```




    Ttest_indResult(statistic=11.800139489541447, pvalue=4.901027148387775e-29)



We see an extremely low p-value, which tells us that we should reject the null hypothesis that there is no statistically significant difference between the revenues from products with no discounts and revenues from discounted products.  

Next, let's take a look at Cohen's d:


```python
df_bev_disc_05to15 = df_bev_disc.loc[(df["Discount"] >= 0.01) & (df["Discount"] < 0.15)]
df_bev_disc_05to15

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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>26</th>
      <td>10905</td>
      <td>WELLI</td>
      <td>2014-02-24</td>
      <td>1</td>
      <td>18.00</td>
      <td>20</td>
      <td>0.05</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>342.000</td>
      <td>0.027019</td>
      <td>0.000270</td>
    </tr>
    <tr>
      <th>33</th>
      <td>11025</td>
      <td>WARTH</td>
      <td>2014-04-15</td>
      <td>1</td>
      <td>18.00</td>
      <td>10</td>
      <td>0.10</td>
      <td>Chai</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>162.000</td>
      <td>0.012798</td>
      <td>0.000128</td>
    </tr>
    <tr>
      <th>50</th>
      <td>10485</td>
      <td>LINOD</td>
      <td>2013-03-25</td>
      <td>2</td>
      <td>15.20</td>
      <td>20</td>
      <td>0.10</td>
      <td>Chang</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>273.600</td>
      <td>0.021615</td>
      <td>0.000216</td>
    </tr>
    <tr>
      <th>54</th>
      <td>10632</td>
      <td>WANDK</td>
      <td>2013-08-14</td>
      <td>2</td>
      <td>19.00</td>
      <td>30</td>
      <td>0.05</td>
      <td>Chang</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>541.500</td>
      <td>0.042780</td>
      <td>0.000428</td>
    </tr>
    <tr>
      <th>61</th>
      <td>10787</td>
      <td>LAMAI</td>
      <td>2013-12-19</td>
      <td>2</td>
      <td>19.00</td>
      <td>15</td>
      <td>0.05</td>
      <td>Chang</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>270.750</td>
      <td>0.021390</td>
      <td>0.000214</td>
    </tr>
    <tr>
      <th>66</th>
      <td>10851</td>
      <td>RICAR</td>
      <td>2014-01-26</td>
      <td>2</td>
      <td>19.00</td>
      <td>5</td>
      <td>0.05</td>
      <td>Chang</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>90.250</td>
      <td>0.007130</td>
      <td>0.000071</td>
    </tr>
    <tr>
      <th>571</th>
      <td>10275</td>
      <td>MAGAA</td>
      <td>2012-08-07</td>
      <td>24</td>
      <td>3.60</td>
      <td>12</td>
      <td>0.05</td>
      <td>Guaraná Fantástica</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>41.040</td>
      <td>0.003242</td>
      <td>0.000032</td>
    </tr>
    <tr>
      <th>577</th>
      <td>10358</td>
      <td>LAMAI</td>
      <td>2012-11-20</td>
      <td>24</td>
      <td>3.60</td>
      <td>10</td>
      <td>0.05</td>
      <td>Guaraná Fantástica</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>34.200</td>
      <td>0.002702</td>
      <td>0.000027</td>
    </tr>
    <tr>
      <th>580</th>
      <td>10446</td>
      <td>TOMSP</td>
      <td>2013-02-14</td>
      <td>24</td>
      <td>3.60</td>
      <td>20</td>
      <td>0.10</td>
      <td>Guaraná Fantástica</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>64.800</td>
      <td>0.005119</td>
      <td>0.000051</td>
    </tr>
    <tr>
      <th>583</th>
      <td>10472</td>
      <td>SEVES</td>
      <td>2013-03-12</td>
      <td>24</td>
      <td>3.60</td>
      <td>80</td>
      <td>0.05</td>
      <td>Guaraná Fantástica</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>273.600</td>
      <td>0.021615</td>
      <td>0.000216</td>
    </tr>
    <tr>
      <th>588</th>
      <td>10541</td>
      <td>HANAR</td>
      <td>2013-05-19</td>
      <td>24</td>
      <td>4.50</td>
      <td>35</td>
      <td>0.10</td>
      <td>Guaraná Fantástica</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>141.750</td>
      <td>0.011199</td>
      <td>0.000112</td>
    </tr>
    <tr>
      <th>590</th>
      <td>10565</td>
      <td>MEREP</td>
      <td>2013-06-11</td>
      <td>24</td>
      <td>4.50</td>
      <td>25</td>
      <td>0.10</td>
      <td>Guaraná Fantástica</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>101.250</td>
      <td>0.007999</td>
      <td>0.000080</td>
    </tr>
    <tr>
      <th>876</th>
      <td>10358</td>
      <td>LAMAI</td>
      <td>2012-11-20</td>
      <td>34</td>
      <td>11.20</td>
      <td>10</td>
      <td>0.05</td>
      <td>Sasquatch Ale</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>106.400</td>
      <td>0.008406</td>
      <td>0.000084</td>
    </tr>
    <tr>
      <th>880</th>
      <td>10483</td>
      <td>WHITC</td>
      <td>2013-03-24</td>
      <td>34</td>
      <td>11.20</td>
      <td>35</td>
      <td>0.05</td>
      <td>Sasquatch Ale</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>372.400</td>
      <td>0.029420</td>
      <td>0.000294</td>
    </tr>
    <tr>
      <th>890</th>
      <td>11008</td>
      <td>ERNSH</td>
      <td>2014-04-08</td>
      <td>34</td>
      <td>14.00</td>
      <td>90</td>
      <td>0.05</td>
      <td>Sasquatch Ale</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>1197.000</td>
      <td>0.094565</td>
      <td>0.000946</td>
    </tr>
    <tr>
      <th>898</th>
      <td>10390</td>
      <td>ERNSH</td>
      <td>2012-12-23</td>
      <td>35</td>
      <td>14.40</td>
      <td>40</td>
      <td>0.10</td>
      <td>Steeleye Stout</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>518.400</td>
      <td>0.040955</td>
      <td>0.000410</td>
    </tr>
    <tr>
      <th>911</th>
      <td>10623</td>
      <td>FRANK</td>
      <td>2013-08-07</td>
      <td>35</td>
      <td>18.00</td>
      <td>30</td>
      <td>0.10</td>
      <td>Steeleye Stout</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>486.000</td>
      <td>0.038395</td>
      <td>0.000384</td>
    </tr>
    <tr>
      <th>919</th>
      <td>10845</td>
      <td>QUICK</td>
      <td>2014-01-21</td>
      <td>35</td>
      <td>18.00</td>
      <td>25</td>
      <td>0.10</td>
      <td>Steeleye Stout</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>405.000</td>
      <td>0.031996</td>
      <td>0.000320</td>
    </tr>
    <tr>
      <th>928</th>
      <td>11046</td>
      <td>WANDK</td>
      <td>2014-04-23</td>
      <td>35</td>
      <td>18.00</td>
      <td>18</td>
      <td>0.05</td>
      <td>Steeleye Stout</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>307.800</td>
      <td>0.024317</td>
      <td>0.000243</td>
    </tr>
    <tr>
      <th>966</th>
      <td>10329</td>
      <td>SPLIR</td>
      <td>2012-10-15</td>
      <td>38</td>
      <td>210.80</td>
      <td>20</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>4005.200</td>
      <td>0.316418</td>
      <td>0.003164</td>
    </tr>
    <tr>
      <th>967</th>
      <td>10351</td>
      <td>ERNSH</td>
      <td>2012-11-11</td>
      <td>38</td>
      <td>210.80</td>
      <td>20</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>4005.200</td>
      <td>0.316418</td>
      <td>0.003164</td>
    </tr>
    <tr>
      <th>976</th>
      <td>10541</td>
      <td>HANAR</td>
      <td>2013-05-19</td>
      <td>38</td>
      <td>263.50</td>
      <td>4</td>
      <td>0.10</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>948.600</td>
      <td>0.074941</td>
      <td>0.000749</td>
    </tr>
    <tr>
      <th>977</th>
      <td>10616</td>
      <td>GREAL</td>
      <td>2013-07-31</td>
      <td>38</td>
      <td>263.50</td>
      <td>15</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>3754.875</td>
      <td>0.296642</td>
      <td>0.002966</td>
    </tr>
    <tr>
      <th>978</th>
      <td>10672</td>
      <td>BERGS</td>
      <td>2013-09-17</td>
      <td>38</td>
      <td>263.50</td>
      <td>15</td>
      <td>0.10</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>3557.250</td>
      <td>0.281029</td>
      <td>0.002810</td>
    </tr>
    <tr>
      <th>981</th>
      <td>10816</td>
      <td>GREAL</td>
      <td>2014-01-06</td>
      <td>38</td>
      <td>263.50</td>
      <td>30</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>7509.750</td>
      <td>0.593284</td>
      <td>0.005933</td>
    </tr>
    <tr>
      <th>985</th>
      <td>10865</td>
      <td>QUICK</td>
      <td>2014-02-02</td>
      <td>38</td>
      <td>263.50</td>
      <td>60</td>
      <td>0.05</td>
      <td>Côte de Blaye</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>15019.500</td>
      <td>1.186568</td>
      <td>0.011866</td>
    </tr>
    <tr>
      <th>993</th>
      <td>10305</td>
      <td>OLDWO</td>
      <td>2012-09-13</td>
      <td>39</td>
      <td>14.40</td>
      <td>30</td>
      <td>0.10</td>
      <td>Chartreuse verte</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>388.800</td>
      <td>0.030716</td>
      <td>0.000307</td>
    </tr>
    <tr>
      <th>996</th>
      <td>10361</td>
      <td>QUICK</td>
      <td>2012-11-22</td>
      <td>39</td>
      <td>14.40</td>
      <td>54</td>
      <td>0.10</td>
      <td>Chartreuse verte</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>699.840</td>
      <td>0.055289</td>
      <td>0.000553</td>
    </tr>
    <tr>
      <th>1006</th>
      <td>10654</td>
      <td>BERGS</td>
      <td>2013-09-02</td>
      <td>39</td>
      <td>18.00</td>
      <td>20</td>
      <td>0.10</td>
      <td>Chartreuse verte</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>324.000</td>
      <td>0.025597</td>
      <td>0.000256</td>
    </tr>
    <tr>
      <th>1009</th>
      <td>10764</td>
      <td>ERNSH</td>
      <td>2013-12-03</td>
      <td>39</td>
      <td>18.00</td>
      <td>130</td>
      <td>0.10</td>
      <td>Chartreuse verte</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>2106.000</td>
      <td>0.166378</td>
      <td>0.001664</td>
    </tr>
    <tr>
      <th>1014</th>
      <td>10865</td>
      <td>QUICK</td>
      <td>2014-02-02</td>
      <td>39</td>
      <td>18.00</td>
      <td>80</td>
      <td>0.05</td>
      <td>Chartreuse verte</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>1368.000</td>
      <td>0.108075</td>
      <td>0.001081</td>
    </tr>
    <tr>
      <th>1019</th>
      <td>11077</td>
      <td>RATTC</td>
      <td>2014-05-06</td>
      <td>39</td>
      <td>18.00</td>
      <td>2</td>
      <td>0.05</td>
      <td>Chartreuse verte</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>34.200</td>
      <td>0.002702</td>
      <td>0.000027</td>
    </tr>
    <tr>
      <th>1143</th>
      <td>10340</td>
      <td>BONAP</td>
      <td>2012-10-29</td>
      <td>43</td>
      <td>36.80</td>
      <td>40</td>
      <td>0.05</td>
      <td>Ipoh Coffee</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>1398.400</td>
      <td>0.110476</td>
      <td>0.001105</td>
    </tr>
    <tr>
      <th>1893</th>
      <td>10420</td>
      <td>WELLI</td>
      <td>2013-01-21</td>
      <td>70</td>
      <td>12.00</td>
      <td>8</td>
      <td>0.10</td>
      <td>Outback Lager</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>86.400</td>
      <td>0.006826</td>
      <td>0.000068</td>
    </tr>
    <tr>
      <th>1894</th>
      <td>10453</td>
      <td>AROUT</td>
      <td>2013-02-21</td>
      <td>70</td>
      <td>12.00</td>
      <td>25</td>
      <td>0.10</td>
      <td>Outback Lager</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>270.000</td>
      <td>0.021331</td>
      <td>0.000213</td>
    </tr>
    <tr>
      <th>1896</th>
      <td>10485</td>
      <td>LINOD</td>
      <td>2013-03-25</td>
      <td>70</td>
      <td>12.00</td>
      <td>60</td>
      <td>0.10</td>
      <td>Outback Lager</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>648.000</td>
      <td>0.051193</td>
      <td>0.000512</td>
    </tr>
    <tr>
      <th>1897</th>
      <td>10506</td>
      <td>KOENE</td>
      <td>2013-04-15</td>
      <td>70</td>
      <td>15.00</td>
      <td>14</td>
      <td>0.10</td>
      <td>Outback Lager</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>189.000</td>
      <td>0.014931</td>
      <td>0.000149</td>
    </tr>
    <tr>
      <th>1900</th>
      <td>10616</td>
      <td>GREAL</td>
      <td>2013-07-31</td>
      <td>70</td>
      <td>15.00</td>
      <td>15</td>
      <td>0.05</td>
      <td>Outback Lager</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>213.750</td>
      <td>0.016887</td>
      <td>0.000169</td>
    </tr>
    <tr>
      <th>1903</th>
      <td>10659</td>
      <td>QUEE</td>
      <td>2013-09-05</td>
      <td>70</td>
      <td>15.00</td>
      <td>40</td>
      <td>0.05</td>
      <td>Outback Lager</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>570.000</td>
      <td>0.045031</td>
      <td>0.000450</td>
    </tr>
    <tr>
      <th>1906</th>
      <td>10698</td>
      <td>ERNSH</td>
      <td>2013-10-09</td>
      <td>70</td>
      <td>15.00</td>
      <td>8</td>
      <td>0.05</td>
      <td>Outback Lager</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>114.000</td>
      <td>0.009006</td>
      <td>0.000090</td>
    </tr>
    <tr>
      <th>2039</th>
      <td>10436</td>
      <td>BLONP</td>
      <td>2013-02-05</td>
      <td>75</td>
      <td>6.20</td>
      <td>24</td>
      <td>0.10</td>
      <td>Rhönbräu Klosterbier</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>133.920</td>
      <td>0.010580</td>
      <td>0.000106</td>
    </tr>
    <tr>
      <th>2043</th>
      <td>10510</td>
      <td>SAVEA</td>
      <td>2013-04-18</td>
      <td>75</td>
      <td>7.75</td>
      <td>36</td>
      <td>0.10</td>
      <td>Rhönbräu Klosterbier</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>251.100</td>
      <td>0.019837</td>
      <td>0.000198</td>
    </tr>
    <tr>
      <th>2047</th>
      <td>10572</td>
      <td>BERGS</td>
      <td>2013-06-18</td>
      <td>75</td>
      <td>7.75</td>
      <td>15</td>
      <td>0.10</td>
      <td>Rhönbräu Klosterbier</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>104.625</td>
      <td>0.008266</td>
      <td>0.000083</td>
    </tr>
    <tr>
      <th>2053</th>
      <td>10631</td>
      <td>LAMAI</td>
      <td>2013-08-14</td>
      <td>75</td>
      <td>7.75</td>
      <td>8</td>
      <td>0.10</td>
      <td>Rhönbräu Klosterbier</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>55.800</td>
      <td>0.004408</td>
      <td>0.000044</td>
    </tr>
    <tr>
      <th>2063</th>
      <td>10788</td>
      <td>QUICK</td>
      <td>2013-12-22</td>
      <td>75</td>
      <td>7.75</td>
      <td>40</td>
      <td>0.05</td>
      <td>Rhönbräu Klosterbier</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>294.500</td>
      <td>0.023266</td>
      <td>0.000233</td>
    </tr>
    <tr>
      <th>2065</th>
      <td>10894</td>
      <td>SAVEA</td>
      <td>2014-02-18</td>
      <td>75</td>
      <td>7.75</td>
      <td>120</td>
      <td>0.05</td>
      <td>Rhönbräu Klosterbier</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>883.500</td>
      <td>0.069798</td>
      <td>0.000698</td>
    </tr>
    <tr>
      <th>2069</th>
      <td>10932</td>
      <td>BONAP</td>
      <td>2014-03-06</td>
      <td>75</td>
      <td>7.75</td>
      <td>20</td>
      <td>0.10</td>
      <td>Rhönbräu Klosterbier</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>139.500</td>
      <td>0.011021</td>
      <td>0.000110</td>
    </tr>
    <tr>
      <th>2070</th>
      <td>10951</td>
      <td>RICSU</td>
      <td>2014-03-16</td>
      <td>75</td>
      <td>7.75</td>
      <td>50</td>
      <td>0.05</td>
      <td>Rhönbräu Klosterbier</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>368.125</td>
      <td>0.029083</td>
      <td>0.000291</td>
    </tr>
    <tr>
      <th>2079</th>
      <td>10273</td>
      <td>QUICK</td>
      <td>2012-08-05</td>
      <td>76</td>
      <td>14.40</td>
      <td>33</td>
      <td>0.05</td>
      <td>Lakkalikööri</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>451.440</td>
      <td>0.035665</td>
      <td>0.000357</td>
    </tr>
    <tr>
      <th>2095</th>
      <td>10604</td>
      <td>FURIB</td>
      <td>2013-07-18</td>
      <td>76</td>
      <td>18.00</td>
      <td>10</td>
      <td>0.10</td>
      <td>Lakkalikööri</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>162.000</td>
      <td>0.012798</td>
      <td>0.000128</td>
    </tr>
    <tr>
      <th>2115</th>
      <td>11050</td>
      <td>FOLKO</td>
      <td>2014-04-27</td>
      <td>76</td>
      <td>18.00</td>
      <td>50</td>
      <td>0.10</td>
      <td>Lakkalikööri</td>
      <td>1</td>
      <td>Beverages</td>
      <td>Soft drinks, coffees, teas, beers, and ales</td>
      <td>810.000</td>
      <td>0.063992</td>
      <td>0.000640</td>
    </tr>
  </tbody>
</table>
</div>




```python
# rev_disc_01to05 = df.loc[(df["Discount"] >= .01) & (df["Discount"] < .05)]

```

### Condiments:  Discounts vs. no discounts

Create dataframes for Revenues from no discounts and Revenues due to all discounts


```python
df_cond_no_disc = df_cond.loc[df["Discount"] == 0]
df_cond_no_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>82</th>
      <td>10289</td>
      <td>BSBEV</td>
      <td>2012-08-26</td>
      <td>3</td>
      <td>8.0</td>
      <td>30</td>
      <td>0.0</td>
      <td>Aniseed Syrup</td>
      <td>2</td>
      <td>Condiments</td>
      <td>Sweet and savory sauces, relishes, spreads, an...</td>
      <td>240.0</td>
      <td>0.018960</td>
      <td>0.000190</td>
    </tr>
    <tr>
      <th>83</th>
      <td>10405</td>
      <td>LINOD</td>
      <td>2013-01-06</td>
      <td>3</td>
      <td>8.0</td>
      <td>50</td>
      <td>0.0</td>
      <td>Aniseed Syrup</td>
      <td>2</td>
      <td>Condiments</td>
      <td>Sweet and savory sauces, relishes, spreads, an...</td>
      <td>400.0</td>
      <td>0.031601</td>
      <td>0.000316</td>
    </tr>
    <tr>
      <th>85</th>
      <td>10540</td>
      <td>QUICK</td>
      <td>2013-05-19</td>
      <td>3</td>
      <td>10.0</td>
      <td>60</td>
      <td>0.0</td>
      <td>Aniseed Syrup</td>
      <td>2</td>
      <td>Condiments</td>
      <td>Sweet and savory sauces, relishes, spreads, an...</td>
      <td>600.0</td>
      <td>0.047401</td>
      <td>0.000474</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_cond_disc = df_cond.loc[df["Discount"] > 0]
df_cond_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>84</th>
      <td>10485</td>
      <td>LINOD</td>
      <td>2013-03-25</td>
      <td>3</td>
      <td>8.0</td>
      <td>20</td>
      <td>0.1</td>
      <td>Aniseed Syrup</td>
      <td>2</td>
      <td>Condiments</td>
      <td>Sweet and savory sauces, relishes, spreads, an...</td>
      <td>144.00</td>
      <td>0.011376</td>
      <td>0.000114</td>
    </tr>
    <tr>
      <th>89</th>
      <td>10764</td>
      <td>ERNSH</td>
      <td>2013-12-03</td>
      <td>3</td>
      <td>10.0</td>
      <td>20</td>
      <td>0.1</td>
      <td>Aniseed Syrup</td>
      <td>2</td>
      <td>Condiments</td>
      <td>Sweet and savory sauces, relishes, spreads, an...</td>
      <td>180.00</td>
      <td>0.014220</td>
      <td>0.000142</td>
    </tr>
    <tr>
      <th>96</th>
      <td>10336</td>
      <td>PRINI</td>
      <td>2012-10-23</td>
      <td>4</td>
      <td>17.6</td>
      <td>18</td>
      <td>0.1</td>
      <td>Chef Anton's Cajun Seasoning</td>
      <td>2</td>
      <td>Condiments</td>
      <td>Sweet and savory sauces, relishes, spreads, an...</td>
      <td>285.12</td>
      <td>0.022525</td>
      <td>0.000225</td>
    </tr>
  </tbody>
</table>
</div>



### Confections:  Discounts vs. no discounts

Create dataframes for Revenues from no discounts and Revenues due to all discounts


```python
df_confect_no_disc = df_confect.loc[df["Discount"] == 0]
df_confect_no_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>336</th>
      <td>10255</td>
      <td>RICSU</td>
      <td>2012-07-12</td>
      <td>16</td>
      <td>13.9</td>
      <td>35</td>
      <td>0.0</td>
      <td>Pavlova</td>
      <td>3</td>
      <td>Confections</td>
      <td>Desserts, candies, and sweet breads</td>
      <td>486.5</td>
      <td>0.038434</td>
      <td>0.000384</td>
    </tr>
    <tr>
      <th>339</th>
      <td>10296</td>
      <td>LILAS</td>
      <td>2012-09-03</td>
      <td>16</td>
      <td>13.9</td>
      <td>30</td>
      <td>0.0</td>
      <td>Pavlova</td>
      <td>3</td>
      <td>Confections</td>
      <td>Desserts, candies, and sweet breads</td>
      <td>417.0</td>
      <td>0.032944</td>
      <td>0.000329</td>
    </tr>
    <tr>
      <th>340</th>
      <td>10310</td>
      <td>THEBI</td>
      <td>2012-09-20</td>
      <td>16</td>
      <td>13.9</td>
      <td>10</td>
      <td>0.0</td>
      <td>Pavlova</td>
      <td>3</td>
      <td>Confections</td>
      <td>Desserts, candies, and sweet breads</td>
      <td>139.0</td>
      <td>0.010981</td>
      <td>0.000110</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_confect_disc = df_confect.loc[df["Discount"] > 0]
df_confect_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>337</th>
      <td>10263</td>
      <td>ERNSH</td>
      <td>2012-07-23</td>
      <td>16</td>
      <td>13.9</td>
      <td>60</td>
      <td>0.25</td>
      <td>Pavlova</td>
      <td>3</td>
      <td>Confections</td>
      <td>Desserts, candies, and sweet breads</td>
      <td>625.500</td>
      <td>0.049416</td>
      <td>0.000494</td>
    </tr>
    <tr>
      <th>338</th>
      <td>10287</td>
      <td>RICAR</td>
      <td>2012-08-22</td>
      <td>16</td>
      <td>13.9</td>
      <td>40</td>
      <td>0.15</td>
      <td>Pavlova</td>
      <td>3</td>
      <td>Confections</td>
      <td>Desserts, candies, and sweet breads</td>
      <td>472.600</td>
      <td>0.037336</td>
      <td>0.000373</td>
    </tr>
    <tr>
      <th>341</th>
      <td>10324</td>
      <td>SAVEA</td>
      <td>2012-10-08</td>
      <td>16</td>
      <td>13.9</td>
      <td>21</td>
      <td>0.15</td>
      <td>Pavlova</td>
      <td>3</td>
      <td>Confections</td>
      <td>Desserts, candies, and sweet breads</td>
      <td>248.115</td>
      <td>0.019602</td>
      <td>0.000196</td>
    </tr>
  </tbody>
</table>
</div>



### Dairy:  Discounts vs. no discounts

Create dataframes for Revenues from no discounts and Revenues due to all discounts


```python
df_dairy_no_disc = df_dairy.loc[df["Discount"] == 0]
df_dairy_no_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>216</th>
      <td>10248</td>
      <td>VINET</td>
      <td>2012-07-04</td>
      <td>11</td>
      <td>14.0</td>
      <td>12</td>
      <td>0.0</td>
      <td>Queso Cabrales</td>
      <td>4</td>
      <td>Dairy Products</td>
      <td>Cheeses</td>
      <td>168.0</td>
      <td>0.013272</td>
      <td>0.000133</td>
    </tr>
    <tr>
      <th>217</th>
      <td>10296</td>
      <td>LILAS</td>
      <td>2012-09-03</td>
      <td>11</td>
      <td>16.8</td>
      <td>12</td>
      <td>0.0</td>
      <td>Queso Cabrales</td>
      <td>4</td>
      <td>Dairy Products</td>
      <td>Cheeses</td>
      <td>201.6</td>
      <td>0.015927</td>
      <td>0.000159</td>
    </tr>
    <tr>
      <th>220</th>
      <td>10365</td>
      <td>ANTO</td>
      <td>2012-11-27</td>
      <td>11</td>
      <td>16.8</td>
      <td>24</td>
      <td>0.0</td>
      <td>Queso Cabrales</td>
      <td>4</td>
      <td>Dairy Products</td>
      <td>Cheeses</td>
      <td>403.2</td>
      <td>0.031854</td>
      <td>0.000319</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_dairy_disc = df_dairy.loc[df["Discount"] > 0]
df_dairy_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>218</th>
      <td>10327</td>
      <td>FOLKO</td>
      <td>2012-10-11</td>
      <td>11</td>
      <td>16.8</td>
      <td>50</td>
      <td>0.2</td>
      <td>Queso Cabrales</td>
      <td>4</td>
      <td>Dairy Products</td>
      <td>Cheeses</td>
      <td>672.00</td>
      <td>0.053089</td>
      <td>0.000531</td>
    </tr>
    <tr>
      <th>219</th>
      <td>10353</td>
      <td>PICCO</td>
      <td>2012-11-13</td>
      <td>11</td>
      <td>16.8</td>
      <td>12</td>
      <td>0.2</td>
      <td>Queso Cabrales</td>
      <td>4</td>
      <td>Dairy Products</td>
      <td>Cheeses</td>
      <td>161.28</td>
      <td>0.012741</td>
      <td>0.000127</td>
    </tr>
    <tr>
      <th>224</th>
      <td>10443</td>
      <td>REGGC</td>
      <td>2013-02-12</td>
      <td>11</td>
      <td>16.8</td>
      <td>6</td>
      <td>0.2</td>
      <td>Queso Cabrales</td>
      <td>4</td>
      <td>Dairy Products</td>
      <td>Cheeses</td>
      <td>80.64</td>
      <td>0.006371</td>
      <td>0.000064</td>
    </tr>
  </tbody>
</table>
</div>



### Meat/Poultry:  Discounts vs. no discounts

Create dataframes for Revenues from no discounts and Revenues due to all discounts


```python
df_meat_no_disc = df_meat.loc[df["Discount"] == 0]
df_meat_no_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>181</th>
      <td>10693</td>
      <td>WHITC</td>
      <td>2013-10-06</td>
      <td>9</td>
      <td>97.0</td>
      <td>6</td>
      <td>0.0</td>
      <td>Mishi Kobe Niku</td>
      <td>6</td>
      <td>Meat/Poultry</td>
      <td>Prepared meats</td>
      <td>582.0</td>
      <td>0.045979</td>
      <td>0.000460</td>
    </tr>
    <tr>
      <th>182</th>
      <td>10848</td>
      <td>CONSH</td>
      <td>2014-01-23</td>
      <td>9</td>
      <td>97.0</td>
      <td>3</td>
      <td>0.0</td>
      <td>Mishi Kobe Niku</td>
      <td>6</td>
      <td>Meat/Poultry</td>
      <td>Prepared meats</td>
      <td>291.0</td>
      <td>0.022990</td>
      <td>0.000230</td>
    </tr>
    <tr>
      <th>379</th>
      <td>10265</td>
      <td>BLONP</td>
      <td>2012-07-25</td>
      <td>17</td>
      <td>31.2</td>
      <td>30</td>
      <td>0.0</td>
      <td>Alice Mutton</td>
      <td>6</td>
      <td>Meat/Poultry</td>
      <td>Prepared meats</td>
      <td>936.0</td>
      <td>0.073946</td>
      <td>0.000739</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_meat_disc = df_meat.loc[df["Discount"] > 0]
df_meat_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>178</th>
      <td>10420</td>
      <td>WELLI</td>
      <td>2013-01-21</td>
      <td>9</td>
      <td>77.6</td>
      <td>20</td>
      <td>0.10</td>
      <td>Mishi Kobe Niku</td>
      <td>6</td>
      <td>Meat/Poultry</td>
      <td>Prepared meats</td>
      <td>1396.8</td>
      <td>0.110350</td>
      <td>0.001103</td>
    </tr>
    <tr>
      <th>179</th>
      <td>10515</td>
      <td>QUICK</td>
      <td>2013-04-23</td>
      <td>9</td>
      <td>97.0</td>
      <td>16</td>
      <td>0.15</td>
      <td>Mishi Kobe Niku</td>
      <td>6</td>
      <td>Meat/Poultry</td>
      <td>Prepared meats</td>
      <td>1319.2</td>
      <td>0.104219</td>
      <td>0.001042</td>
    </tr>
    <tr>
      <th>180</th>
      <td>10687</td>
      <td>HUNGO</td>
      <td>2013-09-30</td>
      <td>9</td>
      <td>97.0</td>
      <td>50</td>
      <td>0.25</td>
      <td>Mishi Kobe Niku</td>
      <td>6</td>
      <td>Meat/Poultry</td>
      <td>Prepared meats</td>
      <td>3637.5</td>
      <td>0.287369</td>
      <td>0.002874</td>
    </tr>
  </tbody>
</table>
</div>



### Grains/Cereals:  Discounts vs. no discounts

Create dataframes for Revenues from no discounts and Revenues due to all discounts


```python
df_grains_no_disc = df_grains.loc[df["Discount"] == 0]
df_grains_no_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>536</th>
      <td>10435</td>
      <td>CONSH</td>
      <td>2013-02-04</td>
      <td>22</td>
      <td>16.8</td>
      <td>12</td>
      <td>0.0</td>
      <td>Gustaf's Knäckebröd</td>
      <td>5</td>
      <td>Grains/Cereals</td>
      <td>Breads, crackers, pasta, and cereal</td>
      <td>201.6</td>
      <td>0.015927</td>
      <td>0.000159</td>
    </tr>
    <tr>
      <th>537</th>
      <td>10553</td>
      <td>WARTH</td>
      <td>2013-05-30</td>
      <td>22</td>
      <td>21.0</td>
      <td>24</td>
      <td>0.0</td>
      <td>Gustaf's Knäckebröd</td>
      <td>5</td>
      <td>Grains/Cereals</td>
      <td>Breads, crackers, pasta, and cereal</td>
      <td>504.0</td>
      <td>0.039817</td>
      <td>0.000398</td>
    </tr>
    <tr>
      <th>538</th>
      <td>10603</td>
      <td>SAVEA</td>
      <td>2013-07-18</td>
      <td>22</td>
      <td>21.0</td>
      <td>48</td>
      <td>0.0</td>
      <td>Gustaf's Knäckebröd</td>
      <td>5</td>
      <td>Grains/Cereals</td>
      <td>Breads, crackers, pasta, and cereal</td>
      <td>1008.0</td>
      <td>0.079634</td>
      <td>0.000796</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_grains_disc = df_grains.loc[df["Discount"] > 0]
df_grains_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>535</th>
      <td>10251</td>
      <td>VICTE</td>
      <td>2012-07-08</td>
      <td>22</td>
      <td>16.8</td>
      <td>6</td>
      <td>0.05</td>
      <td>Gustaf's Knäckebröd</td>
      <td>5</td>
      <td>Grains/Cereals</td>
      <td>Breads, crackers, pasta, and cereal</td>
      <td>95.76</td>
      <td>0.007565</td>
      <td>0.000076</td>
    </tr>
    <tr>
      <th>542</th>
      <td>10651</td>
      <td>WANDK</td>
      <td>2013-09-01</td>
      <td>22</td>
      <td>21.0</td>
      <td>20</td>
      <td>0.25</td>
      <td>Gustaf's Knäckebröd</td>
      <td>5</td>
      <td>Grains/Cereals</td>
      <td>Breads, crackers, pasta, and cereal</td>
      <td>315.00</td>
      <td>0.024886</td>
      <td>0.000249</td>
    </tr>
    <tr>
      <th>556</th>
      <td>10543</td>
      <td>LILAS</td>
      <td>2013-05-21</td>
      <td>23</td>
      <td>9.0</td>
      <td>70</td>
      <td>0.15</td>
      <td>Tunnbröd</td>
      <td>5</td>
      <td>Grains/Cereals</td>
      <td>Breads, crackers, pasta, and cereal</td>
      <td>535.50</td>
      <td>0.042305</td>
      <td>0.000423</td>
    </tr>
  </tbody>
</table>
</div>



### Produce:  Discounts vs. no discounts

Create dataframes for Revenues from no discounts and Revenues due to all discounts


```python
df_produce_no_disc = df_produce.loc[df["Discount"] == 0]
df_produce_no_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>136</th>
      <td>10262</td>
      <td>RATTC</td>
      <td>2012-07-22</td>
      <td>7</td>
      <td>24.0</td>
      <td>15</td>
      <td>0.0</td>
      <td>Uncle Bob's Organic Dried Pears</td>
      <td>7</td>
      <td>Produce</td>
      <td>Dried fruit and bean curd</td>
      <td>360.0</td>
      <td>0.028441</td>
      <td>0.000284</td>
    </tr>
    <tr>
      <th>139</th>
      <td>10471</td>
      <td>BSBEV</td>
      <td>2013-03-11</td>
      <td>7</td>
      <td>24.0</td>
      <td>30</td>
      <td>0.0</td>
      <td>Uncle Bob's Organic Dried Pears</td>
      <td>7</td>
      <td>Produce</td>
      <td>Dried fruit and bean curd</td>
      <td>720.0</td>
      <td>0.056881</td>
      <td>0.000569</td>
    </tr>
    <tr>
      <th>141</th>
      <td>10546</td>
      <td>VICTE</td>
      <td>2013-05-23</td>
      <td>7</td>
      <td>30.0</td>
      <td>10</td>
      <td>0.0</td>
      <td>Uncle Bob's Organic Dried Pears</td>
      <td>7</td>
      <td>Produce</td>
      <td>Dried fruit and bean curd</td>
      <td>300.0</td>
      <td>0.023701</td>
      <td>0.000237</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_produce_disc = df_produce.loc[df["Discount"] > 0]
df_produce_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>137</th>
      <td>10385</td>
      <td>SPLIR</td>
      <td>2012-12-17</td>
      <td>7</td>
      <td>24.0</td>
      <td>10</td>
      <td>0.20</td>
      <td>Uncle Bob's Organic Dried Pears</td>
      <td>7</td>
      <td>Produce</td>
      <td>Dried fruit and bean curd</td>
      <td>192.0</td>
      <td>0.015168</td>
      <td>0.000152</td>
    </tr>
    <tr>
      <th>138</th>
      <td>10459</td>
      <td>VICTE</td>
      <td>2013-02-27</td>
      <td>7</td>
      <td>24.0</td>
      <td>16</td>
      <td>0.05</td>
      <td>Uncle Bob's Organic Dried Pears</td>
      <td>7</td>
      <td>Produce</td>
      <td>Dried fruit and bean curd</td>
      <td>364.8</td>
      <td>0.028820</td>
      <td>0.000288</td>
    </tr>
    <tr>
      <th>140</th>
      <td>10511</td>
      <td>BONAP</td>
      <td>2013-04-18</td>
      <td>7</td>
      <td>30.0</td>
      <td>50</td>
      <td>0.15</td>
      <td>Uncle Bob's Organic Dried Pears</td>
      <td>7</td>
      <td>Produce</td>
      <td>Dried fruit and bean curd</td>
      <td>1275.0</td>
      <td>0.100727</td>
      <td>0.001007</td>
    </tr>
  </tbody>
</table>
</div>



### Seafood:  Discounts vs. no discounts

Create dataframes for Revenues from no discounts and Revenues due to all discounts


```python
df_seafood_no_disc = df_seafood.loc[df["Discount"] == 0]
df_seafood_no_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>184</th>
      <td>10276</td>
      <td>TORTU</td>
      <td>2012-08-08</td>
      <td>10</td>
      <td>24.8</td>
      <td>15</td>
      <td>0.0</td>
      <td>Ikura</td>
      <td>8</td>
      <td>Seafood</td>
      <td>Seaweed and fish</td>
      <td>372.0</td>
      <td>0.029389</td>
      <td>0.000294</td>
    </tr>
    <tr>
      <th>186</th>
      <td>10389</td>
      <td>BOTTM</td>
      <td>2012-12-20</td>
      <td>10</td>
      <td>24.8</td>
      <td>16</td>
      <td>0.0</td>
      <td>Ikura</td>
      <td>8</td>
      <td>Seafood</td>
      <td>Seaweed and fish</td>
      <td>396.8</td>
      <td>0.031348</td>
      <td>0.000313</td>
    </tr>
    <tr>
      <th>187</th>
      <td>10449</td>
      <td>BLONP</td>
      <td>2013-02-18</td>
      <td>10</td>
      <td>24.8</td>
      <td>14</td>
      <td>0.0</td>
      <td>Ikura</td>
      <td>8</td>
      <td>Seafood</td>
      <td>Seaweed and fish</td>
      <td>347.2</td>
      <td>0.027429</td>
      <td>0.000274</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_seafood_disc = df_seafood.loc[df["Discount"] > 0]
df_seafood_disc.head(3)
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
      <th>Id</th>
      <th>CustomerId</th>
      <th>OrderDate</th>
      <th>ProductId</th>
      <th>OrderUnitPrice</th>
      <th>OrderQty</th>
      <th>Discount</th>
      <th>ProductName</th>
      <th>CategoryId</th>
      <th>CategoryName</th>
      <th>CatDescription</th>
      <th>Revenue</th>
      <th>RevPercentTotal</th>
      <th>RevFractionTotal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>183</th>
      <td>10273</td>
      <td>QUICK</td>
      <td>2012-08-05</td>
      <td>10</td>
      <td>24.8</td>
      <td>24</td>
      <td>0.05</td>
      <td>Ikura</td>
      <td>8</td>
      <td>Seafood</td>
      <td>Seaweed and fish</td>
      <td>565.44</td>
      <td>0.044671</td>
      <td>0.000447</td>
    </tr>
    <tr>
      <th>185</th>
      <td>10357</td>
      <td>LILAS</td>
      <td>2012-11-19</td>
      <td>10</td>
      <td>24.8</td>
      <td>30</td>
      <td>0.20</td>
      <td>Ikura</td>
      <td>8</td>
      <td>Seafood</td>
      <td>Seaweed and fish</td>
      <td>595.20</td>
      <td>0.047022</td>
      <td>0.000470</td>
    </tr>
    <tr>
      <th>188</th>
      <td>10450</td>
      <td>VICTE</td>
      <td>2013-02-19</td>
      <td>10</td>
      <td>24.8</td>
      <td>20</td>
      <td>0.20</td>
      <td>Ikura</td>
      <td>8</td>
      <td>Seafood</td>
      <td>Seaweed and fish</td>
      <td>396.80</td>
      <td>0.031348</td>
      <td>0.000313</td>
    </tr>
  </tbody>
</table>
</div>



While we could look at the simple question of whether the average revenue per order is larger for discounted products than for non-discounted products in each product category, I believe that it would be more informative to look at how the level of discount correlates to average revenues per order.  We already know from question 1 that discounts drive revenues overall vs. no discounts, and we know from question 2 that discounts in the 5-9.9% range maximize revenues summed up across all categories.  I'd like to find out if the result we see across all categories holds true within each category.  

Originally, I thought about dividing up each category into  discount levels within categories of 5%, 10%, 15%, 20%, and more than 20%.  However, in many cases, the sample sizes are small at each of these levels (often with an n < 30).  However, we can get larger samples by consolidating discounts levels into two categories: 1-14.9%, and 15%+.  

Depending on the findings (and time permitting) it would be really interesting to compare product categories to look for interesting trends or differences in terms of discounts applied by product.  

### Null and alternative hypotheses  

-  **Null hypothesis:**  No significant difference between revenues from non-discounted products in a category versus revenues from discounts of 1% - 14.9% and discounts of 15% and up in a category

    -  Ho:  $\mu_1 = \mu_2 = \mu_3 ... = \mu_i$)


-  **Alternative hypothesis:** Differences among revenues from non-discounted products in a category versus revenues from discounts of 1% - 14.9% and discounts of 15% and up in a category are unlikely to be due to chance  

    -  Ha:  $\mu_1 \neq \mu_2 \neq \mu_3 \neq ... \neq \mu_i$


