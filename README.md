# Enterprise Telecom Data Warehouse & ETL Pipeline (SSIS & T-SQL)

An end-to-end Enterprise Data Warehouse (EDW) solution built using **SQL Server (OLTP & DWH)** and orchestrated via **SQL Server Integration Services (SSIS)**. This project implements advanced Kimball dimensional modeling techniques including **Slowly Changing Dimensions (SCD Type 2)**, **Incremental Loading via Advanced Lookups**, and specialized data transformations.

---

## 🏗️ System Architecture & Schema Design

The pipeline follows a textbook **Star Schema architecture**, transferring raw telecommunication transaction logs from an operational transactional system (OLTP) to an analytical Data Warehouse (DWH).

### 1. Source Database (`telecom_oltp`)
Tracks real-time operations across four primary entities:
* `Customers`: Relational customer contact location and base subscription profile.
* `Plans`: Catalog of voice/data tariff plans and billing models.
* `Networks`: Infrastructural logs capturing network generation types (3G/4G/5G) and towers.
* `Call_Records`: Central transactional engine storing active durations, fees, and call types.

### 2. Destination Data Warehouse (`telecom_dwh`)
Optimized for high-fidelity analytical querying:
* `Customer_dim`: Modeled using **SCD Type 2** tracking historical profile movements (Start/End Dates, Current Flag).
* `Plan_dim`: Categorized dimension for tariff asset distributions.
* `Call_type_dim`: Standardized data-cleansed lookups.
* `Location_dim`: Regional location aggregation mapping geographic layers.
* `Network_dim`: Network assets and infrastructure dimension.
* `Date_dim`: Time intelligence dimension populated via a T-SQL loop.
* `Call_fact`: The central transaction repository holding numeric metrics (`duration_minutes`, `charge`) mapped directly to Surrogate Keys.

---

## 🚀 Advanced Pipeline Transformations & Key Refactors

### 1. Orchestration Logic (`Master.dtsx`)
Runs a layered workflow topology to enforce structural integrity constraints:
* **Sequence Container (`Load Dimensions`)**: Runs all dimension loading parallel pipelines synchronously (`Load_Location_Dim`, `Load_Network_Dim`, `Load_CallType_Dim`, `Load_Plan_Dim`, `Load_Customer_Dim`).
* **Precedence Constraint**: A strictly defined success boundary ensuring **Facts only load after all Dimension transformations finish successfully**.
* **Execute Package Task (`Load Fact`)**: Resolves keys and loads transactions into `Call_fact`.

### 2. Incremental Load Architecture via Lookups
To move away from resource-heavy Full Cleansing routines, all dimensions and the final fact tables implement **Incremental Loading**. 
* **Mechanism**: The incoming pipeline streams into a `Lookup Transformation` pointing directly to the Target DWH table.
* **Routing Execution**: Configured with `Redirect rows to no match output`. If the Business/Composite keys match an existing entity row, the row is discarded (**skipped**). If it is a `No Match`, it flags a brand new entry and routes directly to the `OLE DB Destination` for an `INSERT`.

### 3. Complex Multi-Stream Pipeline (`Load_Plan_Dim.dtsx`)
Instead of using standard sequential transforms, advanced data distribution routing was injected here:
* **Derived Columns**: Standardizes strings via `UPPER(TRIM())`.
* **Conditional Split**: Evaluates the `monthly_fee` metric. Routes rows with `monthly_fee >= 100` into a `Premium` stream, and the remainder into a `Default` (Economy) stream.
* **Union All Transformation**: Merges both diverging data paths cleanly back into a unified stream before executing the lookups, optimizing parallel row inspection.

### 4. Case-Insensitive Source Extraction (`Load_CallType_Dim.dtsx`)
    ```
* This ensures the lookup caches uniquely distinct items (`LOCAL`, `INTERNATIONAL`, `ROAMING`), preventing stream collision.

### 5. Enterprise Slowly Changing Dimensions (`Load_Customer_Dim.dtsx`)

### 6. Fact Table Duplication Resolution & Referential Integrity
    ```

---

## 💻 Tech Stack
* **Database Engine:** Microsoft SQL Server
* **ETL Framework:** SQL Server Integration Services (SSIS)
* **Modeling Framework:** Kimball Star Schema Modeling
