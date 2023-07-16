---
description: >-
  Deepnote is a versatile notebook that revolutionizes analytics and data
  science workflow by seamlessly combining the power of SQL and Python with a
  wide range of data frameworks.
---

# Introduction

The focus of this guide is to display the ease of integrating Hydra into [Deepnote](https://deepnote.com/docs) to perform analytics using Python scripts which can be used to anticipate customer behavior, improve operational efficiency and enhance your business decisions.

To cater to a broad audience, the data and KPIs were chosen to be as agnostic as possible. We will leverage a simple salary dataset on [Kaggle](https://www.kaggle.com/datasets/rsadiq/salary) with two columns:

  * 'Years of experience'
  * 'Salary'

Your datasets will likely differ in scope and complexity, but this will hopefully serve as a general tutorial on how to leverage your data with Hydra & Deepnote for predictive analytics.


## Setup

* Ensure you have your Hydra login details handy or follow the [Setup guide](https://docs.hydra.so/getting-started/setup-guide) to create your initial Hydra data warehouse.

* Sign up for your free instance at [Deepnote](https://deepnote.com/sign-up)

### Building the table with Hydra

1. Sample data from [Kaggle](https://www.kaggle.com/datasets/rsadiq/salary) would be used for this guide. Download the CSV file.
2. Connect to your Hydra database and import the CSV file to the database using the steps for importing a CSV file outlined in the [Documentation](https://docs.hydra.so/centralize-data/load/from-local-csv-file)

### Creating the Connection

Now we can connect Deepnote to our sample data in Hydra.

1. Start by [signing up](https://deepnote.com/sign-up) for Deepnote if you haven't done so already.
2. Within your Deepnote account, create a workspace name. In the case of this tutorial, the workspace name will be "Hydra-Team".
3. Next, choose 'PostgreSQL' as the data source as shown in the image below.

<figure><img src="../.gitbook/assets/.predictive-analytics/datasource.png" alt=""><figcaption></figcaption></figure>

4. Create a new project by clicking on '+' under 'Projects' within the left panel of the Deepnote console.

<figure><img src="../.gitbook/assets/.predictive-analytics/create project.png" alt=""><figcaption></figcaption></figure>

5. Within your new project, navigate to 'Integrations' on the right panel and choose 'PostgreSQL'.

<figure><img src="../.gitbook/assets/.predictive-analytics/Integrating postgres.png" alt=""><figcaption></figcaption></figure>

6. At the 'Connect to PostgresSQL' popup, add your Hydra database credentials including Hostname, User, Password, and Database.

<figure><img src="../.gitbook/assets/.predictive-analytics/connecting postgres.png" alt=""><figcaption></figcaption></figure>

7. After establishing the connection, you can view the imported salary table within Deepnote.

Using the SELECT statement, you are able to run SQL queries on the salary table which enables you to retrieve and analyze specific data from the table.

```sql
SELECT * FROM public.salary LIMIT 100
```

<figure><img src="../.gitbook/assets/.predictive-analytics/Hydra dashboard.png" alt=""><figcaption></figcaption></figure>

8. To access the table using Python scripts because predictions can not be done using SQL, an engine has to be created with the data warehouse credentials

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

Now that we have connected Deepnote to Hydra, we can use Python and SQL scripts to access the data. We will use the Linear Regression model to create predictions leveraging the salary data.

The goal of linear regression model is to find a mathematical relationship or equation that best fits the data and helps us predict salaries based on the other factors.

The necessary models and libraries are being imported.
The linear regression model and matplotlib for our visualization are imported as shown below&#x20;

```python
#import Classes and lib needed
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression
```

The dataset is divided into dependent and independent variables(X, Y), in order for the linear regression model to understand the relationship between the varibles and make predictions.We train the linear regression model using the dataset. During the training process, the model learns from the data by identifying the patterns and relationships between the independent variables (years of experience) and the dependent variable (the salary).

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

Visualizations are created to show the difference between the actual salary versus the predicted salary.

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

<figure><img src="../.gitbook/assets/.predictive-analytics/Output.png" alt="" width="464"><figcaption><p>Visualization</p></figcaption></figure>

Your first prediction is complete! You can determine the range of a person's salary based on years of experience.

## Conclusion
At this point, we have performed a basic predictive analytics project by integrating the Hydra database with Deepnote. This basic guide can be leveraged to experiment with your own business metrics, but don't hesitate to reach out for more advance support through the Hydra #experts channel in Discord.