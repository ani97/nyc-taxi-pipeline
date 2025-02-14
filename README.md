**NYC Taxi Data Pipeline Documentation**

**Overview**

This project focuses on building a data pipeline to process NYC taxi trip data using Azure Data Factory, Azure Data Lake Storage (ADLS), and Databricks. The pipeline follows the Medallion architecture, moving data through Bronze, Silver, and Gold layers.

**Data Sources**

**1. Trip Type File**

Contains trip type and description.

Example:

trip_type,description

1,Street-hail
2,Dispatch

**2. Trip Zone Lookup File**

Contains location details.

Example:

LocationID,Borough,Zone,service_zone

1,EWR,Newark Airport,EWR
2,Queens,Jamaica Bay,Boro Zone

**3. Green Taxi Trip Data**

Pulled using an API for the year 2023.

Stored in ADLS in Parquet format.

**Data Pipeline Steps**

**1. Ingest Data into Bronze Layer**

Created a Linked Service (LS) in Azure Data Factory (ADF) to pull Green Taxi Trip records via API.

Stored the data in ADLS Bronze Layer in Parquet format.

Configured a dynamic pipeline to pull data efficiently.

Configured GitHub integration in ADF to push pipeline configurations to the repository.

**2. Moving Data from Bronze to Silver**

Connecting Databricks with ADLS

Created an Application Service Principal in Entra ID:

Navigate to Manage -> App registration

Registered an app: nycservice_principle

Assigned access to ADLS:

IAM -> Add role assignment

Role: Storage Blob Contributor

Assigned to nycservice_principle

Created a Databricks resource:

Collected Application Client ID, Tenant ID.

Generated a Client Secret under Manage -> Certificates & Secrets.

Used Microsoft documentation to set up a Databricks-ADLS connection using Python.

Transformations in Silver Layer

Read CSV data from Bronze using recursive file lookup().

Performed transformations:

Trip Type Data: Renamed columns and saved as Parquet.

Trip Zone Data: Split Zone column values with / (e.g., Bloomfield/Emerson Hill) using string indexing. Saved as Parquet.

Trip Data:

Extracted pickup timestamp into three separate columns: Date, Year, Month.

Selected relevant fields: Vendor, PULocationID, DOLocationID, Fare_Amount, Total_Amount.

Files executed:

bronze_to_silver_notebook.ipynb

**3. Moving Data from Silver to Gold**

Gold Layer Format

Gold Layer Format: Converted Parquet to Delta format.

Used External Tables (Delta Tables) since the gold container is owned by us.

Created an external location and mounted it to the Gold container.

Created Unity Catalog External Location for Delta tables.

Transformations in Gold Layer

Moved Trip Type, Trip Zone, and Trip Data (2023) from Silver to Gold in Delta Table format.

Created Delta tables on top of the data for optimized querying.

Files executed:

silver_to_gold_delta.ipynb
