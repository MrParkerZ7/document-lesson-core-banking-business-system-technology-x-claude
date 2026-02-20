# Lesson 07: Payments & Transfers

## Overview

This lesson covers payment systems and fund transfer mechanisms in core banking. You'll learn about domestic and international payment rails, clearing and settlement processes, and how modern payment infrastructures work.

## Learning Objectives

By the end of this lesson, you will be able to:

- Understand different payment types and their characteristics
- Explain clearing and settlement processes
- Design payment processing workflows
- Integrate with payment networks (SWIFT, RTGS, ACH)
- Handle payment exceptions and reconciliation

## Sub-Lessons

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Payment Types](./01-payment-types/README.md) | Internal, domestic, international transfers |
| 02 | [Payment Rails](./02-payment-rails/README.md) | RTGS, ACH, SWIFT, card networks |
| 03 | [Clearing & Settlement](./03-clearing-settlement/README.md) | Netting, settlement cycles, finality |
| 04 | [Payment Processing](./04-payment-processing/README.md) | Validation, routing, execution |
| 05 | [Reconciliation](./05-reconciliation/README.md) | Nostro/Vostro, exceptions handling |

## Key Concepts

### Payment Types

| Type | Description | Settlement |
|------|-------------|------------|
| Internal Transfer | Between accounts in same bank | Immediate |
| Domestic Wire | Same-day domestic transfer | Same day (RTGS) |
| ACH Transfer | Batch clearing | 1-2 business days |
| International Wire | Cross-border transfer | 1-5 business days |
| Instant Payment | Real-time domestic | Seconds |
| Card Payment | Debit/credit card | 1-3 business days |

### Payment Flow

```
┌──────────────┐
│   INITIATE   │  Customer/System initiates payment
└──────┬───────┘
       ▼
┌──────────────┐
│   VALIDATE   │  AML, sanctions, balance, limits
└──────┬───────┘
       ▼
┌──────────────┐
│    ROUTE     │  Determine payment rail/method
└──────┬───────┘
       ▼
┌──────────────┐
│   EXECUTE    │  Debit sender, credit receiver/nostro
└──────┬───────┘
       ▼
┌──────────────┐
│    CLEAR     │  Exchange with other banks/networks
└──────┬───────┘
       ▼
┌──────────────┐
│   SETTLE     │  Final settlement of funds
└──────┬───────┘
       ▼
┌──────────────┐
│  RECONCILE   │  Match and confirm all entries
└──────────────┘
```

### Payment Rails

| Rail | Type | Speed | Use Case |
|------|------|-------|----------|
| RTGS | Real-time gross | Immediate | High-value domestic |
| ACH | Batch net | T+1/T+2 | Payroll, bills |
| SWIFT | Messaging | T+1 to T+5 | International |
| FedNow/Faster Payments | Real-time | Seconds | Instant domestic |
| SEPA | EU zone | T+1 | Euro transfers |
| Card Networks | Authorization/Settlement | T+1 to T+3 | Card transactions |

### SWIFT Message Types

| MT Code | Description | Use |
|---------|-------------|-----|
| MT103 | Single Customer Credit Transfer | Standard wire |
| MT202 | Bank-to-Bank Transfer | Cover payments |
| MT199 | Free Format Message | Ad-hoc communication |
| MT940 | Customer Statement | EOD statements |
| MT950 | Statement Message | Nostro reconciliation |

### ISO 20022 Migration

```
Legacy SWIFT MT ────────────► ISO 20022 (MX)

MT103 (Text-based)           pacs.008 (XML/JSON)
┌─────────────────┐          ┌─────────────────────────┐
│ :20:TXN123456   │          │ <CdtTrfTxInf>           │
│ :32A:240115USD  │   ──►    │   <PmtId>               │
│      1000,00    │          │     <InstrId>TXN123456  │
│ :50K:/12345678  │          │   </PmtId>              │
│    JOHN SMITH   │          │   <Amt>                 │
└─────────────────┘          │     <InstdAmt Ccy="USD">│
                             │       1000.00</InstdAmt>│
                             │   </Amt>                │
                             └─────────────────────────┘
```

## Code Example: Payment Data Model

```sql
-- Payment Instructions
CREATE TABLE payments (
    payment_id UUID PRIMARY KEY,
    payment_reference VARCHAR(50) NOT NULL UNIQUE,
    payment_type VARCHAR(30) NOT NULL,
    payment_method VARCHAR(30) NOT NULL,  -- INTERNAL, RTGS, ACH, SWIFT
    status VARCHAR(30) NOT NULL DEFAULT 'INITIATED',

    -- Debit Side
    debit_account_id UUID NOT NULL REFERENCES accounts(account_id),
    debit_amount DECIMAL(18,2) NOT NULL,
    debit_currency VARCHAR(3) NOT NULL,

    -- Credit Side
    credit_account_id UUID REFERENCES accounts(account_id),
    credit_amount DECIMAL(18,2) NOT NULL,
    credit_currency VARCHAR(3) NOT NULL,
    beneficiary_name VARCHAR(200),
    beneficiary_account VARCHAR(50),
    beneficiary_bank_code VARCHAR(20),
    beneficiary_bank_name VARCHAR(200),

    -- FX Details (if applicable)
    fx_rate DECIMAL(18,8),
    fx_reference VARCHAR(50),

    -- Processing Details
    value_date DATE NOT NULL,
    purpose_code VARCHAR(10),
    payment_details VARCHAR(500),
    charge_type VARCHAR(20),  -- OUR, BEN, SHA

    -- Network Details
    network_reference VARCHAR(50),
    clearing_system VARCHAR(20),

    -- Timestamps
    initiated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    processed_at TIMESTAMP WITH TIME ZONE,
    settled_at TIMESTAMP WITH TIME ZONE,
    initiated_by VARCHAR(100) NOT NULL,
    channel VARCHAR(30) NOT NULL,

    CONSTRAINT valid_payment_type CHECK (payment_type IN (
        'INTERNAL', 'DOMESTIC_WIRE', 'DOMESTIC_ACH',
        'INTERNATIONAL_WIRE', 'INSTANT_PAYMENT'
    )),
    CONSTRAINT valid_status CHECK (status IN (
        'INITIATED', 'VALIDATED', 'PENDING_APPROVAL',
        'APPROVED', 'PROCESSING', 'SENT', 'CLEARED',
        'SETTLED', 'COMPLETED', 'FAILED', 'RETURNED', 'CANCELLED'
    ))
);

-- Payment Routing Rules
CREATE TABLE payment_routing_rules (
    rule_id UUID PRIMARY KEY,
    rule_name VARCHAR(100) NOT NULL,
    priority INTEGER NOT NULL,
    payment_type VARCHAR(30),
    currency VARCHAR(3),
    amount_min DECIMAL(18,2),
    amount_max DECIMAL(18,2),
    destination_country VARCHAR(3),
    clearing_system VARCHAR(20) NOT NULL,
    correspondent_bank_code VARCHAR(20),
    nostro_account_id UUID REFERENCES accounts(account_id),
    effective_from DATE NOT NULL,
    effective_to DATE,
    status VARCHAR(20) DEFAULT 'ACTIVE'
);

-- Nostro/Vostro Accounts
CREATE TABLE correspondent_accounts (
    correspondent_id UUID PRIMARY KEY,
    account_type VARCHAR(10) NOT NULL,  -- NOSTRO, VOSTRO
    our_account_id UUID NOT NULL REFERENCES accounts(account_id),
    their_bank_code VARCHAR(20) NOT NULL,
    their_bank_name VARCHAR(200) NOT NULL,
    their_account_number VARCHAR(50) NOT NULL,
    currency VARCHAR(3) NOT NULL,
    swift_bic VARCHAR(11),
    status VARCHAR(20) DEFAULT 'ACTIVE',

    CONSTRAINT valid_account_type CHECK (account_type IN ('NOSTRO', 'VOSTRO'))
);

-- Payment Status History
CREATE TABLE payment_status_history (
    history_id UUID PRIMARY KEY,
    payment_id UUID NOT NULL REFERENCES payments(payment_id),
    status VARCHAR(30) NOT NULL,
    status_reason VARCHAR(500),
    status_timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_by VARCHAR(100)
);
```

```json
{
  "payment": {
    "paymentId": "pmt-uuid-12345",
    "paymentReference": "PAY2024011500001",
    "paymentType": "INTERNATIONAL_WIRE",
    "paymentMethod": "SWIFT",
    "status": "COMPLETED",
    "debitSide": {
      "accountId": "acc-uuid-12345",
      "accountNumber": "1234567890",
      "amount": 10000.00,
      "currency": "USD"
    },
    "creditSide": {
      "beneficiaryName": "ACME Corporation Ltd",
      "beneficiaryAccount": "GB29NWBK60161331926819",
      "beneficiaryBank": "NWBKGB2L",
      "amount": 7850.00,
      "currency": "GBP"
    },
    "fxDetails": {
      "rate": 0.785,
      "fxReference": "FX2024011500001"
    },
    "valueDate": "2024-01-15",
    "networkReference": "SWIFT123456789",
    "chargeType": "SHA",
    "charges": {
      "ourCharge": 25.00,
      "correspondentCharge": 15.00
    },
    "timestamps": {
      "initiated": "2024-01-15T09:30:00Z",
      "sent": "2024-01-15T09:35:00Z",
      "settled": "2024-01-16T14:00:00Z"
    }
  }
}
```

## Key Takeaways

1. Different payment types have different speed, cost, and processing characteristics
2. Payment rails determine the underlying infrastructure for moving funds
3. Clearing aggregates transactions; settlement finalizes fund movement
4. ISO 20022 is becoming the global standard for payment messaging
5. Nostro/Vostro reconciliation is critical for international payments

## Further Reading

- SWIFT Standards documentation
- ISO 20022 Message Definitions
- Federal Reserve Payment System publications
- CPMI-IOSCO Principles for Financial Market Infrastructures

## Next Lesson

[Lesson 08: General Ledger & Accounting](../lesson-08-general-ledger-and-accounting/README.md)
