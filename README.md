# Bronze Layer Data Warehouse

## Overview
A SQL Server-based ETL pipeline that implements the bronze layer of a medallion architecture data warehouse. This project ingests raw data from multiple enterprise source systems (CRM and ERP) into a centralized bronze schema for downstream processing and analytics.

## Architecture

### Medallion Architecture - Bronze Layer
The bronze layer serves as the initial landing zone for raw data ingestion:
- **Purpose**: Store raw, unprocessed data from source systems
- **Pattern**: Full refresh (truncate and load)
- **Data Sources**: CRM system and ERP system
- **Load Method**: Bulk insert from CSV files

### System Components

```
Source Systems
├── CRM System
│   ├── Customer Information (cust_info.csv)
│   ├── Product Information (prd_info.csv)
│   └── Sales Details (sales_details.csv)
└── ERP System
    ├── Customer Data (cust_az12.csv)
    ├── Location Data (loc_a101.csv)
    └── Product Category Data (px_cat_g1v2.csv)
```

## Database Schema

### Bronze Schema Tables

#### CRM Tables

**bronze.crm_cust_info**
- Customer master data from CRM system
- Fields: customer ID, key, name, marital status, gender, creation date

**bronze.crm_prd_info**
- Product master data from CRM system
- Fields: product ID, key, name, cost, product line, start/end dates

**bronze.crm_sales_details**
- Transactional sales data
- Fields: order number, product key, customer ID, dates (order/ship/due), sales amount, quantity, price

#### ERP Tables

**bronze.erp_loc_a101**
- Location/country reference data
- Fields: customer ID, country

**bronze.erp_cust_az12**
- Customer demographic data from ERP
- Fields: customer ID, birthdate, gender

**bronze.erp_px_cat_g1v2**
- Product category and maintenance data
- Fields: product ID, category, subcategory, maintenance flag

## Features

### ETL Pipeline
- **Automated Loading**: Single stored procedure orchestrates all data loads
- **Performance Tracking**: Captures load duration for each table
- **Error Handling**: Comprehensive try-catch blocks with detailed error logging
- **Batch Processing**: Tracks total pipeline execution time

### Optimization Techniques
- **BULK INSERT**: High-performance data loading from CSV files
- **TABLOCK**: Table-level locking for improved bulk insert performance
- **Truncate Pattern**: Ensures clean slate for each load cycle

### Monitoring & Logging
- Real-time console output during execution
- Per-table load duration tracking
- Total batch execution time
- Error details including message and error number

## Installation

### Prerequisites
- SQL Server (2016 or later recommended)
- Appropriate file system permissions to access CSV files
- Database permissions to create schemas, tables, and stored procedures

### Setup Steps

1. **Create Database Structure**
```
-- Run ddl.sql to create bronze schema and tables
-- This will drop existing tables if they exist
```

2. **Create Stored Procedure**
```
-- Run proc_load.sql to create the load procedure
```

3. **Update File Paths**
```
-- Modify CSV file paths in proc_load.sql to match your environment
```

## Usage

### Execute Bronze Layer Load
```
EXEC bronze.load_bronze
```

### Expected Output
```
===========================
Loading Bronze Layer
===========================

---------------------------
Loading CRM Tables
---------------------------
>> Truncating Table: bronze.crm_cust_info
>> Inserting Data Into: bronze.crm_cust_info
>> Load Duration: 2 seconds
>> ----------------
...
========================
Loading Bronze Layer is completed
>> Total Load Duration: 15 seconds
========================
```

## Data Flow

```
CSV Files → BULK INSERT → Bronze Tables → [Silver Layer (Future)]
```

1. **Extract**: CSV files from source systems
2. **Load**: BULK INSERT into bronze tables
3. **Validate**: Error handling catches load failures
4. **Monitor**: Duration tracking for performance analysis

## File Structure

```
project/
├── ddl.sql                 # Schema and table definitions
├── proc_load.sql          # ETL stored procedure
└── datasets/
    ├── source_crm/
    │   ├── cust_info.csv
    │   ├── prd_info.csv
    │   └── sales_details.csv
    └── source_erp/
        ├── cust_az12.csv
        ├── loc_a101.csv
        └── px_cat_g1v2.csv
```

## Technical Details

### Load Pattern
- **Strategy**: Full refresh (truncate and load)
- **Frequency**: On-demand execution
- **Idempotency**: Tables are truncated before each load

### Performance Considerations
- TABLOCK hint reduces locking overhead during bulk operations
- FIRSTROW = 2 skips CSV headers
- Field terminator set to comma for CSV parsing
- Sequential table loading (not parallel)

### Error Handling
```
BEGIN TRY
    -- Load operations
END TRY
BEGIN CATCH
    -- Error logging with message and number
END CATCH
```

## Future Enhancements

- [ ] Implement incremental loading strategy
- [ ] Add data quality checks
- [ ] Create silver layer transformation logic
- [ ] Implement audit logging to persistent tables
- [ ] Add parameter support for dynamic file paths
- [ ] Parallel table loading for improved performance
- [ ] Data validation and cleansing rules
- [ ] Integration with orchestration tools (Azure Data Factory, Apache Airflow)

## Data Lineage

**Source Systems** → **Bronze Layer** → Silver Layer → Gold Layer

The bronze layer maintains raw data fidelity, enabling:
- Historical data reconstruction
- Reprocessing capabilities
- Audit trails
- Source system reconciliation

## Contributing
This is a personal/portfolio project demonstrating ETL and data warehousing concepts.


## Author
Ishvir Chopra

## Acknowledgments
- Medallion architecture pattern (Databricks)
- Microsoft SQL Server documentation
- Data warehouse design best practices
