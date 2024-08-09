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