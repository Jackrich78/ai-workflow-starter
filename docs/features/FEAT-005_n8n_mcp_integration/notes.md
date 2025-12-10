# FEAT-005: Installation Notes

## Quick Reference

| Setting | Value |
|---------|-------|
| Docker Image | `ghcr.io/czlonkowski/n8n-mcp:latest` |
| Transport | STDIO |
| n8n Instance | `https://sherpasdurden.app.n8n.cloud` |
| MCP Mode | `stdio` (required) |
| Usage Model | **On-demand** (add when needed, remove when done) |

---

## On-Demand Usage Model

We use an **add/remove** approach rather than keeping the MCP always configured. This avoids:
- Docker container running in every Claude Code session
- ~150-200MB RAM usage when not needed
- Docker Desktop needing to be running for unrelated work

### To Start Using n8n-mcp

**Prerequisites:**
1. Docker Desktop must be running
2. Have your n8n API key ready

**Add the MCP by editing the config file directly:**

> **Note:** The `claude mcp add` CLI has a bug where complex Docker commands with multiple `-e` flags get truncated. We edit the JSON directly instead.

**Config file location (Claude Code):**
```
~/.claude.json
```
Full path: `/Users/builder/.claude.json`

**Find the `mcpServers` section and add the `n8n-mcp` entry:**

```json
"mcpServers": {
  "archon": {
    "type": "http",
    "url": "http://localhost:8051/mcp"
  },
  "n8n-mcp": {
    "type": "stdio",
    "command": "docker",
    "args": [
      "run",
      "-i",
      "--rm",
      "--init",
      "-e", "MCP_MODE=stdio",
      "-e", "LOG_LEVEL=error",
      "-e", "DISABLE_CONSOLE_OUTPUT=true",
      "-e", "N8N_API_URL=https://sherpasdurden.app.n8n.cloud",
      "-e", "N8N_API_KEY=<YOUR_API_KEY>",
      "ghcr.io/czlonkowski/n8n-mcp:latest"
    ],
    "env": {}
  }
}
```

**Replace `<YOUR_API_KEY>` with your actual n8n API key.**

**Verify it's added:**
```bash
claude mcp list
```

**Restart Claude Code** to load the new MCP.

### To Stop Using n8n-mcp

When you're done with n8n work, either:

**Option A: Use CLI**
```bash
claude mcp remove n8n-mcp
```

**Option B: Edit JSON**
Remove the entire `"n8n-mcp": { ... }` block from `~/.claude.json`

The Docker image remains cached for next time.

---

## Risk Assessment

### What Gets Installed Where

| Component | Location | Runs When | Risk |
|-----------|----------|-----------|------|
| Docker image | Docker's managed storage (`~/Library/Containers/com.docker.docker/`) | Never (just files) | None - passive storage |
| MCP config entry | `~/.claude.json` | Never (just text) | None - config only |
| Docker container | Memory | Only when Claude Code is open AND MCP is configured | Low - isolated, no privileged access |

### Security Considerations

- **Docker image source:** GitHub Container Registry (`ghcr.io`), built from public repo
- **What the container can access:** Only what you pass via environment variables (API key, URL)
- **Network access:** Container can reach your n8n instance (required for API calls)
- **File system:** No volume mounts - container cannot access your files
- **Cleanup:** `--rm` flag ensures container is deleted when Claude Code closes

### API Key Storage

When you run `claude mcp add`, your API key is stored in plain text in `~/.claude.json`.

**Mitigations:**
- File is in your home directory with standard user permissions
- Remove the MCP config when not in use (`claude mcp remove n8n-mcp`)
- Consider creating a limited-scope API key in n8n

---

## Claude Desktop Configuration

If you also want n8n-mcp in Claude Desktop, edit the config file directly.

**Config file location (Claude Desktop):**
```
~/Library/Application Support/Claude/claude_desktop_config.json
```
Full path: `/Users/builder/Library/Application Support/Claude/claude_desktop_config.json`

**Open in Finder:**
```bash
open ~/Library/Application\ Support/Claude/
```

**Add or merge into the file:**

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--init",
        "-e", "MCP_MODE=stdio",
        "-e", "LOG_LEVEL=error",
        "-e", "DISABLE_CONSOLE_OUTPUT=true",
        "-e", "N8N_API_URL=https://sherpasdurden.app.n8n.cloud",
        "-e", "N8N_API_KEY=<YOUR_API_KEY>",
        "ghcr.io/czlonkowski/n8n-mcp:latest"
      ]
    }
  }
}
```

**Replace `<YOUR_API_KEY>` with your actual n8n API key.**

**Note:** Unlike Claude Code, there's no CLI for Claude Desktop. You must edit the JSON file directly. Remove the `n8n-mcp` entry when not needed. Restart Claude Desktop after changes.

---

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `MCP_MODE` | Yes | Must be `stdio` for Claude integrations |
| `LOG_LEVEL` | No | Set to `error` to reduce noise |
| `DISABLE_CONSOLE_OUTPUT` | No | Set to `true` to prevent JSON parse errors |
| `N8N_API_URL` | For management | Your n8n instance URL (no trailing slash) |
| `N8N_API_KEY` | For management | API key from n8n Settings → API |

---

## What n8n-mcp Provides

**Without API Key (documentation only):**
- 543 n8n node documentation
- 2,709 workflow templates
- Node property and operation details

**With API Key (full management):**
- All documentation features
- Create workflows
- Update workflows
- Execute workflows
- List and manage existing workflows

---

## Updating the Docker Image

```bash
docker pull ghcr.io/czlonkowski/n8n-mcp:latest
```

Then restart Claude Code (if MCP is currently configured).

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| JSON parsing errors | Ensure `MCP_MODE=stdio` is set |
| MCP not appearing | Run `claude mcp list` to verify config |
| Docker errors | Ensure Docker Desktop is running |
| API connection fails | Check URL has no trailing slash |
| "command not found" | Run in a separate terminal, not inside Claude Code |
| Container keeps running | The `--rm` flag should auto-cleanup; check `docker ps` |

---

## Source Repository

- GitHub: https://github.com/czlonkowski/n8n-mcp
- Docker: `ghcr.io/czlonkowski/n8n-mcp:latest`

We use the pre-built Docker image rather than cloning the repository.
