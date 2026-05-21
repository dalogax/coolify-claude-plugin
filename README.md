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

## CLI tool

A `coolify` CLI is bundled with this plugin and automatically available in your shell when the plugin is active. It reads the same `.env` variables:

```bash
coolify status             # App status + health
coolify logs [N]           # Last N log lines (default 100)
coolify deploy             # Trigger a new build + deployment
coolify deploy true        # Force rebuild (ignores Docker cache)
coolify restart            # Restart container without rebuild
coolify start / stop       # Start or stop the container
coolify deployments        # Recent deployment history
coolify envs               # List all production env vars
coolify set-env KEY VALUE  # Create or update a production env var
coolify del-env KEY        # Delete a production env var
coolify version            # Coolify server version
```

## Plugin structure

```
coolify/
├── .claude-plugin/
│   └── plugin.json     # Plugin metadata
├── commands/
│   └── coolify.md      # Skill (model-invoked + /coolify command)
├── bin/
│   └── coolify         # Bash CLI (auto-added to PATH by the plugin)
└── README.md
```

## License

MIT
