I'll read the repository files to provide a comprehensive project analysis including data modeling and architecture.Based on the repository structure and code analysis, here's a comprehensive overview of the **ecommerce_data_pipeline** project:

## 📊 **ecommerce_data_pipeline**

**Overview:** A Databricks-based ETL (Extract, Transform, Load) pipeline for e-commerce data processing, building a dimensional data warehouse for profit analysis and business intelligence.

---

## 🏗️ **Architecture & Data Pipeline Flow**

```
Raw Source Data (CSV/JSON/Excel)
         ↓
    [INGESTION] (src/ingestion.ipynb)
         ↓
Staging Tables (stg_products, stg_customers, stg_orders)
         ↓
    [TRANSFORMATION] (src/transformation.ipynb)
         ↓
Dimensional Models (dim_products, dim_customers, fact_orders)
         ↓
    [ENRICHMENT] (src/enrich.ipynb)
         ↓
Enriched Facts & Analytics (enriched_orders, profit_by_year)
```

---

## 🗂️ **Repository Structure**

```
ecommerce_data_pipeline/
├── README.md                          # Project documentation
├── requirements.txt                   # Python dependencies
├── config/                            # Configuration files
│   └── ingestion_configs.json        # Data ingestion parameters
├── resources/                         # Shared utilities
│   └── utils.ipynb                   # Reusable functions library
├── src/                              # ETL pipeline notebooks
│   ├── ingestion.ipynb               # Data ingestion layer
│   ├── transformation.ipynb          # Data transformation & cleansing
│   └── enrich.ipynb                  # Data enrichment & aggregation
└── tests/                            # Test directory (reserved)
```

---

## 📋 **Data Modeling**

### **1. Staging Layer (Raw Data)**
Source tables created from raw files:
- `stg_products` – Raw product information
- `stg_customers` – Raw customer information  
- `stg_orders` – Raw order transactions

### **2. Dimensional Layer (Star Schema)**

#### **dim_products** (Dimension Table)
| Column | Type | Notes |
|--------|------|-------|
| product_id | String | Primary Key |
| product_name | String | Defaults to 'unknown' if NULL |
| category | String | Product category |
| sub_category | String | Product sub-category |
| state | String | Product state/status |
| price_per_product | Double | Unit price (default 0) |

#### **dim_customers** (Dimension Table)
| Column | Type | Notes |
|--------|------|-------|
| customer_id | String | Primary Key |
| customer_name | String | Customer full name |
| email | String | Email address |
| phone | String | Phone number |
| address | String | Full address |
| segment | String | Customer segment |
| country | String | Country of residence |
| city | String | City |
| state | String | State/Province |
| postal_code | String | Postal code |
| region | String | Geographic region |

#### **fact_orders** (Fact Table)
| Column | Type | Notes |
|--------|------|-------|
| order_id | String | Primary Key |
| order_date | Date | Order placement date (format: d/M/yyyy) |
| ship_date | Date | Shipment date (format: d/M/yyyy) |
| ship_mode | String | Shipping method |
| customer_id | String | Foreign Key → dim_customers |
| product_id | String | Foreign Key → dim_products |
| quantity | Integer | Units ordered |
| price | Double | Unit sale price |
| discount | Double | Discount applied |
| profit | Decimal(10,2) | Net profit |

### **3. Enriched Layer**

#### **enriched_orders** (Denormalized Fact Table)
Joins `fact_orders` with dimensions using broadcast optimization:
- All fact_orders columns
- `customer_name`, `country` (from dim_customers)
- `category`, `sub_category` (from dim_products)

#### **profit_by_year** (Aggregated Analytics Table)
Pre-aggregated profit metrics by:
- Year (extracted from order_date)
- Category & Sub-category
- Customer Name
- Total Profit (summed)

---

## 🔧 **Core Components**

### **1. Utilities Library** (`resources/utils.ipynb`)
Reusable functions for:
- **File Readers:** JSON, CSV, Excel parsing
- **Schema Management:** Column aliasing and schema application
- **Data Quality:** NULL validation, duplicate detection, schema validation, row count checks
- **Delta Writers:** Writing to Unity Catalog with overwrite safety
- **Ingestion Orchestration:** Batch processing with error handling
- **Transformation:** Standardized cleaning pipeline (NULL filtering, deduplication, type casting, date conversion)

### **2. Ingestion Pipeline** (`src/ingestion.ipynb`)
- Reads configuration from JSON file
- Supports multiple file formats (CSV, JSON, Excel)
- Creates staging tables in Unity Catalog
- Validates configuration before processing

### **3. Transformation Pipeline** (`src/transformation.ipynb`)
- Applies business rules and data quality standards
- **dim_products:** Filters NULL product_ids, fills missing categories/prices, deduplicates
- **dim_customers:** Normalizes customer attributes, defaults missing fields
- **fact_orders:** Parses dates, enforces referential integrity, validates key fields
- All transformations use the `clean_dataframe()` utility

### **4. Enrichment Pipeline** (`src/enrich.ipynb`)
- **Broadcast Optimization:** Uses `F.broadcast()` on smaller dimension tables to avoid expensive shuffle joins
- Creates denormalized enriched_orders table
- Computes `profit_by_year` aggregate table with year extraction
- Includes SQL analytics queries for business insights (profit by year, category, customer)

---

## 📊 **Data Quality Checks**

The pipeline includes comprehensive validation:

1. **Not Null Validation** – Ensures primary keys and critical fields have no NULLs
2. **Duplicate Detection** – Identifies and removes duplicate rows
3. **Schema Validation** – Verifies column presence and data types
4. **Row Count Validation** – Confirms minimum/maximum row thresholds

**Example Check:**
```python
validate_not_null(product_df_cleaned, ['product_id'], label='dim_products')
validate_row_count(product_df_cleaned, min_rows=1, label='dim_products')
```

---

## 🎯 **Key Features**

✅ **Modular Design** – Reusable utilities library separates concerns  
✅ **Error Handling** – Try-catch blocks with descriptive error messages  
✅ **Logging** – Structured logging for debugging and monitoring  
✅ **Data Quality Gates** – Validations at each pipeline stage  
✅ **Performance Optimization** – Broadcast joins for large-to-small table relationships  
✅ **Schema Management** – Dynamic column aliasing and type casting  
✅ **Configuration-Driven** – External JSON configuration for ingestion parameters  
✅ **Delta Lake Integration** – ACID transactions and schema evolution support  

---

## 🚀 **Execution Order**

```
1. ingestion.ipynb      → Creates stg_* tables
2. transformation.ipynb → Creates dim_* and fact_* tables
3. enrich.ipynb         → Creates enriched_* and profit_by_year tables
```

---

## 📌 **Use Cases**

- **Profit Analysis** – Aggregate profit by year, category, customer
- **Customer Analytics** – Segment analysis and spending patterns
- **Product Performance** – Category-level sales and profit metrics
- **Business Intelligence** – Pre-computed aggregates for BI dashboards