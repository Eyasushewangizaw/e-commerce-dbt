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

