# Lesson 11: Security & Compliance

## Overview

This lesson covers security controls and regulatory compliance in core banking systems. You'll learn about authentication, authorization, data protection, and how to meet regulatory requirements such as PCI-DSS, GDPR, and banking regulations.

## Learning Objectives

By the end of this lesson, you will be able to:

- Implement authentication and authorization mechanisms
- Design data protection and encryption strategies
- Understand key banking regulations and compliance requirements
- Implement audit logging and monitoring
- Handle security incidents and fraud prevention

## Sub-Lessons

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Authentication & Authorization](./01-authentication-authorization/README.md) | MFA, RBAC, session management |
| 02 | [Data Protection](./02-data-protection/README.md) | Encryption, masking, tokenization |
| 03 | [Regulatory Compliance](./03-regulatory-compliance/README.md) | PCI-DSS, GDPR, banking laws |
| 04 | [Audit & Monitoring](./04-audit-monitoring/README.md) | Logging, SIEM, alerting |
| 05 | [Fraud Prevention](./05-fraud-prevention/README.md) | Detection, rules, machine learning |

## Key Concepts

### Security Layers

```
┌─────────────────────────────────────────────────────────────┐
│                     PERIMETER SECURITY                       │
│        (Firewalls, WAF, DDoS Protection, IDS/IPS)           │
├─────────────────────────────────────────────────────────────┤
│                    NETWORK SECURITY                          │
│      (Network Segmentation, VPN, TLS, mTLS)                 │
├─────────────────────────────────────────────────────────────┤
│                  APPLICATION SECURITY                        │
│   (Authentication, Authorization, Input Validation)         │
├─────────────────────────────────────────────────────────────┤
│                      DATA SECURITY                           │
│   (Encryption at Rest, Encryption in Transit, Masking)      │
├─────────────────────────────────────────────────────────────┤
│                  OPERATIONAL SECURITY                        │
│   (Logging, Monitoring, Incident Response, Backup)          │
└─────────────────────────────────────────────────────────────┘
```

### Authentication Methods

| Method | Security Level | Use Case |
|--------|---------------|----------|
| Password | Basic | Initial factor |
| OTP (SMS/Email) | Medium | Second factor |
| TOTP (Authenticator App) | High | Second factor |
| Hardware Token | Very High | Critical operations |
| Biometrics | High | Mobile banking |
| Push Notification | High | Transaction approval |
| FIDO2/WebAuthn | Very High | Passwordless |

### Role-Based Access Control (RBAC)

```
┌──────────────────────────────────────────────────────────────┐
│                         PERMISSIONS                           │
│                                                               │
│  account:read   account:write   payment:initiate             │
│  loan:approve   customer:view   report:generate              │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                           ROLES                               │
│                                                               │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐             │
│  │   Teller   │  │  Manager   │  │   Admin    │             │
│  │            │  │            │  │            │             │
│  │ account:   │  │ account:*  │  │    *:*     │             │
│  │   read     │  │ payment:*  │  │            │             │
│  │ payment:   │  │ loan:      │  │            │             │
│  │  initiate  │  │  approve   │  │            │             │
│  └────────────┘  └────────────┘  └────────────┘             │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                          USERS                                │
│                                                               │
│     John (Teller)    Jane (Manager)    Admin (Admin)         │
└──────────────────────────────────────────────────────────────┘
```

### Key Regulations

| Regulation | Scope | Key Requirements |
|------------|-------|------------------|
| PCI-DSS | Card data | Encryption, access control, monitoring |
| GDPR | EU personal data | Consent, right to erasure, data portability |
| SOX | Financial reporting | Internal controls, audit trails |
| Basel III | Banking capital | Risk management, reporting |
| AML/KYC | Financial crime | Customer verification, transaction monitoring |
| PSD2 | EU payments | Strong authentication, open banking |

### Data Classification

| Level | Examples | Controls |
|-------|----------|----------|
| Public | Marketing materials | None |
| Internal | Policies, procedures | Basic access control |
| Confidential | Customer PII | Encryption, logging |
| Highly Confidential | PINs, passwords, keys | HSM, strict access |

## Code Example: Security Implementation

```sql
-- User Authentication Table
CREATE TABLE user_accounts (
    user_id UUID PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,  -- Argon2 or bcrypt
    mfa_enabled BOOLEAN DEFAULT FALSE,
    mfa_secret_encrypted BYTEA,
    status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE',
    failed_login_attempts INTEGER DEFAULT 0,
    locked_until TIMESTAMP WITH TIME ZONE,
    password_changed_at TIMESTAMP WITH TIME ZONE,
    last_login_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Roles and Permissions
CREATE TABLE roles (
    role_id UUID PRIMARY KEY,
    role_name VARCHAR(100) NOT NULL UNIQUE,
    description VARCHAR(500),
    is_system_role BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE permissions (
    permission_id UUID PRIMARY KEY,
    permission_code VARCHAR(100) NOT NULL UNIQUE,
    permission_name VARCHAR(200) NOT NULL,
    resource_type VARCHAR(50) NOT NULL,
    action VARCHAR(50) NOT NULL
);

CREATE TABLE role_permissions (
    role_id UUID NOT NULL REFERENCES roles(role_id),
    permission_id UUID NOT NULL REFERENCES permissions(permission_id),
    PRIMARY KEY (role_id, permission_id)
);

CREATE TABLE user_roles (
    user_id UUID NOT NULL REFERENCES user_accounts(user_id),
    role_id UUID NOT NULL REFERENCES roles(role_id),
    assigned_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    assigned_by UUID REFERENCES user_accounts(user_id),
    PRIMARY KEY (user_id, role_id)
);

-- Audit Log
CREATE TABLE audit_logs (
    log_id UUID PRIMARY KEY,
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    user_id UUID REFERENCES user_accounts(user_id),
    session_id VARCHAR(100),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id VARCHAR(100),
    old_value JSONB,
    new_value JSONB,
    ip_address INET,
    user_agent VARCHAR(500),
    status VARCHAR(20) NOT NULL,  -- SUCCESS, FAILURE
    failure_reason VARCHAR(500)
);

-- Sensitive Data (Encrypted)
CREATE TABLE encrypted_data (
    data_id UUID PRIMARY KEY,
    reference_type VARCHAR(50) NOT NULL,
    reference_id VARCHAR(100) NOT NULL,
    field_name VARCHAR(100) NOT NULL,
    encrypted_value BYTEA NOT NULL,
    key_version INTEGER NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    CONSTRAINT unique_encrypted_field UNIQUE (reference_type, reference_id, field_name)
);

-- API Keys
CREATE TABLE api_keys (
    key_id UUID PRIMARY KEY,
    key_hash VARCHAR(255) NOT NULL,  -- SHA-256 hash of key
    key_prefix VARCHAR(10) NOT NULL, -- First 10 chars for identification
    name VARCHAR(200) NOT NULL,
    owner_id UUID NOT NULL,
    scopes JSONB NOT NULL,
    rate_limit INTEGER DEFAULT 1000,
    expires_at TIMESTAMP WITH TIME ZONE,
    last_used_at TIMESTAMP WITH TIME ZONE,
    status VARCHAR(20) DEFAULT 'ACTIVE',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### Data Masking Example

```typescript
interface MaskingRule {
  pattern: RegExp;
  replacement: (match: string) => string;
}

const maskingRules: Record<string, MaskingRule> = {
  accountNumber: {
    pattern: /(\d{4})(\d+)(\d{4})/,
    replacement: (_, first, middle, last) => `${first}${'*'.repeat(middle.length)}${last}`
  },
  email: {
    pattern: /(.{2})(.*)(@.*)/,
    replacement: (_, start, middle, domain) => `${start}${'*'.repeat(middle.length)}${domain}`
  },
  phone: {
    pattern: /(\d{3})(\d+)(\d{2})/,
    replacement: (_, start, middle, end) => `${start}${'*'.repeat(middle.length)}${end}`
  },
  ssn: {
    pattern: /(\d{3})-?(\d{2})-?(\d{4})/,
    replacement: () => '***-**-****'
  }
};

function maskSensitiveData(value: string, type: keyof typeof maskingRules): string {
  const rule = maskingRules[type];
  if (!rule) return value;
  return value.replace(rule.pattern, rule.replacement);
}

// Examples:
// maskSensitiveData('1234567890123456', 'accountNumber') => '1234********3456'
// maskSensitiveData('john.doe@email.com', 'email') => 'jo*****@email.com'
// maskSensitiveData('555-123-4567', 'phone') => '555*****67'
```

### Security Headers Configuration

```json
{
  "securityHeaders": {
    "Content-Security-Policy": "default-src 'self'; script-src 'self'; style-src 'self'",
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "X-XSS-Protection": "1; mode=block",
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains",
    "Cache-Control": "no-store, no-cache, must-revalidate",
    "Pragma": "no-cache",
    "Referrer-Policy": "strict-origin-when-cross-origin"
  }
}
```

## Key Takeaways

1. Defense in depth provides multiple security layers
2. Strong authentication requires multiple factors
3. RBAC enables granular access control
4. Data protection requires encryption and masking
5. Audit logging is essential for compliance and forensics

## Further Reading

- OWASP Banking Security Guidelines
- PCI-DSS Requirements
- NIST Cybersecurity Framework
- ISO 27001 Information Security

## Next Lesson

[Lesson 12: Data Management](../lesson-12-data-management/README.md)
