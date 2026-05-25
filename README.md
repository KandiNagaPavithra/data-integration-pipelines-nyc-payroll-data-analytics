# Data Integration Pipelines for NYC Payroll Data Analytics

## Project Description

Built data pipelines to analyze how New York City allocates its financial resources and how much of the budget goes to overtime. The pipelines integrate payroll data from multiple sources and aggregate it into a summary that can be queried by the public.

## Tools Used

- Azure Data Lake Storage Gen2
- Azure SQL Database
- Azure Data Factory
- Azure Synapse Analytics (Serverless SQL Pool)

## Pipeline Architecture

Data flows from CSV source files in ADLS Gen2 into Azure SQL Database, gets transformed and aggregated using Azure Data Factory, and the summary results are exported back to ADLS Gen2 staging directory. Azure Synapse Analytics queries the output via an external table.

## Steps Completed

**1. Data Infrastructure**
- Created ADLS Gen2 storage with three directories: `dirpayrollfiles`, `dirhistoryfiles`, `dirstaging`
- Uploaded source CSV files
- Created Azure SQL Database with master and transaction tables
- Created Synapse workspace with external table pointing to dirstaging

**2. Linked Services**
- `ls_adls` — connects ADF to Azure Data Lake Gen2
- `ls_sqldb` — connects ADF to SQL Database (Version: Legacy)

**3. Datasets**
- 5 source datasets from ADLS CSV files
- 5 target datasets for SQL DB tables
- 1 output dataset for dirstaging folder

**4. Data Flows**
- 5 simple load flows: CSV files → SQL DB tables
- 1 aggregation flow: unions 2020 + 2021 payroll data, filters by fiscal year, calculates TotalPaid, aggregates by AgencyName and FiscalYear, writes to SQL DB summary table and dirstaging

**TotalPaid = RegularGrossPaid + TotalOTPaid + TotalOtherPay**

**5. Pipeline**

One pipeline with three phases:
- Phase 1 (parallel): Load Agency, Employee, Title master data
- Phase 2 (parallel, runs after phase 1): Load 2020 and 2021 payroll data
- Phase 3 (runs after phase 2): Run aggregation

**6. Verification**
- SQL DB: queried NYC_Payroll_Summary table — data present
- ADLS: dirstaging directory contains output files from pipeline run
- Synapse: queried external table via OPENROWSET — data present

## Data Schema

Master tables: `NYC_Payroll_AGENCY_MD`, `NYC_Payroll_EMP_MD`, `NYC_Payroll_TITLE_MD`

Transaction table: `NYC_Payroll_Data` (holds both 2020 and 2021 data)

Output table: `NYC_Payroll_Summary` (FiscalYear, AgencyName, TotalPaid)
