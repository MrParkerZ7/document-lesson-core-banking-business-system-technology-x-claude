# Lesson 09: Transaction Processing

## Overview

This lesson covers transaction processing in core banking systems, including real-time and batch processing, End of Day (EOD) operations, and transaction management patterns. You'll learn how banks process millions of transactions reliably.

## Learning Objectives

By the end of this lesson, you will be able to:

- Design real-time transaction processing systems
- Implement batch processing and scheduling
- Understand EOD/EOM/EOY processing cycles
- Handle transaction reversal and correction
- Ensure transaction integrity and consistency

## Sub-Lessons

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Real-Time Processing](./01-real-time-processing/README.md) | OLTP, synchronous operations |
| 02 | [Batch Processing](./02-batch-processing/README.md) | Scheduling, job management |
| 03 | [EOD Operations](./03-eod-operations/README.md) | Interest, charges, statements |
| 04 | [Transaction Lifecycle](./04-transaction-lifecycle/README.md) | States, reversals, amendments |
| 05 | [Error Handling](./05-error-handling/README.md) | Recovery, reconciliation |

## Key Concepts

### Processing Modes

| Mode | Characteristics | Use Cases |
|------|-----------------|-----------|
| Real-Time (OLTP) | Immediate, synchronous | Transfers, withdrawals, payments |
| Near Real-Time | Seconds delay, async | Notifications, fraud checks |
| Batch | Scheduled, bulk | Interest posting, statements |
| Event-Driven | Triggered by events | Account updates, alerts |

### Transaction Processing Flow

```
┌──────────────┐
│   INITIATE   │  Create transaction request
└──────┬───────┘
       ▼
┌──────────────┐
│   VALIDATE   │  Business rules, limits, AML
└──────┬───────┘
       ▼
┌──────────────┐
│  AUTHORIZE   │  Balance check, approval
└──────┬───────┘
       ▼
┌──────────────┐
│   EXECUTE    │  Debit/Credit accounts
└──────┬───────┘
       ▼
┌──────────────┐
│     POST     │  GL entries, update balances
└──────┬───────┘
       ▼
┌──────────────┐
│   COMPLETE   │  Confirmations, notifications
└──────────────┘
```

### End of Day (EOD) Processing

```
┌─────────────────────────────────────────────────────────────┐
│                    EOD PROCESSING SEQUENCE                   │
├─────────────────────────────────────────────────────────────┤
│  1. Cut-off Processing                                       │
│     ├── Stop real-time transactions                         │
│     ├── Queue pending transactions                          │
│     └── Set business date                                   │
├─────────────────────────────────────────────────────────────┤
│  2. Interest Accrual                                         │
│     ├── Calculate daily interest (deposits)                 │
│     ├── Calculate daily interest (loans)                    │
│     └── Post accrual entries                                │
├─────────────────────────────────────────────────────────────┤
│  3. Charges & Fees                                           │
│     ├── Calculate account maintenance fees                  │
│     ├── Apply penalty charges                               │
│     └── Process service charges                             │
├─────────────────────────────────────────────────────────────┤
│  4. Balance Updates                                          │
│     ├── Calculate closing balances                          │
│     ├── Update GL positions                                 │
│     └── Archive daily data                                  │
├─────────────────────────────────────────────────────────────┤
│  5. Statement Generation                                     │
│     ├── Generate daily statements                           │
│     └── Queue statement delivery                            │
├─────────────────────────────────────────────────────────────┤
│  6. Reporting                                                │
│     ├── Generate regulatory reports                         │
│     ├── Create management reports                           │
│     └── Update data warehouse                               │
├─────────────────────────────────────────────────────────────┤
│  7. Housekeeping                                             │
│     ├── Archive old data                                    │
│     ├── Clean temporary files                               │
│     └── Advance business date                               │
└─────────────────────────────────────────────────────────────┘
```

### Transaction States

```
┌──────────┐
│ INITIATED│
└────┬─────┘
     │
     ▼
┌──────────┐    ┌──────────┐
│ PENDING  │───►│ REJECTED │
└────┬─────┘    └──────────┘
     │
     ▼
┌──────────┐    ┌──────────┐
│AUTHORIZED│───►│ DECLINED │
└────┬─────┘    └──────────┘
     │
     ▼
┌──────────┐    ┌──────────┐
│ EXECUTING│───►│  FAILED  │
└────┬─────┘    └──────────┘
     │
     ▼
┌──────────┐
│ COMPLETED│──────────────────┐
└────┬─────┘                  │
     │                        ▼
     │               ┌──────────────┐
     │               │   REVERSED   │
     │               └──────────────┘
     ▼
┌──────────┐
│ SETTLED  │
└──────────┘
```

## Code Example: Transaction Processing

```sql
-- Transactions Table
CREATE TABLE transactions (
    transaction_id UUID PRIMARY KEY,
    transaction_reference VARCHAR(50) NOT NULL UNIQUE,
    transaction_type VARCHAR(50) NOT NULL,
    transaction_subtype VARCHAR(50),
    status VARCHAR(30) NOT NULL DEFAULT 'INITIATED',

    -- Accounts
    debit_account_id UUID NOT NULL REFERENCES accounts(account_id),
    credit_account_id UUID NOT NULL REFERENCES accounts(account_id),

    -- Amounts
    amount DECIMAL(18,2) NOT NULL,
    currency VARCHAR(3) NOT NULL,

    -- Processing Details
    value_date DATE NOT NULL,
    posting_date DATE NOT NULL,
    narrative VARCHAR(500),
    channel VARCHAR(30) NOT NULL,

    -- Audit
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_by VARCHAR(100) NOT NULL,
    processed_at TIMESTAMP WITH TIME ZONE,
    version INTEGER DEFAULT 1,

    CONSTRAINT valid_status CHECK (status IN (
        'INITIATED', 'PENDING', 'AUTHORIZED', 'EXECUTING',
        'COMPLETED', 'SETTLED', 'REJECTED', 'DECLINED',
        'FAILED', 'REVERSED', 'CANCELLED'
    ))
);

-- Transaction Status History
CREATE TABLE transaction_history (
    history_id UUID PRIMARY KEY,
    transaction_id UUID NOT NULL REFERENCES transactions(transaction_id),
    status VARCHAR(30) NOT NULL,
    status_reason VARCHAR(500),
    status_timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_by VARCHAR(100)
);

-- EOD Batch Jobs
CREATE TABLE batch_jobs (
    job_id UUID PRIMARY KEY,
    job_name VARCHAR(100) NOT NULL,
    job_type VARCHAR(50) NOT NULL,
    business_date DATE NOT NULL,
    sequence_number INTEGER NOT NULL,
    status VARCHAR(30) NOT NULL DEFAULT 'PENDING',
    started_at TIMESTAMP WITH TIME ZONE,
    completed_at TIMESTAMP WITH TIME ZONE,
    records_processed INTEGER DEFAULT 0,
    records_failed INTEGER DEFAULT 0,
    error_message TEXT,

    CONSTRAINT unique_job_sequence UNIQUE (business_date, job_name, sequence_number)
);

-- Batch Job Dependencies
CREATE TABLE batch_job_dependencies (
    job_id UUID NOT NULL REFERENCES batch_jobs(job_id),
    depends_on_job_id UUID NOT NULL REFERENCES batch_jobs(job_id),

    PRIMARY KEY (job_id, depends_on_job_id)
);
```

```json
{
  "transaction": {
    "transactionId": "txn-uuid-12345",
    "transactionReference": "TXN2024011500001",
    "transactionType": "INTERNAL_TRANSFER",
    "status": "COMPLETED",
    "debitAccount": {
      "accountId": "acc-uuid-001",
      "accountNumber": "1234567890"
    },
    "creditAccount": {
      "accountId": "acc-uuid-002",
      "accountNumber": "0987654321"
    },
    "amount": 5000.00,
    "currency": "USD",
    "valueDate": "2024-01-15",
    "postingDate": "2024-01-15",
    "narrative": "Transfer to savings",
    "channel": "MOBILE_APP",
    "timestamps": {
      "initiated": "2024-01-15T10:30:00Z",
      "authorized": "2024-01-15T10:30:01Z",
      "completed": "2024-01-15T10:30:02Z"
    }
  }
}
```

### Transaction Processing with Saga Pattern

```typescript
interface TransactionStep {
  name: string;
  execute: () => Promise<void>;
  compensate: () => Promise<void>;
}

class TransactionSaga {
  private steps: TransactionStep[] = [];
  private executedSteps: TransactionStep[] = [];

  addStep(step: TransactionStep): void {
    this.steps.push(step);
  }

  async execute(): Promise<void> {
    try {
      for (const step of this.steps) {
        await step.execute();
        this.executedSteps.push(step);
      }
    } catch (error) {
      await this.compensate();
      throw error;
    }
  }

  private async compensate(): Promise<void> {
    // Execute compensating actions in reverse order
    for (const step of this.executedSteps.reverse()) {
      try {
        await step.compensate();
      } catch (compensateError) {
        // Log and continue with other compensations
        console.error(`Compensation failed for ${step.name}`, compensateError);
      }
    }
  }
}

// Usage
const transferSaga = new TransactionSaga();

transferSaga.addStep({
  name: 'validateTransaction',
  execute: async () => { /* validate */ },
  compensate: async () => { /* no-op */ }
});

transferSaga.addStep({
  name: 'debitAccount',
  execute: async () => { /* debit source */ },
  compensate: async () => { /* credit back source */ }
});

transferSaga.addStep({
  name: 'creditAccount',
  execute: async () => { /* credit destination */ },
  compensate: async () => { /* debit destination */ }
});

await transferSaga.execute();
```

## Key Takeaways

1. Real-time processing requires low latency and high availability
2. Batch processing handles bulk operations efficiently
3. EOD processing is critical for accurate financial reporting
4. Transaction states must be clearly defined and tracked
5. Saga pattern enables reliable distributed transactions

## Further Reading

- "Transaction Processing: Concepts and Techniques" by Jim Gray
- Enterprise Integration Patterns
- Saga Pattern documentation
- Bank reconciliation best practices

## Next Lesson

[Lesson 10: Integration & APIs](../lesson-10-integration-and-apis/README.md)
