# KingBase MCP Configuration Guide

This reference provides detailed configuration options and advanced settings for the KingBase MCP Server.

## Table of Contents

1. [Basic Configuration](#basic-configuration)
2. [Environment Variables](#environment-variables)
3. [Access Modes](#access-modes)
4. [Transport Modes](#transport-modes)
5. [Security Configuration](#security-configuration)
6. [Advanced Settings](#advanced-settings)
7. [Troubleshooting](#troubleshooting)
8. [Examples](#examples)

## Basic Configuration

### Minimal Configuration

The absolute minimum configuration requires only database connection details:

```json
{
  "mcpServers": {
    "kingbase": {
      "command": "npx",
      "args": ["-y", "kingbase-mcp-server"],
      "env": {
        "DB_HOST": "localhost",
        "DB_PORT": "54321",
        "DB_USER": "system",
        "DB_PASSWORD": "your-password",
        "DB_NAME": "kingbase"
      }
    }
  }
}
```

### Using .env File

Instead of environment variables in MCP config, you can use a `.env` file:

```bash
# .env file
DB_HOST=192.168.1.100
DB_PORT=54321
DB_USER=system
DB_PASSWORD=your-secure-password
DB_NAME=mydb
DB_SCHEMA=public
ACCESS_MODE=readonly
TRANSPORT=stdio
```

Then start the server:
```bash
npx kingbase-mcp-server
```

## Environment Variables

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `DB_HOST` | Database server hostname or IP | `localhost`, `192.168.1.100` |
| `DB_PORT` | Database server port | `54321` |
| `DB_USER` | Database username | `system` |
| `DB_PASSWORD` | Database password | (your password) |
| `DB_NAME` | Database name | `kingbase` |

### Optional Variables

| Variable | Description | Default | Options |
|----------|-------------|---------|---------|
| `DB_SCHEMA` | Default schema for unqualified table names | `public` | Any valid schema |
| `ACCESS_MODE` | Permission level for operations | `readonly` | `readonly`, `readwrite`, `full`, `admin` |
| `TRANSPORT` | Communication protocol | `stdio` | `stdio`, `http` |
| `MCP_PORT` | HTTP mode listening port | `3000` | Any available port |
| `MCP_HOST` | HTTP mode bind address | `0.0.0.0` | `0.0.0.0`, `127.0.0.1`, specific IP |
| `SKIP_CONFIRM` | Skip write operation confirmation | `false` | `true`, `false` |

## Access Modes

Access modes control what operations are allowed. They are cumulative - each level includes all permissions from lower levels.

### readonly (Default)

**Recommended for:** Data analysis, reporting, exploration

**Allowed Operations:**
- `kb_query` - SELECT, WITH, SHOW statements
- `kb_list_schemas` - List all schemas
- `kb_list_tables` - List tables and views
- `kb_describe_table` - View table structure
- `kb_list_indexes` - View indexes
- `kb_list_constraints` - View constraints
- `kb_explain` - Query execution plans
- `kb_table_data` - Preview table data
- `kb_table_stats` - Table statistics
- Resources and Prompts

**Blocked Operations:**
- Any INSERT, UPDATE, DELETE
- Any CREATE, ALTER, DROP, TRUNCATE

### readwrite

**Recommended for:** Data maintenance, batch updates

**Includes:** All `readonly` operations plus:
- `kb_execute` with INSERT, UPDATE

**Blocked Operations:**
- DELETE statements
- DDL operations (CREATE, ALTER, DROP, TRUNCATE)

### full

**Recommended for:** Data cleanup, ETL operations

**Includes:** All `readwrite` operations plus:
- `kb_execute` with DELETE

**Blocked Operations:**
- DDL operations (CREATE, ALTER, DROP, TRUNCATE)

### admin

**Recommended for:** Schema management, migrations

**Includes:** All `full` operations plus:
- `kb_execute_ddl` - CREATE, ALTER, DROP, TRUNCATE

**Warning:** This mode allows destructive operations. Use with caution.

## Transport Modes

### stdio Mode (Default)

**Use when:** Running locally, direct MCP client connection

**How it works:**
- MCP client spawns the server process
- Communication via stdin/stdout
- Server lifecycle tied to client

**Configuration:**
```json
{
  "mcpServers": {
    "kingbase": {
      "command": "npx",
      "args": ["-y", "kingbase-mcp-server"],
      "env": {
        "DB_HOST": "localhost",
        "TRANSPORT": "stdio"
      }
    }
  }
}
```

**Pros:**
- Simple setup
- No network configuration needed
- Secure (no exposed ports)

**Cons:**
- Server runs per client session
- Cannot share between multiple clients
- Must restart to apply config changes

### HTTP Mode

**Use when:** Remote deployment, shared server, team environment

**How it works:**
- Server runs as a standalone service
- MCP clients connect via HTTP
- Multiple clients can share one server

**Server Configuration:**
```bash
# Environment variables
export TRANSPORT=http
export MCP_PORT=3000
export MCP_HOST=0.0.0.0

# Start server
npx kingbase-mcp-server
```

**Client Configuration:**
```json
{
  "mcpServers": {
    "kingbase": {
      "url": "http://your-server:3000/mcp"
    }
  }
}
```

**Pros:**
- Shared across multiple clients
- Can run on separate machine
- Easier to monitor and manage

**Cons:**
- Requires network access
- Need to secure the endpoint
- Additional setup complexity

**Security Considerations:**
- Use firewall to restrict access
- Consider reverse proxy with authentication
- Use VPN for remote access
- Monitor access logs

## Security Configuration

### Password Security

**Good practices:**
1. Use `.env` file instead of command-line environment variables
2. Set restrictive file permissions: `chmod 600 .env`
3. Use strong passwords
4. Rotate passwords regularly
5. Never commit `.env` to version control

### Network Security

**For HTTP mode:**

1. **Firewall Rules:**
```bash
# Allow only specific IPs
iptables -A INPUT -p tcp --dport 3000 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 3000 -j DROP
```

2. **Reverse Proxy with nginx:**
```nginx
server {
    listen 80;
    server_name kingbase-mcp.example.com;
    
    location / {
        auth_basic "KingBase MCP";
        auth_basic_user_file /etc/nginx/.htpasswd;
        
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

3. **SSL/TLS with nginx:**
```nginx
server {
    listen 443 ssl;
    server_name kingbase-mcp.example.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    location / {
        proxy_pass http://localhost:3000;
        # ... other proxy settings
    }
}
```

### Access Control

**Database-level:**
- Create dedicated database user for MCP
- Grant minimal required permissions:

```sql
-- For readonly mode
CREATE USER mcp_readonly WITH PASSWORD 'secure-password';
GRANT CONNECT ON DATABASE kingbase TO mcp_readonly;
GRANT USAGE ON SCHEMA public TO mcp_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO mcp_readonly;

-- For readwrite mode
CREATE USER mcp_readwrite WITH PASSWORD 'secure-password';
GRANT CONNECT ON DATABASE kingbase TO mcp_readwrite;
GRANT USAGE ON SCHEMA public TO mcp_readwrite;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO mcp_readwrite;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE ON TABLES TO mcp_readwrite;
```

### Confirmation Bypass (Use with Caution)

**SKIP_CONFIRM=true**

When enabled, DML and DDL operations execute immediately without user confirmation.

**⚠️ WARNING:** Only use this if:
- You fully understand the risks
- You're in a development/test environment
- You have database backups
- Your MCP client doesn't support Elicitation

```bash
export SKIP_CONFIRM=true
```

## Advanced Settings

### Connection Pooling

The KingBase MCP server uses a connection pool. Default settings are usually sufficient, but you can tune:

```bash
# Maximum number of connections in pool
export DB_MAX_CONNECTIONS=10

# Connection timeout (ms)
export DB_CONNECTION_TIMEOUT=5000

# Idle timeout (ms)
export DB_IDLE_TIMEOUT=30000
```

### Query Timeouts

Set maximum query execution time:

```bash
# Statement timeout in milliseconds (PostgreSQL/KingBase)
export DB_STATEMENT_TIMEOUT=30000
```

This is passed to the database via `SET statement_timeout = ...`

### Logging

Control log verbosity:

```bash
# Log level: debug, info, warn, error
export LOG_LEVEL=info

# Enable query logging (may expose sensitive data)
export LOG_QUERIES=false
```

## Troubleshooting

### Connection Refused

**Symptoms:** Error connecting to database

**Solutions:**
1. Verify `DB_HOST` and `DB_PORT`
2. Check if KingBase is running: `pg_isready -h <host> -p <port>`
3. Verify firewall rules allow connection
4. Check KingBase logs for connection attempts

### Authentication Failed

**Symptoms:** Password authentication failed

**Solutions:**
1. Verify `DB_USER` and `DB_PASSWORD`
2. Check if user exists: `SELECT * FROM pg_user WHERE usename = 'username';`
3. Reset password if needed
4. Check `pg_hba.conf` for authentication method

### Permission Denied

**Symptoms:** Cannot execute operation

**Solutions:**
1. Check current `ACCESS_MODE`
2. Verify database user has required permissions
3. Check schema/table-level permissions
4. Increase `ACCESS_MODE` if appropriate

### Query Timeout

**Symptoms:** Query hangs or times out

**Solutions:**
1. Add appropriate WHERE clauses to limit data
2. Use `LIMIT` to reduce result set
3. Check for missing indexes: `kb_explain` + `kb_list_indexes`
4. Increase `DB_STATEMENT_TIMEOUT`
5. Consider running during off-peak hours

### Memory Issues

**Symptoms:** Server crashes or becomes unresponsive

**Solutions:**
1. Reduce `DB_MAX_CONNECTIONS`
2. Add `LIMIT` to large queries
3. Process data in smaller batches
4. Monitor server resources

## Examples

### Development Environment

```json
{
  "mcpServers": {
    "kingbase-dev": {
      "command": "npx",
      "args": ["-y", "kingbase-mcp-server"],
      "env": {
        "DB_HOST": "localhost",
        "DB_PORT": "54321",
        "DB_USER": "dev_user",
        "DB_PASSWORD": "dev-password",
        "DB_NAME": "dev_db",
        "ACCESS_MODE": "admin",
        "LOG_LEVEL": "debug"
      }
    }
  }
}
```

### Production Read-only

```json
{
  "mcpServers": {
    "kingbase-prod-readonly": {
      "command": "npx",
      "args": ["-y", "kingbase-mcp-server"],
      "env": {
        "DB_HOST": "prod-db.company.com",
        "DB_PORT": "54321",
        "DB_USER": "mcp_readonly",
        "DB_PASSWORD": "secure-password",
        "DB_NAME": "production",
        "DB_SCHEMA": "public",
        "ACCESS_MODE": "readonly",
        "DB_STATEMENT_TIMEOUT": "60000",
        "LOG_LEVEL": "warn"
      }
    }
  }
}
```

### Team Shared Server (HTTP Mode)

**Server side (.env):**
```bash
TRANSPORT=http
MCP_PORT=3000
MCP_HOST=0.0.0.0
ACCESS_MODE=readonly
DB_HOST=db-server.internal
DB_USER=mcp_team
DB_PASSWORD=team-password
DB_NAME=shared_db
SKIP_CONFIRM=false
```

**Client configurations:**
```json
{
  "mcpServers": {
    "kingbase-team": {
      "url": "http://mcp-server.internal:3000/mcp"
    }
  }
}
```

### Multiple Environments

```json
{
  "mcpServers": {
    "kingbase-local": {
      "command": "npx",
      "args": ["-y", "kingbase-mcp-server"],
      "env": {
        "DB_HOST": "localhost",
        "ACCESS_MODE": "admin"
      }
    },
    "kingbase-staging": {
      "command": "npx",
      "args": ["-y", "kingbase-mcp-server"],
      "env": {
        "DB_HOST": "staging-db.company.com",
        "ACCESS_MODE": "readwrite"
      }
    },
    "kingbase-production": {
      "command": "npx",
      "args": ["-y", "kingbase-mcp-server"],
      "env": {
        "DB_HOST": "prod-db.company.com",
        "ACCESS_MODE": "readonly"
      }
    }
  }
}
```

### systemd Service (HTTP Mode)

Create `/etc/systemd/system/kingbase-mcp.service`:

```ini
[Unit]
Description=KingBase MCP Server
After=network.target

[Service]
Type=simple
Environment=TRANSPORT=http
Environment=MCP_PORT=3000
Environment=ACCESS_MODE=readonly
Environment=DB_HOST=localhost
Environment=DB_PORT=54321
Environment=DB_USER=mcp_service
Environment=DB_PASSWORD=your-password
Environment=DB_NAME=kingbase
WorkingDirectory=/opt/kingbase-mcp
ExecStart=/usr/bin/npx kingbase-mcp-server
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable and start:
```bash
sudo systemctl daemon-reload
sudo systemctl enable kingbase-mcp
sudo systemctl start kingbase-mcp
sudo systemctl status kingbase-mcp
```

View logs:
```bash
sudo journalctl -u kingbase-mcp -f
```
