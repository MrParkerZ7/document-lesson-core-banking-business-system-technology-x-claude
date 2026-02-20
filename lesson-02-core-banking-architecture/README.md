# Lesson 02: Core Banking Architecture

## Overview

This lesson explores the architectural foundations of Core Banking Systems, covering design patterns, system layers, and component interactions. You'll learn how to design scalable, resilient, and maintainable banking systems that meet both functional and non-functional requirements.

## Learning Objectives

By the end of this lesson, you will be able to:

- Describe the layered architecture of a modern CBS
- Explain key architectural patterns used in banking systems
- Understand the role of each architectural component
- Evaluate trade-offs between monolithic and microservices architectures
- Design for scalability, resilience, and compliance

## Sub-Lessons

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Architectural Layers](./01-architectural-layers/README.md) | Presentation, business logic, data layers |
| 02 | [Design Patterns](./02-design-patterns/README.md) | Event sourcing, CQRS, saga patterns |
| 03 | [Microservices vs Monolith](./03-microservices-vs-monolith/README.md) | Architecture comparison and trade-offs |
| 04 | [Data Architecture](./04-data-architecture/README.md) | Database design, partitioning, replication |
| 05 | [Integration Architecture](./05-integration-architecture/README.md) | ESB, API gateway, message brokers |

## Key Concepts

### Layered Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                        │
│  (Mobile Apps, Web Portal, Branch Systems, ATM, API Gateway)│
├─────────────────────────────────────────────────────────────┤
│                     API LAYER                                │
│         (REST APIs, GraphQL, gRPC, WebSocket)               │
├─────────────────────────────────────────────────────────────┤
│                  BUSINESS LOGIC LAYER                        │
│    (Domain Services, Rules Engine, Workflow Engine)         │
├─────────────────────────────────────────────────────────────┤
│                   SERVICE LAYER                              │
│  (Customer, Account, Loan, Payment, GL Services)            │
├─────────────────────────────────────────────────────────────┤
│                    DATA LAYER                                │
│  (Databases, Cache, Message Queues, Event Store)            │
├─────────────────────────────────────────────────────────────┤
│                 INFRASTRUCTURE LAYER                         │
│       (Containers, Kubernetes, Cloud Services)              │
└─────────────────────────────────────────────────────────────┘
```

### Key Architectural Patterns

| Pattern | Use Case | Benefits |
|---------|----------|----------|
| Event Sourcing | Transaction history, audit trails | Complete history, temporal queries |
| CQRS | High-read/write separation | Optimized read/write paths |
| Saga Pattern | Distributed transactions | Eventual consistency, compensation |
| Domain-Driven Design | Complex business logic | Clear boundaries, ubiquitous language |
| API Gateway | External integrations | Security, rate limiting, routing |

### Non-Functional Requirements

Banking systems must meet stringent non-functional requirements:

| Requirement | Target | Rationale |
|-------------|--------|-----------|
| Availability | 99.99% (52 min/year downtime) | 24/7 banking services |
| Latency | < 100ms for transactions | Real-time user experience |
| Throughput | 10,000+ TPS | Peak transaction volumes |
| Data Durability | Zero data loss | Financial accuracy |
| Security | Defense in depth | Regulatory compliance |
| Auditability | Complete trail | Compliance, forensics |

### Component Interactions

```
Customer ──► API Gateway ──► Authentication Service
                   │
                   ▼
              Load Balancer
                   │
         ┌────────┼────────┐
         ▼        ▼        ▼
    Account   Payment   Loan
    Service   Service   Service
         │        │        │
         ▼        ▼        ▼
    ┌─────────────────────────┐
    │     Message Broker      │
    │   (Kafka/RabbitMQ)      │
    └─────────────────────────┘
              │
    ┌─────────┼─────────┐
    ▼         ▼         ▼
  GL      Notification  Audit
Service    Service    Service
```

## Code Example: Event Sourcing for Account

```sql
-- Event Store Schema
CREATE TABLE account_events (
    event_id UUID PRIMARY KEY,
    aggregate_id VARCHAR(50) NOT NULL,  -- Account ID
    aggregate_type VARCHAR(50) NOT NULL,
    event_type VARCHAR(100) NOT NULL,
    event_data JSONB NOT NULL,
    metadata JSONB,
    version INTEGER NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_by VARCHAR(100) NOT NULL,

    CONSTRAINT unique_aggregate_version
        UNIQUE (aggregate_id, version)
);

-- Sample Events
INSERT INTO account_events (event_id, aggregate_id, aggregate_type, event_type, event_data, version, created_by)
VALUES
    ('evt-001', 'ACC-123', 'Account', 'AccountOpened',
     '{"accountNumber": "1234567890", "customerId": "CIF-001", "accountType": "SAVINGS", "currency": "USD"}',
     1, 'system'),
    ('evt-002', 'ACC-123', 'Account', 'DepositMade',
     '{"amount": 1000.00, "reference": "Initial deposit"}',
     2, 'teller-001'),
    ('evt-003', 'ACC-123', 'Account', 'WithdrawalMade',
     '{"amount": 200.00, "reference": "ATM withdrawal"}',
     3, 'atm-central-01');
```

```json
{
  "eventType": "AccountOpened",
  "aggregateId": "ACC-123",
  "version": 1,
  "timestamp": "2024-01-15T09:00:00Z",
  "payload": {
    "accountNumber": "1234567890",
    "customerId": "CIF-001",
    "accountType": "SAVINGS",
    "currency": "USD",
    "initialBalance": 0.00,
    "branch": "BRANCH-001"
  },
  "metadata": {
    "correlationId": "req-abc-123",
    "causationId": "cmd-open-account",
    "userId": "teller-001"
  }
}
```

## Key Takeaways

1. Layered architecture separates concerns and enables independent scaling
2. Event sourcing provides complete audit trails essential for banking
3. CQRS optimizes read and write operations for different access patterns
4. Microservices enable agility but add complexity - choose based on team and scale
5. Non-functional requirements drive architectural decisions in banking

## Further Reading

- "Building Microservices" by Sam Newman
- "Domain-Driven Design" by Eric Evans
- "Designing Data-Intensive Applications" by Martin Kleppmann
- The Twelve-Factor App methodology

## Next Lesson

[Lesson 03: Customer Management](../lesson-03-customer-management/README.md)
