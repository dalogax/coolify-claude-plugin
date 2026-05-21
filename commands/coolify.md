---
name: coolify
description: Manage, monitor, and troubleshoot applications hosted on a self-hosted Coolify instance. Invoke this skill whenever the user asks about deployment status, production logs, triggering a redeploy, restarting or stopping a container, production environment variables (viewing, setting, deleting), deployment history, rollbacks, or any Coolify-related operations. Also use when diagnosing production issues like 502 errors, container crashes, failed deployments, or health check failures.
version: 1.0.0
---

# Coolify Application Management

Coolify is a self-hosted PaaS (like Heroku/Render on your own server). It manages Docker containers, SSL, environment variables, and deployments through a REST API.

## Configuration

All config is read from the project's `.env` file. Required variables:

| Variable | Purpose |
|----------|---------|
| `COOLIFY_URL` | Base URL of the Coolify instance (e.g. `https://coolify.example.com`) |
| `COOLIFY_API_TOKEN` | Bearer token — generate at `$COOLIFY_URL/keys-and-tokens` (not the UI password) |
| `COOLIFY_APP_UUID` | Application UUID — visible in the Coolify dashboard URL |

Optional variables:

| Variable | Purpose |
|----------|---------|
| `COOLIFY_APPLICATION_NAME` | Display name for log output |
| `COOLIFY_PROJECT_ID` | Project UUID (used to build direct UI links) |
| `COOLIFY_ENVIRONMENT_ID` | Environment UUID (used to build direct UI links) |

**Finding the App UUID**: navigate to the application in the Coolify UI — the UUID is the last path segment of the URL. Or list all apps via API:
```bash
source .env
curl -s -H "Authorization: Bearer $COOLIFY_API_TOKEN" \
  "$COOLIFY_URL/api/v1/applications" | jq '.[] | {name, uuid}'
```

## CLI tool

A `coolify` CLI is bundled with this plugin and automatically available in your shell (via the plugin's `bin/` directory). Use it for common operations:

```bash
coolify status             # App status + health
coolify logs [N]           # Last N log lines (default 100)
coolify deploy             # Trigger a new build + deployment
coolify deploy true        # Force rebuild (ignores Docker cache)
coolify restart            # Restart container without rebuild
coolify start              # Start a stopped container
coolify stop               # Stop the container
coolify deployments        # Recent deployment history
coolify envs               # List all production env vars
coolify set-env KEY VALUE  # Create or update a production env var
coolify del-env KEY        # Delete a production env var
coolify version            # Coolify server version
```

After `set-env`, always remind the user to run `coolify deploy` for the change to take effect.

## Direct API calls

When the CLI is not installed, use `curl` directly. Always `source .env` first.

```bash
source .env
BASE="$COOLIFY_URL/api/v1"
AUTH="Authorization: Bearer $COOLIFY_API_TOKEN"
UUID="$COOLIFY_APP_UUID"

# Status
curl -s -H "$AUTH" "$BASE/applications/$UUID" \
  | jq '{name, status, health_status, fqdn, updated_at}'

# Logs (last 200 lines)
curl -s -H "$AUTH" "$BASE/applications/$UUID/logs?lines=200" \
  | jq -r '.logs'

# Trigger deployment
curl -s -H "$AUTH" "$BASE/deploy?uuid=$UUID&force=false" | jq

# Force rebuild
curl -s -H "$AUTH" "$BASE/deploy?uuid=$UUID&force=true" | jq

# Restart
curl -s -H "$AUTH" "$BASE/applications/$UUID/restart"

# Start / Stop
curl -s -H "$AUTH" "$BASE/applications/$UUID/start"
curl -s -H "$AUTH" "$BASE/applications/$UUID/stop"

# Recent deployments
curl -s -H "$AUTH" "$BASE/deployments/applications/$UUID" \
  | jq -r '(if type == "array" then . else .deployments // [] end)
    | .[:10][]
    | "[\(.created_at | split("T")[0])] \(.status) — \(.commit[:7] // "-")"'

# List env vars
curl -s -H "$AUTH" "$BASE/applications/$UUID/envs" \
  | jq -r '.[] | "\(.key)=\(.value // "")"' | sort

# Set env var (create or update)
KEY="MY_VAR"; VALUE="my-value"
EXISTING=$(curl -s -H "$AUTH" "$BASE/applications/$UUID/envs" \
  | jq -r --arg k "$KEY" '.[] | select(.key == $k) | .uuid // empty')
if [[ -n "$EXISTING" ]]; then
  curl -s -X PATCH -H "$AUTH" -H "Content-Type: application/json" \
    -d "{\"uuid\":\"$EXISTING\",\"key\":\"$KEY\",\"value\":\"$VALUE\",\"is_preview\":false,\"is_build_time\":false}" \
    "$BASE/applications/$UUID/envs"
else
  curl -s -X POST -H "$AUTH" -H "Content-Type: application/json" \
    -d "{\"key\":\"$KEY\",\"value\":\"$VALUE\",\"is_preview\":false,\"is_build_time\":false}" \
    "$BASE/applications/$UUID/envs"
fi

# Delete env var
ENV_UUID=$(curl -s -H "$AUTH" "$BASE/applications/$UUID/envs" \
  | jq -r --arg k "MY_VAR" '.[] | select(.key == $k) | .uuid')
curl -s -X DELETE -H "$AUTH" "$BASE/applications/$UUID/envs/$ENV_UUID"

# Coolify server version
curl -s -H "$AUTH" "$BASE/version" | jq -r '.'
```

## Troubleshooting playbook

When diagnosing a production issue, follow this order:

1. Check status — is the container running and healthy?
2. Check logs (200+ lines) — look for errors, crashes, unhandled exceptions
3. Check recent deployments — did a recent deploy break it?
4. Check env vars — missing or wrong configuration?

Common symptoms:

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| `502 Bad Gateway` | Container not running / health check failing | Check status + logs; verify `PORT` env var matches app |
| Container keeps crashing | App startup error | Fetch 300+ log lines — look for the crash reason |
| Build failure | Dockerfile or dependency error | Check deployments tab; test `docker build` locally |
| Env var not taking effect | Deployed before applying the change | Confirm env var is set, then redeploy |
| Certificate error | Traefik/Let's Encrypt issue | Check server proxy logs in Coolify UI |

## Authentication errors

If you get a 401 error:
1. Open `$COOLIFY_URL/keys-and-tokens`
2. Generate a new API token (the UI login password does NOT work for API calls)
3. Update `COOLIFY_API_TOKEN` in `.env`
