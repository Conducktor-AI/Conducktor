# OAuth Guide

This guide has two parts:

1. **[Connecting to Conducktor](#part-1-connecting-to-conducktor)**: Conducktor's own OAuth
   server, for an MCP client or agent that signs a user in to Conducktor.
2. **[Letting users connect your MCP server](#part-2-letting-users-connect-your-mcp-server)**:
   what your OAuth server should support so users can add your service to Conducktor as a
   connector.

In the examples, `https://your-client.example/callback` stands for your own registered
redirect URI.

## Part 1: Connecting to Conducktor

### Endpoints

| Endpoint | Purpose |
|----------|---------|
| `GET https://api.conducktor.com/.well-known/oauth-authorization-server` | RFC 8414 server metadata |
| `POST https://api.conducktor.com/api/oauth/register` | RFC 7591 Dynamic Client Registration |
| `GET https://api.conducktor.com/api/oauth/authorize` | Authorization code flow |
| `POST https://api.conducktor.com/api/oauth/token` | Code exchange, refresh, client credentials, device code |
| `POST https://api.conducktor.com/api/oauth/revoke` | RFC 7009 token revocation |
| `POST https://api.conducktor.com/api/oauth/introspect` | RFC 7662 token introspection |
| `POST https://api.conducktor.com/api/oauth/device/authorize` | RFC 8628 device authorization |

Most MCP clients need none of this by hand: point the client at
`https://mcp.conducktor.com/mcp`, choose OAuth, and it discovers the endpoints, registers
itself and opens the sign-in page.

### Server metadata

The metadata document advertises:

- scopes: `mcp:tools`, `mcp:read`, `mcp:write`;
- response type: `code`;
- grant types: `authorization_code`, `refresh_token`, `client_credentials`,
  `urn:ietf:params:oauth:grant-type:device_code`;
- token endpoint authentication: `client_secret_post` or `none` (a public client);
- PKCE method: `S256`;
- `authorization_response_iss_parameter_supported: true` (RFC 9207).

### Register a client

Registration needs no authentication.

```bash
curl -X POST https://api.conducktor.com/api/oauth/register \
  -H "Content-Type: application/json" \
  -d '{
    "client_name": "My Agent",
    "redirect_uris": ["https://your-client.example/callback"],
    "scope": "mcp:tools mcp:read"
  }'
```

| Field | Required | Default |
|-------|----------|---------|
| `client_name` | yes | — |
| `redirect_uris` | yes | — |
| `grant_types` | no | `["authorization_code", "refresh_token"]` |
| `response_types` | no | `["code"]` |
| `scope` | no | `mcp:tools mcp:read mcp:write`; a scope outside these three fails registration |
| `token_endpoint_auth_method` | no | `client_secret_post`; send `none` for a public client |

The response carries `client_id`, `client_secret` (omitted for a public client),
`client_id_issued_at`, `client_secret_expires_at` (`0`: the secret does not expire), the
registered values, and a `registration_access_token` with its `registration_client_uri`.
The secret and the registration access token are shown once; store them.

With the registration access token as a bearer token you can manage the client:

```
GET    /api/oauth/register/{client_id}   # read its metadata
PUT    /api/oauth/register/{client_id}   # update it
DELETE /api/oauth/register/{client_id}   # delete it
```

### Authorize and exchange the code

```bash
# 1. Make a PKCE verifier and challenge.
CODE_VERIFIER=$(openssl rand -base64 48 | tr -d '=+/' | cut -c1-64)
CODE_CHALLENGE=$(printf '%s' "$CODE_VERIFIER" | openssl dgst -sha256 -binary | base64 | tr '+/' '-_' | tr -d '=')

# 2. Send the user's browser here. They sign in to Conducktor and approve the client.
echo "https://api.conducktor.com/api/oauth/authorize?response_type=code&client_id=$CLIENT_ID&redirect_uri=https://your-client.example/callback&scope=mcp:tools%20mcp:read&state=$STATE&code_challenge=$CODE_CHALLENGE&code_challenge_method=S256"

# 3. Exchange the code that comes back on your redirect URI.
curl -X POST https://api.conducktor.com/api/oauth/token \
  -d grant_type=authorization_code \
  -d code="$CODE" \
  -d redirect_uri=https://your-client.example/callback \
  -d client_id="$CLIENT_ID" \
  -d client_secret="$CLIENT_SECRET" \
  -d code_verifier="$CODE_VERIFIER"
```

- The `redirect_uri` must be one you registered, and the token request must send the same
  one.
- An authorization code is valid for 10 minutes.
- Send a PKCE challenge. When the authorization request carried one, the token request
  must carry the matching `code_verifier`.
- Check that the `state` you get back is the one you sent.

The token response:

```json
{
  "access_token": "…",
  "token_type": "Bearer",
  "expires_in": 86400,
  "refresh_token": "refresh_…",
  "scope": "mcp:tools mcp:read"
}
```

Access tokens last 24 hours by default; read `expires_in` rather than assuming it.

### Refresh, revoke, introspect

```bash
curl -X POST https://api.conducktor.com/api/oauth/token \
  -d grant_type=refresh_token \
  -d refresh_token="$REFRESH_TOKEN"
```

A refresh returns a new access token. It does not return a new refresh token: keep using
the one you have. Refresh tokens do not expire by default, and can be revoked.

Revoke a token with `POST /api/oauth/revoke` and inspect one with
`POST /api/oauth/introspect`.

### Client credentials

A confidential client that registered `client_credentials` in its `grant_types` can get a
token for itself, with no user:

```bash
curl -X POST https://api.conducktor.com/api/oauth/token \
  -d grant_type=client_credentials \
  -d client_id="$CLIENT_ID" \
  -d client_secret="$CLIENT_SECRET"
```

No refresh token is issued for this grant.

### Use the token

Send the access token as `Authorization: Bearer <token>` to the MCP endpoint,
`https://mcp.conducktor.com/mcp`. See [Platform APIs](../developers/PLATFORM_APIS.md#mcp-endpoint).

## Part 2: Letting users connect your MCP server

When a user connects your service, Conducktor acts as the OAuth client: it sends the user
to your authorization page, receives the code on its callback, exchanges it, and stores
the tokens encrypted in the user's workspace.

### What Conducktor uses

- **RFC 8414 metadata.** Conducktor can read
  `https://your-service.example/.well-known/oauth-authorization-server` to find your
  authorization, token and registration endpoints and whether you support PKCE.
- **PKCE (S256).** Conducktor sends a PKCE challenge to providers that support it.
- **Dynamic Client Registration (RFC 7591).** When your metadata names a
  `registration_endpoint`, Conducktor can register its own client with you, so the user
  does not need to create one.
- **`state`.** Conducktor sends a `state` value and checks it on the callback.

### What your server should support

| Capability | Standard | Recommendation |
|------------|----------|----------------|
| Authorization code flow | RFC 6749 | Required |
| PKCE with `S256` | RFC 7636 | Required |
| Token refresh | RFC 6749 | Required for long-lived access |
| Server metadata | RFC 8414 | Recommended |
| Dynamic Client Registration | RFC 7591 | Optional |
| Token revocation | RFC 7009 | Recommended |

Your metadata document might look like this:

```json
{
  "issuer": "https://your-service.example",
  "authorization_endpoint": "https://your-service.example/oauth/authorize",
  "token_endpoint": "https://your-service.example/oauth/token",
  "registration_endpoint": "https://your-service.example/oauth/register",
  "revocation_endpoint": "https://your-service.example/oauth/revoke",
  "response_types_supported": ["code"],
  "grant_types_supported": ["authorization_code", "refresh_token"],
  "code_challenge_methods_supported": ["S256"],
  "token_endpoint_auth_methods_supported": ["client_secret_post", "none"]
}
```

### Implementation notes

- **Authorization codes:** single use, short-lived (10 minutes or less), bound to the
  `client_id`, the `redirect_uri` and the `code_challenge`.
- **Redirect URIs:** match registered URIs exactly.
- **PKCE check:** `BASE64URL(SHA256(code_verifier))` must equal the stored
  `code_challenge`. A verifier is 43 to 128 characters from `A-Z a-z 0-9 - . _ ~`.

  ```python
  import base64, hashlib, hmac

  def pkce_matches(code_verifier: str, code_challenge: str) -> bool:
      digest = hashlib.sha256(code_verifier.encode("ascii")).digest()
      computed = base64.urlsafe_b64encode(digest).decode("ascii").rstrip("=")
      return hmac.compare_digest(computed, code_challenge)
  ```

- **Access tokens:** short-lived; return `expires_in` so the client knows when to refresh.
- **Refresh tokens:** stored so they can be revoked; rotating them on use is good practice.
- **Scopes:** document them, and return `insufficient_scope` (403) when a token lacks one.
- **Errors:** use the standard OAuth error format:

  ```json
  { "error": "invalid_request", "error_description": "Missing code_challenge" }
  ```

- **Transport:** HTTPS only, and rate limits on the authorization and token endpoints.

### Test your server

```bash
# Metadata
curl https://your-service.example/.well-known/oauth-authorization-server

# Code exchange, after authorizing in a browser with the challenge from Part 1
curl -X POST https://your-service.example/oauth/token \
  -d grant_type=authorization_code \
  -d code="$CODE" \
  -d redirect_uri=https://your-client.example/callback \
  -d client_id="$CLIENT_ID" \
  -d code_verifier="$CODE_VERIFIER"

# Refresh
curl -X POST https://your-service.example/oauth/token \
  -d grant_type=refresh_token \
  -d refresh_token="$REFRESH_TOKEN" \
  -d client_id="$CLIENT_ID"
```

## References

- RFC 6749: [OAuth 2.0](https://datatracker.ietf.org/doc/html/rfc6749)
- RFC 7636: [PKCE](https://datatracker.ietf.org/doc/html/rfc7636)
- RFC 8414: [Authorization Server Metadata](https://datatracker.ietf.org/doc/html/rfc8414)
- RFC 7591: [Dynamic Client Registration](https://datatracker.ietf.org/doc/html/rfc7591)
- RFC 7009: [Token Revocation](https://datatracker.ietf.org/doc/html/rfc7009)
- RFC 7662: [Token Introspection](https://datatracker.ietf.org/doc/html/rfc7662)
- RFC 8628: [Device Authorization Grant](https://datatracker.ietf.org/doc/html/rfc8628)
