# Linear Integration Guide

**Integrating Linear's MCP Server with Conducktor**

This guide explains how to connect Linear's Model Context Protocol (MCP) server to Conducktor, enabling AI applications like ChatGPT to interact with your Linear workspace through a centralized, governed gateway.

## Overview

Linear provides an MCP server that allows AI applications to:
- Search and read issues
- Create and update issues
- Query projects and teams
- Manage labels and workflows
- Access roadmaps and cycles

By connecting Linear to Conducktor, you gain:
- **Centralized governance** - Policy enforcement and approval workflows
- **Cost controls** - Budget limits and spending alerts
- **Audit logging** - Complete trail of all Linear operations
- **Multi-user access** - Team-wide access with role-based permissions

## Prerequisites

Before you begin, ensure you have:
- [ ] A Conducktor account
- [ ] A Linear workspace with admin access
- [ ] OAuth application credentials from Linear (or use automatic registration)

## Integration Methods

Linear MCP supports two authentication methods:

### Method 1: OAuth 2.1 (Recommended)

OAuth 2.1 with PKCE provides secure, user-authorized access to Linear data.

**Advantages:**
- Scoped permissions (read/write control)
- Token refresh for long-lived access
- User attribution in Linear audit logs
- Revocable access

**Use when:** You want AI applications to act on behalf of specific users

### Method 2: API Key

Personal API keys provide direct access with your user's permissions.

**Advantages:**
- Simpler setup
- No OAuth flow required
- Works for automation

**Use when:** You want service-level access or testing

## Setup Instructions

### Option A: OAuth 2.1 with Pre-Configured Credentials

**Step 1: Create Linear OAuth Application**

1. Go to Linear Settings → API → OAuth Applications
2. Click "New OAuth Application"
3. Configure:
   - **Name:** `Conducktor MCP Gateway`
   - **Description:** `Enterprise MCP integration platform`
   - **Callback URLs:** `https://app.conducktor.com/oauth/callback`
   - **Scopes:** Select `read`, `write` (as needed)
4. Save and copy your **Client ID** and **Client Secret**

**Step 2: Add Linear Connection in Conducktor**

1. Log in to [Conducktor Dashboard](https://app.conducktor.com)
2. Navigate to **Connections** → **Catalog**
3. Find "Linear MCP" and click **Deploy**
4. In the wizard:
   - **Name:** `Linear Production` (or your preferred name)
   - **Client ID:** Paste from Step 1
   - **Client Secret:** Paste from Step 1
   - **Scopes:** `read`, `write` (must match Linear OAuth app)
5. Click **Deploy Now**

**Step 3: Authorize Connection**

1. A new tab opens with Linear's authorization page
2. Review the requested permissions
3. Click **Authorize**
4. You'll be redirected back to Conducktor
5. Connection status changes to **ACTIVE**

### Option B: OAuth 2.1 with Dynamic Registration

Linear supports RFC 7591 Dynamic Client Registration, allowing automatic OAuth app creation.

**Step 1: Deploy Connection with Empty Credentials**

1. Log in to [Conducktor Dashboard](https://app.conducktor.com)
2. Navigate to **Connections** → **Catalog**
3. Find "Linear MCP" and click **Deploy**
4. In the wizard:
   - **Name:** `Linear Production`
   - **Client ID:** *(leave empty)*
   - **Client Secret:** *(leave empty)*
5. Click **Deploy Now**

**Step 2: Automatic Registration & Authorization**

1. Conducktor automatically registers an OAuth application with Linear
2. Linear's authorization page opens
3. Review permissions and click **Authorize**
4. Connection status changes to **ACTIVE**

### Option C: API Key (Quick Start)

**Step 1: Generate Linear API Key**

1. Go to Linear Settings → API → Personal API Keys
2. Click "Create new key"
3. Give it a name: `Conducktor MCP Gateway`
4. Copy the generated key (starts with `lin_api_`)

**Step 2: Add Connection in Conducktor**

1. Log in to [Conducktor Dashboard](https://app.conducktor.com)
2. Navigate to **Connections** → **Catalog**
3. Find "Linear RestAPI" (API Key version)
4. Click **Deploy**
5. Paste your API key
6. Click **Deploy Now**
7. Connection status changes to **ACTIVE** immediately

## Configuration

### MCP Server Endpoint

Linear's MCP server is hosted at:
```
https://api.linear.app
```

This endpoint provides:
- OAuth discovery (RFC 8414): `/.well-known/oauth-authorization-server`
- MCP manifest: `/.well-known/mcp-manifest.json`
- Tool endpoints: `/mcp/*`

### Available Tools

Once connected, Conducktr exposes Linear tools with the prefix `linear-prod_` (or your connection name):

**Issue Management:**
- `linear-prod_search_issues` - Search issues with filters
- `linear-prod_get_issue` - Get issue details by ID
- `linear-prod_create_issue` - Create a new issue
- `linear-prod_update_issue` - Update issue fields
- `linear-prod_add_comment` - Add comment to issue

**Project Management:**
- `linear-prod_list_projects` - List all projects
- `linear-prod_list_teams` - List teams in workspace
- `linear-prod_list_cycles` - List active cycles
- `linear-prod_get_roadmap` - Get roadmap view

**Workflow:**
- `linear-prod_list_states` - List workflow states
- `linear-prod_update_state` - Move issue to different state
- `linear-prod_list_labels` - List available labels
- `linear-prod_apply_label` - Apply label to issue

### Permissions & Scopes

**OAuth Scopes:**

| Scope | Permissions | Recommended For |
|-------|-------------|-----------------|
| `read` | Read issues, projects, teams | Reporting, search, analytics |
| `write` | Create/update issues, comments | Full issue management |
| `admin` | Workspace settings, API config | Internal tools only |

**API Key Permissions:**
- Inherits all permissions from the user who created the key
- Cannot be scoped down
- Use dedicated service account for production

## Testing Your Connection

### Using Conducktr Dashboard

1. Navigate to **Connections** → **Your Linear Connection**
2. Click **Test Connection**
3. You'll see:
   - Health status (HEALTHY/DEGRADED/ERROR)
   - Available tools count
   - Token expiration (for OAuth)
   - Last successful call timestamp

### Using MCP Inspector

For detailed testing:

```bash
npx @modelcontextprotocol/inspector
```

Connect to:
```
URL: https://api.conducktr.com/api/conducktr-mcp
Auth: OAuth 2.0 (Client ID: chatgpt-mcp-client)
```

Test Linear tools:
```json
{
  "tool": "linear-prod_search_issues",
  "params": {
    "query": "status:in-progress",
    "limit": 10
  }
}
```

### Example: Search Issues

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "linear-prod_search_issues",
    "arguments": {
      "query": "assignee:me",
      "team": "ENG"
    }
  }
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Found 5 issues:\n\n1. ENG-123: Fix login bug (In Progress)\n2. ENG-124: Add OAuth support (To Do)\n..."
      }
    ]
  }
}
```

## Policy Configuration

Add governance policies to your Linear connection:

### Example: Approval Required for Issue Creation

```yaml
Policy Name: Require Approval for Linear Issue Creation
Connection: Linear Production
Rules:
  - Tool Pattern: "*_create_issue"
  - Action: REQUIRE_APPROVAL
  - Approvers: [Admin, Ops]
  - Reason: "Prevent accidental issue creation by AI"
```

### Example: Budget Limit

```yaml
Policy Name: Monthly Linear API Budget
Connection: Linear Production
Rules:
  - Budget Limit: $50/month
  - Action: ALERT_AND_BLOCK
  - Alert Threshold: 80%
  - Notification Channel: Slack #alerts
```

### Example: Tool Allowlist

```yaml
Policy Name: Read-Only Linear Access
Connection: Linear Production
Rules:
  - Allowed Tools:
      - "*_search_issues"
      - "*_get_issue"
      - "*_list_projects"
  - Denied Tools: "*_create_*", "*_update_*"
  - Action: DENY
```

## Troubleshooting

### Connection Status: PENDING_AUTH

**Cause:** OAuth authorization not completed

**Solution:**
1. Go to Connections → Your Linear Connection
2. Click **Re-authorize**
3. Complete OAuth flow

### Connection Status: ERROR

**Cause:** OAuth token expired or revoked

**Solution:**
1. Check token expiration in connection details
2. Click **Re-authorize** to refresh token
3. For API keys, generate a new key in Linear

### Tool Calls Return 401 Unauthorized

**Cause:** Invalid credentials or expired token

**Solution:**
1. Verify connection status is **ACTIVE**
2. Check token expiration date
3. Re-authorize if expired
4. For API keys, verify key hasn't been revoked in Linear

### Tool Calls Return 403 Forbidden

**Cause:** Insufficient permissions for requested operation

**Solution:**
1. Check OAuth scopes - ensure `write` scope for create/update operations
2. For API keys, verify user has appropriate Linear permissions
3. Review Linear audit logs for permission denials

### Dynamic Registration Fails

**Cause:** Linear's registration endpoint not responding

**Solution:**
1. Fall back to pre-configured credentials (Option A)
2. Check Linear API status: [status.linear.app](https://status.linear.app)
3. Contact Linear support if persistent

## Security Best Practices

### For Production Deployments

**Use OAuth 2.1 (not API Keys):**
- Scoped permissions
- Token expiration and rotation
- User attribution in audit logs

**Rotate Credentials Regularly:**
- OAuth secrets: Every 90 days
- API keys: Monthly (or use OAuth instead)

**Enable Audit Logging:**
- Track all Linear operations
- Export logs to SIEM
- Monitor for anomalies

**Apply Least Privilege:**
- Request only needed scopes
- Use read-only access when possible
- Separate connections for different use cases

**Monitor Token Expiration:**
- Set up alerts for expiring tokens
- Auto-refresh for OAuth (enabled by default in Conducktr)
- Keep backup API key for emergency access

## Support & Resources

**Linear Documentation:**
- MCP Server: [linear.app/docs/mcp](https://linear.app/docs/mcp)
- OAuth Guide: [linear.app/docs/oauth](https://linear.app/docs/oauth)
- API Reference: [linear.app/docs/api](https://linear.app/docs/api)

**Conducktr Resources:**
- Documentation: [docs.conducktr.com](https://docs.conducktr.com)
- Community: [community.conducktr.com](https://community.conducktr.com)
- Support: support@conducktr.com

**Linear Support:**
- Support: support@linear.app
- AI Client Partner Program: partners@linear.app

## Appendix

### OAuth Flow Diagram

```
User → Conducktr → Linear OAuth
  ↓         ↓           ↓
Deploy  Configure  Authorize
  ↓         ↓           ↓
Create  Register  Grant Access
  ↓         ↓           ↓
Initiate  Redirect  Callback
  ↓         ↓           ↓
Code    Exchange  Active ✓
```

### API Key vs OAuth Comparison

| Feature | API Key | OAuth 2.1 |
|---------|---------|-----------|
| Setup Complexity | Low | Medium |
| Security | Medium | High |
| Scoped Permissions | ❌ | ✅ |
| User Attribution | ❌ | ✅ |
| Token Refresh | ❌ | ✅ |
| Revocation | Manual | Automatic |
| Recommended For | Testing | Production |

### Compliance Considerations

**GDPR:**
- OAuth provides user consent tracking
- API keys lack user attribution
- Enable audit logging for data access

**SOC 2:**
- OAuth preferred for access controls
- Credential rotation required
- Enable Conducktr's audit exports

**HIPAA:**
- Use OAuth with workspace isolation
- Enable encryption at rest
- Sign BAA with Conducktr

---

**Last Updated:** November 18, 2025  
**Version:** 1.0
