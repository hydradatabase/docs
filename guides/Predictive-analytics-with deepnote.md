---
description: Deepnote is a versatile notebook that revolutionizes analytics and data science workflow by seamlessly combining the power of SQL and Python with a wide range of data frameworks. 
---

# Predictive analytics from application Telementry

## Scope
The focus of this guide is to display the ease of integrating Hydra to [Deepnote](https://deepnote.com/docs) to perform analytics and predictions using python scripts.

In order to make predictions a simple salary dataset was chosen with two columns 'Years of experience' and 'Salary'.

## Setup

- Ensure you have the details of your hydra instance handy.
- Sign up for your free instance at [Deepnote](https://deepnote.com/sign-up)

### Building the table with Hydra

1. A sample data from [Kaggle](https://www.kaggle.com/datasets/rsadiq/salary) would be used for this guide.Download the csv file.

2. Connect to your Hydra database and import the CSV file to the database using the steps for importing a CSV file outlined in the [Documentation](https://docs.hydra.so/centralize-data/load/from-local-csv-file)

### Creating the Connection

Now we can connect the Postgres Database to Deepnote

1. Start by [Signing Up](https://deepnote.com/sign-up) and using the 14 days trial or Logging in if you already have an account.

2.  Create a workspace name in this Demo it is called Hydra-Team and choose data source as PostreSQL

Create a new project by clicking the plus sign by the side of projects on the left of the notebook
click on integration on the right of the notebook and select postgres and put in your hydra credentials in the image below
Once that is onnected you can be able to view the tables in your datawarehouse in deepnote and run SQL queries on the salary table that has been imported earlier as shown below

In order to access the table using python scripts,an engine has to be created with the datawarehouse credentials

```python
from sqlalchemy import create_engine
#user = username
#pass = password
#server = hostname
#database = databasename
engine = create_engine('postgresql://user:pass@server:5432/database')
pd.read_sql_table('table_name',con= engine, schema= database)
```
The data can now be imported and python scripts run to perform predictions
The linear regression model is imported
The dataset is split into dependent and independent variables(X,Y)
The model is fitted on the dataset and prediction on the salary is made
Matplotlib is imported to make a visualization of the actual salary and the predicted 

