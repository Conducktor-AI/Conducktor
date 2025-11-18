# Security Policy

## Supported Versions

We actively maintain security updates for the following versions:

| Version | Supported          |
| ------- | ------------------ |
| 1.x.x   | :white_check_mark: |
| < 1.0   | :x:                |

## Security Practices

### Data Protection

**Encryption at Rest:**
- All credentials and secrets are encrypted using Fernet (AES-256-CBC)
- Database encryption for sensitive fields
- Multi-key encryption support for zero-downtime key rotation

**Encryption in Transit:**
- TLS 1.3 for all API communications
- Certificate pinning for production environments
- HSTS (HTTP Strict Transport Security) enabled

**Authentication:**
- OAuth 2.1 with PKCE (Proof Key for Code Exchange)
- JWT tokens with short expiration (60 minutes default)
- Refresh token rotation
- Multi-factor authentication support (coming soon)

### Access Controls

**Principle of Least Privilege:**
- Role-based access control (RBAC)
- Persona-based UI filtering (ADMIN, OPS, BUILDER, APPROVER)
- Workspace-level isolation
- Connection-level policy enforcement

**Audit Logging:**
- Comprehensive audit trail for all actions
- Immutable event logs
- Retention configurable per compliance requirements
- Export capabilities for SIEM integration

### Network Security

**Infrastructure:**
- Deployed on Google Cloud Platform (GKE)
- Private networking between services
- DDoS protection via Cloud Armor
- Web Application Firewall (WAF) enabled

**API Security:**
- Rate limiting per user/workspace
- Request size limits
- IP whitelisting for admin operations
- CORS policy enforcement

### Credential Management

**Vault Architecture:**
- Centralized credential vault with encryption
- Automatic credential rotation support
- Expiration tracking and renewal alerts
- Zero-knowledge credential storage

**OAuth Token Handling:**
- Tokens stored encrypted, never logged
- Automatic refresh before expiration
- Secure token revocation
- State parameter validation (CSRF protection)

## Reporting a Vulnerability

We take security seriously. If you discover a security vulnerability, please follow these steps:

### 1. DO NOT Create a Public Issue

Please **do not** create a public GitHub issue for security vulnerabilities. This helps protect Conducktr users from exploitation.

### 2. Report via Email

Send your vulnerability report to: **security@conducktr.com**

Include the following information:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if available)
- Your contact information

### 3. Expected Response Time

| Severity | Initial Response | Status Update | Fix Timeline |
|----------|------------------|---------------|--------------|
| Critical | 24 hours | Daily | 7 days |
| High | 48 hours | Every 3 days | 30 days |
| Medium | 5 days | Weekly | 60 days |
| Low | 10 days | Bi-weekly | 90 days |

### 4. Responsible Disclosure

We follow a responsible disclosure process:

1. **Acknowledge:** We confirm receipt within the timeframes above
2. **Investigate:** We investigate and validate the vulnerability
3. **Fix:** We develop and test a fix
4. **Notify:** We notify affected customers (if applicable)
5. **Disclose:** We publicly disclose after fix is deployed (with your credit, if desired)

### 5. Bug Bounty Program

We currently do not have a formal bug bounty program, but we:
- Acknowledge security researchers in our hall of fame
- Provide swag for valid security reports
- Offer references for security professionals
- Consider bounties for critical vulnerabilities on a case-by-case basis

## Security Features for Partners

### For MCP Server Providers (like Linear)

**OAuth 2.1 Requirements:**
- PKCE support (RFC 7636) - Required
- OAuth Discovery (RFC 8414) - Recommended
- Dynamic Client Registration (RFC 7591) - Optional
- Token refresh endpoint - Required

**Security Expectations:**
- HTTPS-only endpoints
- Valid SSL/TLS certificates
- Rate limiting on OAuth endpoints
- Webhook signature verification (if applicable)

**Data Handling:**
- Minimal data collection principle
- Clear data retention policies
- GDPR compliance for EU users
- User data deletion on request

### For AI Client Applications (like ChatGPT)

**Integration Security:**
- OAuth 2.0 for user authentication to Conducktr
- Token storage in secure, httponly cookies
- CORS properly configured
- No credential exposure in client-side code

**Best Practices:**
- Validate all MCP responses
- Implement timeout handling
- Use exponential backoff for retries
- Log security events (without sensitive data)

## Compliance & Certifications

### Current Status

- **GDPR:** Fully compliant
- **SOC 2 Type II:** In progress (expected Q1 2026)
- **HIPAA:** Available for BAA customers
- **ISO 27001:** Planned for 2026

### Data Processing Agreements

We offer Data Processing Agreements (DPAs) for:
- Enterprise customers
- GDPR-covered organizations
- HIPAA-covered entities
- Custom compliance requirements

Contact: legal@conducktr.com

## Security Updates & Advisories

**Notification Channels:**
- Email notifications to registered admin users
- Security advisories published at: security.conducktr.com
- Status page: status.conducktr.com
- RSS feed: status.conducktr.com/feed

**Severity Levels:**

| Level | Description | Example |
|-------|-------------|---------|
| **Critical** | Immediate action required | Remote code execution, credential leakage |
| **High** | Urgent update recommended | Authentication bypass, privilege escalation |
| **Medium** | Update when convenient | XSS, CSRF vulnerabilities |
| **Low** | Non-urgent, informational | Minor information disclosure |

## Third-Party Security

### Dependencies

We maintain security for our dependencies:
- Automated vulnerability scanning via Dependabot
- Regular dependency updates
- Security patches applied within SLA timeframes
- Bill of Materials (SBOM) available on request

### Infrastructure Providers

**Google Cloud Platform (GCP):**
- Shared responsibility model
- GCP Security Command Center enabled
- Regular infrastructure security audits
- Compliance certifications inherited

## Security Hardening Guide

For self-hosted deployments (enterprise customers):

### Environment Configuration

```bash
# Use strong encryption keys (minimum 32 bytes)
TOKEN_ENCRYPTION_KEY=<generate-with-cryptography.fernet>
JWT_SECRET_KEY=<generate-with-openssl-rand>

# Enforce HTTPS
FORCE_HTTPS=true

# Restrict CORS origins
ALLOW_ORIGINS=https://app.yourdomain.com

# Enable rate limiting
RATE_LIMIT_ENABLED=true
RATE_LIMIT_PER_MINUTE=60
```

### Network Security

- Deploy behind a reverse proxy (nginx, Cloudflare)
- Use private subnets for database
- Enable VPC peering for service communication
- Implement network policies in Kubernetes

### Database Security

- Use strong database passwords (minimum 32 characters)
- Enable SSL/TLS for database connections
- Restrict database access to application subnet
- Enable database audit logging

## Security Contact

**General Security Inquiries:** security@conducktr.com  
**Vulnerability Reports:** security@conducktr.com  
**Compliance Questions:** compliance@conducktr.com  
**Legal/DPA Requests:** legal@conducktr.com

**PGP Key:** Available at [keys.conducktr.com/security.asc](https://keys.conducktr.com/security.asc)

---

**Last Updated:** November 18, 2025  
**Version:** 1.0
