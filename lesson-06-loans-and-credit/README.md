# Lesson 06: Loans & Credit

## Overview

This lesson covers lending products and credit management in core banking systems. You'll learn about the loan lifecycle from origination through servicing, including amortization calculations, collections, and portfolio management.

## Learning Objectives

By the end of this lesson, you will be able to:

- Design loan origination and servicing systems
- Implement amortization schedules and payment calculations
- Understand credit scoring and decisioning processes
- Handle loan disbursement and repayment workflows
- Manage delinquency and collections processes

## Sub-Lessons

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Loan Product Types](./01-loan-product-types/README.md) | Personal, mortgage, business, revolving credit |
| 02 | [Loan Origination](./02-loan-origination/README.md) | Application, underwriting, approval workflow |
| 03 | [Amortization & Payments](./03-amortization-payments/README.md) | Schedules, calculations, prepayments |
| 04 | [Loan Servicing](./04-loan-servicing/README.md) | Disbursement, statements, modifications |
| 05 | [Collections & Recovery](./05-collections-recovery/README.md) | Delinquency, workout, write-off |

## Key Concepts

### Loan Product Categories

| Category | Products | Characteristics |
|----------|----------|-----------------|
| Consumer | Personal, Auto, Education | Shorter terms, unsecured/secured |
| Mortgage | Home purchase, Refinance, HELOC | Long terms, property secured |
| Commercial | Working capital, Equipment, Real estate | Business purpose, varied structures |
| Revolving | Credit cards, Lines of credit | Ongoing availability, variable balance |
| Specialized | Trade finance, Project finance | Complex structures |

### Loan Lifecycle

```
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ APPLICATION  │──►│ UNDERWRITING │──►│   APPROVAL   │
│              │   │              │   │              │
└──────────────┘   └──────────────┘   └──────────────┘
                                             │
┌──────────────────────────────────────────────┘
│
▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ DOCUMENTATION│──►│ DISBURSEMENT │──►│   ACTIVE/    │
│              │   │              │   │  SERVICING   │
└──────────────┘   └──────────────┘   └──────────────┘
                                             │
         ┌───────────────┬───────────────────┼───────────────┐
         ▼               ▼                   ▼               ▼
   ┌──────────┐   ┌──────────┐        ┌──────────┐   ┌──────────┐
   │  PAID    │   │DELINQUENT│        │RESTRUCTURE│  │ WRITTEN  │
   │   OFF    │   │          │        │           │  │   OFF    │
   └──────────┘   └──────────┘        └──────────┘   └──────────┘
```

### Amortization Methods

| Method | Description | Use Case |
|--------|-------------|----------|
| Equal Payment (EMI) | Fixed payment amount | Most consumer loans |
| Equal Principal | Decreasing payments | Some mortgages |
| Bullet/Balloon | Interest only, principal at end | Short-term commercial |
| Graduated | Increasing payments over time | Income-growth borrowers |
| Flexible | Variable based on cash flow | Seasonal businesses |

### EMI Calculation

```
EMI = P × r × (1 + r)^n / [(1 + r)^n - 1]

Where:
  P = Principal loan amount
  r = Monthly interest rate (annual rate / 12)
  n = Total number of payments

Example:
  Loan: $100,000 at 6% annual for 30 years
  r = 0.06 / 12 = 0.005
  n = 30 × 12 = 360
  EMI = 100000 × 0.005 × (1.005)^360 / [(1.005)^360 - 1]
  EMI = $599.55
```

### Loan Status Classification

| Status | Days Past Due | Provisioning Impact |
|--------|---------------|---------------------|
| Current | 0 | Stage 1 (12-month ECL) |
| Watch | 1-30 | Stage 1 |
| Substandard | 31-60 | Stage 2 (Lifetime ECL) |
| Doubtful | 61-90 | Stage 2 |
| Loss | 90+ | Stage 3 (Credit impaired) |

## Code Example: Loan Data Model

```sql
-- Loan Products
CREATE TABLE loan_products (
    product_code VARCHAR(20) PRIMARY KEY,
    product_name VARCHAR(200) NOT NULL,
    product_type VARCHAR(30) NOT NULL,
    loan_purpose VARCHAR(50),
    minimum_amount DECIMAL(18,2),
    maximum_amount DECIMAL(18,2),
    minimum_tenure_months INTEGER,
    maximum_tenure_months INTEGER,
    interest_type VARCHAR(20) NOT NULL,  -- FIXED, FLOATING, HYBRID
    base_rate_code VARCHAR(20),
    spread_rate DECIMAL(7,4),
    amortization_method VARCHAR(30) NOT NULL,
    collateral_required BOOLEAN DEFAULT FALSE,
    status VARCHAR(20) DEFAULT 'ACTIVE',

    CONSTRAINT valid_loan_type CHECK (product_type IN (
        'PERSONAL', 'AUTO', 'MORTGAGE', 'BUSINESS',
        'CREDIT_LINE', 'OVERDRAFT', 'CREDIT_CARD'
    ))
);

-- Loan Accounts
CREATE TABLE loans (
    loan_id UUID PRIMARY KEY,
    loan_number VARCHAR(30) NOT NULL UNIQUE,
    account_id UUID NOT NULL REFERENCES accounts(account_id),
    cif_number VARCHAR(20) NOT NULL REFERENCES customers(cif_number),
    product_code VARCHAR(20) NOT NULL REFERENCES loan_products(product_code),
    application_id UUID,
    sanctioned_amount DECIMAL(18,2) NOT NULL,
    disbursed_amount DECIMAL(18,2) DEFAULT 0,
    outstanding_principal DECIMAL(18,2) DEFAULT 0,
    interest_rate DECIMAL(7,4) NOT NULL,
    tenure_months INTEGER NOT NULL,
    emi_amount DECIMAL(18,2),
    disbursement_date DATE,
    first_emi_date DATE,
    maturity_date DATE,
    repayment_account_id UUID REFERENCES accounts(account_id),
    loan_status VARCHAR(20) NOT NULL DEFAULT 'PENDING_DISBURSEMENT',
    dpd INTEGER DEFAULT 0,  -- Days Past Due
    npa_date DATE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Amortization Schedule
CREATE TABLE amortization_schedule (
    schedule_id UUID PRIMARY KEY,
    loan_id UUID NOT NULL REFERENCES loans(loan_id),
    installment_number INTEGER NOT NULL,
    due_date DATE NOT NULL,
    principal_due DECIMAL(18,2) NOT NULL,
    interest_due DECIMAL(18,2) NOT NULL,
    total_due DECIMAL(18,2) NOT NULL,
    principal_paid DECIMAL(18,2) DEFAULT 0,
    interest_paid DECIMAL(18,2) DEFAULT 0,
    penalty_due DECIMAL(18,2) DEFAULT 0,
    penalty_paid DECIMAL(18,2) DEFAULT 0,
    payment_date DATE,
    status VARCHAR(20) DEFAULT 'PENDING',

    CONSTRAINT unique_installment UNIQUE (loan_id, installment_number)
);

-- Loan Disbursements
CREATE TABLE loan_disbursements (
    disbursement_id UUID PRIMARY KEY,
    loan_id UUID NOT NULL REFERENCES loans(loan_id),
    tranche_number INTEGER NOT NULL,
    disbursement_amount DECIMAL(18,2) NOT NULL,
    disbursement_date DATE NOT NULL,
    credit_account_id UUID REFERENCES accounts(account_id),
    payment_mode VARCHAR(30),
    status VARCHAR(20) DEFAULT 'PENDING',
    approved_by VARCHAR(100),

    CONSTRAINT unique_tranche UNIQUE (loan_id, tranche_number)
);

-- Loan Collections/Payments
CREATE TABLE loan_payments (
    payment_id UUID PRIMARY KEY,
    loan_id UUID NOT NULL REFERENCES loans(loan_id),
    payment_date DATE NOT NULL,
    payment_amount DECIMAL(18,2) NOT NULL,
    principal_applied DECIMAL(18,2) NOT NULL,
    interest_applied DECIMAL(18,2) NOT NULL,
    penalty_applied DECIMAL(18,2) DEFAULT 0,
    excess_amount DECIMAL(18,2) DEFAULT 0,
    payment_type VARCHAR(20) NOT NULL,  -- REGULAR, PREPAYMENT, SETTLEMENT
    payment_mode VARCHAR(30),
    debit_account_id UUID,
    transaction_reference VARCHAR(50)
);
```

```json
{
  "loan": {
    "loanId": "loan-uuid-12345",
    "loanNumber": "LN2024000001",
    "cifNumber": "CIF-2024-000001",
    "productCode": "HOME_LOAN_FIXED",
    "sanctionedAmount": 250000.00,
    "disbursedAmount": 250000.00,
    "outstandingPrincipal": 245123.45,
    "interestRate": 6.50,
    "tenureMonths": 240,
    "emiAmount": 1862.85,
    "disbursementDate": "2024-01-15",
    "firstEmiDate": "2024-02-15",
    "maturityDate": "2044-01-15",
    "status": "ACTIVE",
    "dpd": 0,
    "nextDueDate": "2024-03-15",
    "amortizationSummary": {
      "totalInstallments": 240,
      "paidInstallments": 1,
      "remainingInstallments": 239,
      "totalInterestPayable": 197084.00,
      "interestPaidToDate": 1354.17
    }
  }
}
```

### Amortization Schedule Generation (TypeScript)

```typescript
interface AmortizationEntry {
  installmentNumber: number;
  dueDate: Date;
  principalDue: number;
  interestDue: number;
  totalDue: number;
  openingBalance: number;
  closingBalance: number;
}

function generateAmortizationSchedule(
  principal: number,
  annualRate: number,
  tenureMonths: number,
  startDate: Date
): AmortizationEntry[] {
  const monthlyRate = annualRate / 12 / 100;
  const emi = principal * monthlyRate *
    Math.pow(1 + monthlyRate, tenureMonths) /
    (Math.pow(1 + monthlyRate, tenureMonths) - 1);

  const schedule: AmortizationEntry[] = [];
  let balance = principal;
  let dueDate = new Date(startDate);

  for (let i = 1; i <= tenureMonths; i++) {
    dueDate.setMonth(dueDate.getMonth() + 1);

    const interestDue = balance * monthlyRate;
    const principalDue = emi - interestDue;
    const closingBalance = balance - principalDue;

    schedule.push({
      installmentNumber: i,
      dueDate: new Date(dueDate),
      principalDue: Math.round(principalDue * 100) / 100,
      interestDue: Math.round(interestDue * 100) / 100,
      totalDue: Math.round(emi * 100) / 100,
      openingBalance: Math.round(balance * 100) / 100,
      closingBalance: Math.round(closingBalance * 100) / 100
    });

    balance = closingBalance;
  }

  return schedule;
}
```

## Key Takeaways

1. Loan origination involves multiple stages of verification and approval
2. Different amortization methods suit different loan types and borrower needs
3. Interest accrual methods must align with accounting standards (IFRS 9)
4. Collections processes require clear escalation paths and workout options
5. Portfolio management requires continuous monitoring and provisioning

## Further Reading

- IFRS 9 Expected Credit Loss (ECL) model
- Basel III Credit Risk Framework
- Consumer lending regulations (TILA, RESPA)
- Credit bureau integration standards

## Next Lesson

[Lesson 07: Payments & Transfers](../lesson-07-payments-and-transfers/README.md)
