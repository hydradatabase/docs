---
description: >-
  Deepnote is a versatile notebook that revolutionizes analytics and data
  science workflow by seamlessly combining the power of SQL and Python with a
  wide range of data frameworks.
---

# Predictive analytics with Hydra using Deepnote

The focus of this guide is to display the ease of integrating Hydra into [Deepnote](https://deepnote.com/docs) to perform analytics and predictions using Python scripts.

A simple salary dataset was chosen with two columns 'Years of experience' and 'Salary' to make predictions.

## Setup

* Ensure you have the details of your Hydra instance handy by creating a data warehouse in Hydra as shown in the [Setup guide](https://docs.hydra.so/getting-started/setup-guide).
* Sign up for your free instance at [Deepnote](https://deepnote.com/sign-up)

### Building the table with Hydra

1. Sample data from [Kaggle](https://www.kaggle.com/datasets/rsadiq/salary) would be used for this guide. Download the CSV file.
2. Connect to your Hydra database and import the CSV file to the database using the steps for importing a CSV file outlined in the [Documentation](https://docs.hydra.so/centralize-data/load/from-local-csv-file)

### Creating the Connection

Now we can connect the Postgres Database to Deepnote

1. Start by [Signing Up](https://deepnote.com/sign-up) or Logging in if you already have an account.
2. Create a workspace name, in this Demo, it is called Hydra-Team, and choose the data source as PostgreSQL as shown in the image below.

<figure><img src="../../.gitbook/assets/.predictive-analytics/datasource.png" alt=""><figcaption><p>Selecting PostreSQL</p></figcaption></figure>

3. Create a new project by clicking on projects at the left panel of the screen

<figure><img src="../../.gitbook/assets/.predictive-analytics/create project.png" alt=""><figcaption><p>Create a new project</p></figcaption></figure>

4. Make an integration to your Hydra data warehouse by integrating PostgreSQL.Retrieve your database credentials from your Hydra dashboard. You will need Hostname, User, Password, and Database.

<figure><img src="../../.gitbook/assets/.predictive-analytics/Integrating postgres.png" alt=""><figcaption><p>Adding integrations</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/.predictive-analytics/connecting postgres.png" alt=""><figcaption><p>Inputing your Hydra credentials</p></figcaption></figure>

5. Once that is connected you can be able to view the tables in your data warehouse in Deepnote and run SQL queries on the salary table that has been imported earlier as shown below using the SELECT statement on the salary table

```sql
SELECT * FROM public.salary LIMIT 100
```

<figure><img src="../../.gitbook/assets/.predictive-analytics/Hydra dashboard.png" alt=""><figcaption><p>Deepnotes dashboard after connection</p></figcaption></figure>

6. To access the table using Python scripts, an engine has to be created with the data warehouse credentials

```python
import pandas as pd
from sqlalchemy import create_engine
#user = username
#pass = password
#server = hostname
#database = database-name
engine = create_engine('postgresql://user:pass@server:5432/database')
#table_name = Salary
#database = public
data = pd.read_sql_table('table_name',con= engine, schema= 'database')
```

**Congratulations!** You've successfully connected Hydra with Deepnote! What next?

## Making the predictions

Now that the Database is connected we can use Python and SQL scripts to access the data. We can now predict the Salary data using the Linear Regression model.&#x20;

```python
#import Classes and lib needed
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression
```

The dataset is split into dependent and independent variables(X, Y) and the model is fitted on the dataset, and prediction on the salary table  is made\


```python
#defining variables X and Y
X = data.iloc[:,0].values.reshape(-1,1)
print(X)
Y = data.iloc[:,1].values.reshape(-1,1)
print(Y)
```

```python
#creating a Linear Regression model for prediction
model = LinearRegression()

#Fitting the model on the Dataset
model.fit(X,Y)

Y_pred = model.predict(X)  # make predictions
```

Visualizations are made on the actual salary and the predicted to view the difference.

```python
Y_pred = Y_pred.reshape(-1)
difference = pd.DataFrame({'Real' : data. salary, 'Predicted': Y_pred, })
print(difference)
```

```python
graph = difference.head(10)
graph.plot(kind='bar')
plt.title('Actual vs Predicted')
plt.ylabel('salary')
```

<figure><img src="../../.gitbook/assets/.predictive-analytics/Output.png" alt="" width="464"><figcaption><p>Visuslisation</p></figcaption></figure>

You have now made your first prediction with Hydra using Deepnote!
