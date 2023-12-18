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

   
