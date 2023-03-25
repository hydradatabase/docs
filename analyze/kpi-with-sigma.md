

  

  

---

  

description: Sigma is a cloud-native analytics platform that uses a familiar spreadsheet interface to give business users instant access to explore and get insights from their cloud data warehouse. It requires no code or special training to explore billions or rows, augment with new data, or perform “what if” analysis on all data in real⁠-⁠time.

  

---

  

  

# Integrating Hydra with [Sigma](https://www.sigmacomputing.com/)

  

  

## Setup

  

  

- Ensure you have the details of your Hydra instance handy.

  

- Sign up for a free trial at [Sigma](https://www.sigmacomputing.com/free-trial).

  

  

### Building the Table Hydra

  

1. We will use sample sales data from [IBM](https://relational.fit.cvut.cz/dataset/GOSales). Head to the following [repository](https://github.com/IBM/watson-machine-learning-samples) and download the .csv located in `watson-machine-learning-samples/cpd4.6/data/go_sales/go_daily_sales.csv`.

  

2. Connect to your Hydra database and upload the .csv. If you're unfamiliar with the process of connecting via. psql and creating/populating a table, please follow this [guide](https://docs.hydra.so/centralize-data/load/from-local-csv-file).

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

  

  

Now we can configure Sigma to access our sample data in Hydra.

  

  

1. As mentioned earlier, create a [trial account](https://www.sigmacomputing.com/free-trial).

  

2. Open your Admin Portal by selecting **Administration** in the user menu at the top right of your screen.

  

3. Select the **Connections** page from the top left-hand panel.

  

4. Click the **Create Connection** button on the top right.

  

5. Choose a name for the connection (we went with **hydra-sales_data_2023**) and select PostgreSQL as the type of database you'd like to connect to.
    
    ![Step_1](https://user-images.githubusercontent.com/71795488/227726542-f797223a-ba13-46ab-8640-507101f2fa3f.png)

  

  

6. Retrieve your database credentials from your Hydra dashboard. You will need Hostname, User, Password, and Database. Keep all other options unchanged. Once all the information has been entered, select **Create** on the top right.
    
    ![Step_2](https://user-images.githubusercontent.com/71795488/227726544-8070700a-1525-4cb6-8ed3-24d0ab62155f.png)

  

  

7. Once created, you will be able to view the connection details.

    ![Step_3](https://user-images.githubusercontent.com/71795488/227726572-198b26f7-da24-428c-a0c4-7b6acf13c760.png)

  

  

### Creating Your First Hydra-backend Dashboard

  

1. Back on the main page, click **Create New** on the top left followed by selecting **Workbook**. You will see a banner entitled **DATA ELEMENTS** on the toolbar to the left.

  

2. Within the aforementioned toolbar, select **VIZ** followed by clicking on **TABLES AND DATASETS**. This will being you to the Hydra database and will show you the available schemas. Your data should exist within the **public** schema.

    ![Step_4](https://user-images.githubusercontent.com/71795488/227726590-1a295fdc-6d96-4a48-b50b-7fb2ae161b25.png)

  

  

3. Navigate to the aforementioned schema, select **sales_data**, and then confirm your selection by clicking the button on the bottom right.

  

  

### Congratulations! You've connected Hydra with Sigma! What next?

Now that your data is connected, you're able to utilize your Hydra tables and create insights using Sigma's visualization. Let's experiment with creating the below dashboard showcasing some important KPI's relating to YoY revenue and profit.

   ![Step_5](https://user-images.githubusercontent.com/71795488/227729058-83647447-0536-4016-a04c-cdb09b8fcc1e.png)

  
  
  

1. Within the dashboard, click the **+** symbol on the top left.

2. Navigate to **Layouts** and click on the layout showcasing two visuals side-by-side (third from the bottom).

    ![PIC1](https://user-images.githubusercontent.com/71795488/227729006-01f0bb7b-c368-4165-9655-6f05df711e73.png)

  

4. Select the left empty visual, and then select **sales_data** as the table/data source.

    ![PIC2](https://user-images.githubusercontent.com/71795488/227729013-7e32450d-26e9-4bc1-8384-617014f35b15.png)

  

6. Create two new columns, one for *Revenue* and one for *Profit Margin*, by clicking the **Add Column** button on the toolbar to the left.

    ![PIC3](https://user-images.githubusercontent.com/71795488/227729027-073a7099-9bfe-4e4e-94d0-eb082fccce9a.png)

  
  

      * Revenue: Change the column name to Revenue and enter the following formula onto the formula bar at the top: `[Unit Price] * [Quantity]`.

      * Profit Margin: Change the column name to Profit Margin and enter the following formula onto the formula bar at the top: `([Unit Price] - [Unit Sale Price]) * [Quantity]`.

7. Now that we have the two columns used for these KPI's, let's add them to the metrics:

      * Click on the left empty visual and, from the top left toolbar, change its type from **Bar** to **Single Value**.

		  ![PIC5](https://user-images.githubusercontent.com/71795488/227732966-279516a6-27c5-459e-8410-195268490d77.png)


      * Drag **Revenue** into both the **Value** and **Comparison** section in the toolbar.


		  ![PIC6](https://user-images.githubusercontent.com/71795488/227732960-0c065170-0564-4dca-8923-42d6a993f517.png)


      * Select the newly added column under **Value** (should be automatically renamed to **Sum of Revenue** and change the formula, located in the top-middle, to `SumIf([Revenue], Year([Date]) = 2017)`. Change the formula of the newly added column under **Comparison**, now entitled **Sum of SumIf of Revenue**, to `SumIf([Revenue], Year([Date]) = 2016)`.

          ![PIC7](https://user-images.githubusercontent.com/71795488/227732979-784967e9-7df9-4c67-805f-d0eed1c37b43.png)
          ![PIC8](https://user-images.githubusercontent.com/71795488/227732982-241f448e-bdb7-4163-8ee2-5a38e87b641b.png)

	

      * Rename **Sum of Revenue**, under **Value**, to **Annual Sales** and Format the type to *currency* by hovering over the column and clicking on the arrow to the right. You should now have something looking like this:

          ![PIC4](https://user-images.githubusercontent.com/71795488/227729038-a8ec4a33-1fd3-407c-952e-055a055faa69.png)

  
  

**Repeat the above steps for YoY Profit**, but use the **Profit Margin** column instead. When adding a new visual, choose **Annual Sales** as your data source when prompted, as it will include the new columns created in the visual prior. 

   ![PIC10](https://user-images.githubusercontent.com/71795488/227733082-da1f89e8-92ba-479c-b6b5-c14e74d684c5.png)



The Quarterly revenue graph is also made in similar fashion, with the addition of adding a date range and aggregation filter.
1. Add a new visual.
2. Change the visual type from **Bar** to **Line**.
3. Drag Revenue into the y-axis, and Date into the x-axis.
4. On the x-axis toolbar, click on the arrow and navigate to **Truncate date**. Select **Quarter**.

      ![PIC11](https://user-images.githubusercontent.com/71795488/227733056-5f8d994e-e27d-47b7-b6cd-f013677b5b55.png)
  
5. Hovering over the data element again, click the arrow and then **Filter**. Change the filter type from **Between** to **Before**, and select December 31, 2017 as the date.

      ![PIC12](https://user-images.githubusercontent.com/71795488/227733063-c5fcc5a3-e1f4-4d4d-ba8e-962fb294e0c7.png)


### You're now done! Congratulations on creating your first Hydra x Sigma KPI dashboard!

![Step_5](https://user-images.githubusercontent.com/71795488/227729058-83647447-0536-4016-a04c-cdb09b8fcc1e.png)
