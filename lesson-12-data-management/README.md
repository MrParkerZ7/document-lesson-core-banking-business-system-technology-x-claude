# Lesson 12: Data Management

## Overview

This lesson covers data management strategies in core banking systems, including data modeling, data warehousing, analytics, and data governance. You'll learn how to design data architectures that support both operational and analytical needs.

## Learning Objectives

By the end of this lesson, you will be able to:

- Design data models for banking systems
- Implement data warehousing solutions
- Build reporting and analytics capabilities
- Establish data governance practices
- Handle data migration and archival

## Sub-Lessons

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Data Modeling](./01-data-modeling/README.md) | Relational, dimensional, NoSQL models |
| 02 | [Data Warehousing](./02-data-warehousing/README.md) | ETL, star schema, data lakes |
| 03 | [Analytics & Reporting](./03-analytics-reporting/README.md) | BI, dashboards, regulatory reports |
| 04 | [Data Governance](./04-data-governance/README.md) | Quality, lineage, master data |
| 05 | [Data Migration](./05-data-migration/README.md) | Strategies, validation, cutover |

## Key Concepts

### Data Architecture Layers

```
┌─────────────────────────────────────────────────────────────┐
│                   PRESENTATION LAYER                         │
│        (Dashboards, Reports, Self-Service BI)               │
├─────────────────────────────────────────────────────────────┤
│                    SEMANTIC LAYER                            │
│     (Business Definitions, Metrics, Aggregations)           │
├─────────────────────────────────────────────────────────────┤
│                   DATA MART LAYER                            │
│    (Customer, Product, Risk, Finance, Compliance)           │
├─────────────────────────────────────────────────────────────┤
│                  DATA WAREHOUSE LAYER                        │
│       (Integrated, Historical, Subject-Oriented)            │
├─────────────────────────────────────────────────────────────┤
│                   DATA LAKE LAYER                            │
│        (Raw, Structured, Semi-Structured Data)              │
├─────────────────────────────────────────────────────────────┤
│                    SOURCE SYSTEMS                            │
│  (Core Banking, Payments, Cards, Loans, Channels)           │
└─────────────────────────────────────────────────────────────┘
```

### Data Modeling Approaches

| Approach | Use Case | Characteristics |
|----------|----------|-----------------|
| 3NF (Normalized) | OLTP systems | Minimal redundancy, fast writes |
| Star Schema | Data warehouse | Optimized reads, simple joins |
| Data Vault | Enterprise DW | Flexibility, auditability |
| NoSQL Document | Flexible schemas | Schema-less, horizontal scaling |
| Time Series | Transaction history | Time-based queries, compression |

### Star Schema for Banking

```
                    ┌─────────────────┐
                    │  DIM_DATE       │
                    │                 │
                    │  date_key       │
                    │  full_date      │
                    │  day_of_week    │
                    │  month          │
                    │  quarter        │
                    │  year           │
                    └────────┬────────┘
                             │
┌─────────────────┐         │         ┌─────────────────┐
│  DIM_ACCOUNT    │         │         │  DIM_PRODUCT    │
│                 │         │         │                 │
│  account_key    │    ┌────┴────┐    │  product_key    │
│  account_number │    │  FACT_  │    │  product_code   │
│  account_type   │───►│TRANSACTIONS│◄──│  product_name   │
│  open_date      │    │         │    │  product_type   │
│  status         │    │ txn_key │    │  category       │
└─────────────────┘    │ date_key│    └─────────────────┘
                       │ acct_key│
                       │ prod_key│
┌─────────────────┐    │ cust_key│    ┌─────────────────┐
│  DIM_CUSTOMER   │    │ amount  │    │  DIM_CHANNEL    │
│                 │    │ txn_type│    │                 │
│  customer_key   │───►│ channel │◄───│  channel_key    │
│  cif_number     │    │         │    │  channel_name   │
│  customer_name  │    └─────────┘    │  channel_type   │
│  segment        │                   │                 │
└─────────────────┘                   └─────────────────┘
```

### ETL Pipeline

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   EXTRACT    │───►│  TRANSFORM   │───►│    LOAD      │
│              │    │              │    │              │
│ Core Banking │    │ Clean        │    │ Data         │
│ Payments     │    │ Deduplicate  │    │ Warehouse    │
│ Channels     │    │ Conform      │    │              │
│ External     │    │ Enrich       │    │ Data Marts   │
└──────────────┘    └──────────────┘    └──────────────┘
       │                   │                   │
       ▼                   ▼                   ▼
┌──────────────────────────────────────────────────────┐
│                    ORCHESTRATION                      │
│           (Scheduling, Monitoring, Alerting)         │
└──────────────────────────────────────────────────────┘
```

### Data Quality Dimensions

| Dimension | Description | Example Check |
|-----------|-------------|---------------|
| Completeness | No missing values | Account must have customer |
| Accuracy | Data reflects reality | Balance matches transactions |
| Consistency | Same across systems | Customer name matches |
| Timeliness | Data is current | EOD balance is today's |
| Uniqueness | No duplicates | One CIF per customer |
| Validity | Conforms to rules | Valid email format |

## Code Example: Data Warehouse Schema

```sql
-- Dimension: Customer
CREATE TABLE dim_customer (
    customer_key SERIAL PRIMARY KEY,
    cif_number VARCHAR(20) NOT NULL,
    customer_name VARCHAR(500) NOT NULL,
    customer_type VARCHAR(20) NOT NULL,
    segment VARCHAR(50),
    risk_rating VARCHAR(20),
    onboarding_date DATE,
    status VARCHAR(20),
    -- SCD Type 2 columns
    effective_from DATE NOT NULL,
    effective_to DATE,
    is_current BOOLEAN DEFAULT TRUE,
    source_system VARCHAR(50),
    loaded_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Dimension: Account
CREATE TABLE dim_account (
    account_key SERIAL PRIMARY KEY,
    account_id VARCHAR(50) NOT NULL,
    account_number VARCHAR(30) NOT NULL,
    account_type VARCHAR(30) NOT NULL,
    product_code VARCHAR(20),
    currency VARCHAR(3) NOT NULL,
    branch_code VARCHAR(10),
    open_date DATE,
    close_date DATE,
    status VARCHAR(20),
    -- SCD Type 2 columns
    effective_from DATE NOT NULL,
    effective_to DATE,
    is_current BOOLEAN DEFAULT TRUE
);

-- Dimension: Date
CREATE TABLE dim_date (
    date_key INTEGER PRIMARY KEY,  -- YYYYMMDD
    full_date DATE NOT NULL UNIQUE,
    day_of_week INTEGER NOT NULL,
    day_name VARCHAR(10) NOT NULL,
    day_of_month INTEGER NOT NULL,
    day_of_year INTEGER NOT NULL,
    week_of_year INTEGER NOT NULL,
    month_number INTEGER NOT NULL,
    month_name VARCHAR(10) NOT NULL,
    quarter INTEGER NOT NULL,
    year INTEGER NOT NULL,
    is_weekend BOOLEAN NOT NULL,
    is_holiday BOOLEAN DEFAULT FALSE,
    fiscal_year INTEGER,
    fiscal_quarter INTEGER
);

-- Fact: Daily Balances
CREATE TABLE fact_daily_balance (
    balance_key BIGSERIAL PRIMARY KEY,
    date_key INTEGER NOT NULL REFERENCES dim_date(date_key),
    account_key INTEGER NOT NULL REFERENCES dim_account(account_key),
    customer_key INTEGER NOT NULL REFERENCES dim_customer(customer_key),
    opening_balance DECIMAL(18,2) NOT NULL,
    closing_balance DECIMAL(18,2) NOT NULL,
    total_credits DECIMAL(18,2) DEFAULT 0,
    total_debits DECIMAL(18,2) DEFAULT 0,
    transaction_count INTEGER DEFAULT 0,
    interest_accrued DECIMAL(18,2) DEFAULT 0,
    average_balance DECIMAL(18,2),

    CONSTRAINT unique_daily_balance UNIQUE (date_key, account_key)
);

-- Fact: Transactions
CREATE TABLE fact_transaction (
    transaction_key BIGSERIAL PRIMARY KEY,
    date_key INTEGER NOT NULL REFERENCES dim_date(date_key),
    account_key INTEGER NOT NULL REFERENCES dim_account(account_key),
    customer_key INTEGER NOT NULL REFERENCES dim_customer(customer_key),
    product_key INTEGER REFERENCES dim_product(product_key),
    channel_key INTEGER REFERENCES dim_channel(channel_key),
    transaction_id VARCHAR(50) NOT NULL,
    transaction_type VARCHAR(30) NOT NULL,
    transaction_amount DECIMAL(18,2) NOT NULL,
    currency VARCHAR(3) NOT NULL,
    local_currency_amount DECIMAL(18,2),
    exchange_rate DECIMAL(18,8),
    transaction_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    running_balance DECIMAL(18,2),
    source_system VARCHAR(50)
);

-- Aggregate: Monthly Summary
CREATE TABLE agg_monthly_account_summary (
    summary_key BIGSERIAL PRIMARY KEY,
    year_month INTEGER NOT NULL,  -- YYYYMM
    account_key INTEGER NOT NULL REFERENCES dim_account(account_key),
    customer_key INTEGER NOT NULL REFERENCES dim_customer(customer_key),
    month_start_balance DECIMAL(18,2),
    month_end_balance DECIMAL(18,2),
    average_balance DECIMAL(18,2),
    minimum_balance DECIMAL(18,2),
    maximum_balance DECIMAL(18,2),
    total_credits DECIMAL(18,2),
    total_debits DECIMAL(18,2),
    credit_count INTEGER,
    debit_count INTEGER,
    interest_earned DECIMAL(18,2),
    fees_charged DECIMAL(18,2),

    CONSTRAINT unique_monthly_summary UNIQUE (year_month, account_key)
);
```

### Data Lineage Tracking

```json
{
  "dataLineage": {
    "target": {
      "system": "Data Warehouse",
      "table": "fact_transaction",
      "column": "transaction_amount"
    },
    "source": {
      "system": "Core Banking",
      "table": "transactions",
      "column": "amount"
    },
    "transformations": [
      {
        "type": "CAST",
        "description": "Convert to DECIMAL(18,2)"
      },
      {
        "type": "ABSOLUTE",
        "description": "Take absolute value for debit amounts"
      }
    ],
    "quality_rules": [
      {
        "rule": "NOT_NULL",
        "severity": "ERROR"
      },
      {
        "rule": "POSITIVE_VALUE",
        "severity": "WARNING"
      }
    ],
    "last_updated": "2024-01-15T06:00:00Z",
    "schedule": "DAILY_EOD"
  }
}
```

### Regulatory Reporting Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│                   REGULATORY REPORTS                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Basel     │  │   IFRS 9    │  │   AML/      │         │
│  │   Reports   │  │   Reports   │  │   CTR       │         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │
│         │                │                │                 │
│         ▼                ▼                ▼                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              REGULATORY DATA MART                    │   │
│  │   (Pre-calculated metrics, validated data)          │   │
│  └─────────────────────────────────────────────────────┘   │
│                          │                                  │
│                          ▼                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              ENTERPRISE DATA WAREHOUSE               │   │
│  │   (Integrated, reconciled, historical data)         │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Key Takeaways

1. Data architecture must support both operational and analytical workloads
2. Star schema optimizes analytical queries while maintaining simplicity
3. Data quality is foundational for regulatory compliance and business decisions
4. ETL pipelines require robust orchestration and monitoring
5. Data governance ensures consistent definitions and trusted data

## Further Reading

- "The Data Warehouse Toolkit" by Ralph Kimball
- BCBS 239 - Principles for Risk Data Aggregation
- DAMA Data Management Body of Knowledge
- Data Mesh Principles

## Conclusion

Congratulations on completing the Core Banking Business System Technology curriculum! You now have a comprehensive understanding of:

- Core banking fundamentals and architecture
- Customer and account management
- Deposit and lending operations
- Payment processing and settlement
- General ledger and accounting
- Transaction processing patterns
- API integration and open banking
- Security and compliance requirements
- Data management and analytics

Continue learning by exploring the further reading resources in each lesson and applying these concepts to real-world banking technology projects.
