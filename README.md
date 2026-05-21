# Coolify — Claude Code Plugin

Manage, monitor, and troubleshoot applications hosted on a self-hosted [Coolify](https://coolify.io) instance — directly from Claude Code.

## What it does

Claude automatically invokes this skill when you ask about:

- Deployment status, health checks, container state
- Production logs and crash diagnostics
- Triggering redeploys or restarts
- Production environment variables (view, set, delete)
- Deployment history and rollbacks
- Diagnosing production issues (502 errors, failed builds, etc.)

## Requirements

- A running [Coolify](https://coolify.io) instance (v4+)
- `curl` and `jq` available in your shell
- A project `.env` file with the following variables:

```env
COOLIFY_URL=https://coolify.example.com
COOLIFY_API_TOKEN=your-api-token
COOLIFY_APP_UUID=your-app-uuid
```

**Getting your API token**: open `$COOLIFY_URL/keys-and-tokens` in the Coolify UI and generate a token. The UI login password does not work for API calls.

**Getting your App UUID**: navigate to your application in the Coolify UI — the UUID is the last path segment of the URL.

## Usage

Just describe what you want in natural language. Claude will handle the API calls:

```
"what's the status of the app?"
"show me the last 200 log lines"
"redeploy the app"
"set the DATABASE_URL env var to postgres://..."
"show me recent deployments"
"the app is returning 502, help me debug it"
```

## Optional: install the CLI

Run `/coolify-setup` to install the `coolify` CLI to `~/.local/bin/coolify`. This lets you run Coolify commands directly in your terminal without going through Claude:

```bash
coolify status
coolify logs 200
coolify deploy
coolify envs
coolify set-env KEY value
```

The CLI reads the same `.env` variables as the skill.

## Plugin structure

```
coolify/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── skills/
│   └── coolify/
│       └── SKILL.md         # Model-invoked skill
├── commands/
│   └── coolify-setup.md     # /coolify-setup slash command
├── scripts/
│   └── coolify              # Optional bash CLI
└── README.md
```

## License

MIT
