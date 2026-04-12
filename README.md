# databricks-lakehouse

> Personal data engineering project implementing a full medallion architecture lakehouse on Databricks — from raw file ingestion to analytics-ready gold layer tables.

---

## Architecture

```
Raw Files (Volumes)
       │
       ▼
┌─────────────┐
│   BRONZE    │  Raw ingestion — metadata-driven, schema-preserved
└─────────────┘
       │
       ▼
┌─────────────┐
│   SILVER    │  Cleansed & standardized — trimming, type casting,
│             │  normalization, data healing
└─────────────┘
       │
       ▼
┌─────────────┐
│    GOLD     │  Analytics-ready — dimensional model (dim/fact)
│             │  built from integrated Silver sources
└─────────────┘
```

### Gold Layer Model

```
dim_customers ──┐
                ├──► fact_sales
dim_products  ──┘
```

---

## Project Structure

```
databricks-lakehouse/
├── datasets/                        # Raw source files
├── scripts/
│   ├── bronze/
│   │   └── bronze_layer.ipynb       # Metadata-driven ingestion from Volumes to Delta tables
│   ├── silver/
│   │   ├── silver.crm_customer_info.ipynb
│   │   ├── silver.crm_products.ipynb
│   │   ├── silver.crm_sales.ipynb
│   │   ├── silver.erp_customers.ipynb
│   │   ├── silver.erp_customer_location.ipynb
│   │   └── silver.erp_product_category.ipynb
│   └── gold/
│       ├── gold.dim_customers.ipynb
│       ├── gold.dim_products.ipynb
│       └── gold.fact_sales.ipynb
├── configs/
│   └── jobs/
│       └── lakehouse_pipeline.yml   # Databricks job definition (full pipeline)
├── LICENSE
└── README.md
```

---

## Pipeline

The full pipeline is orchestrated as a Databricks Job with task dependencies:

```
bronze_ingestion
       │
       ├──► silver_crm_customer_info  ──┐
       ├──► silver_crm_products         │
       ├──► silver_crm_sales            ├──► gold_dim_customers ──┐
       ├──► silver_erp_customers        │                         ├──► gold_fact_sales
       ├──► silver_erp_customer_location│──► gold_dim_products ───┘
       └──► silver_erp_product_category ┘
```

- **Bronze** runs first
- All **Silver** tasks run in parallel after Bronze completes
- **Gold dimensions** run after all Silver tasks complete
- **Gold fact** runs after both dimensions are ready

The job definition can be found at `configs/jobs/lakehouse_pipeline.yml` and imported directly into any Databricks workspace.

---

## Setup & Usage

### Prerequisites
- Databricks workspace with Unity Catalog enabled
- A cluster running Databricks Runtime 13.0+
- Source files uploaded to a Databricks Volume

### Running the Pipeline

**Option 1 — Import the job definition**
1. Go to **Workflows → Create Job → Import**
2. Upload `configs/jobs/lakehouse_pipeline.yml`
3. Update notebook paths to match your workspace
4. Click **Run now**

**Option 2 — Run notebooks manually**
Execute notebooks in this order:
1. `scripts/bronze/bronze_layer.ipynb`
2. Any/all notebooks under `scripts/silver/` (order independent)
3. `scripts/gold/gold.dim_customers.ipynb` and `gold.dim_products.ipynb`
4. `scripts/gold/gold.fact_sales.ipynb`

---

## Tech Stack

| Tool | Purpose |
|---|---|
| Databricks | Compute & orchestration |
| Apache Spark / PySpark | Data transformation |
| Delta Lake | Storage layer |
| Unity Catalog | Data governance |
| GitHub | Version control |

