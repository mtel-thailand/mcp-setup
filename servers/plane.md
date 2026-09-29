# Plane

Lets your agent read and create work items in the team's Plane workspace at
`task.mtel.cloud`, through the upstream
[`plane-mcp-server`](https://pypi.org/project/plane-mcp-server/).

## 1. Check the prerequisites

| Need | Check | If missing |
| ---- | ----- | ---------- |
| `uv` | `uvx --version` | `brew install uv`, or see [docs.astral.sh/uv](https://docs.astral.sh/uv/) |
| A Plane account on `task.mtel.cloud` | log in | ask a workspace admin |

## 2. Create a token

On `task.mtel.cloud`, go to **Profile Settings → Personal Access Tokens** and
create one. Create a separate token for agent use, so you can revoke it on its
own.

## 3. Register the server

In your terminal — not in a chat with an agent:

```bash
claude mcp add plane --scope user \
  --env PLANE_API_KEY=<your token> \
  --env PLANE_WORKSPACE_SLUG=mtel \
  --env PLANE_BASE_URL=https://task.mtel.cloud \
  -- uvx plane-mcp-server stdio
```

`--scope user` makes it available in every project. Drop it to register only for
the current one.

For any other client, these are the values to put in its config (see
[Adding a server to your client](../README.md#adding-a-server-to-your-client)):

```
command: uvx
args:    plane-mcp-server  stdio
env:     PLANE_API_KEY=<your token>
         PLANE_WORKSPACE_SLUG=mtel
         PLANE_BASE_URL=https://task.mtel.cloud
```

Then restart your session.

## 4. Verify

Ask your agent: *"list my Plane projects"*. You should get back the projects you
are a member of. In Claude Code, `claude mcp list` also shows
`plane: ... - ✔ Connected`.

## Things that will bite

- **Do not put `/api/v1` in `PLANE_BASE_URL`.** The SDK appends it; including it
  makes every call 404.
- **A project you can't see returns 403.** The token reaches only the projects
  your account is a member of. Ask to be added; do not borrow someone else's
  token.
- **The token carries your whole account's permissions.** There is no read-only
  scope and no per-project limit, so your agent can write to and delete anything
  you can.

## Why your own token, and not a shared one

Everyone creates their own. There is no team token to ask for.

- **Attribution.** Plane records `created_by` on every work item. Your token is
  the only thing that ties a card an agent created back to the person who asked
  for it. A shared token puts the same name on everything.
- **Project permissions.** A token can only reach the projects you are a member
  of — anything else returns 403. That boundary is doing real work; a shared
  token would flatten it to one permission set that is either too wide or too
  narrow for everybody.
- **Revocation and offboarding.** A leaked token is revoked without stopping
  anyone else, and it dies with the account when someone leaves.

A shared service account is the right answer for *unattended* automation — CI, a
cron job, a bot filing cards from alerts — where no person is behind the action.
Give that its own account with deliberately narrow project membership. It is not
the right answer for an agent a human is driving.

A separate agent-only token limits who you have to tell when you revoke it, not
what it could have reached.
