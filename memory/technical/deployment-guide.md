---
created: 2026-01-31
modified: 2026-01-31
author: AI Assistant
tags: [deployment, configuration, installation, docker, openclaw]
---

# OpenClaw Deployment & Configuration Guide

## Table of Contents
1. [Installation Methods](#installation-methods)
2. [Configuration Guide](#configuration-guide)
3. [Docker Deployment](#docker-deployment)
4. [Remote Gateway Setup](#remote-gateway-setup)
5. [Production Considerations](#production-considerations)
6. [Troubleshooting](#troubleshooting)

---

## Installation Methods

### Method 1: npm Global Install (Recommended)

The simplest way to install OpenClaw for end users.

#### Requirements
- Node.js ≥ 22.12.0
- npm or pnpm

#### Installation

```bash
# Using npm
npm install -g openclaw@latest

# Using pnpm (recommended)
pnpm add -g openclaw@latest

# Using bun
bun add -g openclaw@latest
```

#### Post-Installation

```bash
# Run onboarding wizard
openclaw onboard --install-daemon

# Verify installation
openclaw doctor

# Check version
openclaw --version
```

### Method 2: From Source (Development)

For developers who want to contribute or customize.

#### Requirements
- Node.js ≥ 22.12.0
- pnpm (preferred) or bun
- Git

#### Clone and Build

```bash
# Clone repository
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Install dependencies
pnpm install

# Build the project
pnpm build

# Build UI (first time only)
pnpm ui:build

# Run in development mode
pnpm gateway:watch
```

#### Development Commands

```bash
# Run CLI in dev mode
pnpm openclaw --help

# Run tests
pnpm test

# Run tests with coverage
pnpm test:coverage

# Run linting
pnpm lint

# Format code
pnpm format:fix
```

### Method 3: Docker

For containerized deployments.

#### Using Pre-built Image

```bash
# Pull image (if available)
docker pull openclaw/openclaw:latest

# Or build locally
docker build -t openclaw:local .
```

#### Running with Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  openclaw-gateway:
    image: openclaw:local
    environment:
      HOME: /home/node
      OPENCLAW_GATEWAY_TOKEN: ${OPENCLAW_GATEWAY_TOKEN}
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      - "18789:18789"
      - "18790:18790"
    init: true
    restart: unless-stopped
    command:
      - node
      - dist/index.js
      - gateway
      - --bind
      - lan
      - --port
      - "18789"
```

```bash
# Start services
docker-compose up -d

# View logs
docker-compose logs -f openclaw-gateway

# Stop services
docker-compose down
```

---

## Configuration Guide

### Configuration File Location

```
Linux/macOS: ~/.openclaw/openclaw.json
Windows:     %USERPROFILE%\.openclaw\openclaw.json
```

### Minimal Configuration

```json5
{
  "agents": {
    "defaults": {
      "model": "anthropic/claude-opus-4-5"
    }
  }
}
```

### Full Configuration Reference

```json5
{
  // ==========================================
  // GATEWAY CONFIGURATION
  // ==========================================
  gateway: {
    // Network binding
    // Options: "loopback" (127.0.0.1), "lan" (0.0.0.0), or specific IP
    bind: "loopback",
    
    // WebSocket port
    port: 18789,
    
    // Authentication
    auth: {
      // Mode: "none", "password", or "token"
      mode: "none",
      
      // Global gateway token (for "token" mode)
      token: null,
      
      // Password for Web UI (for "password" mode)
      password: null,
      
      // Allow Tailscale identity headers
      allowTailscale: true,
    },
    
    // Tailscale integration
    tailscale: {
      // Mode: "off", "serve", or "funnel"
      mode: "off",
      
      // Reset on gateway exit
      resetOnExit: false,
    },
    
    // CORS settings
    cors: {
      origins: ["*"],
    },
  },
  
  // ==========================================
  // AGENT CONFIGURATION
  // ==========================================
  agents: {
    defaults: {
      // Default model (format: provider/model)
      model: "anthropic/claude-opus-4-5",
      
      // Thinking level: "off", "minimal", "low", "medium", "high", "xhigh"
      thinking: "medium",
      
      // Verbose output
      verbose: false,
      
      // Workspace directory
      workspace: "~/.openclaw/workspace",
      
      // Sandbox configuration
      sandbox: {
        // Mode: "off" or "non-main"
        mode: "off",
        
        // Allowed tools in sandbox
        allowlist: [
          "bash",
          "process",
          "read",
          "write",
          "edit",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn"
        ],
        
        // Denied tools in sandbox
        denylist: [
          "browser",
          "canvas",
          "nodes",
          "cron",
          "discord",
          "gateway"
        ],
      },
      
      // Max characters for bootstrap files
      bootstrapMaxChars: 20000,
      
      // Agent timeout in seconds
      timeoutSeconds: 600,
    },
    
    // Per-agent overrides
    overrides: {
      "work-agent": {
        model: "openai/gpt-4o",
        workspace: "~/work/projects",
      },
      "personal-agent": {
        model: "anthropic/claude-opus-4-5",
        thinking: "high",
      },
    },
  },
  
  // ==========================================
  // CHANNEL CONFIGURATION
  // ==========================================
  channels: {
    // WhatsApp (via Baileys)
    whatsapp: {
      enabled: true,
      
      // Allow specific phone numbers
      allowFrom: [
        "+1234567890",
        "+0987654321"
      ],
      
      // Group configuration
      groups: {
        "group-id-1": {
          // Require @mention to trigger agent
          requireMention: true,
          
          // Allow specific members
          allowFrom: ["+1234567890"]
        },
        "group-id-2": {
          requireMention: false,
          // Allow all members
          allowFrom: ["*"]
        }
      },
    },
    
    // Telegram (via grammY)
    telegram: {
      enabled: true,
      
      // Bot token (can use env var: ${TELEGRAM_BOT_TOKEN})
      botToken: "${TELEGRAM_BOT_TOKEN}",
      
      // Webhook URL (optional, defaults to polling)
      webhookUrl: null,
      
      // Allowed usernames
      allowFrom: ["@username1", "@username2"],
      
      // Group configuration
      groups: {
        "*": {
          requireMention: true,
        }
      },
      
      // Direct message policy
      dm: {
        policy: "pairing",  // "pairing", "open", "closed"
        allowFrom: [],
      },
    },
    
    // Slack (via Bolt)
    slack: {
      enabled: true,
      
      // Bot token
      botToken: "${SLACK_BOT_TOKEN}",
      
      // App token (for Socket Mode)
      appToken: "${SLACK_APP_TOKEN}",
      
      // DM policy
      dm: {
        policy: "pairing",
        allowFrom: [],
      },
      
      // Channel allowlist
      channels: [
        "C1234567890"
      ],
    },
    
    // Discord (via discord.js)
    discord: {
      enabled: true,
      
      // Bot token
      token: "${DISCORD_BOT_TOKEN}",
      
      // DM policy
      dm: {
        policy: "pairing",
        allowFrom: [],
      },
      
      // Guild configuration
      guilds: {
        "guild-id-1": {
          channels: ["channel-id-1", "channel-id-2"],
        }
      },
      
      // Native slash commands
      commands: {
        native: false,
        text: true,
        useAccessGroups: false,
      },
      
      // Media size limit (MB)
      mediaMaxMb: 25,
    },
    
    // Signal (via signal-cli)
    signal: {
      enabled: false,
      
      // Signal account (phone number)
      account: "+1234567890",
      
      // signal-cli socket path
      socketPath: "/var/run/signal-cli/socket",
    },
    
    // iMessage (macOS only)
    imessage: {
      enabled: false,
      
      // Allow specific contacts
      allowFrom: [
        "email@example.com",
        "+1234567890"
      ],
      
      // Group configuration
      groups: {
        "*": {
          requireMention: false,
        }
      },
    },
    
    // Google Chat
    googlechat: {
      enabled: false,
      // Requires service account setup
    },
  },
  
  // ==========================================
  // MODEL CONFIGURATION
  // ==========================================
  models: {
    // Model profiles
    profiles: [
      {
        id: "default",
        provider: "anthropic",
        model: "claude-opus-4-5",
        authProfile: "anthropic-main",
      },
      {
        id: "fast",
        provider: "anthropic",
        model: "claude-sonnet-4",
        authProfile: "anthropic-main",
      },
      {
        id: "backup",
        provider: "openai",
        model: "gpt-4o",
        authProfile: "openai-main",
      },
    ],
    
    // Failover configuration
    failovers: [
      {
        from: "anthropic/claude-opus-4-5",
        to: "openai/gpt-4o",
        on: ["rate-limit", "error", "timeout"],
        maxRetries: 2,
      },
    ],
    
    // Provider-specific settings
    providers: {
      anthropic: {
        baseUrl: "https://api.anthropic.com",
        timeout: 120000,
      },
      openai: {
        baseUrl: "https://api.openai.com",
        timeout: 60000,
      },
    },
  },
  
  // ==========================================
  // AUTHENTICATION PROFILES
  // ==========================================
  authProfiles: {
    "anthropic-main": {
      type: "oauth",
      provider: "anthropic",
    },
    "openai-main": {
      type: "apiKey",
      key: "${OPENAI_API_KEY}",
    },
    "bedrock-main": {
      type: "aws",
      region: "us-east-1",
      accessKeyId: "${AWS_ACCESS_KEY_ID}",
      secretAccessKey: "${AWS_SECRET_ACCESS_KEY}",
    },
  },
  
  // ==========================================
  // BROWSER CONFIGURATION
  // ==========================================
  browser: {
    enabled: true,
    
    // Browser accent color
    color: "#FF4500",
    
    // Run headless
    headless: false,
    
    // Chrome/Chromium executable path
    executablePath: null,  // Auto-detect
    
    // User data directory
    userDataDir: "~/.openclaw/browser-profiles",
    
    // Default viewport
    viewport: {
      width: 1280,
      height: 720,
    },
    
    // Additional arguments
    args: [],
  },
  
  // ==========================================
  // CANVAS CONFIGURATION
  // ==========================================
  canvas: {
    enabled: true,
    
    // Canvas host port
    port: 18793,
    
    // A2UI settings
    a2ui: {
      enabled: true,
      bundlePath: "./dist/canvas-host/a2ui",
    },
  },
  
  // ==========================================
  // CRON CONFIGURATION
  // ==========================================
  cron: {
    enabled: true,
    
    jobs: [
      {
        name: "morning-summary",
        schedule: "0 9 * * *",  // 9 AM daily
        command: "openclaw agent --message 'Daily briefing'",
        timezone: "America/New_York",
      },
      {
        name: "weekly-cleanup",
        schedule: "0 0 * * 0",  // Midnight Sunday
        command: "openclaw sessions compact --all",
      },
    ],
  },
  
  // ==========================================
  // LOGGING CONFIGURATION
  // ==========================================
  logging: {
    // Log level: "debug", "info", "warn", "error"
    level: "info",
    
    // Pretty print logs
    pretty: true,
    
    // Log destination: "console", "file", or both
    destination: "console",
    
    // File log settings (if destination includes "file")
    file: {
      path: "~/.openclaw/logs/openclaw.log",
      maxSize: "10m",
      maxFiles: 5,
    },
    
    // Include timestamp
    timestamp: true,
    
    // Redact sensitive info
    redact: ["apiKey", "token", "password"],
  },
  
  // ==========================================
  // HOOKS CONFIGURATION
  // ==========================================
  hooks: {
    enabled: true,
    
    // Hook scripts directory
    directory: "~/.openclaw/hooks",
    
    // Individual hooks
    scripts: {
      "agent:bootstrap": "./hooks/bootstrap.sh",
      "agent:end": "./hooks/agent-end.sh",
      "message:received": "./hooks/message-received.sh",
    },
  },
  
  // ==========================================
  // REMOTE ACCESS
  // ==========================================
  remote: {
    // Enable remote access
    enabled: false,
    
    // Tailscale settings
    tailscale: {
      enabled: false,
      funnel: false,
    },
    
    // SSH tunnel settings
    ssh: {
      enabled: false,
      host: "gateway.example.com",
      port: 22,
      keyPath: "~/.ssh/id_rsa",
    },
  },
}
```

### Environment Variables

Create a `.env` file in your workspace or export these variables:

```bash
# Gateway
export OPENCLAW_GATEWAY_TOKEN="your-secret-token"
export OPENCLAW_GATEWAY_BIND="loopback"
export OPENCLAW_GATEWAY_PORT="18789"

# LLM Providers
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."

# Channels
export TELEGRAM_BOT_TOKEN="123456:ABC..."
export SLACK_BOT_TOKEN="xoxb-..."
export SLACK_APP_TOKEN="xapp-..."
export DISCORD_BOT_TOKEN="..."

# OAuth (for web provider)
export CLAUDE_AI_SESSION_KEY="..."
export CLAUDE_WEB_SESSION_KEY="..."
export CLAUDE_WEB_COOKIE="..."
```

---

## Docker Deployment

### Building Docker Image

```bash
# Build from Dockerfile
docker build -t openclaw:local .

# Build with specific Node version
docker build --build-arg NODE_VERSION=22 -t openclaw:local .
```

### Running Container

```bash
# Run basic container
docker run -d \
  --name openclaw-gateway \
  -p 18789:18789 \
  -v ~/.openclaw:/home/node/.openclaw \
  openclaw:local

# Run with environment variables
docker run -d \
  --name openclaw-gateway \
  -p 18789:18789 \
  -e OPENCLAW_GATEWAY_TOKEN="secret" \
  -e ANTHROPIC_API_KEY="sk-ant-..." \
  -v ~/.openclaw:/home/node/.openclaw \
  openclaw:local

# Run with custom config
docker run -d \
  --name openclaw-gateway \
  -p 18789:18789 \
  -v $(pwd)/custom-config.json:/home/node/.openclaw/openclaw.json:ro \
  -v ~/.openclaw/workspace:/home/node/.openclaw/workspace \
  openclaw:local
```

### Docker Compose Production Setup

```yaml
version: '3.8'

services:
  openclaw-gateway:
    image: openclaw:local
    container_name: openclaw-gateway
    hostname: openclaw-gateway
    
    environment:
      # Node environment
      HOME: /home/node
      NODE_ENV: production
      
      # Gateway auth
      OPENCLAW_GATEWAY_TOKEN: ${OPENCLAW_GATEWAY_TOKEN}
      
      # LLM providers
      ANTHROPIC_API_KEY: ${ANTHROPIC_API_KEY}
      OPENAI_API_KEY: ${OPENAI_API_KEY}
      
      # Channel tokens
      TELEGRAM_BOT_TOKEN: ${TELEGRAM_BOT_TOKEN}
      SLACK_BOT_TOKEN: ${SLACK_BOT_TOKEN}
      SLACK_APP_TOKEN: ${SLACK_APP_TOKEN}
      DISCORD_BOT_TOKEN: ${DISCORD_BOT_TOKEN}
    
    volumes:
      # Config persistence
      - openclaw-config:/home/node/.openclaw
      # Workspace
      - openclaw-workspace:/home/node/.openclaw/workspace
      # Logs
      - openclaw-logs:/home/node/.openclaw/logs
    
    ports:
      - "127.0.0.1:18789:18789"  # Gateway (localhost only)
      - "127.0.0.1:18793:18793"  # Canvas (localhost only)
    
    networks:
      - openclaw-network
    
    init: true
    restart: unless-stopped
    
    # Resource limits
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
    
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:18789/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    
    command:
      - node
      - dist/index.js
      - gateway
      - --bind
      - "0.0.0.0"
      - --port
      - "18789"
      - --verbose

  # Optional: Redis for caching
  redis:
    image: redis:7-alpine
    container_name: openclaw-redis
    volumes:
      - redis-data:/data
    networks:
      - openclaw-network
    restart: unless-stopped

volumes:
  openclaw-config:
  openclaw-workspace:
  openclaw-logs:
  redis-data:

networks:
  openclaw-network:
    driver: bridge
```

### Docker Best Practices

1. **Use volumes for persistence**
   ```bash
   -v openclaw-config:/home/node/.openclaw
   ```

2. **Set resource limits**
   ```yaml
   deploy:
     resources:
       limits:
         cpus: '2.0'
         memory: 2G
   ```

3. **Enable health checks**
   ```yaml
   healthcheck:
     test: ["CMD", "curl", "-f", "http://localhost:18789/health"]
   ```

4. **Use environment files**
   ```bash
   docker-compose --env-file .env.production up -d
   ```

---

## Remote Gateway Setup

### Scenario 1: VPS/Cloud Server

Running OpenClaw Gateway on a remote server with local clients.

#### Server Setup (Remote)

```bash
# SSH to server
ssh user@gateway-server

# Install Node.js 22+
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install OpenClaw
npm install -g openclaw@latest

# Configure
openclaw config set gateway.bind lan
echo 'export OPENCLAW_GATEWAY_TOKEN="secret-token"' >> ~/.bashrc
source ~/.bashrc

# Start with systemd (create service)
sudo tee /etc/systemd/system/openclaw.service > /dev/null <<EOF
[Unit]
Description=OpenClaw Gateway
After=network.target

[Service]
Type=simple
User=$USER
Environment="OPENCLAW_GATEWAY_TOKEN=secret-token"
Environment="HOME=/home/$USER"
ExecStart=/usr/bin/openclaw gateway --bind lan --port 18789
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable openclaw
sudo systemctl start openclaw
```

#### Client Setup (Local)

```bash
# Method 1: SSH Tunnel
ssh -N -L 18789:localhost:18789 user@gateway-server

# Then use locally
openclaw agent --message "Hello"

# Method 2: Tailscale (preferred)
# On server
sudo tailscale up --advertise-routes=10.0.0.0/24

# On client
sudo tailscale up
tailscale ssh user@gateway-server

# Connect directly
curl http://gateway-server:18789/health
```

### Scenario 2: Tailscale Funnel

Expose Gateway publicly via Tailscale Funnel (with password protection).

```json5
// openclaw.json on server
{
  gateway: {
    bind: "loopback",  // Important: keep bound to localhost
    port: 18789,
    auth: {
      mode: "password",
      password: "strong-password",
    },
    tailscale: {
      mode: "funnel",  // Enable public access
      resetOnExit: true,
    },
  },
}
```

```bash
# On server
openclaw gateway

# Tailscale will output public URL
# https://gateway-server.tailnet-name.ts.net

# On client anywhere
openclaw config set gateway.host https://gateway-server.tailnet-name.ts.net
openclaw agent --message "Hello" --auth-password strong-password
```

### Scenario 3: Split Gateway and Nodes

Gateway runs on server, device nodes run locally.

```
┌─────────────────────────────────────────────────────────────┐
│                      Remote Server                          │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   Gateway   │◀───│  Session    │───▶│   LLM API   │     │
│  │   (18789)   │    │  Manager    │    │  (Claude)   │     │
│  └──────┬──────┘    └─────────────┘    └─────────────┘     │
│         │                                                   │
└─────────┼───────────────────────────────────────────────────┘
          │ WebSocket
          │
┌─────────┼───────────────────────────────────────────────────┐
│         │              Local Network                          │
│         │                                                   │
│  ┌──────▼──────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   Node      │───▶│  macOS      │───▶│  Camera/    │     │
│  │   (iOS)     │    │  System     │    │  Screen     │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│                                                            │
│  ┌─────────────┐    ┌─────────────┐                        │
│  │   CLI       │───▶│  Local      │                        │
│  │   Client    │    │  Commands   │                        │
│  └─────────────┘    └─────────────┘                        │
└─────────────────────────────────────────────────────────────┘
```

---

## Production Considerations

### Security Checklist

- [ ] Set strong gateway token (`gateway.auth.token`)
- [ ] Enable authentication (`gateway.auth.mode: "token"` or `"password"`)
- [ ] Configure DM policies to "pairing" for all channels
- [ ] Set up allowlists for channels
- [ ] Use sandboxing for non-main sessions
- [ ] Enable audit logging
- [ ] Regularly rotate API keys
- [ ] Keep OpenClaw updated

### Performance Optimization

1. **Session Pruning**
   ```json5
   {
     agents: {
       defaults: {
         // Compact sessions after N messages
         compactionThreshold: 50,
       }
     }
   }
   ```

2. **Resource Limits**
   ```bash
   # Docker memory limits
   --memory=2g --memory-swap=2g
   
   # Node.js memory
   NODE_OPTIONS="--max-old-space-size=1536"
   ```

3. **Connection Pooling**
   - Channels maintain persistent connections
   - Gateway reuses WebSocket connections
   - HTTP keep-alive for API calls

### Monitoring

1. **Health Checks**
   ```bash
   # Gateway health
   curl http://localhost:18789/health
   
   # Channel status
   openclaw channels status
   
   # Full status
   openclaw status --all
   ```

2. **Logging**
   ```json5
   {
     logging: {
       level: "info",
       destination: "file",
       file: {
         path: "/var/log/openclaw/openclaw.log",
         maxSize: "100m",
         maxFiles: 10,
       }
     }
   }
   ```

3. **Metrics** (if available)
   - Session count
   - Message throughput
   - Tool usage stats
   - API latency

### Backup Strategy

```bash
# Backup script
#!/bin/bash
BACKUP_DIR="/backups/openclaw/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Config
cp ~/.openclaw/openclaw.json "$BACKUP_DIR/"

# Sessions
tar czf "$BACKUP_DIR/sessions.tar.gz" ~/.openclaw/sessions/

# Workspace
tar czf "$BACKUP_DIR/workspace.tar.gz" ~/.openclaw/workspace/

# Credentials (encrypted)
tar czf - ~/.openclaw/credentials/ | gpg --encrypt --recipient backup@example.com > "$BACKUP_DIR/credentials.tar.gz.gpg"
```

---

## Troubleshooting

### Gateway Won't Start

```bash
# Check port availability
lsof -i :18789
netstat -tlnp | grep 18789

# Check logs
openclaw gateway --verbose

# Check config validity
openclaw doctor
```

### Channel Connection Issues

```bash
# Check channel status
openclaw channels status

# Reconnect specific channel
openclaw channels restart whatsapp

# View channel logs
openclaw logs --channel whatsapp
```

### Model/API Errors

```bash
# Check auth profiles
openclaw auth profiles

# Test model connection
openclaw models test anthropic/claude-opus-4-5

# Check rate limits
openclaw usage
```

### Permission Issues

```bash
# Fix config permissions
chmod 600 ~/.openclaw/openclaw.json

# Fix workspace permissions
chmod -R 700 ~/.openclaw/workspace

# Fix session permissions
chmod -R 700 ~/.openclaw/sessions
```

### Docker Issues

```bash
# View container logs
docker-compose logs -f openclaw-gateway

# Exec into container
docker-compose exec openclaw-gateway sh

# Check container health
docker-compose ps
```

---

*Generated from source code analysis on 2026-01-31*
