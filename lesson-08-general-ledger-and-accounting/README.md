# Lesson 08: General Ledger & Accounting

## Overview

This lesson covers the General Ledger (GL) and accounting principles in core banking systems. You'll learn about double-entry bookkeeping, chart of accounts design, and how banking transactions are recorded for financial reporting.

## Learning Objectives

By the end of this lesson, you will be able to:

- Design a chart of accounts for a banking institution
- Implement double-entry accounting in banking systems
- Understand GL posting and journal entries
- Handle multi-currency accounting
- Perform reconciliation and period-end closing

## Sub-Lessons

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Chart of Accounts](./01-chart-of-accounts/README.md) | Structure, coding, hierarchy |
| 02 | [Double-Entry Bookkeeping](./02-double-entry-bookkeeping/README.md) | Debits, credits, accounting equation |
| 03 | [GL Posting Rules](./03-gl-posting-rules/README.md) | Transaction codes, posting logic |
| 04 | [Multi-Currency Accounting](./04-multi-currency-accounting/README.md) | FX revaluation, position management |
| 05 | [Period-End Processing](./05-period-end-processing/README.md) | Closing, accruals, reporting |

## Key Concepts

### Accounting Equation

```
Assets = Liabilities + Equity

In Banking:
┌─────────────────────────────────────────────────────────────┐
│                          ASSETS                              │
│  Cash + Loans + Securities + Fixed Assets + Other Assets    │
├─────────────────────────────────────────────────────────────┤
│                        LIABILITIES                           │
│  Deposits + Borrowings + Accrued Expenses + Other Liab.     │
├─────────────────────────────────────────────────────────────┤
│                          EQUITY                              │
│  Share Capital + Retained Earnings + Reserves               │
└─────────────────────────────────────────────────────────────┘
```

### Chart of Accounts Structure

| Code | Category | Type | Example |
|------|----------|------|---------|
| 1XXXX | Assets | Debit Normal | Cash, Loans, Securities |
| 2XXXX | Liabilities | Credit Normal | Deposits, Borrowings |
| 3XXXX | Equity | Credit Normal | Capital, Reserves |
| 4XXXX | Income | Credit Normal | Interest Income, Fees |
| 5XXXX | Expenses | Debit Normal | Interest Expense, Salaries |

### Chart of Accounts Hierarchy

```
1 - ASSETS
├── 10 - Cash & Cash Equivalents
│   ├── 1010 - Cash on Hand
│   ├── 1020 - Balances with Central Bank
│   └── 1030 - Nostro Accounts
├── 11 - Loans & Advances
│   ├── 1110 - Consumer Loans
│   ├── 1120 - Mortgage Loans
│   └── 1130 - Commercial Loans
└── 12 - Investments
    ├── 1210 - Government Securities
    └── 1220 - Corporate Bonds

2 - LIABILITIES
├── 20 - Customer Deposits
│   ├── 2010 - Demand Deposits
│   ├── 2020 - Savings Deposits
│   └── 2030 - Term Deposits
└── 21 - Borrowings
    └── 2110 - Interbank Borrowings
```

### Double-Entry Examples

| Transaction | Debit | Credit |
|-------------|-------|--------|
| Customer Deposit | Cash (1010) | Deposits (2010) |
| Loan Disbursement | Loans (1110) | Customer Account (2010) |
| Interest Income (Loan) | Interest Receivable (1199) | Interest Income (4110) |
| Interest Expense (Deposit) | Interest Expense (5110) | Interest Payable (2199) |
| Fee Collection | Customer Account (2010) | Fee Income (4210) |

### GL Posting Flow

```
Transaction ──► Posting Engine ──► GL Entries
                     │
                     ▼
              ┌─────────────┐
              │  Posting    │
              │   Rules     │
              └─────────────┘
                     │
         ┌───────────┼───────────┐
         ▼           ▼           ▼
    ┌─────────┐ ┌─────────┐ ┌─────────┐
    │ Account │ │ Internal│ │   GL    │
    │ Balance │ │ Account │ │  Entry  │
    └─────────┘ └─────────┘ └─────────┘
```

## Code Example: General Ledger Data Model

```sql
-- Chart of Accounts
CREATE TABLE gl_accounts (
    gl_code VARCHAR(20) PRIMARY KEY,
    gl_name VARCHAR(200) NOT NULL,
    gl_type VARCHAR(20) NOT NULL,  -- ASSET, LIABILITY, EQUITY, INCOME, EXPENSE
    parent_gl_code VARCHAR(20) REFERENCES gl_accounts(gl_code),
    level_number INTEGER NOT NULL,
    normal_balance VARCHAR(10) NOT NULL,  -- DEBIT, CREDIT
    currency VARCHAR(3),  -- NULL for multi-currency
    is_header BOOLEAN DEFAULT FALSE,
    is_posting BOOLEAN DEFAULT TRUE,
    status VARCHAR(20) DEFAULT 'ACTIVE',

    CONSTRAINT valid_gl_type CHECK (gl_type IN (
        'ASSET', 'LIABILITY', 'EQUITY', 'INCOME', 'EXPENSE'
    ))
);

-- GL Journal Entries
CREATE TABLE gl_entries (
    entry_id UUID PRIMARY KEY,
    journal_id UUID NOT NULL,
    entry_date DATE NOT NULL,
    value_date DATE NOT NULL,
    gl_code VARCHAR(20) NOT NULL REFERENCES gl_accounts(gl_code),
    branch_code VARCHAR(10) NOT NULL,
    currency VARCHAR(3) NOT NULL,
    debit_amount DECIMAL(18,2),
    credit_amount DECIMAL(18,2),
    local_currency_amount DECIMAL(18,2),  -- For reporting currency
    exchange_rate DECIMAL(18,8),
    narrative VARCHAR(500),
    transaction_reference VARCHAR(50),
    source_system VARCHAR(50),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_by VARCHAR(100),

    CONSTRAINT debit_or_credit CHECK (
        (debit_amount IS NOT NULL AND credit_amount IS NULL) OR
        (debit_amount IS NULL AND credit_amount IS NOT NULL)
    )
);

-- Journal Headers
CREATE TABLE gl_journals (
    journal_id UUID PRIMARY KEY,
    journal_number VARCHAR(30) NOT NULL UNIQUE,
    journal_type VARCHAR(30) NOT NULL,
    journal_date DATE NOT NULL,
    posting_date DATE NOT NULL,
    description VARCHAR(500),
    source_system VARCHAR(50),
    source_reference VARCHAR(50),
    total_debit DECIMAL(18,2) NOT NULL,
    total_credit DECIMAL(18,2) NOT NULL,
    status VARCHAR(20) DEFAULT 'POSTED',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    posted_at TIMESTAMP WITH TIME ZONE,
    posted_by VARCHAR(100),

    CONSTRAINT balanced_journal CHECK (total_debit = total_credit)
);

-- GL Balances (Aggregated)
CREATE TABLE gl_balances (
    gl_code VARCHAR(20) NOT NULL REFERENCES gl_accounts(gl_code),
    branch_code VARCHAR(10) NOT NULL,
    currency VARCHAR(3) NOT NULL,
    balance_date DATE NOT NULL,
    opening_balance DECIMAL(18,2) NOT NULL,
    debit_movement DECIMAL(18,2) DEFAULT 0,
    credit_movement DECIMAL(18,2) DEFAULT 0,
    closing_balance DECIMAL(18,2) NOT NULL,
    local_currency_balance DECIMAL(18,2),

    PRIMARY KEY (gl_code, branch_code, currency, balance_date)
);

-- Transaction Posting Rules
CREATE TABLE gl_posting_rules (
    rule_id UUID PRIMARY KEY,
    transaction_type VARCHAR(50) NOT NULL,
    rule_sequence INTEGER NOT NULL,
    debit_gl_code VARCHAR(20) NOT NULL REFERENCES gl_accounts(gl_code),
    credit_gl_code VARCHAR(20) NOT NULL REFERENCES gl_accounts(gl_code),
    description VARCHAR(200),
    status VARCHAR(20) DEFAULT 'ACTIVE',

    CONSTRAINT unique_rule UNIQUE (transaction_type, rule_sequence)
);
```

```json
{
  "journalEntry": {
    "journalId": "jnl-uuid-12345",
    "journalNumber": "JNL2024011500001",
    "journalType": "SYSTEM",
    "journalDate": "2024-01-15",
    "description": "Loan Disbursement - LN2024000001",
    "entries": [
      {
        "entryId": "ent-001",
        "glCode": "1110",
        "glName": "Consumer Loans",
        "debitAmount": 50000.00,
        "creditAmount": null,
        "currency": "USD",
        "narrative": "Loan disbursement to John Smith"
      },
      {
        "entryId": "ent-002",
        "glCode": "2010",
        "glName": "Demand Deposits",
        "debitAmount": null,
        "creditAmount": 50000.00,
        "currency": "USD",
        "narrative": "Credit to customer account 1234567890"
      }
    ],
    "totalDebit": 50000.00,
    "totalCredit": 50000.00,
    "status": "POSTED"
  }
}
```

### GL Balance Calculation

```typescript
interface GLBalance {
  glCode: string;
  currency: string;
  openingBalance: number;
  debitMovement: number;
  creditMovement: number;
  closingBalance: number;
}

function calculateClosingBalance(
  glType: 'ASSET' | 'LIABILITY' | 'EQUITY' | 'INCOME' | 'EXPENSE',
  opening: number,
  debits: number,
  credits: number
): number {
  // Debit-normal accounts: Assets, Expenses
  // Credit-normal accounts: Liabilities, Equity, Income

  if (glType === 'ASSET' || glType === 'EXPENSE') {
    return opening + debits - credits;
  } else {
    return opening - debits + credits;
  }
}

// Example: Deposit account (Liability - Credit Normal)
// Opening: 10,000 (credit balance)
// Debits: 2,000 (withdrawals)
// Credits: 5,000 (deposits)
// Closing: 10,000 - 2,000 + 5,000 = 13,000
```

## Key Takeaways

1. Double-entry bookkeeping ensures every transaction is balanced
2. Chart of accounts provides structure for financial reporting
3. GL posting rules automate transaction recording
4. Multi-currency accounting requires FX position management
5. Period-end closing ensures accurate financial statements

## Further Reading

- IFRS 9 Financial Instruments
- IAS 21 Effects of Changes in Foreign Exchange Rates
- Basel III Capital Adequacy requirements
- GAAP vs IFRS differences in banking

## Next Lesson

[Lesson 09: Transaction Processing](../lesson-09-transaction-processing/README.md)
