# Security Policy

Conducktor is a hosted service. Fixes are deployed to the service; there is no version for
you to install or upgrade.

## Reporting a vulnerability

### Do not open a public issue

Please do not report a vulnerability in a public GitHub issue, pull request or discussion.
That exposes Conducktor users before a fix exists.

### Email us

Send the report to **security@conducktor.com**. Include:

- what the vulnerability is and where it is (URL, endpoint, MCP tool, or document);
- steps to reproduce it;
- what an attacker could do with it;
- a suggested fix, if you have one;
- how to reach you.

Do not access, change or delete data that is not yours while you investigate, and stop as
soon as you have shown the issue.

### What happens next

| Severity | First response | Status updates | Fix target |
|----------|----------------|----------------|------------|
| Critical | 24 hours | Daily | 7 days |
| High | 48 hours | Every 3 days | 30 days |
| Medium | 5 days | Weekly | 60 days |
| Low | 10 days | Every 2 weeks | 90 days |

1. **Acknowledge.** We confirm we received the report, within the times above.
2. **Investigate.** We reproduce and assess it.
3. **Fix.** We build, test and deploy a fix.
4. **Notify.** We tell affected customers, where that applies.
5. **Disclose.** We disclose the issue publicly after the fix is deployed, and credit you
   if you want to be credited.

We do not run a formal bug bounty program.

## How Conducktor protects your data

### Credentials and secrets

- Credentials you give Conducktor (OAuth tokens, connector API keys, your own model keys)
  are stored encrypted with Fernet (AES-128-CBC with an HMAC-SHA256 check). Several
  encryption keys can be active at once, so keys can be rotated without losing access to
  existing secrets.
- Account passwords are stored as bcrypt hashes, never in plain text.
- A secret you set on a Build cannot be read back through the API; reads return a masked
  preview.

### Accounts and workspaces

- Each account owns exactly one workspace, and every workspace has a single owner. Your
  data, connections and Builds stay inside your workspace.
- Requests to the REST API carry a user-session bearer token. MCP clients get OAuth access
  tokens through the authorization code flow, with PKCE (S256) supported; tokens can be
  revoked (RFC 7009) and introspected (RFC 7662).
- When you connect a service over OAuth, Conducktor checks the `state` parameter on the
  callback and uses PKCE (S256) with providers that support it.

### Builds

- Launched Builds run in an isolated, workspace-scoped environment.
- On the default launch path a Build's container runs as a non-root user with all Linux
  capabilities dropped and no privilege escalation.
- A Build's token (`CONDUCKTOR_APP_TOKEN`) is scoped to that Build in that workspace. One
  Build cannot read another Build's data, and the token is accepted only by the data store
  and the platform services the user has allowed that Build.
- A Build calls the workspace's connectors by reference. It never holds their credentials;
  Conducktor adds them on its side and replaces any echo of them in the answer with
  `[redacted]`.

### API protection

- Requests are rate limited per caller, with tighter limits on sign-in, registration and
  OAuth endpoints.
- Request bodies are size-limited.
- Browser access is limited to an allowlist of origins (CORS).
- In production every response carries HSTS (`Strict-Transport-Security`).

### Governance and audit

- Tool calls go through Conducktor's governance checks before they run: the caller's
  permission, rate limits and argument validation.
- Tool calls are audit-logged and recorded in the workspace's ledger.

## For MCP server providers

If users connect your MCP server or API to Conducktor:

- serve every endpoint over HTTPS;
- support OAuth 2.1 with PKCE (S256) and token refresh;
- publish RFC 8414 authorization server metadata, so Conducktor can discover your
  endpoints and PKCE support;
- optionally support RFC 7591 Dynamic Client Registration;
- rate-limit your OAuth endpoints;
- verify signatures on webhooks you send or receive.

See the [OAuth guide](docs/authentication/OAUTH_GUIDE.md).

## For AI clients connecting to Conducktor

- Use the OAuth authorization code flow with PKCE (S256). See
  [Platform APIs § Authentication](docs/developers/PLATFORM_APIS.md#authentication).
- Keep tokens server-side or in secure storage; never put them in client-side code.
- Handle timeouts, and retry with exponential backoff.
- Log security events without logging tokens or other secrets.

## Contact

- Vulnerability reports and security questions: **security@conducktor.com**
- Legal questions: **legal@conducktor.com**
- Privacy questions: **privacy@conducktor.com**
