# OAuth 2.1 Implementation Guide

**Implementing OAuth 2.1 with PKCE for MCP Server Integration**

This guide is for service providers who want to integrate their MCP server with Conducktor using OAuth 2.1 with Proof Key for Code Exchange (PKCE).

## Overview

OAuth 2.1 is the modern authorization standard that consolidates best practices from OAuth 2.0 and adds mandatory security features. Conducktor requires OAuth 2.1 with PKCE for all third-party MCP server integrations.

### Why OAuth 2.1 + PKCE?

**Security Benefits:**
- **PKCE Required** - Prevents authorization code interception attacks
- **No Implicit Flow** - Eliminates token leakage via URL fragments
- **Refresh Token Rotation** - Detects token theft
- **State Parameter** - CSRF protection built-in

**User Benefits:**
- Scoped permissions (read/write separation)
- Revocable access
- User attribution in audit logs
- Token expiration and refresh

## Prerequisites

Before implementing OAuth 2.1 for your MCP server:

- [ ] HTTPS-only endpoints (required for OAuth)
- [ ] User management system
- [ ] Token generation and validation logic
- [ ] Database for storing OAuth clients and tokens
- [ ] Understanding of OAuth 2.1 and PKCE (RFC 6749, RFC 7636)

## Required Standards

Conducktor's OAuth 2.1 implementation follows these RFCs:

| RFC | Title | Status |
|-----|-------|--------|
| RFC 6749 | OAuth 2.0 Authorization Framework | ✅ Required |
| RFC 7636 | PKCE | ✅ Required |
| RFC 8414 | OAuth 2.0 Authorization Server Metadata | ⚠️ Recommended |
| RFC 7591 | Dynamic Client Registration | 🔄 Optional |

## Implementation Steps

### Step 1: OAuth Discovery Endpoint (RFC 8414)

Provide OAuth server metadata at:
```
https://your-api.example.com/.well-known/oauth-authorization-server
```

**Response Format:**
```json
{
  "issuer": "https://your-api.example.com",
  "authorization_endpoint": "https://your-api.example.com/oauth/authorize",
  "token_endpoint": "https://your-api.example.com/oauth/token",
  "revocation_endpoint": "https://your-api.example.com/oauth/revoke",
  "registration_endpoint": "https://your-api.example.com/oauth/register",
  "scopes_supported": ["read", "write", "admin"],
  "response_types_supported": ["code"],
  "grant_types_supported": ["authorization_code", "refresh_token"],
  "code_challenge_methods_supported": ["S256"],
  "token_endpoint_auth_methods_supported": ["client_secret_post", "none"]
}
```

**Key Fields:**

- `authorization_endpoint` - Where users authorize (required)
- `token_endpoint` - Where codes are exchanged for tokens (required)
- `code_challenge_methods_supported` - Must include `"S256"` for SHA-256 PKCE
- `response_types_supported` - Must include `"code"` (authorization code flow)

### Step 2: Authorization Endpoint

**Endpoint:** `POST /oauth/authorize`

**Request Parameters:**
```
response_type=code (required)
client_id=<client_id> (required)
redirect_uri=<uri> (required)
scope=<scopes> (required, space-separated)
state=<random_string> (required for CSRF protection)
code_challenge=<challenge> (required for PKCE)
code_challenge_method=S256 (required, must be SHA-256)
```

**Example Request:**
```http
GET /oauth/authorize
  ?response_type=code
  &client_id=conducktor_client_abc123
  &redirect_uri=https://app.conducktor.com/oauth/callback
  &scope=read%20write
  &state=random_32_byte_state
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
```

**Implementation Logic:**

```python
def authorize(request):
    # 1. Validate client_id
    client = get_client(request.client_id)
    if not client:
        return error("invalid_client")
    
    # 2. Validate redirect_uri matches registered URI
    if request.redirect_uri not in client.redirect_uris:
        return error("invalid_redirect_uri")
    
    # 3. Validate PKCE parameters
    if not request.code_challenge or request.code_challenge_method != "S256":
        return error("invalid_request", "PKCE required")
    
    # 4. Validate scopes
    requested_scopes = request.scope.split()
    if not all(scope in SUPPORTED_SCOPES for scope in requested_scopes):
        return error("invalid_scope")
    
    # 5. Show authorization page to user
    return render_authorization_page(
        client_name=client.name,
        requested_scopes=requested_scopes,
        state=request.state
    )
```

**User Authorization Page:**

Display to user:
- Client application name (e.g., "Conducktor")
- Requested permissions (e.g., "Read your issues", "Create issues")
- Approve/Deny buttons

**On Approval:**

```python
def user_approves():
    # 1. Generate authorization code (random 32 bytes, base64url)
    code = generate_random_code()
    
    # 2. Store code with metadata (expires in 10 minutes)
    store_authorization_code(
        code=code,
        client_id=request.client_id,
        user_id=current_user.id,
        redirect_uri=request.redirect_uri,
        scope=request.scope,
        code_challenge=request.code_challenge,  # Store for verification
        expires_at=now() + timedelta(minutes=10)
    )
    
    # 3. Redirect back to client with code and state
    redirect_to = f"{request.redirect_uri}?code={code}&state={request.state}"
    return redirect(redirect_to)
```

### Step 3: Token Endpoint

**Endpoint:** `POST /oauth/token`

**Request Parameters:**
```
grant_type=authorization_code (required)
code=<authorization_code> (required)
redirect_uri=<uri> (required, must match)
client_id=<client_id> (required)
client_secret=<secret> (if client has secret)
code_verifier=<verifier> (required for PKCE verification)
```

**Example Request:**
```http
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=abc123_authorization_code
&redirect_uri=https://app.conducktor.com/oauth/callback
&client_id=conducktor_client_abc123
&client_secret=secret_xyz (if applicable)
&code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
```

**PKCE Verification:**

```python
import hashlib
import base64

def verify_pkce(code_verifier: str, code_challenge: str) -> bool:
    # 1. Hash the code_verifier with SHA-256
    hash_digest = hashlib.sha256(code_verifier.encode('utf-8')).digest()
    
    # 2. Base64url encode (no padding)
    computed_challenge = base64.urlsafe_b64encode(hash_digest).decode('utf-8').rstrip('=')
    
    # 3. Compare with stored code_challenge
    return computed_challenge == code_challenge
```

**Token Generation Logic:**

```python
def exchange_code_for_token(request):
    # 1. Validate authorization code
    code_data = get_authorization_code(request.code)
    if not code_data or code_data.expires_at < now():
        return error("invalid_grant", "Code expired or invalid")
    
    # 2. Verify PKCE code_verifier
    if not verify_pkce(request.code_verifier, code_data.code_challenge):
        return error("invalid_grant", "PKCE verification failed")
    
    # 3. Verify client_id and redirect_uri match
    if request.client_id != code_data.client_id:
        return error("invalid_client")
    if request.redirect_uri != code_data.redirect_uri:
        return error("invalid_grant")
    
    # 4. Generate access token (JWT or random string)
    access_token = generate_access_token(
        user_id=code_data.user_id,
        client_id=code_data.client_id,
        scope=code_data.scope,
        expires_in=3600  # 1 hour
    )
    
    # 5. Generate refresh token
    refresh_token = generate_refresh_token(
        user_id=code_data.user_id,
        client_id=code_data.client_id,
        scope=code_data.scope
    )
    
    # 6. Delete authorization code (single use only)
    delete_authorization_code(request.code)
    
    # 7. Return token response
    return {
        "access_token": access_token,
        "token_type": "Bearer",
        "expires_in": 3600,
        "refresh_token": refresh_token,
        "scope": code_data.scope
    }
```

**Response Format:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "refresh_token_abc123",
  "scope": "read write"
}
```

### Step 4: Token Refresh

**Endpoint:** `POST /oauth/token`

**Request Parameters:**
```
grant_type=refresh_token (required)
refresh_token=<token> (required)
client_id=<client_id> (required)
client_secret=<secret> (if applicable)
scope=<scopes> (optional, must not exceed original scope)
```

**Implementation:**

```python
def refresh_access_token(request):
    # 1. Validate refresh token
    token_data = get_refresh_token(request.refresh_token)
    if not token_data or token_data.revoked:
        return error("invalid_grant", "Refresh token invalid or revoked")
    
    # 2. Verify client_id
    if request.client_id != token_data.client_id:
        return error("invalid_client")
    
    # 3. Check if scope is being reduced (allowed)
    if request.scope:
        requested_scopes = set(request.scope.split())
        original_scopes = set(token_data.scope.split())
        if not requested_scopes.issubset(original_scopes):
            return error("invalid_scope", "Cannot expand scope")
    
    # 4. Generate new access token
    access_token = generate_access_token(
        user_id=token_data.user_id,
        client_id=token_data.client_id,
        scope=request.scope or token_data.scope,
        expires_in=3600
    )
    
    # 5. OPTIONAL: Rotate refresh token (recommended)
    new_refresh_token = generate_refresh_token(
        user_id=token_data.user_id,
        client_id=token_data.client_id,
        scope=token_data.scope
    )
    revoke_refresh_token(request.refresh_token)
    
    return {
        "access_token": access_token,
        "token_type": "Bearer",
        "expires_in": 3600,
        "refresh_token": new_refresh_token,  # New token
        "scope": request.scope or token_data.scope
    }
```

### Step 5: Token Validation

**For MCP API Endpoints:**

```python
from functools import wraps
from flask import request, jsonify

def require_oauth(required_scope: str):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            # 1. Extract Bearer token from Authorization header
            auth_header = request.headers.get('Authorization', '')
            if not auth_header.startswith('Bearer '):
                return jsonify({"error": "unauthorized"}), 401
            
            token = auth_header[7:]  # Remove "Bearer "
            
            # 2. Validate token (JWT verification or database lookup)
            token_data = validate_access_token(token)
            if not token_data or token_data.expires_at < now():
                return jsonify({"error": "token_expired"}), 401
            
            # 3. Check scope
            token_scopes = set(token_data.scope.split())
            if required_scope not in token_scopes:
                return jsonify({"error": "insufficient_scope"}), 403
            
            # 4. Attach user context to request
            request.oauth_user = token_data.user_id
            request.oauth_client = token_data.client_id
            request.oauth_scopes = token_scopes
            
            return f(*args, **kwargs)
        return decorated_function
    return decorator

# Usage
@app.route('/mcp/issues')
@require_oauth('read')
def list_issues():
    user_id = request.oauth_user
    # ... return issues for user
```

### Step 6: Token Revocation

**Endpoint:** `POST /oauth/revoke`

**Request Parameters:**
```
token=<token> (required, access or refresh token)
token_type_hint=access_token|refresh_token (optional)
client_id=<client_id> (required)
client_secret=<secret> (if applicable)
```

**Implementation:**

```python
def revoke_token(request):
    # 1. Validate client credentials
    client = authenticate_client(request.client_id, request.client_secret)
    if not client:
        return error("invalid_client")
    
    # 2. Determine token type
    if request.token_type_hint == "refresh_token":
        token_data = get_refresh_token(request.token)
    else:
        # Try access token first, then refresh token
        token_data = get_access_token(request.token) or get_refresh_token(request.token)
    
    # 3. Verify token belongs to client
    if token_data and token_data.client_id == client.id:
        # 4. Revoke token
        if token_data.type == "refresh":
            revoke_refresh_token(request.token)
        else:
            revoke_access_token(request.token)
    
    # 5. Always return 200 (even if token not found, per RFC)
    return {"status": "revoked"}, 200
```

## Dynamic Client Registration (RFC 7591) - Optional

Allow Conducktor to automatically register OAuth clients:

**Endpoint:** `POST /oauth/register`

**Request:**
```json
{
  "client_name": "Conducktor",
  "redirect_uris": ["https://app.conducktor.com/oauth/callback"],
  "grant_types": ["authorization_code", "refresh_token"],
  "response_types": ["code"],
  "scope": "read write",
  "token_endpoint_auth_method": "client_secret_post"
}
```

**Response:**
```json
{
  "client_id": "auto_generated_client_id",
  "client_secret": "auto_generated_secret",
  "client_id_issued_at": 1700000000,
  "client_secret_expires_at": 0,
  "redirect_uris": ["https://app.conducktor.com/oauth/callback"],
  "grant_types": ["authorization_code", "refresh_token"],
  "response_types": ["code"],
  "scope": "read write"
}
```

## Security Best Practices

### PKCE Requirements

**Code Verifier:**
- Minimum 43 characters
- Maximum 128 characters
- Characters: `[A-Z]`, `[a-z]`, `[0-9]`, `-`, `.`, `_`, `~`
- Generated with cryptographically secure random

**Code Challenge:**
- SHA-256 hash of code_verifier
- Base64url encoded (no padding)
- `code_challenge = BASE64URL(SHA256(code_verifier))`

### Token Security

**Access Tokens:**
- Short-lived (1 hour recommended)
- Use JWT for stateless validation or database for revocation
- Include user_id, client_id, scope, expiration in JWT claims

**Refresh Tokens:**
- Long-lived (30-90 days)
- Store in database for revocation support
- Rotate on refresh (recommended)
- Single-use only (detect token theft)

**Authorization Codes:**
- Single-use only (delete after exchange)
- Short-lived (10 minutes maximum)
- Bind to client_id, redirect_uri, code_challenge

### Error Handling

**OAuth Error Response Format:**
```json
{
  "error": "invalid_request",
  "error_description": "Missing required parameter: code_challenge",
  "error_uri": "https://docs.your-api.com/oauth/errors/invalid_request"
}
```

**Standard Error Codes:**
- `invalid_request` - Malformed request
- `invalid_client` - Client authentication failed
- `invalid_grant` - Authorization code or refresh token invalid
- `unauthorized_client` - Client not authorized for requested grant type
- `unsupported_grant_type` - Grant type not supported
- `invalid_scope` - Requested scope invalid or exceeds granted scope
- `server_error` - Internal server error
- `temporarily_unavailable` - Server temporarily unavailable

## Testing Your Implementation

### 1. Test OAuth Discovery

```bash
curl https://your-api.example.com/.well-known/oauth-authorization-server
```

Verify response includes `authorization_endpoint`, `token_endpoint`, and `code_challenge_methods_supported: ["S256"]`.

### 2. Test Authorization Flow

```bash
# Generate PKCE challenge
CODE_VERIFIER=$(openssl rand -base64 32 | tr -d "=+/" | cut -c1-43)
CODE_CHALLENGE=$(echo -n "$CODE_VERIFIER" | openssl dgst -sha256 -binary | base64 | tr -d "=+/" | tr -- "+/" "-_")

# Request authorization
open "https://your-api.example.com/oauth/authorize?response_type=code&client_id=test&redirect_uri=http://localhost:3000/callback&scope=read&state=test123&code_challenge=$CODE_CHALLENGE&code_challenge_method=S256"
```

### 3. Test Token Exchange

```bash
curl -X POST https://your-api.example.com/oauth/token \
  -d "grant_type=authorization_code" \
  -d "code=<authorization_code>" \
  -d "redirect_uri=http://localhost:3000/callback" \
  -d "client_id=test" \
  -d "code_verifier=$CODE_VERIFIER"
```

### 4. Test Token Validation

```bash
curl https://your-api.example.com/mcp/issues \
  -H "Authorization: Bearer <access_token>"
```

### 5. Test Token Refresh

```bash
curl -X POST https://your-api.example.com/oauth/token \
  -d "grant_type=refresh_token" \
  -d "refresh_token=<refresh_token>" \
  -d "client_id=test"
```

## Conducktor Integration Checklist

Before submitting your MCP server for Conducktor integration:

- [ ] OAuth discovery endpoint returns valid metadata
- [ ] Authorization endpoint supports PKCE (S256 only)
- [ ] Token endpoint validates code_verifier
- [ ] Tokens include standard fields (access_token, expires_in, refresh_token)
- [ ] Token refresh supported
- [ ] Token revocation supported
- [ ] HTTPS enforced for all OAuth endpoints
- [ ] Error responses follow OAuth 2.0 format
- [ ] Scopes clearly documented
- [ ] Rate limiting implemented (recommended: 100 req/min per user)
- [ ] Audit logging for all OAuth operations

## Support & Resources

**OAuth Specifications:**
- RFC 6749: [OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749)
- RFC 7636: [PKCE](https://datatracker.ietf.org/doc/html/rfc7636)
- RFC 8414: [OAuth Discovery](https://datatracker.ietf.org/doc/html/rfc8414)
- RFC 7591: [Dynamic Registration](https://datatracker.ietf.org/doc/html/rfc7591)

**Conducktor Resources:**
- Integration Portal: [partners.conducktor.com](https://partners.conducktor.com)
- API Documentation: [docs.conducktor.com/api](https://docs.conducktor.com/api)
- Partner Support: partners@conducktor.com

**Tools:**
- OAuth Debugger: [oauthdebugger.com](https://oauthdebugger.com)
- JWT Decoder: [jwt.io](https://jwt.io)
- PKCE Generator: [tonyxu-io.github.io/pkce-generator](https://tonyxu-io.github.io/pkce-generator/)

---

**Last Updated:** November 18, 2025  
**Version:** 1.0
