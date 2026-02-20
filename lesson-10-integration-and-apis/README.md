# Lesson 10: Integration & APIs

## Overview

This lesson covers integration patterns and API design for core banking systems. You'll learn about Open Banking standards, API security, and how to connect banking systems with external partners and services.

## Learning Objectives

By the end of this lesson, you will be able to:

- Design RESTful APIs for banking services
- Implement Open Banking standards (PSD2, Open Banking UK)
- Secure banking APIs with proper authentication
- Handle third-party integrations and webhooks
- Implement API versioning and lifecycle management

## Sub-Lessons

| # | Topic | Description |
|---|-------|-------------|
| 01 | [API Design Principles](./01-api-design-principles/README.md) | REST, GraphQL, API standards |
| 02 | [Open Banking](./02-open-banking/README.md) | PSD2, consent, TPP integration |
| 03 | [API Security](./03-api-security/README.md) | OAuth 2.0, mTLS, API keys |
| 04 | [Integration Patterns](./04-integration-patterns/README.md) | ESB, events, webhooks |
| 05 | [API Management](./05-api-management/README.md) | Gateway, versioning, documentation |

## Key Concepts

### Open Banking Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    THIRD-PARTY PROVIDERS                     │
│         (AISPs, PISPs, Fintechs, Aggregators)              │
└─────────────────────────────────────────────────────────────┘
                              │
                    Open Banking APIs
                              │
┌─────────────────────────────────────────────────────────────┐
│                      API GATEWAY                             │
│     (Rate Limiting, Auth, Routing, Monitoring)              │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                   CONSENT MANAGEMENT                         │
│        (Customer Permissions, Scope, Validity)              │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                   BANKING SERVICES                           │
│    Account │ Payment │ Product │ Customer Services          │
└─────────────────────────────────────────────────────────────┘
```

### API Categories in Banking

| Category | Purpose | Examples |
|----------|---------|----------|
| Account Information | Read account data | Balances, transactions, statements |
| Payment Initiation | Execute payments | Transfers, bill pay, standing orders |
| Product Catalog | View products | Rates, terms, availability |
| Customer Onboarding | New customers | KYC, account opening |
| Card Management | Card operations | Activate, block, limits |

### OAuth 2.0 Flow for Open Banking

```
┌──────────┐                              ┌──────────┐
│   TPP    │                              │   Bank   │
│  (AISP)  │                              │          │
└────┬─────┘                              └────┬─────┘
     │                                         │
     │  1. Authorization Request               │
     │────────────────────────────────────────►│
     │                                         │
     │  2. Redirect to Bank Login              │
     │◄────────────────────────────────────────│
     │                                         │
     │         ┌──────────────┐                │
     │         │   Customer   │                │
     │         │   Consent    │                │
     │         └──────────────┘                │
     │                                         │
     │  3. Authorization Code                  │
     │◄────────────────────────────────────────│
     │                                         │
     │  4. Exchange Code for Token             │
     │────────────────────────────────────────►│
     │                                         │
     │  5. Access Token + Refresh Token        │
     │◄────────────────────────────────────────│
     │                                         │
     │  6. API Request with Access Token       │
     │────────────────────────────────────────►│
     │                                         │
     │  7. API Response                        │
     │◄────────────────────────────────────────│
```

### Integration Patterns

| Pattern | Description | Use Case |
|---------|-------------|----------|
| Request-Response | Synchronous API calls | Real-time queries |
| Event-Driven | Async message publishing | Transaction notifications |
| Webhook | Callback to third party | Payment status updates |
| Batch File | Scheduled file exchange | Regulatory reporting |
| Message Queue | Async processing | High-volume transactions |

## Code Example: Banking API Specifications

### OpenAPI Specification

```yaml
openapi: 3.0.3
info:
  title: Core Banking API
  version: 1.0.0
  description: RESTful API for core banking operations

servers:
  - url: https://api.bank.com/v1
    description: Production

security:
  - OAuth2:
      - accounts:read
      - payments:write

paths:
  /accounts:
    get:
      summary: List customer accounts
      operationId: getAccounts
      tags:
        - Accounts
      parameters:
        - name: customerId
          in: query
          required: true
          schema:
            type: string
      responses:
        '200':
          description: List of accounts
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/AccountList'
        '401':
          $ref: '#/components/responses/Unauthorized'

  /accounts/{accountId}/balance:
    get:
      summary: Get account balance
      operationId: getAccountBalance
      parameters:
        - name: accountId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Account balance
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Balance'

  /payments:
    post:
      summary: Initiate payment
      operationId: initiatePayment
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PaymentRequest'
      responses:
        '201':
          description: Payment initiated
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PaymentResponse'

components:
  schemas:
    Account:
      type: object
      properties:
        accountId:
          type: string
          example: "acc-uuid-12345"
        accountNumber:
          type: string
          example: "1234567890"
        accountType:
          type: string
          enum: [SAVINGS, CURRENT, TERM_DEPOSIT]
        currency:
          type: string
          example: "USD"
        status:
          type: string
          enum: [ACTIVE, DORMANT, BLOCKED, CLOSED]

    Balance:
      type: object
      properties:
        accountId:
          type: string
        balanceType:
          type: string
          enum: [LEDGER, AVAILABLE]
        amount:
          type: number
          format: decimal
        currency:
          type: string
        asOfDateTime:
          type: string
          format: date-time

    PaymentRequest:
      type: object
      required:
        - debitAccountId
        - creditAccountId
        - amount
        - currency
      properties:
        debitAccountId:
          type: string
        creditAccountId:
          type: string
        amount:
          type: number
        currency:
          type: string
        reference:
          type: string
        paymentDate:
          type: string
          format: date

  securitySchemes:
    OAuth2:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://auth.bank.com/authorize
          tokenUrl: https://auth.bank.com/token
          scopes:
            accounts:read: Read account information
            payments:write: Initiate payments

  responses:
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
```

### API Response Examples

```json
{
  "accounts": [
    {
      "accountId": "acc-uuid-12345",
      "accountNumber": "1234567890",
      "accountType": "SAVINGS",
      "currency": "USD",
      "status": "ACTIVE",
      "balances": {
        "ledger": {
          "amount": 15000.00,
          "currency": "USD",
          "asOfDateTime": "2024-01-15T23:59:59Z"
        },
        "available": {
          "amount": 14500.00,
          "currency": "USD",
          "asOfDateTime": "2024-01-15T23:59:59Z"
        }
      }
    }
  ],
  "_links": {
    "self": "/accounts?customerId=cif-001",
    "next": "/accounts?customerId=cif-001&cursor=abc123"
  }
}
```

### Consent Management

```sql
-- Consent Records
CREATE TABLE api_consents (
    consent_id UUID PRIMARY KEY,
    cif_number VARCHAR(20) NOT NULL REFERENCES customers(cif_number),
    tpp_id VARCHAR(50) NOT NULL,
    tpp_name VARCHAR(200) NOT NULL,
    consent_type VARCHAR(30) NOT NULL,  -- ACCOUNT_ACCESS, PAYMENT_INITIATION
    status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE',
    scopes JSONB NOT NULL,  -- ["accounts:read", "balances:read"]
    accounts JSONB,  -- Specific accounts consented
    valid_from TIMESTAMP WITH TIME ZONE NOT NULL,
    valid_until TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    revoked_at TIMESTAMP WITH TIME ZONE,
    revoked_by VARCHAR(100),

    CONSTRAINT valid_consent_type CHECK (consent_type IN (
        'ACCOUNT_ACCESS', 'PAYMENT_INITIATION', 'FUNDS_CONFIRMATION'
    ))
);

-- API Access Logs
CREATE TABLE api_access_logs (
    log_id UUID PRIMARY KEY,
    consent_id UUID REFERENCES api_consents(consent_id),
    tpp_id VARCHAR(50) NOT NULL,
    api_endpoint VARCHAR(200) NOT NULL,
    http_method VARCHAR(10) NOT NULL,
    request_timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    response_status INTEGER,
    response_time_ms INTEGER,
    client_ip VARCHAR(50),
    user_agent VARCHAR(500)
);
```

## Key Takeaways

1. Open Banking enables secure third-party access to banking data
2. OAuth 2.0 with proper scopes controls API access
3. Consent management is central to regulatory compliance
4. API design must follow industry standards (OpenAPI, ISO 20022)
5. API versioning ensures backward compatibility

## Further Reading

- Open Banking Implementation Entity (OBIE) standards
- PSD2 Technical Regulatory Standards
- OAuth 2.0 for Banking (FAPI)
- ISO 20022 API guidelines

## Next Lesson

[Lesson 11: Security & Compliance](../lesson-11-security-and-compliance/README.md)
