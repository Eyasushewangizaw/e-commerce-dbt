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
