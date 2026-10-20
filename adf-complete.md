```markdown
# 🚀 Azure Data Factory (ADF) - Complete Hands-On Guide for Beginners

Welcome to the end-to-end practical guide on **Azure Data Factory (ADF)**. In this lab, we build a complete data pipeline that extracts raw CSV data from **Azure Blob Storage** and loads it into an **Azure SQL Database**.

---

## 📌 Table of Contents
- [Overview & Architecture](#overview--architecture)
- [Key ADF Terminology](#key-adf-terminology)
- [Lab Prerequisites](#lab-prerequisites)
- [Step-by-Step Hands-On Implementation](#step-by-step-hands-on-implementation)
  - [Step 1: Create Resource Group & Blob Storage](#step-1-create-resource-group--blob-storage)
  - [Step 2: Create Azure SQL Server & Database](#step-2-create-azure-sql-server--database)
  - [Step 3: Configure SQL Firewall Rules](#step-3-configure-sql-firewall-rules)
  - [Step 4: Create Destination Table in SQL](#step-4-create-destination-table-in-sql)
  - [Step 5: Deploy Azure Data Factory (ADF)](#step-5-deploy-azure-data-factory-adf)
  - [Step 6: Build Pipeline using Ingest Tool](#step-6-build-pipeline-using-ingest-tool)
  - [Step 7: Troubleshooting & Updating Pipeline](#step-7-troubleshooting--updating-pipeline)
  - [Step 8: Handling Daily / Multiple Files](#step-8-handling-daily--multiple-files)
- [ETL vs ELT Overview](#etl-vs-elt-overview)
- [Future Scope: Microsoft Fabric](#future-scope-microsoft-fabric)
- [Resource Cleanup](#resource-cleanup)

---

## 🏗️ Overview & Architecture

**Azure Data Factory (ADF)** ek fully managed, serverless, cloud-based data integration service hai. Iska main purpose data ko alag-alag sources se collect karna, transform karna, aur destination system men load karna hai.


```

+------------------------+          +-------------------------+          +-----------------------+
|   Source Data Store    |  =====>  |   Azure Data Factory    |  =====>  | Destination Data Store|
|   (Azure Blob Storage) |          |  (Pipeline Execution)   |          |  (Azure SQL Database) |
+------------------------+          +-------------------------+          +-----------------------+

```

---

## 🔑 Key ADF Terminology

| Component | Description |
| :--- | :--- |
| **Linked Service** | External services (Blob Storage, SQL DB) ke sath connection establish karne wali configuration. |
| **Dataset** | Asal data ka structural representation (e.g., CSV file path ya SQL table ka reference). |
| **Activity** | Pipeline ke andar execute hone wala specific task (e.g., Copy Data, Stored Procedure). |
| **Pipeline** | Activities ka logical grouping jo ek specific workflow run karti hai. |
| **Integration Runtime (IR)** | Compute infrastructure jahan ADF pipelines physical taur par run hoti hain. |
| **Trigger** | Pipeline ko schedule (e.g., daily basis) ya event-driven trigger karne wala engine. |

---

## 🛠️ Lab Prerequisites

1. Active **Microsoft Azure Subscription**.
2. Sample sales data file: `sales_data.csv`.
   * Sample schema: `SaleID, SaleDate, CustomerName, Amount`

---

## 📝 Step-by-Step Hands-On Implementation

### Step 1: Create Resource Group & Blob Storage
1. Azure Portal men search karein **Resource Groups** aur naya group create karein:
   * **Name:** `rg-adf-demo`
   * **Region:** Apni preferred location choose karein (e.g., `East Asia`).
2. Search karein **Storage Accounts** aur naya account deploy karein:
   * **Storage Account Name:** `adfdemostorage0926`
   * **Primary Service:** Azure Blob Storage / Data Lake Storage Gen2.
3. Deployment ke baad resource open karein, left menu men **Containers** par jayein:
   * Click **+ Container**.
   * **Name:** `sales-data`
   * Container open karein aur apni sample file `sales_data.csv` upload kar dein.

---

### Step 2: Create Azure SQL Server & Database
1. Azure Portal men search karein **Azure SQL**.
2. Click **Create** aur select karein **SQL Database**:
   * **Database Name:** `adf-demo-db`
   * **Resource Group:** `rg-adf-demo`
3. Click **Create New Server**:
   * **Server Name:** `adf-demo-server`
   * **Location:** `East Asia`
   * **Authentication Method:** SQL and Microsoft Entra authentication.
   * **Server Admin Login:** Apni pasand ka user name rakhein (e.g., `sqladmin`).
   * **Password:** Strong password set karein aur ise yaad rakhein.
4. Review & Create par click karke database deploy hone dein.

---

### Step 3: Configure SQL Firewall Rules
1. Deployed SQL Server open karein aur **Security** section men **Firewall & virtual networks** ya **Networking** par jayein.
2. Check karein: **"Allow Azure services and resources to access this server"** = `Yes`.
3. Click **Add your client IPv4 address** taake aap apne browser se login kar sakein.
4. Click **Save**.

---

### Step 4: Create Destination Table in SQL
1. Azure SQL Database open karein aur left menu se **Query Editor (preview)** open karein.
2. SQL authentication ke zariye credentials daal kar login karein.
3. Destination table create karne ke liye query execute karein:

```sql
CREATE TABLE Sales (
    SaleID INT PRIMARY KEY,
    SaleDate DATE,
    CustomerName VARCHAR(100),
    Amount DECIMAL(10, 2)
);

```

4. Verify karein ke left panel men `Tables` ke andar `Sales` table show ho rahi hai.

---

### Step 5: Deploy Azure Data Factory (ADF)

1. Portal search bar men search karein **Data Factories** (Version 2).
2. Click **Create**:
* **Name:** `adf-demo-factory-0926`
* **Resource Group:** `rg-adf-demo`
* **Version:** `V2`


3. Git configuration ko abhi ke liye select karein: **Configure Git later**.
4. Click **Review + Create**, then **Create**.
5. Resource complete hone ke baad **Launch Studio** par click karein.

---

### Step 6: Build Pipeline using Ingest Tool

1. ADF Studio home screen par click karein **Ingest** (Copy data tool).
2. Select **Built-in copy task** aur task cadence men select karein **Run once now**. Click **Next**.
3. **Configure Source:**
* **Source Type:** Azure Blob Storage.
* **Connection:** Click **New Connection**, apna storage account `adfdemostorage0926` select karein aur create karein.
* **File Path:** Browse karke `sales-data/sales_data.csv` select karein.
* ADF auto-detect karega ke file delimited text hai aur **First row as header** automatically check karega.


4. **Configure Destination (Sink):**
* **Destination Type:** Azure SQL Database.
* **Connection:** Click **New Connection**, apna SQL Server aur Database choose karein.
* Credentials fill karein. (Agar test connection fail ho, toh SQL Server firewall men ADF IP allow karein).


5. Task name specify karein: `CopyBlobToSql`.
6. Review Summary and click **Next** to deploy aur execute the pipeline.

---

### Step 7: Troubleshooting & Updating Pipeline

Agar pipeline execute ho jaye lekin SQL table empty rahe:

1. Left menu men **Monitor** icon par click karein aur pipeline runs check karein.
2. Agar status **Failed** ho, error trace check karein (Maslan: missing parameter `tableName`).
3. Fix karne ke liye:
* Left menu men **Author** par jayein aur pipeline open karein.
* Sink / Destination Dataset settings men jayein.
* **Table Name** drop-down se `Sales` table select karein.
* Up top par click karein **Publish all** taake changes save ho jayein.


4. Wapis **Monitor** men jayein aur pipeline ko **Re-run** karein.
5. SQL Database Query Editor men ja kar verify karein:

```sql
SELECT * FROM Sales;

```

Data successfully show ho jayega!

---

### Step 8: Handling Daily / Multiple Files

Agar daily basis par new files aati hain (e.g., `sales_data_2.csv`):

* **Source Dataset Update:** File path men specific file browse karke change karein.
* **Wildcard Path:** Agar folder ki sari files automatically process karni hon, toh wildcard path use kar sakte hain:
```text
Container: sales-data
Wildcard file name: *.csv

```


* **Schedule Trigger:** ADF men Trigger add karke pipeline ko rozana ke kisi specific time par automate kiya ja sakta hai.

---

## 🔄 ETL vs ELT Overview

Azure Data Factory dono architectural design patterns ko support karta hai:

```text
ETL (Extract -> Transform -> Load):
[Source Data] ---> [In-Memory / Staging Transformation] ---> [Target Database]

ELT (Extract -> Load -> Transform):
[Source Data] ---> [Direct Load into Target] ---> [Database Compute Transformation]

```

* **ETL:** Data pehle intermediate layer men clean aur format hota hai, phir target men jata hai.
* **ELT:** Modern cloud warehouses ke liye best approach, jahan raw data seedha destination men load hota hai aur SQL compute ke zariye transform hota hai.

---

## 💡 Future Scope: Microsoft Fabric

Microsoft ne ADF ki next-generation capabilities ko **Microsoft Fabric** ke andar integrate kar diya hai (**Fabric Data Factory**). Core engineering concepts (Pipelines, Activities, Connections) bilkul identical rehte hain, is liye ADF seekhna modern cloud data engineering ke liye solid foundation hai.

---

## 🧹 Resource Cleanup

Charges aur billing se bachne ke liye lab complete hone ke baad resource group delete karna na bhoolein:

1. Azure Portal men jayein **Resource Groups**.
2. Select karein `rg-adf-demo`.
3. Click **Delete resource group**, group name type karke confirm karein.

---

⭐ **Tip:** Is repo ko star aur fork kar lein taake future data engineering pipelines banate waqt quick reference ke taur par use kar sakein!

```

```

