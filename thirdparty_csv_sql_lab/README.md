To transform data from the provided CSV URLs into the `TransformedSales` table in SQL Server, follow these steps:

### **1. Extract Data from CSV URLs Using Python**

#### **Steps:**

1. **Install Required Libraries:**

   ```bash
   pip install pandas sqlalchemy pyodbc
   ```

2. **Extract Data Using Python:**

   ```python
   import pandas as pd

   # Define the URLs
   urls = {
       'Customers': 'https://raw.githubusercontent.com/bubz90/AzureDataFactoryLab/main/adf_csv_sql_lab/Customers.csv',
       'Sales': 'https://raw.githubusercontent.com/bubz90/AzureDataFactoryLab/main/adf_csv_sql_lab/Sales.csv',
       'Products': 'https://raw.githubusercontent.com/bubz90/AzureDataFactoryLab/main/adf_csv_sql_lab/Products.csv',
       'Stores': 'https://raw.githubusercontent.com/bubz90/AzureDataFactoryLab/main/adf_csv_sql_lab/Stores.csv'
   }

   # Load data from URLs into DataFrames
   customers_df = pd.read_csv(urls['Customers'])
   sales_df = pd.read_csv(urls['Sales'])
   products_df = pd.read_csv(urls['Products'])
   stores_df = pd.read_csv(urls['Stores'])
   ```

### **2. Clean and Transform the Data Using Pandas**

#### **Steps:**

1. **Inspect and Clean Data:**

   ```python
   # Inspect the data
   print(customers_df.head())
   print(sales_df.head())
   print(products_df.head())
   print(stores_df.head())

   # Remove duplicates and handle missing values
   customers_df = customers_df.drop_duplicates().fillna('Unknown')
   sales_df = sales_df.drop_duplicates().fillna(0)
   products_df = products_df.drop_duplicates().fillna(0)
   stores_df = stores_df.drop_duplicates().fillna('Unknown')
   ```

2. **Transform Data for `TransformedSales`:**

   ```python
   # Merge DataFrames to create the TransformedSales table
   transformed_sales_df = sales_df.merge(products_df, on='ProductID') \
                                  .merge(customers_df, on='CustomerID') \
                                  .merge(stores_df, on='StoreID')

   # Select and rename columns to match TransformedSales schema
   transformed_sales_df = transformed_sales_df[['SaleID', 'ProductName', 'Category', 'Brand', 'Price', 
                                                'FirstName', 'LastName', 'Email', 'PhoneNumber', 
                                                'StoreName', 'Location', 'Quantity', 'TotalAmount']]
   ```

### **3. Load the Data into Azure SQL Database**

#### **Prerequisites:**
- Azure SQL Database instance
- SQL Server connection details: hostname, database name, username, and password

#### **Steps:**

1. **Set Up Azure SQL Database:**

   Ensure that your Azure SQL Database is accessible and configured to allow connections from your IP address.

2. **Connect to Azure SQL Database and Load Data:**

   ```python
   from sqlalchemy import create_engine

   # Define connection parameters
   server = 'your_server.database.windows.net'
   database = 'your_database_name'
   username = 'your_username'
   password = 'your_password'

   # Create connection string
   connection_string = f"mssql+pyodbc://{username}:{password}@{server}/{database}?driver=ODBC+Driver+17+for+SQL+Server"

   # Create SQLAlchemy engine
   engine = create_engine(connection_string)

   # Define the table name
   table_name = 'TransformedSales'

   # Load the DataFrame into the Azure SQL Database table
   transformed_sales_df.to_sql(name=table_name, con=engine, if_exists='replace', index=False)
   ```

### **Complete Code Example**

Here’s the complete Python script:

```python
import pandas as pd
from sqlalchemy import create_engine

# Step 1: Extract Data from CSV URLs
urls = {
    'Customers': 'https://raw.githubusercontent.com/bubz90/AzureDataFactoryLab/main/adf_csv_sql_lab/Customers.csv',
    'Sales': 'https://raw.githubusercontent.com/bubz90/AzureDataFactoryLab/main/adf_csv_sql_lab/Sales.csv',
    'Products': 'https://raw.githubusercontent.com/bubz90/AzureDataFactoryLab/main/adf_csv_sql_lab/Products.csv',
    'Stores': 'https://raw.githubusercontent.com/bubz90/AzureDataFactoryLab/main/adf_csv_sql_lab/Stores.csv'
}

customers_df = pd.read_csv(urls['Customers'])
sales_df = pd.read_csv(urls['Sales'])
products_df = pd.read_csv(urls['Products'])
stores_df = pd.read_csv(urls['Stores'])

# Step 2: Clean and Transform Data
customers_df = customers_df.drop_duplicates().fillna('Unknown')
sales_df = sales_df.drop_duplicates().fillna(0)
products_df = products_df.drop_duplicates().fillna(0)
stores_df = stores_df.drop_duplicates().fillna('Unknown')

transformed_sales_df = sales_df.merge(products_df, on='ProductID') \
                               .merge(customers_df, on='CustomerID') \
                               .merge(stores_df, on='StoreID')

transformed_sales_df = transformed_sales_df[['SaleID', 'ProductName', 'Category', 'Brand', 'Price', 
                                             'FirstName', 'LastName', 'Email', 'PhoneNumber', 
                                             'StoreName', 'Location', 'Quantity', 'TotalAmount']]

# Step 3: Load Data into Azure SQL Database
server = 'your_server.database.windows.net'
database = 'your_database_name'
username = 'your_username'
password = 'your_password'

connection_string = f"mssql+pyodbc://{username}:{password}@{server}/{database}?driver=ODBC+Driver+17+for+SQL+Server"
engine = create_engine(connection_string)
table_name = 'TransformedSales'
transformed_sales_df.to_sql(name=table_name, con=engine, if_exists='replace', index=False)
```

### **Additional Notes**

- **Firewall Settings**: Ensure Azure SQL Database firewall settings allow connections from your IP address.
- **ODBC Driver**: Ensure "ODBC Driver 17 for SQL Server" is installed. [Download it here](https://docs.microsoft.com/en-us/sql/connect/odbc/download-odbc-driver-for-sql-server).
- **Security**: Use environment variables or Azure Key Vault to manage credentials securely.

This guide provides a comprehensive approach to extracting data from public CSV URLs, cleaning and transforming it, and loading it into an Azure SQL Database. Adjust the code based on your specific database and API requirements.