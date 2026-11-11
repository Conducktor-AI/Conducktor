# Linear

Connect Linear to Conducktor as a connector, so your Assistants, your Flows and any MCP
client you connect through `ask_conducktor` can search, create and update your Linear
issues.

## Connect

1. In Conducktor, open **Integrations** and find **Linear**.
2. Click **Connect**.
3. Authorize in the window that opens (it goes to linear.app): choose your Linear workspace
   and approve the permissions.
4. The connection is set up when you return to Conducktor.

Linear uses OAuth. Conducktor stores the tokens encrypted in your workspace.

## Use it

Ask in a Conducktor chat, or from a connected MCP client:

```
Show me my open Linear issues for the ENG team.
Create a Linear issue in ENG: "Login fails on Safari", priority high.
```

Tips:

- Issue states differ per team. Ask for a team's states before moving an issue.
- When you create an issue, name the team it belongs to.

## Troubleshooting

| Problem | What to check |
|---------|---------------|
| "Team not found" | The team ID or name is wrong or you cannot access it. Ask for your Linear teams first. |
| Calls fail as unauthorized | The authorization was revoked in Linear, or expired. Reconnect Linear from **Integrations**. |
| A write is refused | Your Linear user lacks permission for it in Linear. |

Linear's own documentation: <https://linear.app/docs>.
