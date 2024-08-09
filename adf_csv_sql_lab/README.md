### Part 1
### Detailed Guide on ETL from CSV to SQL Server Using Azure Data Factory

This guide will walk you through the process of setting up an ETL (Extract, Transform, Load) pipeline using Azure Data Factory (ADF) to load data from a CSV file into a SQL Server database.

#### **Prerequisites**
- **Azure Subscription**: An active Azure account.
- **SQL Server Database**: SQL Server or Azure SQL Database with a table created to receive the data.
- **Azure Blob Storage**: Blob storage to hold the CSV files.

#### **Step 1: Set Up Azure Resources**

1. **Create an Azure Storage Account**:
   - Navigate to the Azure portal.
   - Click on `Create a resource` and select `Storage account`.
   - Fill in the required details, select a resource group, and create the storage account.
   - Once created, go to the storage account, create a new container (e.g., "csv-files"), and upload your CSV file to this container.

2. **Create an Azure SQL Database**:
   - In the Azure portal, go to `Create a resource`, select `SQL Database`.
   - Configure the SQL server, database name, and other settings.
   - Once the database is created, use SQL Server Management Studio (SSMS) or the query editor in the Azure portal to create the necessary table that matches your CSV schema.

#### **Step 2: Create Linked Services in Azure Data Factory**

1. **Create Azure Data Factory**:
   - Navigate to `Create a resource` and select `Data Factory`.
   - Configure the resource by selecting the appropriate subscription, resource group, and region.
   - Once created, click on `Author & Monitor` to open the ADF Studio.

2. **Create Linked Service for Azure Blob Storage**:
   - In ADF Studio, go to `Manage` on the left-hand menu.
   - Under `Linked services`, click `New` and choose `Azure Blob Storage`.
   - Name the linked service, select the subscription, and choose the storage account you created earlier. Test the connection and create.

3. **Create Linked Service for Azure SQL Database**:
   - Similarly, create another linked service but choose `Azure SQL Database`.
   - Provide the necessary connection details such as server name, database name, authentication type, and credentials. Test the connection and create.

#### **Step 3: Create Datasets**

1. **Create Dataset for CSV File**:
   - In ADF Studio, under `Author`, click on `+` and choose `Dataset`.
   - Select `Azure Blob Storage`, then `DelimitedText`.
   - Link it to the Blob Storage linked service you created earlier.
   - Configure the dataset by selecting the container and the CSV file path.

2. **Create Dataset for SQL Table**:
   - Similarly, create another dataset but select `Azure SQL Database` as the type.
   - Link it to the SQL Database linked service.
   - Configure the dataset by selecting the database and the table you created earlier.

#### **Step 4: Create the Pipeline**

1. **Create a New Pipeline**:
   - In ADF Studio, under `Author`, click `+` and select `Pipeline`.
   - Name your pipeline (e.g., `CSV_to_SQL_ETL`).

2. **Add a Copy Data Activity**:
   - Drag the `Copy Data` activity from the activities pane onto the pipeline canvas.
   - In the properties pane, configure the source by selecting the CSV dataset.
   - Configure the sink by selecting the SQL dataset.
   - Map the columns from the CSV to the SQL table columns. If they match exactly, ADF will auto-map them.

3. **Set Pipeline Parameters and Triggers (Optional)**:
   - You can add parameters to make the pipeline more dynamic (e.g., file path, table name).
   - You can also set up a trigger to run the pipeline on a schedule or based on an event.

#### **Step 5: Run and Monitor the Pipeline**

- Click `Debug` to test the pipeline with real data.
- After debugging, publish the pipeline.
- Monitor the pipeline execution by going to the `Monitor` section in ADF Studio. Check for any errors and ensure data has been transferred to the SQL table.

#### **Additional Notes**
- **Error Handling**: You can add additional activities for error handling, like sending notifications or writing logs to a storage account.
- **Data Transformation**: If your data requires transformation, consider using a Data Flow activity instead of a simple Copy Data activity.

### **References**
- **[Azure Data Factory Documentation](https://docs.microsoft.com/en-us/azure/data-factory/introduction)**: Comprehensive guide to all features and best practices.
- **[GitHub Example](https://github.com/sarannng/ETL-pipeline-project)**: Example ETL project from CSV to SQL Server using ADF【11†source】.
- **[Step-by-Step Tutorial](https://github.com/ssquadri/Azure-Data-Factory-Pipeline)**: Detailed guide and code for setting up a similar pipeline【13†source】. 

By following these steps, you should be able to set up a reliable ETL pipeline using Azure Data Factory to move data from CSV files to an SQL Server database.

### Part 2
### Enhanced Guide: ETL from CSV to SQL Server Using Azure Data Factory with Transformed Sales Data Table

This guide expands on the previous ETL pipeline by including a step to create a transformed sales data table that joins the Fact Table (`Sales`) with the Dimension Tables (`Products`, `Customers`, `Stores`).

#### **Updated Data Warehouse Schema Overview**
- **Fact Table: Sales**
  - `SaleID` (Primary Key)
  - `ProductID` (Foreign Key)
  - `CustomerID` (Foreign Key)
  - `StoreID` (Foreign Key)
  - `DateID` (Foreign Key)
  - `Quantity`
  - `TotalAmount`
  
- **Dimension Table: Products**
  - `ProductID` (Primary Key)
  - `ProductName`
  - `Category`
  - `Brand`
  - `Price`

- **Dimension Table: Customers**
  - `CustomerID` (Primary Key)
  - `FirstName`
  - `LastName`
  - `Email`
  - `PhoneNumber`
  - `Address`
  - `City`
  - `State`
  - `Country`

- **Dimension Table: Stores**
  - `StoreID` (Primary Key)
  - `StoreName`
  - `Location`
  - `Manager`

- **Transformed Sales Table**
  - `SaleID` (Primary Key)
  - `ProductName`
  - `Category`
  - `Brand`
  - `Price`
  - `FirstName`
  - `LastName`
  - `Email`
  - `PhoneNumber`
  - `StoreName`
  - `Location`
  - `Quantity`
  - `TotalAmount`

### **Step 1: Set Up the SQL Server Tables**

1. **Create the SQL Server Tables**:
   - Use the following SQL scripts to create the original Fact and Dimension tables and add the Transformed Sales table:

   ```sql
   -- Fact Table: Sales
   CREATE TABLE Sales (
       SaleID int PRIMARY KEY,
       ProductID int,
       CustomerID int,
       StoreID int,
       DateID int,
       Quantity int,
       TotalAmount decimal(18,2),
       FOREIGN KEY (ProductID) REFERENCES Products(ProductID),
       FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID),
       FOREIGN KEY (StoreID) REFERENCES Stores(StoreID)
   );

   -- Dimension Table: Products
   CREATE TABLE Products (
       ProductID int PRIMARY KEY,
       ProductName nvarchar(100),
       Category nvarchar(50),
       Brand nvarchar(50),
       Price decimal(18,2)
   );

   -- Dimension Table: Customers
   CREATE TABLE Customers (
       CustomerID int PRIMARY KEY,
       FirstName nvarchar(50),
       LastName nvarchar(50),
       Email nvarchar(100),
       PhoneNumber nvarchar(20),
       Address nvarchar(200),
       City nvarchar(50),
       State nvarchar(50),
       Country nvarchar(50)
   );

   -- Dimension Table: Stores
   CREATE TABLE Stores (
       StoreID int PRIMARY KEY,
       StoreName nvarchar(100),
       Location nvarchar(100),
       Manager nvarchar(100)
   );

   -- Transformed Sales Table
   CREATE TABLE TransformedSales (
       SaleID int PRIMARY KEY,
       ProductName nvarchar(100),
       Category nvarchar(50),
       Brand nvarchar(50),
       Price decimal(18,2),
       FirstName nvarchar(50),
       LastName nvarchar(50),
       Email nvarchar(100),
       PhoneNumber nvarchar(20),
       StoreName nvarchar(100),
       Location nvarchar(100),
       Quantity int,
       TotalAmount decimal(18,2)
   );
   ```

### **Step 2: Create the ETL Pipeline**

1. **Create a New Pipeline in Azure Data Factory**:
   - Name the pipeline `CSV_to_SQL_ETL_with_Transformation`.

2. **Add Copy Data Activities for Loading CSV Data**:
   - For each table (`Sales`, `Products`, `Customers`, `Stores`), create a `Copy Data` activity that copies data from the corresponding CSV file to the SQL table.

3. **Add Data Flow Activity for Transformation**:
   - **Create a Data Flow**:
     - In ADF Studio, go to `Author` > `Data Flows` and create a new Data Flow.
   - **Source Transformation**:
     - Add source transformations for the `Sales`, `Products`, `Customers`, and `Stores` tables by linking to the respective datasets.
   - **Join Transformations**:
     - Use the `Join` transformation to join the `Sales` table with `Products`, `Customers`, and `Stores` based on the `ProductID`, `CustomerID`, and `StoreID` fields.
   - **Select Transformation**:
     - Use the `Select` transformation to choose the columns you want in the `TransformedSales` table. Rename the columns as needed to match the `TransformedSales` schema.
   - **Sink Transformation**:
     - Add a sink transformation that points to the `TransformedSales` table.

4. **Add Data Flow Activity to the Pipeline**:
   - In the main pipeline, after the `Copy Data` activities, add a `Data Flow` activity and link it to the Data Flow you created for the transformation.

5. **Debug and Publish**:
   - Run the pipeline in debug mode to test the ETL process.
   - Publish the pipeline after successful testing.

### **Step 3: Run and Monitor the Pipeline**

- Monitor the pipeline in the `Monitor` section of ADF Studio to ensure data is correctly loaded into both the original tables and the `TransformedSales` table.

### **References**
- **[Azure Data Factory Documentation](https://docs.microsoft.com/en-us/azure/data-factory/introduction)**: Detailed documentation on using Data Flows and transformations in ADF.
- **[GitHub Example](https://github.com/sarannng/ETL-pipeline-project)**: For additional examples of Data Flow transformations【11†source】.
- **[Step-by-Step Tutorial](https://github.com/ssquadri/Azure-Data-Factory-Pipeline)**: Practical guide to ADF pipelines【13†source】.

This enhanced ETL pipeline will load data from CSV files into the SQL Server, perform transformations to join the fact and dimension tables, and output the transformed data into a new `TransformedSales` table.
