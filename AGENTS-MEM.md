# AGENTS-MEM.md - MCP Memory Service Guide for AI Agents

> **For AI Agents**: This document explains how to connect to and use the MCP Memory Service. Read this first before attempting to store or retrieve memories.

## What Is This?

The **MCP Memory Service** is a semantic memory database that allows AI agents to persistently store and retrieve information across sessions. It uses vector embeddings for intelligent semantic search.

**There are already memories stored here** - search before asking questions that may have been answered before.

---

## Quick Start (Service Already Running)

If the service is already running at `http://localhost:8000`:

```pwsh
# Test if service is up
Invoke-RestMethod -Uri 'http://localhost:8000/api/health'

# Search existing memories
Invoke-RestMethod -Uri 'http://localhost:8000/api/search' -Method Post `
  -ContentType 'application/json' -Body '{"query": "your search terms", "n_results": 10}'

# Store a new memory
$body = @{content="Your content here"; tags=@("tag1","tag2")} | ConvertTo-Json
Invoke-RestMethod -Uri 'http://localhost:8000/api/memories' -Method Post `
  -ContentType 'application/json' -Body $body
```

---

## Critical Locations

| Item | Path |
| ---- | ---- |
| **Project** | `C:\Users\bean\BitBox\source\GitHub\mcp-memory-service` |
| **Database** | `C:\Users\bean\AppData\Local\mcp-memory\sqlite_vec.db` |
| **Config** | `C:\Users\bean\BitBox\source\GitHub\mcp-memory-service\.env` |
| **HTTP Bridge** | `examples\http-mcp-bridge.js` |

---

## Starting the Service

```pwsh
cd C:\Users\bean\BitBox\source\GitHub\mcp-memory-service
uv run python run_server.py
```

The service will be available at:

- **Web Dashboard**: <http://localhost:8000>
- **API**: <http://localhost:8000/api>
- **Health Check**: <http://localhost:8000/api/health>

---

## Fresh Setup (New Installation)

If setting up from scratch on a new machine:

### Prerequisites

- Python 3.13 (or 3.10+)
- `uv` package manager
- Windows Developer Mode enabled (for symlinks)

### Steps

1). Clone the repo
`git clone https://github.com/doobidoo/mcp-memory-service.git`
`cd mcp-memory-service`

2). Create virtual environment with Python 3.13
`uv venv --python 3.13`

3). Sync dependencies
`uv sync`

4). Install python-dotenv (CRITICAL - not in default deps)
`uv pip install python-dotenv`

5). Create .env file (copy this exactly)

```pwsh
@"
MCP_MEMORY_STORAGE_BACKEND=sqlite_vec
MCP_HTTP_ENABLED=true
MCP_HTTP_PORT=8000
MCP_HTTPS_ENABLED=false
MCP_OAUTH_ENABLED=false
MCP_ALLOW_ANONYMOUS_ACCESS=true
MCP_MDNS_ENABLED=true
MCP_DEBUG=true
MCP_CONSOLIDATION_ENABLED=false
MCP_QUALITY_SYSTEM_ENABLED=true
MCP_QUALITY_AI_PROVIDER=none
MCP_QUALITY_BOOST_ENABLED=false
MCP_GRAPH_STORAGE_MODE=dual_write
"@ | Out-File -FilePath .env -Encoding utf8
```

6). Start the server

`uv run python run_server.py`

---

## What Can Go Wrong (And How We Fixed It)

**Issue (2025-12-23)**: Server starts but API returns "Network Error" or 401 Unauthorized.

**Root Cause**:

1. OAuth is enabled by default (`MCP_OAUTH_ENABLED=true`)
2. `python-dotenv` wasn't installed, so `.env` wasn't being read

**Solution**:

1. Install python-dotenv: `uv pip install python-dotenv`
2. Create `.env` with `MCP_OAUTH_ENABLED=false` and `MCP_ALLOW_ANONYMOUS_ACCESS=true`
3. Kill any stale Python processes and restart

---

## Integration Options

### Option 1: HTTP API (Recommended for Scripts/Agents)

```text
POST http://localhost:8000/api/memories      # Store memory
POST http://localhost:8000/api/search        # Semantic search
GET  http://localhost:8000/api/memories      # List all
GET  http://localhost:8000/api/search/by-tag?tag=tagname  # Search by tag
DELETE http://localhost:8000/api/memories/{hash}  # Delete
GET  http://localhost:8000/api/health        # Health check
```

**Store Example**:

```json
POST /api/memories
{"content": "Your content", "tags": ["tag1", "tag2"], "memory_type": "note"}
```

**Search Example**:

```json
POST /api/search
{"query": "what you're looking for", "n_results": 10}
```

### Option 2: CLI Commands

```pwsh
cd C:\Users\bean\BitBox\source\GitHub\mcp-memory-service
uv run memory store "content here" --tags "tag1,tag2"
uv run memory search "query"
uv run memory list
uv run memory delete <content_hash>
```

### Option 3: Claude Desktop MCP (Native)

Add to `~/.claude.json`:

```json
{
  "mcpServers": {
    "memory": {
      "command": "uv",
      "args": ["run", "--directory", "C:\\Users\\bean\\BitBox\\source\\GitHub\\mcp-memory-service", "memory", "server"],
      "env": {
        "MCP_MEMORY_STORAGE_BACKEND": "sqlite_vec"
      }
    }
  }
}
```

### Option 4: Docker/Remote via HTTP Bridge

For Docker Desktop MCP or remote servers:

```json
{
  "mcpServers": {
    "memory": {
      "command": "node",
      "args": ["C:\\Users\\bean\\BitBox\\source\\GitHub\\mcp-memory-service\\examples\\http-mcp-bridge.js"],
      "env": {
        "MCP_MEMORY_HTTP_ENDPOINT": "http://host.docker.internal:8000/api"
      }
    }
  }
}
```

---

## Emergency Database Access

If the service won't start and you need to read your memories directly:

**Database Location**: `C:\Users\bean\AppData\Local\mcp-memory\sqlite_vec.db`

**Schema** (memories table):

| Column | Type | Description |
| ------ | ---- | ----------- |
| id | INTEGER | Auto-increment primary key |
| content_hash | TEXT | SHA256 hash (unique identifier) |
| content | TEXT | The actual memory content |
| tags | TEXT | JSON array of tags |
| memory_type | TEXT | Type classification |
| metadata | TEXT | JSON metadata |
| created_at | REAL | Unix timestamp |
| created_at_iso | TEXT | ISO formatted date |

**Python Emergency Read Script**:

```python
import sqlite3
import json

DB_PATH = r'C:\Users\bean\AppData\Local\mcp-memory\sqlite_vec.db'

db = sqlite3.connect(DB_PATH)
db.row_factory = sqlite3.Row

# List all memories
print("=== ALL MEMORIES ===")
for row in db.execute('SELECT content_hash, content, tags, created_at_iso FROM memories ORDER BY created_at DESC'):
    print(f"\nHash: {row['content_hash'][:16]}...")
    print(f"Date: {row['created_at_iso']}")
    print(f"Tags: {json.loads(row['tags']) if row['tags'] else []}")
    print(f"Content: {row['content'][:200]}...")
    print("-" * 40)

# Search by text (not semantic, just LIKE)
print("\n=== SEARCH RESULTS ===")
search_term = "integration"  # Change this
for row in db.execute("SELECT content FROM memories WHERE content LIKE ?", (f'%{search_term}%',)):
    print(row['content'][:300])
    print("-" * 40)

# Export all to JSON
memories = []
for row in db.execute('SELECT * FROM memories'):
    memories.append({
        'hash': row['content_hash'],
        'content': row['content'],
        'tags': json.loads(row['tags']) if row['tags'] else [],
        'type': row['memory_type'],
        'created': row['created_at_iso']
    })

with open('memories_backup.json', 'w') as f:
    json.dump(memories, f, indent=2)
print(f"\nExported {len(memories)} memories to memories_backup.json")

db.close()
```

**IMPORTANT**: Direct database access is READ-ONLY for emergencies. Don't write directly - it will break the vector embeddings and indexes. Always use the API for modifications.

---

## Best Practices for Agents

1. **Search before storing** - Check if similar information already exists
2. **Use descriptive tags** - Include project name, topic, type (e.g., `["mcp-memory-service", "troubleshooting", "oauth"]`)
3. **Store decisions and learnings** - Future sessions benefit from past context
4. **Retrieve context at session start** - Search for relevant memories when beginning work
5. **Don't duplicate** - The service deduplicates by content hash, but be mindful

---

## Troubleshooting

| Problem | Solution |
| ------- | -------- |
| 401 Unauthorized | Check `.env` has `MCP_OAUTH_ENABLED=false` |
| Network Error on save | Restart server, check port 8000 is free |
| .env not loading | Ensure `python-dotenv` is installed |
| Can't find database | Check `C:\Users\bean\AppData\Local\mcp-memory\` |
| Server won't start | Kill stale Python processes: `Get-Process python* \| Stop-Process` |

---

## API Key Authentication (Security Hardening)

> **Important**: Running without an API key is fine for local development, but if exposing the service to your network (e.g., mini PC, home server), **always set an API key**.

### Why Use an API Key?

- **Without key**: Anyone who can reach the HTTP port can read/write/delete your memories
- **With key**: Only requests with matching `Authorization: Bearer <key>` succeed
- Essential for network-exposed deployments, even on trusted LANs

### Setting the API Key

#### Windows (PowerShell / .env)

```powershell
# Option 1: Environment variable (session)
$env:MCP_API_KEY = "your-secure-key-here"

# Option 2: In .env file (persistent)
MCP_API_KEY=your-secure-key-here

# Option 3: In start_http_debug.bat (uncomment line 68)
set MCP_API_KEY=your-secure-key-here
```

#### Linux / macOS (Bash)

```bash
# Option 1: Environment variable (session)
export MCP_API_KEY="your-secure-key-here"

# Option 2: In .env file (persistent)
echo 'MCP_API_KEY=your-secure-key-here' >> .env

# Option 3: Systemd service file
Environment="MCP_API_KEY=your-secure-key-here"
```

#### Arch Linux (Systemd Service)

```bash
# In /etc/systemd/system/mcp-memory.service
[Service]
Environment="MCP_API_KEY=your-secure-key-here"
# Or use EnvironmentFile for secrets:
EnvironmentFile=/etc/mcp-memory/secrets.env
```

### Suggested Key Format

Generate a strong, random key (32+ characters):

```bash
# Linux/macOS
openssl rand -base64 32

# PowerShell
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Max 256 }) -as [byte[]])

# Example output: K7xP2mN9qR4sT6vW8yZ1aB3cD5eF7gH9
```

**Recommendations**:

- Minimum 32 characters
- Mix of alphanumeric characters
- Avoid special characters that may need escaping in shells
- Never commit to version control (use `.env` which is gitignored)

### Using the API Key in Requests

```bash
# curl
curl -H "Authorization: Bearer YOUR_API_KEY" http://localhost:8000/api/health

# PowerShell
$headers = @{ Authorization = "Bearer YOUR_API_KEY" }
Invoke-RestMethod -Uri 'http://localhost:8000/api/health' -Headers $headers
```

### Security Checklist for Network Deployment

- [ ] Set `MCP_API_KEY` with strong random value
- [ ] Enable HTTPS (see below) - API key alone doesn't encrypt traffic
- [ ] Restrict port access via firewall (only trusted IPs)
- [ ] Don't expose port 8000 directly to internet without reverse proxy
- [ ] Store API key in secrets file, not in scripts committed to git

---

## Remote Deployment & HTTPS

### HTTPS Configuration (Built-in)

Enable secure connections with these environment variables:

```bash
MCP_HTTPS_ENABLED=true
MCP_HTTPS_PORT=8443
MCP_SSL_CERT_FILE=/path/to/cert.pem
MCP_SSL_KEY_FILE=/path/to/key.pem
```

### Generate Certificate (with IP + Hostname SAN)

For corporate/secure environments, create a certificate valid for both IP and hostname:

```bash
openssl req -x509 -newkey rsa:4096 \
  -keyout key.pem -out cert.pem \
  -days 365 -nodes \
  -subj "/CN=yggdrasil" \
  -addext "subjectAltName=DNS:yggdrasil,DNS:yggdrasil.local,IP:192.168.1.18"
```

This cert is valid for:

- `https://yggdrasil:8443`
- `https://yggdrasil.local:8443`
- `https://192.168.1.18:8443`

### Windows Certificate Trust

To avoid warnings on Windows clients:

1. Copy `cert.pem` to Windows machine
2. Run: `certutil -addstore "Root" cert.pem`
3. Or: MMC → Certificates → Trusted Root CAs → Import

### Remote Linux Deployment (Arch/EndeavourOS)

```bash
# Install dependencies
sudo pacman -S python python-pip git
pip install uv

# Clone and setup
git clone https://github.com/doobidoo/mcp-memory-service.git
cd mcp-memory-service
uv venv --python 3.13
uv sync
uv pip install python-dotenv

# Create .env with HTTPS
cat > .env << 'EOF'
MCP_MEMORY_STORAGE_BACKEND=sqlite_vec
MCP_HTTP_ENABLED=false
MCP_HTTPS_ENABLED=true
MCP_HTTPS_PORT=8443
MCP_SSL_CERT_FILE=/home/user/certs/cert.pem
MCP_SSL_KEY_FILE=/home/user/certs/key.pem
MCP_OAUTH_ENABLED=false
MCP_ALLOW_ANONYMOUS_ACCESS=true
EOF

# Systemd service (auto-start)
sudo cp scripts/service/mcp-memory.service /etc/systemd/system/
sudo systemctl enable --now mcp-memory

# Firewall
sudo firewall-cmd --add-port=8443/tcp --permanent
sudo firewall-cmd --reload
```

### Connect from Windows to Remote Server

```json
{
  "mcpServers": {
    "memory-remote": {
      "command": "node",
      "args": ["C:\\path\\to\\http-mcp-bridge.js"],
      "env": {
        "MCP_MEMORY_HTTP_ENDPOINT": "https://192.168.1.18:8443/api"
      }
    }
  }
}
```

---

## Multi-Instance Architecture (Future)

| Instance | Location | Backend | Purpose |
| -------- | -------- | ------- | ------- |
| Local | Windows (this PC) | SQLite | Dev work, personal |
| Home Server | yggdrasil (192.168.1.18) | SQLite | Shared home network |
| Cloud | Cloudflare | D1 + Vectorize | Work/project memories |

Configure different endpoints per instance:

```text
MCP_MEMORY_HTTP_ENDPOINT=http://localhost:8000/api           # Local
MCP_MEMORY_HTTP_ENDPOINT=https://192.168.1.18:8443/api       # Home server
MCP_MEMORY_HTTP_ENDPOINT=https://your-worker.workers.dev/api # Cloud
```

---

## Version Info

- **Service Version**: 8.53.0
- **Backend**: SQLite-vec (local)
- **Last Updated**: 2025-12-23
- **Document Author**: Claude Code (Opus 4.5)
