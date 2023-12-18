# Azure-Data-Engineering-Comprehensive-Pipeline
The use case for this project is building an end to end solution by ingesting the tables from on-premise SQL Server database using Azure Data Factory and then store the data in Azure Data Lake. Then Azure databricks is used to transform the RAW data to the most cleanest form of data and  finally using Microsoft Power BI to integrate with Azure synapse analytics to build an interactive dashboard. 

## Steps:
1. Go to sql server and upload csv file "pizza_sales.csv" in one of the table - sql server
   
![image](https://github.com/davender-singh1/Azure-Data-Engineering-Comprehensive-Pipeline/assets/106000634/15a2498c-3f4f-4ef4-9a1c-5d89612b0806)

2. Create Azure Storage account and create a container in it.
   
3. Create Azure Data Factory then Open the Data Factory studio
   
   Here you can create a pipeline and move data from our on-prem sql server to the cloud and to do that complete the Integration runtime setup.
   
   Step 1: Download and install integration runtime
   
   Step 2: Use the keys to register your integration runtime
   ![image](https://github.com/davender-singh1/Azure-Data-Engineering-Comprehensive-Pipeline/assets/106000634/18e576e0-17a4-4115-86f2-831e35e339da)

4. Load data from SQL server to storage - data pipeline

   After successfully importing data into Data Factory using correct sql server credentials, you will be able to see the data by clicking on preview
   
   ![image](https://github.com/davender-singh1/Azure-Data-Engineering-Comprehensive-Pipeline/assets/106000634/a3d4b1e3-b826-4b5e-86af-444b02c5ca51)

   Complete the Source and Sink dataset and click on publish all to finish the pipeline.

   You can go to the Monitor Tab, to check if your Pipeline is running successfully and this will transfer the data from the on-prem SQL server to the blob storage account

   ![image](https://github.com/davender-singh1/Azure-Data-Engineering-Comprehensive-Pipeline/assets/106000634/1f2ccf7a-c2ba-49ea-8a15-d40eb1b4a048)

5. Connect Databricks to storage
   
   Step 1: Create a compute cluster in Databricks Community edition
   
   Step 2: Create a notebook in the databricks using python as a default language of the notebook. Then use this code with your container-name, storage-account-name, scope and key name in the required fields to mount the Databricks to the Azure Storage account:
   You can also use Access keys of the storage account instead of creating a scope and key. And in the mount point you have write the location of the blob storage or raw container where you want to mount this.
   
   ```python
   dbutils.fs.mount(
    source = "wasbs://<container-name>@<storage-account-name>.blob.core.windows.net",
    mount_point = "/mnt/iotdata",
    extra_configs = {"fs.azure.account.key.<storage-account-name>.blob.core.windows.net":dbutils.secrets.get(scope = "<scope-name", key = "<key-name")}
    )
   ```

   Run this command to check if you have successfully mounted the data:
   
   ```python
   dbutils.fs.ls("/mnt/iotdata")
   ```

   After successfully doing it, we will be able to fetch the data location where it's mounted using the above command.

   ![image](https://github.com/davender-singh1/Azure-Data-Engineering-Comprehensive-Pipeline/assets/106000634/195a6895-ad23-4657-ae1a-15d1a1f41c13)


6. Create a dataframe in it using PySpark to read the data, using this command:

```python
df = spark.read.format("csv").options(header='True',inferSchema="True").load('dbfs:/mnt/iotdata/dbo.pizza_sales.txt')
```

(In the load(), you have enter the location that you can get after mounting the data)
Use this command to display the data:

```python
display(df)
```
![image](https://github.com/davender-singh1/Azure-Data-Engineering-Comprehensive-Pipeline/assets/106000634/27dd5437-f1a5-4e23-89e5-40742744862f)

Now, we will create a TempView table to be able to run SQL queries on the dataframe, using this command:

```python
df.createOrReplaceTempView("pizza_sales_analysis")
```
After creating a TempView, you can use Spark SQL to query the data.

Now we will focus on aggregating the sales data from the pizza_sales_analysis table. The aim is to gain insights into pizza sales distribution by various dimensions such as time and pizza characteristics.

The provided SQL query serves to aggregate order data to summarize key performance metrics. These metrics are crucial for understanding sales trends and customer preferences on a granular level.
The query groups the data by month, day, hour of order, and pizza attributes to provide a multi-dimensional view of the sales data. This allows for a comprehensive analysis of sales patterns.

```sql
%sql
SELECT
  COUNT(DISTINCT order_id) AS order_id,
  SUM(quantity) AS quantity,
  DATE_FORMAT(order_date, 'MMM') AS month_name,
  DATE_FORMAT(order_date, 'EEEE') AS day_name,
  HOUR(order_time) AS order_time,
  SUM(unit_price) AS unit_price,
  SUM(total_price) AS total_sales,
  pizza_size,
  pizza_category,
  pizza_name
FROM pizza_sales_analysis
GROUP BY month_name, day_name, order_time, pizza_size, pizza_category, pizza_name
```

7. Integrating Databricks Aggregated Data with Power BI
   To visualize the aggregated pizza sales data and gain interactive insights, the data is transferred from Databricks to Power BI, a business analytics service by Microsoft.
   The purpose of this integration is to leverage Power BI's data visualization and business intelligence capabilities. This allows for the creation of interactive reports and dashboards based on the aggregated sales data from Databricks.
   Steps to Connect Databricks to Power BI
   
   Power BI Get Data: Start by opening Power BI Desktop and navigate to the 'Home' tab. Click on 'Get Data' to initiate the process of importing data.

   Data Source Selection: In the 'Get Data' window, select 'Azure' from the categories on the left and then choose 'Azure Databricks'. Click 'Connect' to proceed.

   Databricks Connection Details: Enter the details required to connect to your Databricks cluster:
   
    -The Server Hostname (URL of the Databricks cluster)
    -HTTP Path from the Databricks cluster settings
    -Personal Access Token for authentication
   
    Importing Data: Once connected, Power BI will display a navigator pane where you can select the data you wish to import. Choose the aggregated table that was created using the Spark SQL query in Databricks.
   
    Loading Data: After selecting the required table, click 'Load' to import the data into Power BI Desktop. This may take some time depending on the size of the data.
   
    Building Reports: With the data loaded in Power BI, you can now use the drag-and-drop functionality to create reports and dashboards that provide actionable insights.


This is the Power BI Sales Dashboard that I created:

![image](https://github.com/davender-singh1/Azure-Data-Engineering-Comprehensive-Pipeline/assets/106000634/5dead0a3-c939-49e6-afb6-fa8400cc26a3)


The dashboard provides a comprehensive overview of sales performance across multiple dimensions.

Dashboard Components:

KPIs: At the top, Key Performance Indicators (KPIs) are presented, including 'total pizza sold', 'total order', and 'total sales', to provide a quick snapshot of overall sales health.

Sales by Month and Day: Below the KPIs, bar charts display 'total order' and 'total sales' by 'month_name' and 'day_name', offering insights into sales distribution over time.

Sales by Pizza Size: A pie chart breaks down 'total_sales' by 'pizza_size', illustrating the popularity and revenue contribution of each size category.

Sales by Pizza Category: Another pie chart shows 'total_sales' by 'pizza_category', allowing for an analysis of sales by product line.

Sales by Pizza Name: Finally, a horizontal bar chart ranks 'total_sales' by 'pizza_name', highlighting individual product performance.

Data Visualization Process:

Data Import: The aggregated table from Databricks was imported into Power BI using the 'Get Data' functionality, connecting directly to the Databricks cluster.

Data Modeling: The data was then modeled to create relationships and calculated columns as needed for more in-depth analysis.

KPI Visualization: KPIs were created using the Card visualization in Power BI, providing at-a-glance metrics of critical data points.

Time-based Analysis: Bar charts were used to visualize sales trends over months and days. These visualizations were designed to allow for easy comparison across time periods.

Categorical Analysis: Pie charts were chosen to represent the sales distribution among different pizza sizes and categories to quickly understand the revenue composition.

Product Performance: A horizontal bar chart was employed to show the sales performance of individual pizza names, sorted by total sales to identify best-sellers.

Interactivity: Slicers were added for 'month_name', 'day_name', and 'order_time' to enable interactive filtering, allowing users to drill down into the data.

Styling: The dashboard was styled for readability and visual appeal, with consistent color coding to facilitate quick interpretation of the data.

   
