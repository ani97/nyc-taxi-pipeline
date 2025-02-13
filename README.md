# NYC Taxi Data Pipeline Documentation

## Overview

This project focuses on building a data pipeline to process NYC Green Taxi trip data using Azure Data Factory, Azure Data Lake Storage (ADLS), and Databricks. The pipeline follows the Medallion architecture, moving data through Bronze, Silver, and Gold layers.

The data was collected and provided to the NYC Taxi and Limousine Commission (TLC) by technology providers authorized under the Taxicab & Livery Passenger Enhancement Programs (TPEP/LPEP). The trip data was not created by the TLC, and TLC makes no representations as to the accuracy of these data.

## Data Sources

### 1. Trip Type File
- Contains trip type and description.
- Example:
  ```csv
  trip_type,description
  1,Street-hail
  2,Dispatch
  ```

### 2. Trip Zone Lookup File
- Contains location details.
- Example:
  ```csv
  LocationID,Borough,Zone,service_zone
  1,EWR,Newark Airport,EWR
  2,Queens,Jamaica Bay,Boro Zone
  ```

### 3. Green Taxi Trip Data
- Contains the following columns:

VendorID, lpep_pickup_datetime, lpep_dropoff_datetime, store_and_fwd_flag, RatecodeID, PULocationID, DOLocationID, passenger_count, trip_distance, fare_amount, extra, mta_tax, tip_amount, tolls_amount, ehail_fee, improvement_surcharge, total_amount, payment_type, trip_type, congestion_surcharge.

- Pulled using an API for the year 2023.
- Stored in ADLS in Parquet format.

---

## Data Pipeline Steps

### 1. Ingest Data into Bronze Layer
- Created a **Linked Service (LS)** in Azure Data Factory (ADF) to pull Green Taxi Trip records via API.
- Stored the data in ADLS Bronze Layer in Parquet format.
- Configured a **dynamic pipeline** to pull data efficiently.
- Configured **GitHub integration** in ADF to push pipeline configurations to the repository.

### 2. Moving Data from Bronze to Silver

#### Connecting Databricks with ADLS
- Created an **Application Service Principal** in Entra ID:
  1. Navigate to **Manage -> App registration**
  2. Registered an app: `nycservice_principle`
  3. Assigned access to ADLS:
     - **IAM -> Add role assignment**
     - Role: `Storage Blob Contributor`
     - Assigned to `nycservice_principle`

- Created a **Databricks resource**:
  - Collected `Application Client ID`, `Tenant ID`.
  - Generated a **Client Secret** under `Manage -> Certificates & Secrets`.
  - Used Microsoft documentation to set up a **Databricks-ADLS connection** using Python.

#### Transformations in Silver Layer
- Read CSV data from Bronze using **recursive file lookup()**.
- Performed transformations:
  1. **Trip Type Data**: Renamed columns and saved as Parquet.
  2. **Trip Zone Data**: Split `Zone` column values with `/` (e.g., `Bloomfield/Emerson Hill`) using string indexing. Saved as Parquet.
  3. **Trip Data**:
     - Extracted **pickup timestamp** into three separate columns: `Date`, `Year`, `Month`.
     - Selected relevant fields: `Vendor`, `PULocationID`, `DOLocationID`, `Fare_Amount`, `Total_Amount`.

Files executed:
- [`bronze_to_silver_notebook.ipynb`](./bronze_to_silver_notebook.ipynb)

---

### 3. Moving Data from Silver to Gold

#### Gold Layer Format
- **Gold Layer Format:** Converted Parquet to Delta format.
- Used **External Tables (Delta Tables)** since the gold container is owned by us.
- Created an **external location** and mounted it to the Gold container.
- Created **Unity Catalog External Location** for Delta tables.

#### Transformations in Gold Layer
- Moved **Trip Type, Trip Zone, and Trip Data (2023)** from Silver to Gold in Delta Table format.
- Created Delta tables on top of the data for optimized querying.

Files executed:
- [`silver_to_gold_delta.ipynb`](./silver_to_gold_delta.ipynb)

---

## References
- [Azure Data Factory Documentation](https://learn.microsoft.com/en-us/azure/data-factory/)
- [Azure Databricks Documentation](https://learn.microsoft.com/en-us/azure/databricks/)
