# ChatGPT Integration Guide

**Connecting ChatGPT to Conducktor's MCP Gateway**

This guide explains how to configure ChatGPT to use Conducktor as its Model Context Protocol (MCP) server, giving ChatGPT access to all your connected services through a centralized, governed gateway.

## Overview

ChatGPT's MCP support allows it to interact with external services through MCP servers. By using Conducktor as your MCP gateway, ChatGPT gains:

- **Access to 50+ Services** - Linear, GitHub, Slack, Notion, and more through a single connection
- **Enterprise Governance** - Policy enforcement, approval workflows, and audit logging
- **Cost Controls** - Budget limits and spending alerts for all AI-driven API calls
- **Centralized Management** - One connection to manage, not dozens

## Prerequisites

- [ ] A ChatGPT account (ChatGPT Plus or Enterprise)
- [ ] A Conducktor account
- [ ] At least one MCP connection configured in Conducktor (e.g., Linear, GitHub, Slack)

## Quick Start

### Step 1: Get Your Conducktor MCP Endpoint

Your unique MCP endpoint is displayed in your Conducktor dashboard:

```
https://api.conducktor.com/api/conducktor-mcp
```

This endpoint provides access to:
- All your connected MCP servers
- Conducktor management tools
- Policy enforcement layer

### Step 2: Configure ChatGPT

**For ChatGPT Plus:**

1. Open ChatGPT Settings → Extensions → Model Context Protocol
2. Click "Add MCP Server"
3. Configure:
   - **Name:** `Conducktor Gateway`
   - **Server URL:** `https://api.conducktor.com/api/conducktor-mcp`
   - **Authentication:** OAuth 2.0
   - **Client ID:** `chatgpt-mcp-client`
4. Click "Connect"

**For ChatGPT Enterprise:**

1. Contact your ChatGPT Enterprise administrator
2. Provide them with:
   - MCP Server URL: `https://api.conducktor.com/api/conducktor-mcp`
   - OAuth Client ID: `chatgpt-mcp-client`
   - Documentation: This guide
3. Administrator configures organization-wide MCP server

### Step 3: Authorize Connection

1. ChatGPT redirects you to Conducktor's authorization page
2. Log in to your Conducktor account (or create one)
3. Review the requested permissions:
   - Access to your MCP connections
   - Execute tools on your behalf
   - View usage analytics
4. Click **Authorize**
5. You'll be redirected back to ChatGPT
6. Connection status shows **Connected**

### Step 4: Test the Connection

In ChatGPT, try a command that uses one of your connected services:

**Example with Linear:**
```
"Show me my open issues in Linear"
```

ChatGPT will:
1. Detect you want to use Linear
2. Call Conducktr's MCP gateway
3. Execute `linear-prod_search_issues` tool
4. Return results formatted in chat

## Available Tools

Once connected, ChatGPT can use tools from:

### Conducktr Management Tools

**Connection Management:**
- `conducktr_list_connections` - List all your MCP connections
- `conducktr_create_connection` - Add new MCP server
- `conducktr_get_connection_health` - Check connection status

**Workflow Management:**
- `conducktr_list_workflows` - List automation workflows
- `conducktr_create_workflow` - Create new workflow
- `conducktr_execute_workflow` - Run workflow manually

**Policy & Approvals:**
- `conducktr_list_policies` - View governance policies
- `conducktr_create_policy` - Add policy rule
- `conducktr_list_approvals` - Check pending approvals
- `conducktr_approve_workflow_run` - Approve pending operation

**Analytics:**
- `conducktr_get_usage_stats` - View API usage and costs

### Connected Service Tools

All tools from your connected MCP servers are automatically available with connection name prefix:

**Linear (if connected):**
- `linear-prod_search_issues`
- `linear-prod_create_issue`
- `linear-prod_update_issue`

**GitHub (if connected):**
- `github-main_list_repos`
- `github-main_create_issue`
- `github-main_create_pr`

**Slack (if connected):**
- `slack-alerts_post_message`
- `slack-alerts_list_channels`

ChatGPT automatically discovers and uses these tools based on your natural language requests.

## Example Use Cases

### 1. Issue Tracking

**You:** "Create a Linear issue for the login bug"

**ChatGPT:**
1. Calls `linear-prod_create_issue`
2. Sets title, description, priority
3. Responds: "Created issue ENG-456: Fix login bug"

### 2. Code Management

**You:** "What pull requests are open in my GitHub repo?"

**ChatGPT:**
1. Calls `github-main_list_prs`
2. Filters by status: open
3. Returns list with PR titles and authors

### 3. Team Communication

**You:** "Send a Slack message to #engineering about the deployment"

**ChatGPT:**
1. Calls `slack-alerts_post_message`
2. Formats message with context
3. Confirms: "Message sent to #engineering"

### 4. Workflow Automation

**You:** "Run my daily standup workflow"

**ChatGPT:**
1. Calls `conducktr_list_workflows`
2. Finds "Daily Standup" workflow
3. Calls `conducktr_execute_workflow`
4. Reports workflow status

### 5. Usage Monitoring

**You:** "How much have I spent on API calls this month?"

**ChatGPT:**
1. Calls `conducktr_get_usage_stats`
2. Aggregates spending by connection
3. Shows: "Total: $23.45 (Linear: $12, GitHub: $8, Slack: $3.45)"

## Policy & Governance

Conducktr enforces policies even when accessed through ChatGPT:

### Example: Approval Required

**Policy:** "All Linear issue creations require approval"

**What Happens:**
1. You ask ChatGPT to create an issue
2. ChatGPT calls Conducktr
3. Conducktr detects policy rule
4. Creates approval request instead of executing
5. ChatGPT responds: "Approval required. Request sent to Admin team."
6. Admins review in Conducktr dashboard
7. Once approved, issue is created

### Example: Budget Limit

**Policy:** "Monthly budget limit: $100"

**What Happens:**
1. You ask ChatGPT to perform operation
2. Conducktr checks current spend: $95
3. Operation cost: $10
4. Budget would exceed limit
5. Conducktr blocks operation
6. ChatGPT responds: "Operation blocked: Monthly budget limit reached ($100)"

### Example: Tool Allowlist

**Policy:** "Only allow read operations for contractors"

**What Happens:**
1. Contractor asks ChatGPT to update issue
2. Conducktr checks policy rules
3. Update tools not in allowlist
4. Conducktr denies operation
5. ChatGPT responds: "Permission denied: Update operations not allowed"

## Authentication & Security

### OAuth 2.0 Flow

**Initial Authorization:**
```
ChatGPT → Conducktr → User Authorization → Token Issued
```

**Token Lifespan:**
- Access token: 60 minutes
- Refresh automatically by ChatGPT
- Revocable from Conducktr dashboard

### Security Features

**Encrypted Communication:**
- TLS 1.3 for all requests
- Certificate pinning
- HSTS enabled

**Token Security:**
- Tokens never logged or exposed
- httpOnly cookies when applicable
- Automatic expiration and refresh

**Audit Trail:**
- Every ChatGPT request logged
- User attribution maintained
- Export to SIEM for compliance

## Troubleshooting

### ChatGPT Shows "MCP Connection Failed"

**Cause:** Network connectivity or authorization issue

**Solution:**
1. Check Conducktr status: [status.conducktr.com](https://status.conducktr.com)
2. Verify your account is active: [app.conducktr.com](https://app.conducktr.com)
3. Re-authorize connection in ChatGPT settings
4. Contact support if persistent: support@conducktr.com

### ChatGPT Can't Find Specific Tools

**Cause:** Connection not active or tools not synced

**Solution:**
1. Log in to Conducktr dashboard
2. Go to Connections → Your Service Connection
3. Verify status is **ACTIVE**
4. Click **Test Connection** to refresh tools
5. Return to ChatGPT and try again

### Tool Execution Returns Error

**Cause:** Policy blocked, invalid credentials, or service unavailable

**Solution:**
1. Check error message for specific cause
2. Review policies in Conducktr dashboard
3. Verify connection credentials haven't expired
4. Check service status (Linear, GitHub, etc.)
5. Review audit logs for detailed error

### Authorization Keeps Expiring

**Cause:** Token refresh failing or revoked access

**Solution:**
1. Check if Conducktr account is still active
2. Verify email address is confirmed
3. Re-authorize completely:
   - Remove MCP server from ChatGPT
   - Re-add with same URL and Client ID
   - Complete authorization flow again

## Advanced Configuration

### Custom Client ID (Enterprise)

Enterprise customers can request custom OAuth client IDs for:
- Branding in authorization pages
- Separate prod/dev/staging environments
- Enhanced audit logging

Contact: enterprise@conducktr.com

### Workspace Selection

If you have multiple Conducktr workspaces:
1. Authorization page shows workspace selector
2. Choose workspace for this ChatGPT integration
3. Different ChatGPT accounts can use different workspaces
4. Switch workspace by re-authorizing

### SSO Integration (Enterprise)

For ChatGPT Enterprise with Conducktr Enterprise:
- Single sign-on via Entra ID, Okta, or OneLogin
- No separate authorization step
- Automatic user provisioning
- Centralized access management

Contact: enterprise@conducktr.com

## Best Practices

### For Individual Users

**Be Specific:**
- "Create issue in Linear" vs. "Create issue" (clearer intent)
- Specify connection names when you have multiple

**Check Status First:**
- Ask "What connections do I have?" before complex requests
- Verify connection health if operations fail

**Review Before Executing:**
- Ask ChatGPT to "show me what you'll do" for destructive operations
- Approve manually when policies require it

### For Enterprise Deployments

**Centralized Management:**
- Configure Conducktr workspace for entire team
- Use shared connections for team services
- Set organization-wide policies

**Training:**
- Provide team with example ChatGPT prompts
- Document approved use cases
- Share policy guidelines

**Monitoring:**
- Review usage analytics weekly
- Set up budget alerts
- Monitor audit logs for anomalies

## Support & Resources

**Conducktr Resources:**
- Documentation: [docs.conducktr.com](https://docs.conducktr.com)
- Dashboard: [app.conducktr.com](https://app.conducktr.com)
- Status: [status.conducktr.com](https://status.conducktr.com)
- Support: support@conducktr.com

**ChatGPT Resources:**
- MCP Documentation: [platform.openai.com/docs/mcp](https://platform.openai.com/docs/mcp)
- Community: [community.openai.com](https://community.openai.com)
- Support: help.openai.com

## Appendix

### Connection Flow Diagram

```
ChatGPT ←→ Conducktr MCP Gateway ←→ Linear/GitHub/Slack/etc.
   ↓             ↓                         ↓
User      Policy Engine               External APIs
Request   + Audit Logs                (governed access)
```

### Supported ChatGPT Versions

| Version | MCP Support | Conducktr Compatible |
|---------|-------------|----------------------|
| ChatGPT Free | ❌ | N/A |
| ChatGPT Plus | ✅ | ✅ |
| ChatGPT Team | ✅ | ✅ |
| ChatGPT Enterprise | ✅ | ✅ (with SSO) |

### OAuth Client Credentials

**Standard Client (All Users):**
```
Client ID: chatgpt-mcp-client
Client Secret: (not required for PKCE flow)
Redirect URI: https://chat.openai.com/oauth/callback
```

**Enterprise Custom Client:**
Contact enterprise@conducktr.com for dedicated credentials

---

**Last Updated:** November 18, 2025  
**Version:** 1.0
