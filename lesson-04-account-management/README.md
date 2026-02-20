# Lesson 04: Account Management

## Overview

This lesson covers the fundamental concepts of account management in core banking systems, including account structures, lifecycle management, and the various operations that can be performed on accounts.

## Learning Objectives

By the end of this lesson, you will be able to:

- Design account data models supporting multiple product types
- Implement account lifecycle management
- Understand account hierarchies and relationships
- Handle multi-currency accounts
- Manage account operations and restrictions

## Sub-Lessons

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Account Types & Structures](./01-account-types-structures/README.md) | Classification, attributes, configurations |
| 02 | [Account Lifecycle](./02-account-lifecycle/README.md) | Opening, maintenance, closure processes |
| 03 | [Account Hierarchies](./03-account-hierarchies/README.md) | Parent-child, pooling, sweeping |
| 04 | [Multi-Currency Accounts](./04-multi-currency-accounts/README.md) | FX handling, position management |
| 05 | [Account Operations](./05-account-operations/README.md) | Blocks, holds, mandates, dormancy |

## Key Concepts

### Account Structure

```
┌─────────────────────────────────────────────────────────────┐
│                      ACCOUNT                                 │
├─────────────────────────────────────────────────────────────┤
│  Account Number: 1234-5678-9012-3456                        │
│  IBAN: US12 BANK 1234 5678 9012 3456                        │
│  Product: PREMIUM_SAVINGS                                    │
│  Status: ACTIVE                                              │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Balance   │  │   Limits    │  │   Interest  │         │
│  │  Management │  │  & Blocks   │  │   Config    │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  Ownership  │  │  Mandates   │  │  Statements │         │
│  │  & Signing  │  │  & Powers   │  │  & Comms    │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### Account Types

| Category | Types | Characteristics |
|----------|-------|-----------------|
| Deposit Accounts | Savings, Current, Term | Customer-facing, interest-bearing/paying |
| Loan Accounts | Personal, Mortgage, Business | Liability tracking, amortization |
| Internal Accounts | GL, Suspense, Clearing | System operations, reconciliation |
| Nostro/Vostro | Correspondent banking | Interbank positions |

### Account Lifecycle

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  DRAFT   │───►│ PENDING  │───►│  ACTIVE  │───►│  CLOSED  │
│          │    │ APPROVAL │    │          │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
                                      │
                                      ▼
                               ┌──────────┐
                               │ DORMANT  │
                               │          │
                               └──────────┘
                                      │
                               ┌──────┴──────┐
                               ▼             ▼
                         ┌──────────┐  ┌──────────┐
                         │REACTIVATE│  │ESCHEATMENT│
                         └──────────┘  └──────────┘
```

### Balance Types

| Balance Type | Description | Use Case |
|--------------|-------------|----------|
| Ledger Balance | Actual posted balance | Financial reporting |
| Available Balance | Funds available for use | Transaction authorization |
| Hold Amount | Funds reserved but not debited | Pending transactions |
| Minimum Balance | Required minimum | Fee/interest calculation |
| Overdraft Limit | Allowed negative balance | Credit facility |

### Account Number Structure

```
Branch Code   Product Code   Sequence   Check Digit
   ├───┤         ├──┤         ├────┤       ├─┤
   1234    -     56     -    789012   -     3

IBAN Format:
Country | Check | Bank | Branch | Account Number
  US       12     BANK    1234    567890123456
```

## Code Example: Account Data Model

```sql
-- Core Account Table
CREATE TABLE accounts (
    account_id UUID PRIMARY KEY,
    account_number VARCHAR(30) NOT NULL UNIQUE,
    iban VARCHAR(34) UNIQUE,
    cif_number VARCHAR(20) NOT NULL REFERENCES customers(cif_number),
    product_code VARCHAR(20) NOT NULL REFERENCES products(product_code),
    branch_code VARCHAR(10) NOT NULL,
    currency VARCHAR(3) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING_ACTIVATION',
    opened_date DATE,
    closed_date DATE,
    last_transaction_date DATE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    version INTEGER DEFAULT 1,

    CONSTRAINT valid_status CHECK (status IN (
        'DRAFT', 'PENDING_ACTIVATION', 'ACTIVE',
        'DORMANT', 'BLOCKED', 'CLOSED'
    ))
);

-- Account Balances
CREATE TABLE account_balances (
    account_id UUID NOT NULL REFERENCES accounts(account_id),
    balance_type VARCHAR(20) NOT NULL,
    amount DECIMAL(18,2) NOT NULL DEFAULT 0,
    as_of_date DATE NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    PRIMARY KEY (account_id, balance_type, as_of_date),

    CONSTRAINT valid_balance_type CHECK (balance_type IN (
        'LEDGER', 'AVAILABLE', 'HOLD', 'MINIMUM', 'OVERDRAFT_LIMIT'
    ))
);

-- Account Ownership (supports joint accounts)
CREATE TABLE account_ownership (
    account_id UUID NOT NULL REFERENCES accounts(account_id),
    cif_number VARCHAR(20) NOT NULL REFERENCES customers(cif_number),
    ownership_type VARCHAR(20) NOT NULL,
    ownership_percentage DECIMAL(5,2),
    signing_authority VARCHAR(20),
    effective_from DATE NOT NULL,
    effective_to DATE,

    PRIMARY KEY (account_id, cif_number, effective_from),

    CONSTRAINT valid_ownership_type CHECK (ownership_type IN (
        'PRIMARY', 'JOINT', 'AUTHORIZED_SIGNATORY', 'BENEFICIARY'
    ))
);

-- Account Blocks/Holds
CREATE TABLE account_blocks (
    block_id UUID PRIMARY KEY,
    account_id UUID NOT NULL REFERENCES accounts(account_id),
    block_type VARCHAR(30) NOT NULL,
    block_reason VARCHAR(500),
    amount DECIMAL(18,2),  -- NULL for full account block
    effective_from TIMESTAMP WITH TIME ZONE NOT NULL,
    effective_to TIMESTAMP WITH TIME ZONE,
    placed_by VARCHAR(100) NOT NULL,
    reference_number VARCHAR(50),

    CONSTRAINT valid_block_type CHECK (block_type IN (
        'DEBIT_BLOCK', 'CREDIT_BLOCK', 'FULL_BLOCK',
        'LIEN', 'GARNISHMENT', 'FRAUD_HOLD'
    ))
);
```

```json
{
  "account": {
    "accountId": "acc-uuid-12345",
    "accountNumber": "1234567890123456",
    "iban": "US12BANK1234567890123456",
    "cifNumber": "CIF-2024-000001",
    "productCode": "PREMIUM_SAVINGS",
    "branchCode": "NYC001",
    "currency": "USD",
    "status": "ACTIVE",
    "openedDate": "2024-01-15",
    "balances": {
      "ledger": 15000.00,
      "available": 14500.00,
      "hold": 500.00,
      "minimumRequired": 1000.00
    },
    "ownership": [
      {
        "cifNumber": "CIF-2024-000001",
        "type": "PRIMARY",
        "percentage": 100.00,
        "signingAuthority": "SINGLE"
      }
    ],
    "activeBlocks": []
  }
}
```

## Key Takeaways

1. Account numbers follow structured formats enabling routing and validation
2. Multiple balance types serve different operational needs
3. Account lifecycle includes dormancy handling and regulatory requirements
4. Joint accounts require proper ownership and signing authority modeling
5. Blocks and holds are essential for risk management and compliance

## Further Reading

- ISO 13616 (IBAN structure)
- Account portability regulations
- Dormant account handling regulations
- Know Your Account (KYA) best practices

## Next Lesson

[Lesson 05: Deposits & Savings](../lesson-05-deposits-and-savings/README.md)
