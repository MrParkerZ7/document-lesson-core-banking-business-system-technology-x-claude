# Lesson 05: Deposits & Savings

## Overview

This lesson covers deposit and savings products in core banking, including term deposits, savings accounts, and the critical calculations for interest accrual and payment. Understanding these products is fundamental to retail banking operations.

## Learning Objectives

By the end of this lesson, you will be able to:

- Design data models for various deposit products
- Implement interest calculation methods
- Understand accrual accounting for deposits
- Handle deposit maturity and rollover processes
- Configure deposit product parameters

## Sub-Lessons

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Deposit Product Types](./01-deposit-product-types/README.md) | Savings, term deposits, certificates of deposit |
| 02 | [Interest Calculation](./02-interest-calculation/README.md) | Methods, day count conventions, compounding |
| 03 | [Accrual Processing](./03-accrual-processing/README.md) | Daily accruals, month-end processing |
| 04 | [Maturity & Rollover](./04-maturity-rollover/README.md) | Maturity handling, automatic renewal |
| 05 | [Product Configuration](./05-product-configuration/README.md) | Parameters, tiers, promotional rates |

## Key Concepts

### Deposit Product Types

| Product | Characteristics | Interest Calculation |
|---------|-----------------|---------------------|
| Savings Account | Variable balance, withdrawals allowed | Daily balance, monthly credit |
| Term Deposit | Fixed amount, fixed tenure | Fixed rate, maturity payout |
| Certificate of Deposit | Negotiable, tradeable | Various structures |
| Money Market | Higher minimums, better rates | Tiered rates |
| Notice Account | Withdrawal notice required | Better rates than savings |

### Interest Calculation Methods

```
Simple Interest:
Interest = Principal × Rate × Time

Compound Interest:
A = P(1 + r/n)^(nt)
Where:
  A = Final amount
  P = Principal
  r = Annual rate
  n = Compounding frequency
  t = Time in years
```

### Day Count Conventions

| Convention | Days in Year | Use Case |
|------------|--------------|----------|
| ACT/360 | Actual/360 | Money market (US) |
| ACT/365 | Actual/365 | UK, Australia |
| ACT/ACT | Actual/Actual | Bonds, some deposits |
| 30/360 | 30-day months/360 | US corporate bonds |
| 30E/360 | European variant | European markets |

### Interest Accrual Flow

```
Daily Processing:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Get EOD   │───►│  Calculate  │───►│   Post      │
│   Balance   │    │  Daily Rate │    │   Accrual   │
└─────────────┘    └─────────────┘    └─────────────┘
                          │
                          ▼
                   ┌─────────────┐
                   │  Accrual    │
                   │  Ledger     │
                   └─────────────┘

Monthly/Maturity:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│    Sum      │───►│   Apply     │───►│  Credit to  │
│  Accruals   │    │  Withholding│    │   Account   │
└─────────────┘    └─────────────┘    └─────────────┘
```

### Term Deposit Lifecycle

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ BOOKING  │───►│  ACTIVE  │───►│ MATURING │───►│ MATURED  │
│          │    │          │    │          │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
                                      │
                      ┌───────────────┼───────────────┐
                      ▼               ▼               ▼
                ┌──────────┐   ┌──────────┐   ┌──────────┐
                │ AUTO     │   │  MANUAL  │   │  BREAK/  │
                │ ROLLOVER │   │  RENEWAL │   │  CLOSURE │
                └──────────┘   └──────────┘   └──────────┘
```

## Code Example: Deposit Product and Interest Calculation

```sql
-- Deposit Products Configuration
CREATE TABLE deposit_products (
    product_code VARCHAR(20) PRIMARY KEY,
    product_name VARCHAR(200) NOT NULL,
    product_type VARCHAR(30) NOT NULL,
    currency VARCHAR(3) NOT NULL,
    minimum_balance DECIMAL(18,2),
    maximum_balance DECIMAL(18,2),
    minimum_tenure_days INTEGER,
    maximum_tenure_days INTEGER,
    interest_calculation_method VARCHAR(30),
    day_count_convention VARCHAR(20),
    compounding_frequency VARCHAR(20),
    interest_payout_frequency VARCHAR(20),
    withholding_tax_rate DECIMAL(5,4),
    early_withdrawal_penalty_rate DECIMAL(5,4),
    status VARCHAR(20) DEFAULT 'ACTIVE',

    CONSTRAINT valid_product_type CHECK (product_type IN (
        'SAVINGS', 'TERM_DEPOSIT', 'CERTIFICATE_OF_DEPOSIT',
        'MONEY_MARKET', 'NOTICE_ACCOUNT'
    ))
);

-- Interest Rate Tiers
CREATE TABLE interest_rate_tiers (
    tier_id UUID PRIMARY KEY,
    product_code VARCHAR(20) REFERENCES deposit_products(product_code),
    tier_name VARCHAR(100),
    balance_from DECIMAL(18,2) NOT NULL,
    balance_to DECIMAL(18,2),
    tenure_from_days INTEGER,
    tenure_to_days INTEGER,
    interest_rate DECIMAL(7,4) NOT NULL,  -- Annual rate
    effective_from DATE NOT NULL,
    effective_to DATE,

    CONSTRAINT valid_tier_range CHECK (balance_from < COALESCE(balance_to, balance_from + 1))
);

-- Term Deposits
CREATE TABLE term_deposits (
    deposit_id UUID PRIMARY KEY,
    account_id UUID NOT NULL REFERENCES accounts(account_id),
    product_code VARCHAR(20) NOT NULL REFERENCES deposit_products(product_code),
    principal_amount DECIMAL(18,2) NOT NULL,
    interest_rate DECIMAL(7,4) NOT NULL,
    tenure_days INTEGER NOT NULL,
    start_date DATE NOT NULL,
    maturity_date DATE NOT NULL,
    maturity_instruction VARCHAR(20) NOT NULL,
    maturity_account_id UUID REFERENCES accounts(account_id),
    status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE',
    accrued_interest DECIMAL(18,2) DEFAULT 0,
    paid_interest DECIMAL(18,2) DEFAULT 0,

    CONSTRAINT valid_maturity_instruction CHECK (maturity_instruction IN (
        'ROLLOVER_PRINCIPAL_INTEREST', 'ROLLOVER_PRINCIPAL',
        'CREDIT_TO_ACCOUNT', 'HOLD_FOR_INSTRUCTION'
    ))
);

-- Interest Accruals
CREATE TABLE interest_accruals (
    accrual_id UUID PRIMARY KEY,
    account_id UUID NOT NULL REFERENCES accounts(account_id),
    deposit_id UUID REFERENCES term_deposits(deposit_id),
    accrual_date DATE NOT NULL,
    principal_balance DECIMAL(18,2) NOT NULL,
    interest_rate DECIMAL(7,4) NOT NULL,
    accrued_amount DECIMAL(18,6) NOT NULL,
    status VARCHAR(20) DEFAULT 'PENDING',
    capitalized_date DATE,

    CONSTRAINT unique_daily_accrual UNIQUE (account_id, deposit_id, accrual_date)
);
```

```json
{
  "termDeposit": {
    "depositId": "td-uuid-12345",
    "accountId": "acc-uuid-12345",
    "productCode": "TD_12M_STANDARD",
    "principal": 50000.00,
    "currency": "USD",
    "interestRate": 4.50,
    "tenureDays": 365,
    "startDate": "2024-01-15",
    "maturityDate": "2025-01-15",
    "status": "ACTIVE",
    "maturityInstruction": "ROLLOVER_PRINCIPAL_INTEREST",
    "interestSummary": {
      "expectedInterest": 2250.00,
      "accruedToDate": 1125.00,
      "paidToDate": 0.00,
      "withholdingTax": 225.00
    },
    "dayCountConvention": "ACT/365",
    "compoundingFrequency": "ANNUAL"
  }
}
```

### Interest Calculation Example (TypeScript)

```typescript
interface InterestCalculation {
  principal: number;
  rate: number;  // Annual rate as decimal
  days: number;
  dayCountConvention: 'ACT/360' | 'ACT/365' | '30/360';
}

function calculateSimpleInterest(params: InterestCalculation): number {
  const daysInYear = params.dayCountConvention === 'ACT/360' ? 360 : 365;
  return params.principal * params.rate * (params.days / daysInYear);
}

function calculateDailyAccrual(
  balance: number,
  annualRate: number,
  dayCountConvention: string
): number {
  const daysInYear = dayCountConvention === 'ACT/360' ? 360 : 365;
  const dailyRate = annualRate / daysInYear;
  return balance * dailyRate;
}

// Example:
// $50,000 at 4.5% for 365 days (ACT/365)
// Daily accrual = 50000 * (0.045 / 365) = $6.16
// Total interest = $2,250.00
```

## Key Takeaways

1. Different deposit products have distinct characteristics and rate structures
2. Day count conventions significantly impact interest calculations
3. Accrual accounting ensures proper recognition of interest expense
4. Maturity handling requires clear customer instructions
5. Interest rate tiers enable competitive pricing strategies

## Further Reading

- IFRS 9 Financial Instruments for deposit accounting
- Basel III Liquidity Coverage Ratio (LCR) impact on deposits
- Central bank deposit insurance regulations
- Interest rate risk management in banking book

## Next Lesson

[Lesson 06: Loans & Credit](../lesson-06-loans-and-credit/README.md)
