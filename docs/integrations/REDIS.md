# Redis

Connect a Redis database to Conducktor as a connector.

## What you need

- A Redis host that Conducktor can reach over the internet.
- The connection details:

  | Field | Required | Notes |
  |-------|----------|-------|
  | Host | yes | The Redis endpoint's host name |
  | Port | no | Usually `6379` |
  | Database | no | The database number, such as `0` |
  | TLS | no | Turn it on whenever your provider supports it |
  | Credentials | as your Redis requires | Many managed services use the user `default` with a password |

If you have a connection URL, it has the form
`redis://user:password@host:6379/0`, or `rediss://…` with TLS.

## Connect

1. In Conducktor, open **Integrations** and find **Redis**.
2. Enter the host and the other details, then connect.

Conducktor stores the credentials encrypted in your workspace. The tools the connector
offers are listed on it in **Integrations**.

## Recommendations

- Use TLS for anything other than a test database.
- Give Conducktor its own Redis user with only the permissions it needs; read-only if you
  only want to look at data.
- Do not reuse your Redis administrator password.
- If your provider can restrict which addresses may connect, ask Conducktor support
  (support@conducktor.com) for the addresses to allow.

## Troubleshooting

| Problem | What to check |
|---------|---------------|
| Connection refused or timed out | The host is public, the port is right, and your firewall lets Conducktor in. |
| Authentication failed | The username and password, with no extra spaces. |
| TLS errors | TLS is on in Conducktor exactly when your Redis endpoint expects it. |
