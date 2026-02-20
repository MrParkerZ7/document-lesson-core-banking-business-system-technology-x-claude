# Lesson 03: Customer Management

## Overview

This lesson covers Customer Information File (CIF) management, Know Your Customer (KYC) processes, and Anti-Money Laundering (AML) compliance. You'll learn how banks maintain comprehensive customer records while meeting regulatory requirements.

## Learning Objectives

By the end of this lesson, you will be able to:

- Design a Customer Information File (CIF) data model
- Implement KYC workflows and verification processes
- Understand AML screening and transaction monitoring
- Manage customer relationships and hierarchies
- Handle customer data privacy and consent

## Sub-Lessons

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Customer Information File](./01-customer-information-file/README.md) | CIF structure, identifiers, and data model |
| 02 | [KYC Process](./02-kyc-process/README.md) | Verification levels, document management |
| 03 | [AML Compliance](./03-aml-compliance/README.md) | Screening, monitoring, reporting |
| 04 | [Customer Relationships](./04-customer-relationships/README.md) | Hierarchies, linked accounts, mandates |
| 05 | [Data Privacy & Consent](./05-data-privacy-consent/README.md) | GDPR compliance, consent management |

## Key Concepts

### Customer Information File (CIF)

The CIF is the single source of truth for customer data in a banking system:

```
┌─────────────────────────────────────────────────────────────┐
│                    CUSTOMER (CIF)                            │
├─────────────────────────────────────────────────────────────┤
│  CIF Number: CIF-2024-000001                                │
│  Customer Type: INDIVIDUAL / CORPORATE                       │
│  Status: ACTIVE / DORMANT / CLOSED / BLOCKED                │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  Identity   │  │  Contact    │  │  Address    │         │
│  │  Documents  │  │  Details    │  │  History    │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  KYC        │  │  Risk       │  │  Accounts   │         │
│  │  Status     │  │  Profile    │  │  & Products │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### KYC Verification Levels

| Level | Requirements | Products Allowed |
|-------|--------------|------------------|
| Basic | Name, DOB, Address, Photo ID | Basic savings, limited transactions |
| Standard | + Proof of address, income declaration | Full banking services |
| Enhanced | + Source of funds, detailed due diligence | High-value accounts, investments |

### AML Screening Process

```
Customer Data ──► Name Screening ──► Sanctions Lists
                       │                    │
                       ▼                    ▼
                  PEP Check ────────► Risk Scoring
                       │                    │
                       ▼                    ▼
              Transaction Monitoring ──► Alert Generation
                       │                    │
                       ▼                    ▼
              Case Management ──────► SAR Filing
```

### Customer Relationship Hierarchies

| Relationship Type | Description | Example |
|-------------------|-------------|---------|
| Individual | Single natural person | Personal account holder |
| Joint | Multiple individuals | Joint account holders |
| Corporate | Legal entity | Company accounts |
| Group | Corporate hierarchy | Parent-subsidiary |
| Household | Family relationships | Family banking packages |

## Code Example: Customer Data Model

```sql
-- Core Customer Table
CREATE TABLE customers (
    cif_number VARCHAR(20) PRIMARY KEY,
    customer_type VARCHAR(20) NOT NULL CHECK (customer_type IN ('INDIVIDUAL', 'CORPORATE')),
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING_KYC',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_by VARCHAR(100) NOT NULL,
    version INTEGER DEFAULT 1
);

-- Individual Customer Details
CREATE TABLE individual_customers (
    cif_number VARCHAR(20) PRIMARY KEY REFERENCES customers(cif_number),
    title VARCHAR(10),
    first_name VARCHAR(100) NOT NULL,
    middle_name VARCHAR(100),
    last_name VARCHAR(100) NOT NULL,
    date_of_birth DATE NOT NULL,
    gender VARCHAR(10),
    nationality VARCHAR(3),  -- ISO 3166-1 alpha-3
    tax_id VARCHAR(50),
    occupation VARCHAR(100),
    employer VARCHAR(200)
);

-- Corporate Customer Details
CREATE TABLE corporate_customers (
    cif_number VARCHAR(20) PRIMARY KEY REFERENCES customers(cif_number),
    legal_name VARCHAR(500) NOT NULL,
    trading_name VARCHAR(500),
    registration_number VARCHAR(100) NOT NULL,
    registration_country VARCHAR(3),
    incorporation_date DATE,
    legal_structure VARCHAR(50),
    industry_code VARCHAR(20)  -- NAICS or SIC code
);

-- KYC Records
CREATE TABLE kyc_records (
    kyc_id UUID PRIMARY KEY,
    cif_number VARCHAR(20) NOT NULL REFERENCES customers(cif_number),
    kyc_level VARCHAR(20) NOT NULL,
    status VARCHAR(20) NOT NULL,
    verified_at TIMESTAMP WITH TIME ZONE,
    verified_by VARCHAR(100),
    expiry_date DATE,
    risk_rating VARCHAR(20),
    next_review_date DATE
);

-- Identity Documents
CREATE TABLE identity_documents (
    document_id UUID PRIMARY KEY,
    cif_number VARCHAR(20) NOT NULL REFERENCES customers(cif_number),
    document_type VARCHAR(50) NOT NULL,
    document_number VARCHAR(100) NOT NULL,
    issuing_country VARCHAR(3),
    issue_date DATE,
    expiry_date DATE,
    verified BOOLEAN DEFAULT FALSE,
    document_image_ref VARCHAR(500)
);
```

```json
{
  "customer": {
    "cifNumber": "CIF-2024-000001",
    "customerType": "INDIVIDUAL",
    "status": "ACTIVE",
    "individual": {
      "title": "Mr",
      "firstName": "John",
      "lastName": "Smith",
      "dateOfBirth": "1985-03-15",
      "nationality": "USA"
    },
    "kyc": {
      "level": "STANDARD",
      "status": "VERIFIED",
      "riskRating": "LOW",
      "verifiedAt": "2024-01-10T14:30:00Z",
      "nextReviewDate": "2025-01-10"
    },
    "contacts": [
      {
        "type": "EMAIL",
        "value": "john.smith@email.com",
        "primary": true,
        "verified": true
      },
      {
        "type": "MOBILE",
        "value": "+1-555-123-4567",
        "primary": true,
        "verified": true
      }
    ],
    "addresses": [
      {
        "type": "RESIDENTIAL",
        "line1": "123 Main Street",
        "city": "New York",
        "state": "NY",
        "postalCode": "10001",
        "country": "USA",
        "primary": true
      }
    ]
  }
}
```

## Key Takeaways

1. CIF provides a unified view of customer across all banking products
2. KYC is a regulatory requirement with defined verification levels
3. AML screening is continuous throughout the customer relationship
4. Customer hierarchies enable relationship-based services and risk management
5. Data privacy regulations require explicit consent and data handling controls

## Further Reading

- FATF Recommendations on Customer Due Diligence
- Basel Committee on Customer Due Diligence for Banks
- GDPR requirements for financial services
- Wolfsberg Group AML Principles

## Next Lesson

[Lesson 04: Account Management](../lesson-04-account-management/README.md)
