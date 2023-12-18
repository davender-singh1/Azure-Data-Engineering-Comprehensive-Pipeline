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





   
