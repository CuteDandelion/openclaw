---
created: 2026-01-31
modified: 2026-01-31
author: AI Assistant
tags: [architecture, openclaw, system-design, documentation]
---

# OpenClaw System Architecture Documentation

## Executive Summary

**OpenClaw** is a personal AI assistant platform designed to run on your own devices. It provides a unified interface for interacting with AI models across multiple messaging channels (WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, and more) and platforms (macOS, iOS, Android, Web).

### Key Characteristics
- **Local-first architecture**: The Gateway runs locally as the control plane
- **Multi-channel support**: 12+ messaging platforms integrated
- **Multi-platform**: Native apps for macOS, iOS, and Android
- **Extensible**: Plugin system for skills and extensions
- **Secure**: Device pairing, authentication, and sandboxing support

---

## System Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           OpenClaw System Architecture                       │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                               CLIENT LAYER                                   │
├─────────────────┬─────────────────┬─────────────────┬────────────────────────┤
│   macOS App     │   CLI Tool      │   Web UI        │   Mobile Apps          │
│  (Menu Bar)     │  (openclaw)     │  (Control UI)   │  (iOS/Android)         │
│                 │                 │                 │                        │
│ • Voice Wake    │ • Commands      │ • Dashboard     │ • Voice Trigger        │
│ • Canvas        │ • Agent Send    │ • WebChat       │ • Camera/Screen        │
│ • System Tools  │ • Wizard        │ • Config        │ • Node Commands        │
└────────┬────────┴────────┬────────┴────────┬────────┴────────┬───────────────┘
         │                 │                 │                 │
         │                 │                 │                 │
         └─────────────────┴────────┬────────┴─────────────────┘
                                    │
                         ┌──────────▼──────────┐
                         │   WebSocket (WS)    │
                         │   Protocol          │
                         │   Port: 18789       │
                         └──────────┬──────────┘
                                    │
┌───────────────────────────────────▼─────────────────────────────────────────┐
│                              GATEWAY LAYER                                   │
│                         (Control Plane - Single Instance)                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Session    │  │    Agent     │  │   Channel    │  │   Plugin     │     │
│  │   Manager    │  │   Runtime    │  │   Manager    │  │   Manager    │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                 │                 │                 │             │
│  ┌──────▼───────┐  ┌──────▼───────┐  ┌──────▼───────┐  ┌──────▼───────┐     │
│  │   Config     │  │   Pi Agent   │  │   Routing    │  │   Skills     │     │
│  │   Manager    │  │   Core       │  │   Engine     │  │   Registry   │     │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Browser    │  │   Canvas     │  │   Security   │  │   Cron       │     │
│  │   Control    │  │   Host       │  │   Manager    │  │   Jobs       │     │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                                              │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
         ┌─────────────────────────────┼─────────────────────────────┐
         │                             │                             │
┌────────▼────────┐         ┌──────────▼──────────┐      ┌──────────▼──────────┐
│  CHANNEL LAYER  │         │   DEVICE NODES      │      │   EXTERNAL APIs     │
├─────────────────┤         ├─────────────────────┤      ├─────────────────────┤
│ • WhatsApp      │         │ • macOS Node        │      │ • Anthropic         │
│ • Telegram      │         │ • iOS Node          │      │ • OpenAI            │
│ • Slack         │         │ • Android Node      │      │ • AWS Bedrock       │
│ • Discord       │         │ • Headless Node     │      │ • Ollama            │
│ • Signal        │         │                     │      │                     │
│ • iMessage      │         │ Capabilities:       │      │                     │
│ • Google Chat   │         │ • canvas.*          │      │                     │
│ • BlueBubbles   │         │ • camera.*          │      │                     │
│ • Matrix        │         │ • screen.record     │      │                     │
│ • Zalo          │         │ • location.get      │      │                     │
│ • MS Teams      │         │ • system.run        │      │                     │
│ • WebChat       │         │ • system.notify     │      │                     │
└─────────────────┘         └─────────────────────┘      └─────────────────────┘
```

---

## Core Components

### 1. Gateway (Control Plane)

The Gateway is the central hub that manages all connections, sessions, and message routing.

#### Responsibilities
- **WebSocket Server**: Handles client connections on port 18789
- **Session Management**: Manages AI assistant sessions and conversation history
- **Channel Routing**: Routes messages between channels and agents
- **Authentication**: Device pairing and token-based authentication
- **Plugin System**: Loads and manages extensions
- **Cron Scheduling**: Manages scheduled tasks and wakeups

#### Key Files
- `src/gateway/server.ts` - Main server entry point
- `src/gateway/server.impl.ts` - Server implementation
- `src/gateway/client.ts` - WebSocket client handling
- `src/gateway/protocol/` - Protocol definitions

#### Protocol
- **Transport**: WebSocket with JSON frames
- **Handshake**: `connect` frame required as first message
- **Authentication**: Token-based (optional via `OPENCLAW_GATEWAY_TOKEN`)
- **Request/Response**: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- **Events**: `{type:"event", event, payload, seq?, stateVersion?}`

### 2. Agent Runtime

The agent runtime is responsible for executing AI assistant tasks using the Pi Agent Core.

#### Responsibilities
- **Model Integration**: Connects to LLM providers (Anthropic, OpenAI, etc.)
- **Tool Execution**: Manages tool calls and results
- **Context Management**: Builds and maintains conversation context
- **Streaming**: Streams responses in real-time
- **Sandboxing**: Optional Docker sandboxing for security

#### Key Files
- `src/agents/` - Agent runtime and related components
- `src/agents/pi-embedded-runner/` - Pi agent integration
- `src/agents/tools/` - Tool definitions and execution

#### Agent Loop Flow
1. **Intake**: Receive message from channel or CLI
2. **Context Assembly**: Build system prompt with tools, skills, history
3. **Model Inference**: Send to LLM provider
4. **Tool Execution**: Execute any tool calls
5. **Streaming**: Stream assistant responses
6. **Persistence**: Save to session transcript

### 3. Channel Manager

Manages connections to various messaging platforms.

#### Supported Channels

**Core Channels** (built-in):
- **WhatsApp** (`src/whatsapp/`) - Using Baileys library
- **Telegram** (`src/telegram/`) - Using grammY framework
- **Slack** (`src/slack/`) - Using Bolt framework
- **Discord** (`src/discord/`) - Using discord.js
- **Signal** (`src/signal/`) - Using signal-cli
- **iMessage** (`src/imessage/`) - macOS only
- **WebChat** (`src/web/`) - Built-in web interface

**Extension Channels** (`extensions/`):
- BlueBubbles
- Google Chat
- Matrix
- Microsoft Teams
- Zalo (Personal & Business)
- LINE
- Mattermost
- Nostr
- Twitch
- Voice Call

#### Channel Features
- **Allowlists**: Control who can interact with the assistant
- **Group Support**: Handle group conversations with mention gating
- **Media Support**: Images, audio, video, documents
- **DM Policies**: Pairing, open, or closed direct message policies

### 4. Session Management

Manages conversation sessions and state.

#### Session Types
- **Main Session**: Personal 1:1 conversation with the assistant
- **Group Sessions**: Isolated sessions per group/channel
- **Multi-Agent**: Route different channels to different agents

#### Session Features
- **History**: Persistent conversation history
- **Compaction**: Automatic summarization to manage context window
- **Pruning**: Remove old tool results from active context
- **Isolation**: Sessions are isolated by design

### 5. Plugin/Extension System

Extensible architecture for adding new capabilities.

#### Plugin Types
- **Channel Extensions**: Add new messaging platforms
- **Skills**: Add new capabilities via SKILL.md files
- **Hooks**: Intercept lifecycle events

#### Key Extension Points
- `before_agent_start` - Inject context before run
- `agent_end` - Inspect results after completion
- `before_tool_call` / `after_tool_call` - Intercept tool execution
- `message_received` / `message_sent` - Message hooks

### 6. Canvas & A2UI

Agent-driven visual workspace.

#### Components
- **Canvas Host**: Serves agent-editable HTML (port 18793)
- **A2UI**: Agent-to-User interface for rich interactions
- **Node Commands**: Device-local canvas operations

### 7. Device Nodes

Remote device capabilities.

#### Node Types
- **macOS Node**: System commands, notifications, permissions
- **iOS Node**: Camera, screen recording, voice
- **Android Node**: Camera, screen recording, TTS
- **Headless Node**: Server-side capabilities

#### Capabilities
- `canvas.push` / `canvas.reset` / `canvas.eval`
- `camera.snap` / `camera.clip`
- `screen.record`
- `location.get`
- `system.run` / `system.notify`

### 8. Browser Control

Automated browser for web interactions.

#### Features
- **Chrome/Chromium Control**: Via Playwright/CDP
- **Snapshots**: Page screenshots and state
- **Actions**: Click, type, scroll, navigate
- **Uploads**: File upload support
- **Profiles**: Isolated browsing profiles

### 9. Security Manager

Security and access control.

#### Features
- **Device Pairing**: Approve devices before access
- **DM Policies**: Control direct message access
- **Sandboxing**: Docker sandbox for non-main sessions
- **Allowlists**: Per-channel user allowlists
- **Token Auth**: Optional gateway token authentication

### 10. Configuration System

Flexible configuration management.

#### Configuration Sources
- `~/.openclaw/openclaw.json` - Main config file
- Environment variables
- CLI flags
- Workspace-specific configs

#### Key Config Sections
- `gateway` - Gateway settings (port, bind, auth)
- `agents` - Agent defaults (model, thinking level, workspace)
- `channels` - Channel configurations
- `models` - Model profiles and failovers

---

## Data Flow

### Message Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Channel   │────▶│   Gateway   │────▶│   Session   │────▶│    Agent    │
│  (Inbound)  │     │  (Router)   │     │  (Context)  │     │  (Runtime)  │
└─────────────┘     └─────────────┘     └─────────────┘     └──────┬──────┘
                                                                   │
                                                                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Channel   │◀────│   Gateway   │◀────│   Session   │◀────│   Tooling   │
│  (Outbound) │     │  (Router)   │     │  (Update)   │     │  (Results)  │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

### Agent Execution Flow

```
1. Gateway receives message via channel or CLI
        │
        ▼
2. Gateway routes to appropriate session
        │
        ▼
3. Session loads context (history, tools, skills)
        │
        ▼
4. Agent runtime calls LLM provider
        │
        ▼
5. LLM returns response (possibly with tool calls)
        │
        ▼
6. Tools are executed (browser, exec, etc.)
        │
        ▼
7. Tool results returned to LLM
        │
        ▼
8. Final response streamed to user
        │
        ▼
9. Session transcript updated
```

---

## Technology Stack

### Backend (Gateway)
- **Runtime**: Node.js 22+
- **Language**: TypeScript (ESM)
- **Framework**: Express.js (HTTP), ws (WebSocket)
- **AI Runtime**: pi-agent-core (by Mario Zechner)

### Frontend (UI)
- **Framework**: Lit (Web Components)
- **Styling**: CSS with design tokens
- **Build**: Rolldown

### Mobile Apps
- **iOS**: SwiftUI (Observation framework)
- **Android**: Kotlin with Jetpack Compose
- **macOS**: SwiftUI menu bar app

### Key Dependencies
- `@mariozechner/pi-agent-core` - AI agent runtime
- `@whiskeysockets/baileys` - WhatsApp Web API
- `grammy` - Telegram Bot framework
- `@slack/bolt` - Slack app framework
- `discord.js` - Discord API
- `playwright-core` - Browser automation
- `@sinclair/typebox` - JSON Schema validation
- `ws` - WebSocket library

### Development Tools
- **Package Manager**: pnpm (preferred) or bun
- **Build**: TypeScript compiler + custom scripts
- **Testing**: Vitest with V8 coverage
- **Linting**: Oxlint + Oxfmt
- **CI**: GitHub Actions

---

## Deployment Options

### 1. Local Development
```bash
pnpm install
pnpm build
pnpm openclaw onboard --install-daemon
```

### 2. npm Global Install
```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

### 3. Docker
```bash
docker-compose up -d
```

### 4. Remote Server (Linux VPS)
- Run Gateway on remote host
- Connect via Tailscale or SSH tunnel
- Device nodes connect locally for device actions

---

## Security Model

### Authentication Layers
1. **Device Pairing**: Approve new devices via pairing codes
2. **Gateway Token**: Optional global token (`OPENCLAW_GATEWAY_TOKEN`)
3. **Channel Auth**: Per-channel authentication (bot tokens, etc.)

### Sandboxing
- **Main Session**: Full access (for personal use)
- **Non-Main Sessions**: Optional Docker sandbox
- **Sandbox Tools**: Limited toolset (bash, process, read, write, edit)

### DM Policies
- **pairing**: Unknown senders get pairing code
- **open**: Allow all (with allowlist)
- **closed**: Reject unknown senders

### Permissions (macOS TCC)
- Screen Recording
- Camera
- Microphone
- Location
- Notifications
- Accessibility (for system commands)

---

## Scaling Considerations

### Single Gateway Design
- One Gateway per host
- All channels connect through single Gateway
- Sessions are in-memory with persistence

### Remote Gateway Pattern
- Gateway runs on server
- Clients connect via Tailscale/SSH
- Device nodes run locally for device actions

### Session Isolation
- Each session is independent
- No shared state between sessions
- Compaction manages context window per session

---

## Extension Points

### Adding New Channels
1. Create extension in `extensions/<channel>/`
2. Implement channel interface
3. Add to plugin registry
4. Update documentation

### Adding New Tools
1. Define tool schema in `src/agents/tools/`
2. Implement tool executor
3. Register in tool registry
4. Add tests

### Adding New Skills
1. Create skill directory in `~/.openclaw/workspace/skills/<skill>/`
2. Add `SKILL.md` with instructions
3. Reference in `AGENTS.md` if needed

---

## Monitoring & Observability

### Logging
- Structured logging with tslog
- Log levels: DEBUG, INFO, WARN, ERROR
- Configurable via `logging` config section

### Health Checks
- `health` RPC method
- WebSocket heartbeat
- Channel status probes

### Metrics
- Session counts
- Message throughput
- Tool usage stats
- Model token usage

---

## Troubleshooting

### Common Issues
- **Gateway won't start**: Check port availability
- **Channel not connecting**: Verify credentials
- **Messages not routing**: Check allowlists and DM policies
- **Tools not working**: Check permissions and sandbox settings

### Debug Tools
- `openclaw doctor` - Diagnose common issues
- `openclaw channels status` - Check channel health
- `/status` command - Session status
- `/context` command - Context breakdown

---

## References

- [OpenClaw Documentation](https://docs.openclaw.ai)
- [Gateway Protocol](https://docs.openclaw.ai/gateway/protocol)
- [Architecture Overview](https://docs.openclaw.ai/concepts/architecture)
- [Agent Loop](https://docs.openclaw.ai/concepts/agent-loop)
- [Security Guide](https://docs.openclaw.ai/gateway/security)

---

*Generated from source code analysis on 2026-01-31*
