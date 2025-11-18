# Privacy Policy

**Effective Date:** November 18, 2025  
**Last Updated:** November 18, 2025

## Introduction

Conducktr Inc. ("we," "our," or "us") is committed to protecting your privacy. This Privacy Policy explains how we collect, use, disclose, and safeguard your information when you use our Model Context Protocol (MCP) integration platform.

## Information We Collect

### 1. Account Information

When you register for Conducktr, we collect:
- **Email address** - For account identification and communication
- **Display name** - For personalization
- **Password** - Stored as a bcrypt hash, never in plain text
- **Organization name** - For workspace organization
- **Persona selection** - Your role (Admin, Ops, Builder, Approver)

### 2. Connection Credentials

For MCP server integrations, we collect and store:
- **OAuth tokens** - Encrypted access and refresh tokens
- **API keys** - Encrypted authentication credentials
- **Client IDs and secrets** - Encrypted OAuth application credentials
- **Service principal credentials** - For Entra ID integrations

**Encryption:** All credentials are encrypted at rest using Fernet (AES-256) encryption with regularly rotated keys.

### 3. Usage Information

We automatically collect:
- **API call logs** - Endpoints called, timestamps, response times
- **Tool execution data** - Which MCP tools were used and when
- **Cost and usage metrics** - For billing and monitoring
- **Error logs** - For debugging and service improvement
- **Audit events** - For security and compliance

### 4. Technical Information

- **IP addresses** - For security and rate limiting
- **Browser type and version** - For compatibility
- **Device information** - For responsive design
- **Session cookies** - For authentication
- **Performance metrics** - For service optimization

### 5. Third-Party Data

When you connect third-party services (like Linear, GitHub, etc.):
- We receive data from those services per your authorization
- We only request minimum necessary scopes
- We do not access data beyond what you authorize

## How We Use Your Information

### Primary Uses

1. **Service Delivery**
   - Authenticate and authorize API requests
   - Execute MCP tool calls on your behalf
   - Enforce policies and approval workflows
   - Provide usage analytics and monitoring

2. **Account Management**
   - Create and maintain your account
   - Send service notifications and updates
   - Process billing and subscriptions
   - Provide customer support

3. **Security & Compliance**
   - Detect and prevent fraud
   - Monitor for security threats
   - Maintain audit logs for compliance
   - Respond to legal obligations

4. **Service Improvement**
   - Analyze usage patterns
   - Improve performance and reliability
   - Develop new features
   - Debug and troubleshoot issues

### We Do NOT:

- ❌ Sell your personal data to third parties
- ❌ Use your data to train AI models
- ❌ Share your credentials with anyone
- ❌ Access your third-party accounts without authorization
- ❌ Send unsolicited marketing emails (we only send transactional emails)

## Data Sharing & Disclosure

### When We Share Your Data

1. **With Your Consent**
   - When you explicitly authorize data sharing
   - When you connect third-party MCP servers

2. **Service Providers**
   - Cloud infrastructure (Google Cloud Platform)
   - Payment processing (Stripe)
   - Email delivery (SendGrid)
   - Monitoring and analytics (internal only)

3. **Legal Requirements**
   - To comply with applicable laws
   - To respond to legal process (subpoenas, court orders)
   - To protect our rights and property
   - To prevent fraud or security threats

4. **Business Transfers**
   - In connection with a merger, acquisition, or sale of assets
   - Users will be notified via email and privacy policy update

### Data Minimization

We share only the minimum data necessary:
- Service providers receive only data needed for their specific function
- Third-party MCP servers receive only data you authorize
- Legal requests are carefully reviewed for necessity and validity

## Data Storage & Security

### Where We Store Data

**Primary Region:** United States (Google Cloud Platform)

**Data Residency Options:**
- Enterprise customers can request EU data residency
- GDPR compliance for EU users regardless of storage location

### Security Measures

**Technical Safeguards:**
- Encryption at rest (AES-256)
- Encryption in transit (TLS 1.3)
- Multi-key encryption for credential vault
- Regular security audits and penetration testing
- Automated vulnerability scanning

**Organizational Safeguards:**
- Access controls and role-based permissions
- Employee background checks
- Security awareness training
- Incident response procedures
- SOC 2 compliance (in progress)

**Network Security:**
- DDoS protection
- Web Application Firewall (WAF)
- Rate limiting and throttling
- Private networking between services

### Data Breach Notification

In the event of a data breach:
- We will notify affected users within 72 hours
- Notification will include nature of breach and remediation steps
- We will report to relevant authorities as required by law
- We maintain cyber insurance for breach response

## Data Retention

### Account Data

- **Active accounts:** Retained while account is active
- **Inactive accounts:** Deleted after 2 years of inactivity
- **Deleted accounts:** Purged within 30 days of deletion request

### Usage Logs

- **API call logs:** 90 days (configurable for enterprise)
- **Audit events:** 1 year (7 years for enterprise with compliance requirements)
- **Error logs:** 30 days
- **Aggregated analytics:** Indefinitely (anonymized)

### Credentials

- **OAuth tokens:** Until revoked or expired
- **API keys:** Until manually deleted
- **Revoked credentials:** Immediately purged from vault

### Backups

- **Database backups:** 30 days
- **Deleted data:** Removed from backups after retention period
- **Backup restoration:** Respects current deletion requests

## Your Rights

### Access & Portability

You have the right to:
- **Access your data** - Download your account information
- **Export your data** - Receive data in machine-readable format (JSON)
- **View audit logs** - See all actions taken in your workspace

**How to exercise:** Settings → Privacy → Download My Data

### Correction & Deletion

You have the right to:
- **Update your information** - Edit profile, email, organization name
- **Delete connections** - Remove third-party integrations
- **Delete your account** - Complete account deletion with data purge

**How to exercise:** Settings → Account → Delete Account

### Restrict Processing

You have the right to:
- **Limit data collection** - Opt out of optional telemetry
- **Disable features** - Turn off analytics, recommendations
- **Revoke authorizations** - Disconnect third-party services

**How to exercise:** Settings → Privacy → Data Collection

### Object to Processing

You have the right to:
- **Object to automated decisions** - Request manual review of policy denials
- **Opt out of profiling** - Disable usage pattern analysis

**How to exercise:** Contact privacy@conducktr.com

### GDPR Rights (EU Users)

EU users have additional rights under GDPR:
- Right to erasure ("right to be forgotten")
- Right to data portability
- Right to withdraw consent
- Right to lodge a complaint with supervisory authority

**EU Representative:** gdpr@conducktr.com

### CCPA Rights (California Users)

California users have rights under CCPA:
- Right to know what data is collected
- Right to know if data is sold (we do not sell data)
- Right to opt out of sale (not applicable)
- Right to non-discrimination

**California Contact:** ccpa@conducktr.com

## Children's Privacy

Conducktr is not intended for children under 13 (or 16 in the EU). We do not knowingly collect data from children. If you believe we have collected data from a child, contact us immediately at privacy@conducktr.com.

## Cookies & Tracking

### Essential Cookies

**Authentication Cookie:** `conducktr_session`
- **Purpose:** Maintain user session
- **Duration:** 1 hour (configurable)
- **Type:** httpOnly, Secure, SameSite=Lax

### Analytics (Optional)

We use minimal analytics:
- **First-party only** - No third-party trackers
- **Anonymized** - IP addresses hashed
- **Opt-out available** - Settings → Privacy → Analytics

### Do Not Track

We respect browser Do Not Track (DNT) signals by:
- Disabling optional analytics
- Minimizing data collection
- Not sharing data with third parties

## International Data Transfers

### Data Transfer Mechanisms

**For EU Users:**
- Standard Contractual Clauses (SCCs)
- GDPR Article 49 derogations (explicit consent)
- Data Processing Agreements available

**For Other Regions:**
- Appropriate safeguards in place
- Local data residency options (enterprise)

## Third-Party Services

### MCP Server Providers

When you connect third-party MCP servers:
- Their privacy policies apply to data they collect
- We are not responsible for their data practices
- Review their privacy policy before connecting

**Examples:** Linear, GitHub, Slack, etc.

### Service Providers

We use these third-party services:
- **Google Cloud Platform** - Infrastructure ([Privacy Policy](https://cloud.google.com/privacy))
- **Stripe** - Payment processing ([Privacy Policy](https://stripe.com/privacy))
- **SendGrid** - Email delivery ([Privacy Policy](https://www.twilio.com/legal/privacy))

## Changes to This Policy

**Notification of Changes:**
- Email notification to all users
- Banner notification in dashboard
- Updated "Last Modified" date at top of policy

**Material Changes:**
- 30-day notice before taking effect
- Option to delete account if you disagree

## Contact Us

**Privacy Questions:** privacy@conducktr.com  
**Data Subject Requests:** dsr@conducktr.com  
**GDPR (EU):** gdpr@conducktr.com  
**CCPA (California):** ccpa@conducktr.com  
**General Support:** support@conducktr.com

**Mailing Address:**  
Conducktr Inc.  
Privacy Department  
[Address]  
[City, State, ZIP]  
United States

**Data Protection Officer:** dpo@conducktr.com

---

**Document Version:** 1.0  
**Effective Date:** November 18, 2025  
**Last Review:** November 18, 2025
