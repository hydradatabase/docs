
---
description: Sigma is a cloud-native analytics platform that uses a familiar spreadsheet interface to give business users instant access to explore and get insights from their cloud data warehouse. It requires no code or special training to explore billions or rows, augment with new data, or perform “what if” analysis on all data in real⁠-⁠time.
---

# Integrating Hydra with  [Sigma](https://www.sigmacomputing.com/)



## Setup

- Ensure you have the details of your Hydra instance handy.
- Sign up for a free trial at [Sigma](https://www.sigmacomputing.com/free-trial).

### Building the Table Hydra
1.  We will use sample sales data from IBM's Watson Machine Learning repository. Pull to the following [repo](https://github.com/IBM/watson-machine-learning-samples) and download the .csv located in `watson-machine-learning-samples/cpd4.6/data/go_sales/go_daily_sales.csv`.
2. We will now connect to your Hydra database and upload the .csv. If you're unfamiliar with the process of connecting via. psql and creating/populating a table, please follow this [guide](https://docs.hydra.so/centralize-data/load/from-local-csv-file). 
	* The following code is used to create the sales_data table:
	```
	CREATE TABLE sales_data (
	    retailer_code INT,
	    product_number INT,
	    order_method_code INT,
	    date TIMESTAMP,
	    quantity INT,
	    unit_price FLOAT,
	    unit_sale_price FLOAT
	    )
	```
3. You should now have a sales-oriented table populated with approximately 150,000 rows.


### Connecting Sigma with Hydra

Now we can configure Sigma to access our sample data in the Hydra.

1. As mentioned earlier, create a [trial account](https://www.sigmacomputing.com/free-trial).
2. Open your Admin Portal by selecting **Administration** in the user menu at the top right of your screen.
3. Select the **Connections** page from the top left hand panel.
4. Click the **Create Connection** button on the top right.
5. Choose a name for the connection (we went with **hydra-sales_data_2023**) and select PostgreSQL as the type of database you'd like to connect to.

	![Step_1](https://user-images.githubusercontent.com/71795488/227726542-f797223a-ba13-46ab-8640-507101f2fa3f.png)

7. Retrieve your database credentials from your Hydra dashboard. You will need Hostname, User, Password, and Database. Keep all other options unchanged. Once all the information has been entered, select **Create** on the top right.

	![Step_2](https://user-images.githubusercontent.com/71795488/227726544-8070700a-1525-4cb6-8ed3-24d0ab62155f.png)

9. Once created, you will be able to view the connection details.

	![Step_3](https://user-images.githubusercontent.com/71795488/227726572-198b26f7-da24-428c-a0c4-7b6acf13c760.png)


### Creating Your First Hydra-backend Dashboard
1. Back on the main page, click **Create New** on the top left. Select **Workbook**. On the bar to the left, you will see a banner entitled, **DATA ELEMENTS**. 
2. Select **VIZ** within it, then click on **TABLES AND DATASETS**. This will being you to the Hydra database and will show you the available schemas. Your data should exist within **public**.
![Step_4](https://user-images.githubusercontent.com/71795488/227726590-1a295fdc-6d96-4a48-b50b-7fb2ae161b25.png)


4. Navigate to the aforementioned schema, select sales_data, and then confirm your selection by clicking the button on the bottom right.


Now that your data is connected, you're able to utilize your Hydra tables and create insights using Sigma's visualizations!
	![Step_5](https://user-images.githubusercontent.com/71795488/227726614-0a08bbaa-9acd-433b-8350-db5349aec216.png)




#### Cleanup

Once you’re done exploring Metabase and Hydra you can cleanup the sample data be droping the `sample_data` table.

```shell
psql "$PGCONN" -c "DROP TABLE sample_data;"
```
