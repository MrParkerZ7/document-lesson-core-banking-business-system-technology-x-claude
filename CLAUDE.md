# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains educational materials for understanding Core Banking Business System Technology. It covers the fundamental concepts, architecture patterns, and technical implementations used in modern core banking systems. The content bridges the gap between business requirements and technical implementation, providing a comprehensive understanding of how banking technology powers financial institutions.

## Repository Structure

```
lesson-XX-topic-name/
├── README.md                    # Lesson overview, objectives, sub-lesson links
├── XX.0-lesson-overview.drawio  # Main lesson overview diagram (optional)
├── 01-sub-topic/
│   ├── README.md               # Sub-lesson content
│   ├── XX.1-diagram-name.drawio # DrawIO diagram source (numbered)
│   └── XX.1-diagram-name.png   # PNG export of diagram
├── 02-sub-topic/
│   ├── README.md
│   ├── XX.2-diagram-name.drawio
│   └── XX.2-diagram-name.png
└── ...
```

## Lesson Topics

1. **Introduction to Core Banking Systems** - Fundamentals, history, and evolution
2. **Core Banking Architecture** - System design, layers, and components
3. **Customer Management** - CIF, KYC/AML, customer lifecycle
4. **Account Management** - Account types, structures, and operations
5. **Deposits & Savings** - Term deposits, savings products, interest calculation
6. **Loans & Credit** - Lending products, origination, servicing
7. **Payments & Transfers** - Payment rails, clearing, settlement
8. **General Ledger & Accounting** - Chart of accounts, double-entry, reconciliation
9. **Transaction Processing** - Real-time processing, batch operations, EOD/EOM
10. **Integration & APIs** - Open banking, API design, third-party integrations
11. **Security & Compliance** - Authentication, authorization, regulatory compliance
12. **Data Management** - Data models, warehousing, analytics

## Target Audience

This curriculum is designed for professionals working in or transitioning to the banking technology sector.

### IT Developers

- Backend developers building core banking modules
- API developers creating banking integrations
- Database engineers designing financial data models
- DevOps engineers managing banking infrastructure
- QA engineers testing financial systems

### Product Owners

- Product managers defining banking products
- Business analysts translating requirements
- Project managers overseeing banking implementations
- Solution architects designing system integrations
- Compliance officers understanding technical controls

## Working with Diagrams

### Naming Convention
- Diagrams follow the pattern: `[lesson].[sub-lesson]-descriptive-name.drawio`
- Examples: `04.1-account-structure.drawio`, `07.2-payment-flow.drawio`
- Lesson overview diagrams use `.0` suffix: `03.0-customer-management-overview.drawio`

### File Requirements
- All diagrams use DrawIO format (`.drawio` XML files)
- Each diagram must have a corresponding PNG export for viewing
- When modifying diagrams, update both the `.drawio` source and regenerate the `.png` export

### Styling Standards
- Enable shadows on shapes (`shadow=1`)
- Use curved arrows where appropriate (`curved=1`)
- Add flow animation to unidirectional arrows only (`flowAnimation=1`)
- Use consistent color schemes:
  - Blue: Services/APIs
  - Green: Data/Storage
  - Orange: External integrations
  - Purple: Security/Auth
  - Gray: Infrastructure
- Include title and descriptive labels in diagrams

## Content Conventions

### Documentation Structure
- Each lesson README follows consistent structure:
  - Overview and learning objectives
  - Detailed content with diagrams
  - Code examples (where applicable)
  - Key takeaways
  - Further reading

### Banking Domain Terminology
- Use standard ISO 20022 terminology where applicable
- Include glossary references for technical banking terms
- Explain acronyms on first use (CIF, SWIFT, IBAN, RTGS, etc.)

### Code Examples
- Use realistic banking domain examples
- Include schema definitions in SQL format
- API examples in OpenAPI/JSON format
- Event schemas in JSON with clear documentation

## Key Banking Concepts Covered

### Business Processes
- Customer Onboarding: Lead → KYC → Account Opening → Product Enrollment
- Loan Lifecycle: Application → Underwriting → Approval → Disbursement → Servicing → Closure
- Payment Processing: Initiation → Validation → Routing → Clearing → Settlement

### Technical Components
- Core Systems: Customer, Account, Deposit, Loan, Payment, GL
- Integration Points: Payment networks (SWIFT, RTGS, ACH), credit bureaus, regulators
- Data Patterns: Event sourcing, CQRS, double-entry accounting, audit trails

### Regulatory Compliance
- Data Protection: GDPR, PCI-DSS, data encryption, masking
- Financial Reporting: Basel III, IFRS 9, regulatory reporting
- Industry Standards: ISO 20022, SWIFT messaging, Open Banking (PSD2)

## Git Workflow

Commits in this repository include Claude as co-author:
```
Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

## Commands

### Documentation
- No build process required - all content is markdown
- Preview markdown files directly in IDE or GitHub

### Diagram Export
- Open `.drawio` files in Draw.io (desktop or web)
- Export as PNG with transparent background
- Match filename with `.drawio` source file

## Contributing Guidelines

1. Follow existing folder structure and naming conventions
2. Include diagrams for complex concepts
3. Use consistent terminology from glossary
4. Add practical examples relevant to banking domain
5. Update table of contents when adding new sections
6. Ensure all links between documents are valid
