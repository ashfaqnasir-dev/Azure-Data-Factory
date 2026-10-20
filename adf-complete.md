Samajh gaya! Screenshot mein text plain raw markdown ki tarah dikh raha tha. Yahan poora document **directly beautifully formatted visual Markdown** mein present hai taake headers, tables, callouts, aur badges sab visually chamakte hue nazar aayein aur aap directly copy karke GitHub par `README.md` ke taur par use kar sakein:

---

# 🚀 Azure Data Factory (ADF) Masterclass: Zero to Hero Hands-On

> **Level:** Beginner to Intermediate | **Tech Stack:** Azure Data Factory, Blob Storage, Azure SQL Database | **Type:** Hands-On Lab

An end-to-end practical walkthrough on building enterprise data pipelines using **Azure Data Factory (ADF)**. Extract raw transaction data from **Azure Blob Storage**, transform and validate, and ingest directly into an **Azure SQL Database**.

---

## 📌 Quick Navigation

* [🏗️ Architectural Blueprint](https://www.google.com/search?q=%23%EF%B8%8F-architectural-blueprint)
* [🔑 Core ADF Terminology](https://www.google.com/search?q=%23-core-adf-terminology)
* [🛠️ Lab Prerequisites](https://www.google.com/search?q=%23%EF%B8%8F-lab-prerequisites)
* [⚡ Step-by-Step Practical Implementation](https://www.google.com/search?q=%23-step-by-step-practical-implementation)
* [Step 1: Resource Group & Storage Setup](https://www.google.com/search?q=%23step-1-resource-group--storage-setup)
* [Step 2: Provision Azure SQL Server & DB](https://www.google.com/search?q=%23step-2-provision-azure-sql-server--db)
* [Step 3: Network Security & Firewall Rules](https://www.google.com/search?q=%23step-3-network-security--firewall-rules)
* [Step 4: SQL Target Schema Setup](https://www.google.com/search?q=%23step-4-sql-target-schema-setup)
* [Step 5: Spin Up Azure Data Factory](https://www.google.com/search?q=%23step-5-spin-up-azure-data-factory)
* [Step 6: Build End-to-End Ingestion Pipeline](https://www.google.com/search?q=%23step-6-build-end-to-end-ingestion-pipeline)
* [Step 7: Production Debugging & Fixes](https://www.google.com/search?q=%23step-7-production-debugging--fixes)
* [Step 8: Automating Dynamic & Daily Files](https://www.google.com/search?q=%23step-8-automating-dynamic--daily-files)


* [🔄 ETL vs ELT Frameworks](https://www.google.com/search?q=%23-etl-vs-elt-frameworks)
* [🔮 Future Horizon: Microsoft Fabric](https://www.google.com/search?q=%23-future-horizon-microsoft-fabric)
* [🧹 Teardown & Resource Cleanup](https://www.google.com/search?q=%23-teardown--resource-cleanup)

---

## 🏗️ Architectural Blueprint

```
 ┌────────────────────────────────┐
 │     Source: Azure Blob         │  Format: sales_data.csv (Delimited Raw Data)
 └──────────────┬─────────────────┘
                │
                ▼ [Extract]
 ┌────────────────────────────────┐
 │      Azure Data Factory        │  • Linked Services (Auth & Endpoints)
 │     (Orchestration Engine)     │  • Datasets (Schema Definition)
 │                                │  • Copy Activity (Compute via AutoResolveIR)
 └──────────────┬─────────────────┘
                │
                ▼ [Load & Transform]
 ┌────────────────────────────────┐
 │   Destination: Azure SQL DB    │  Target: dbo.Sales Table
 └────────────────────────────────┘

```

---

## 🔑 Core ADF Terminology

| Component | Architecture Role | Practical Context |
| --- | --- | --- |
| **Linked Service** | Connection Manager | Holds connection strings, tokens, credentials for external services (Blob, SQL). |
| **Dataset** | Data Schema & Pointer | Points specifically to `sales_data.csv` inside container or `dbo.Sales` inside SQL DB. |
| **Activity** | Atomic Execution Unit | The actual task executed (e.g., `CopyData`, `Lookup`, `Execute Pipeline`). |
| **Pipeline** | Workflow Orchestrator | A logical container orchestrating multiple interconnected activities sequentially or in parallel. |
| **Integration Runtime (IR)** | Compute Engine | The underlying compute infrastructure executing the copy activities and data flows. |
| **Trigger** | Automation Schedule | Schedules pipelines via wall-clock schedules, tumbling windows, or storage events. |

---

## 🛠️ Lab Prerequisites

* Active **Microsoft Azure Subscription** (Free Tier / Pay-As-You-Go).
* Sample CSV dataset: `sales_data.csv`

```csv
SaleID,SaleDate,CustomerName,Amount
1001,2026-09-01,Ali Khan,1250.50
1002,2026-09-01,Zainab Fatima,890.00
1003,2026-09-02,Hamza Tariq,450.75
1004,2026-09-03,Ayesha Noor,3200.00
1005,2026-09-03,Bilal Ahmed,150.25

```

---

## ⚡ Step-by-Step Practical Implementation

### Step 1: Resource Group & Storage Setup

1. Azure Portal par search karein **Resource Groups** aur create karein:
* **Name:** `rg-adf-demo`
* **Region:** `East Asia` (ya nearest low-latency region)


2. Search karein **Storage Accounts** aur deploy karein:
* **Storage Account Name:** `adfdemostorage0926` (globally unique)
* **Performance:** Standard | **Redundancy:** LRS


3. Deploy hone ke baad left panel se **Data storage > Containers** open karein:
* Click **+ Container** ➔ Name: `sales-data` ➔ Public access level: `Private`.
* Open `sales-data` container aur upload karein `sales_data.csv`.



---

### Step 2: Provision Azure SQL Server & DB

1. Portal search bar mein navigate karein **Azure SQL** ➔ Click **Create** ➔ Choose **SQL Database**.
* **Database Name:** `adf-demo-db`
* **Resource Group:** `rg-adf-demo`


2. **Server Configuration:** Click *Create New*:
* **Server Name:** `adf-demo-server-0926`
* **Location:** `East Asia`
* **Authentication:** SQL & Microsoft Entra Authentication
* **Admin Login:** `sqladmin`
* **Password:** `P@ssw0rd2026#Secure` *(apna strong password set karein)*


3. **Compute + Storage:** Dev/Test workload ke liye Basic ya General Purpose tier select karein.
4. Click **Review + Create** aur deployment complete hone dein.

---

### Step 3: Network Security & Firewall Rules

1. Deployed SQL Server overview page par jayein ➔ Navigate to **Security > Networking**.
2. **Exceptions:** Set **"Allow Azure services and resources to access this server"** to `Enabled` (Mandatory for ADF connectivity).
3. **Firewall Rules:** Click **Add your client IPv4 address** taake aapka local browser Query Editor ko block na kare.
4. Click **Apply / Save**.

---

### Step 4: SQL Target Schema Setup

1. `adf-demo-db` open karein aur left menu se **Query Editor (preview)** par click karein.
2. Login karein using **SQL Server Authentication** credentials.
3. Target schema execute karein:

```sql
-- Target DDL Schema for Sales Ingestion
IF OBJECT_ID('dbo.Sales', 'U') IS NOT NULL
    DROP TABLE dbo.Sales;

CREATE TABLE dbo.Sales (
    SaleID INT NOT NULL PRIMARY KEY,
    SaleDate DATE NOT NULL,
    CustomerName NVARCHAR(100) NOT NULL,
    Amount DECIMAL(10, 2) NOT NULL,
    IngestedAt DATETIME2 DEFAULT SYSUTCDATETIME()
);

-- Validate schema creation
SELECT * FROM sys.tables WHERE name = 'Sales';

```

---

### Step 5: Spin Up Azure Data Factory

1. Search karein **Data Factories** (Version 2) ➔ Click **Create**:
* **Name:** `adf-demo-factory-0926`
* **Resource Group:** `rg-adf-demo`
* **Version:** `V2`


2. **Git Configuration:** Select *Configure Git later*.
3. Review & Create par click karein.
4. Factory ready hone par click karein **Launch Studio**.

---

### Step 6: Build End-to-End Ingestion Pipeline

1. ADF Studio home screen par click karein **Ingest** (Copy data assistant).
2. **Properties:**
* Task type: **Built-in copy task**
* Task cadence: **Run once now**


3. **Source Store:**
* Source Type: **Azure Blob Storage**
* Connection: Click *New Connection* ➔ Select subscription ➔ Pick `adfdemostorage0926` ➔ Click *Create*.
* File Path: Browse karke `sales-data` ➔ `sales_data.csv` pick karein.
* Auto-detection check karein: Format = DelimitedText, Column delimiter = Comma (`,`), First row as header = Checked.


4. **Destination Store (Sink):**
* Destination Type: **Azure SQL Database**
* Connection: Click *New Connection* ➔ Pick `adf-demo-server` & `adf-demo-db` ➔ Enter `sqladmin` credentials.


5. Task Name: `CopyBlobToSql` ➔ Review summary ➔ Click **Next** to deploy & execute.

---

### Step 7: Production Debugging & Fixes

Agar pipeline execute ho jaye lekin target table empty ho:

```
[Issue Encountered]
Status: Failed
Error Details: Missing target table parameter binding / table does not exist.

```

#### Diagnostic & Fix Workflow:

1. Left navigation panel se **Monitor** tab open karein ➔ Inspect pipeline run error tree.
2. Left navigation panel se **Author** tab open karein ➔ Open pipeline `CopyBlobToSql`.
3. Select **Copy Activity** ➔ Go to the **Sink** tab:
* **Sink Dataset:** Select `AzureSqlTable`.
* **Table Name:** Dropdown se explicitly `dbo.Sales` select karein.
* **Mapping Tab:** Verify karein ke CSV columns SQL columns ke sath 1:1 mapped hain.


4. Top navigation menu par click karein **Publish all** (save changes to ADF metadata).
5. Navigate back to **Monitor** ➔ Click **Re-run**.
6. Verify inside SQL Query Editor:

```sql
SELECT 
    COUNT(*) AS TotalRecords,
    SUM(Amount) AS TotalRevenue,
    MIN(SaleDate) AS EarliestSale,
    MAX(SaleDate) AS LatestSale
FROM dbo.Sales;

SELECT TOP 10 * FROM dbo.Sales ORDER BY SaleID ASC;

```

---

### Step 8: Automating Dynamic & Daily Files

Real-world scenarios mein daily files process karne ke do patterns hain:

#### Pattern A: Wildcard Ingestion

Agar daily multiple batches aate hain:

* Source Dataset setting mein **File path type** ko **Wildcard file path** set karein:
* Container: `sales-data`
* Wildcard file name: `sales_*.csv`



#### Pattern B: Event-Driven / Schedule Triggers

* ADF Studio mein **Trigger > New/Edit** select karein.
* **Type:** *Schedule* (e.g., Har raat 02:00 AM UTC) ya *Storage events* (jaise hi Blob container mein nayi file upload ho, pipeline trigger ho jaye).

---

## 🔄 ETL vs ELT Frameworks

```
========================================================================
ETL: Extract ➔ Transform ➔ Load (Traditional Data Warehousing)
========================================================================
[Raw Blob Storage]
       │
       ▼ (Extract)
[In-Memory ADF / Spark Cluster] ───► Transformations (Cleansing / Regex)
       │
       ▼ (Load Cleaned Data)
[Azure SQL DB / Data Warehouse]

========================================================================
ELT: Extract ➔ Load ➔ Transform (Modern Cloud Lakehouse)
========================================================================
[Raw Blob Storage]
       │
       ▼ (Fast Direct Load via PolyBase / Bulk Copy)
[Azure SQL / Synapse / Snowflake]
       │
       ▼ (Transform in-engine via SQL Views / Stored Procedures / dbt)
[Production Star Schema / Reporting Marts]

```

---

## 🔮 Future Horizon: Microsoft Fabric

* Microsoft ne ADF ki cloud-scale ETL capabilities ko **Microsoft Fabric** ke **Data Factory** component mein absorb kar liya hai.
* Agar aap modern data engineering start kar rahe hain, toh foundational concepts (**Pipelines, Activities, Connections, Triggers**) ADF aur Fabric dono mein bilkul same hain.

---

## 🧹 Teardown & Resource Cleanup

To avoid unintended compute and storage billing:

```bash
# Azure CLI cleanup command
az group delete --name rg-adf-demo --yes --no-wait

```

Ya phir Azure Portal par:

1. Open **Resource Groups**.
2. Select `rg-adf-demo`.
3. Click **Delete resource group** ➔ Type group name to confirm ➔ Click **Delete**.

---
