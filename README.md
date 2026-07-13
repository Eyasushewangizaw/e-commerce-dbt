# E-Commerce dbt Analytics Pipeline

A containerized data transformation project that models synthetic e-commerce sales data using [dbt](https://www.getdbt.com/) and PostgreSQL. Raw transactional data flows through a medallion architecture (Bronze → Silver → Gold) into a star schema ready for analytics.

## Overview

This project demonstrates an end-to-end analytics workflow:

1. **Generate** synthetic sales data with intentional data quality issues
2. **Load** the data into PostgreSQL via dbt seeds
3. **Transform** records through Bronze, Silver, and Gold layers
4. **Serve** dimension and fact tables for reporting

## Architecture

<img width="878" height="286" alt="Untitled Diagram drawio (2)" src="https://github.com/user-attachments/assets/d97bdc99-721c-4ab8-a339-96f50a38021c" />

## Tech Stack

| Component | Version |
|-----------|---------|
| Python | 3.11 |
| dbt-core | 1.11.6 |
| dbt-postgres | 1.10.0 |
| PostgreSQL | 15 |
| Docker Compose | 3.9 |

## Project Structure

```
e-commerce-dbt/
├── data_source/
│   └── generate_sales.py       # Synthetic data generator
├── dbt_project/
│   ├── .dbt/
│   │   └── profiles.yml        # Database connection profile
│   └── dbt_ecomm/
│       ├── dbt_project.yml
│       ├── models/
│       │   ├── bronze/         # Raw layer
│       │   ├── silver/         # Cleaned layer
│       │   └── gold/           # Analytics-ready star schema
│       └── seeds/
│           └── ecommerce_sales.csv
├── Dockerfile
├── docker-compose.yml
└── README.md
```

## Data Model

### Source Data (`ecommerce_sales` seed)

Each row represents an e-commerce transaction with fields such as `transaction_id`, `order_date`, `customer_id`, `product_id`, `quantity`, `price`, `payment_method`, and `order_status`.

The included Python generator (`data_source/generate_sales.py`) creates 1,000 records and injects ~15% intentional errors for testing data quality logic:

- Null `customer_id`
- Negative `quantity` or `price`
- Future `order_date`
- Invalid `country` codes
- Duplicate `transaction_id`

### Bronze Layer

| Model | Materialization | Description |
|-------|-----------------|-------------|
| `bronze_eccomerce` | View | Pass-through of raw seed data |

### Silver Layer

| Model | Materialization | Description |
|-------|-----------------|-------------|
| `silver_eccomerce` | Table | Cleans and standardizes data |

Silver transformations include:

- Casting `order_date` to date
- Normalizing invalid countries to `Unknown`
- Replacing non-positive quantities with `1` and prices with `0`
- Filtering out rows with null `customer_id` or future dates
- Deduplicating by `transaction_id` (keeps most recent order)

### Gold Layer (Star Schema)

| Model | Materialization | Description |
|-------|-----------------|-------------|
| `dim_customer` | Table | Distinct customers with name and country |
| `dim_product` | Table | Distinct products with category |
| `fct_order` | Table | Order facts with `total_amount` (`quantity * price`) |


## Prerequisites

- [Docker](https://www.docker.com/) and Docker Compose
- (Optional) Python 3.11+ with `pandas`, `numpy`, and `faker` if you want to regenerate source data locally

## Getting Started

### 1. Start the infrastructure

From the project root:

```bash
docker compose up -d --build
```

This starts:

- **PostgreSQL** on port `5432` (database: `e-commerce-dbt`)
- **dbt** container with the project mounted at `/usr/app`

### 2. Run the dbt pipeline

Execute dbt commands inside the dbt container:

```bash
# Load seed data into PostgreSQL
docker exec -it dbt_core seed --project-dir dbt_ecomm

# Build all models (bronze → silver → gold)
docker exec -it dbt_core run --project-dir dbt_ecomm

# (Optional) Run tests if defined
docker exec -it dbt_core test --project-dir dbt_ecomm
```

To open an interactive shell in the dbt container:

```bash
docker exec -it dbt_core bash
cd dbt_ecomm
dbt seed
dbt run
```

### 3. Query the results

Connect to PostgreSQL using any SQL client:

| Setting | Value |
|---------|-------|
| Host | `localhost` |
| Port | `5432` |
| Database | `e-commerce-dbt` |
| User | `eyasu` |
| Schema | `dev_schema` |

Example query:

```sql
SELECT
    c.customer_name,
    p.product_category,
    f.total_amount,
    f.order_status
FROM dev_schema.fct_order f
JOIN dev_schema.dim_customer c ON f.customer_id = c.customer_id
JOIN dev_schema.dim_product p ON f.product_id = p.product_id
LIMIT 10;
```
## Configuration

Database connection settings live in `dbt_project/.dbt/profiles.yml`. Docker Compose mounts this file into the dbt container automatically.

stgreSQL credentials are defined in `docker-compose.yml` and must match `profiles.yml`. Update both files if you change the database user or password.


> **Note:** Default credentials are intended for local development only. Change them before deploying to any shared or production environment.

## Stopping the Environment

```bash
docker compose down
```

To remove persisted database data as well:

```bash
docker compose down -v
```

## pipeline execution


<img width="1888" height="963" alt="Screenshot 2026-06-29 195106" src="https://github.com/user-attachments/assets/4788b60a-ba12-4ec6-9db3-009583785215" />
<img width="1892" height="946" alt="Screenshot 2026-06-29 195150" src="https://github.com/user-attachments/assets/2dc242f3-7a76-4276-a55d-5820dd1be41a" />
<img width="1913" height="932" alt="Screenshot 2026-06-29 195204" src="https://github.com/user-attachments/assets/d504f04f-d936-4e51-8d5a-995aee5cabb7" />
<img width="1346" height="711" alt="Screenshot 2026-06-29 194938" src="https://github.com/user-attachments/assets/3fb952fc-4c67-486f-a34b-955118f98de4" />



