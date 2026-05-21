---
name: coolify-setup
description: Install the Coolify CLI to ~/.local/bin/coolify so you can run coolify commands directly in your terminal
allowed-tools: [Bash, Read, Write]
---

# Coolify CLI Setup

Install the `coolify` CLI from this plugin to `~/.local/bin/coolify`.

## Steps

1. Find this plugin's install path:

```bash
find ~/.claude/plugins/cache -name "coolify" -path "*/scripts/coolify" 2>/dev/null | head -1
```

2. Create the symlink:

```bash
SCRIPT=$(find ~/.claude/plugins/cache -name "coolify" -path "*/scripts/coolify" 2>/dev/null | head -1)
if [[ -z "$SCRIPT" ]]; then
  echo "Error: Could not find the coolify script in plugin cache."
  exit 1
fi
chmod +x "$SCRIPT"
mkdir -p ~/.local/bin
ln -sf "$SCRIPT" ~/.local/bin/coolify
echo "Installed: ~/.local/bin/coolify -> $SCRIPT"
```

3. Verify it works:

```bash
coolify help
```

4. If `coolify: command not found`, add `~/.local/bin` to your PATH:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc  # or ~/.zshrc
source ~/.bashrc
```

## After setup

Run `coolify help` from any project directory that has a `.env` with `COOLIFY_URL`, `COOLIFY_API_TOKEN`, and `COOLIFY_APP_UUID` set.
