---
created: 2026-01-31
modified: 2026-01-31
author: AI Assistant
tags: [architecture, diagrams, mermaid, openclaw]
---

# OpenClaw Architecture Diagrams

## Table of Contents
1. [C4 Context Diagram](#c4-context-diagram)
2. [C4 Container Diagram](#c4-container-diagram)
3. [Component Diagram](#component-diagram)
4. [Data Flow Diagrams](#data-flow-diagrams)
5. [Sequence Diagrams](#sequence-diagrams)
6. [Deployment Diagrams](#deployment-diagrams)

---

## C4 Context Diagram

Shows the OpenClaw system in context of users and external systems.

```mermaid
C4Context
    title OpenClaw System Context Diagram
    
    Person(user, "User", "Person using the AI assistant")
    
    System_Boundary(openclaw, "OpenClaw") {
        System(gateway, "OpenClaw Gateway", "Control plane for AI assistant")
    }
    
    System_Ext(whatsapp, "WhatsApp", "Messaging platform")
    System_Ext(telegram, "Telegram", "Messaging platform")
    System_Ext(slack, "Slack", "Team collaboration")
    System_Ext(discord, "Discord", "Community chat")
    System_Ext(signal, "Signal", "Secure messaging")
    System_Ext(imessage, "iMessage", "Apple messaging")
    
    System_Ext(anthropic, "Anthropic API", "Claude AI models")
    System_Ext(openai, "OpenAI API", "GPT models")
    System_Ext(bedrock, "AWS Bedrock", "Amazon AI models")
    
    System_Boundary(devices, "Device Apps") {
        System(macos, "macOS App", "Menu bar companion")
        System(ios, "iOS App", "Mobile node")
        System(android, "Android App", "Mobile node")
    }
    
    Rel(user, gateway, "Uses via CLI or Web UI")
    Rel(user, macos, "Controls")
    Rel(user, ios, "Uses")
    Rel(user, android, "Uses")
    
    Rel(gateway, whatsapp, "Sends/Receives messages")
    Rel(gateway, telegram, "Sends/Receives messages")
    Rel(gateway, slack, "Sends/Receives messages")
    Rel(gateway, discord, "Sends/Receives messages")
    Rel(gateway, signal, "Sends/Receives messages")
    Rel(gateway, imessage, "Sends/Receives messages")
    
    Rel(gateway, anthropic, "Calls LLM APIs")
    Rel(gateway, openai, "Calls LLM APIs")
    Rel(gateway, bedrock, "Calls LLM APIs")
    
    Rel(macos, gateway, "Connects via WebSocket")
    Rel(ios, gateway, "Connects as node")
    Rel(android, gateway, "Connects as node")
    
    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

---

## C4 Container Diagram

Shows the containers (applications/data stores) that make up OpenClaw.

```mermaid
C4Container
    title OpenClaw Container Diagram
    
    Person(user, "User", "Person using the AI assistant")
    
    Container_Boundary(cli, "CLI/Web Interface") {
        Container(cli_app, "CLI Tool", "Node.js/TypeScript", "Command-line interface")
        Container(web_ui, "Web UI", "Lit/Web Components", "Browser-based control panel")
    }
    
    Container_Boundary(gateway, "OpenClaw Gateway") {
        Container(ws_server, "WebSocket Server", "Node.js/ws", "Handles all client connections")
        Container(session_mgr, "Session Manager", "TypeScript", "Manages conversation sessions")
        Container(agent_runtime, "Agent Runtime", "TypeScript/pi-agent-core", "Executes AI tasks")
        Container(channel_mgr, "Channel Manager", "TypeScript", "Manages messaging channels")
        Container(plugin_mgr, "Plugin Manager", "TypeScript", "Loads extensions")
        Container(cron, "Cron Scheduler", "TypeScript", "Scheduled tasks")
        
        ContainerDb(sessions, "Session Store", "File-based", "Conversation history")
        ContainerDb(config, "Config Store", "JSON", "User configuration")
        ContainerDb(credentials, "Credentials", "Encrypted", "API keys and tokens")
    }
    
    Container_Boundary(channels, "Channel Adapters") {
        Container(whatsapp_adapter, "WhatsApp Adapter", "TypeScript/Baileys", "WhatsApp Web integration")
        Container(telegram_adapter, "Telegram Adapter", "TypeScript/grammY", "Telegram Bot API")
        Container(slack_adapter, "Slack Adapter", "TypeScript/Bolt", "Slack integration")
        Container(discord_adapter, "Discord Adapter", "TypeScript/discord.js", "Discord bot")
    }
    
    Container_Boundary(apps, "Device Apps") {
        Container(macos_app, "macOS App", "SwiftUI", "Menu bar companion app")
        Container(ios_app, "iOS App", "SwiftUI", "Mobile node app")
        Container(android_app, "Android App", "Kotlin/Jetpack", "Mobile node app")
    }
    
    Container_Boundary(external, "External Services") {
        Container_Ext(anthropic, "Anthropic", "REST API", "Claude models")
        Container_Ext(openai, "OpenAI", "REST API", "GPT models")
    }
    
    Rel(user, cli_app, "Runs commands")
    Rel(user, web_ui, "Uses browser")
    Rel(user, macos_app, "Controls")
    
    Rel(cli_app, ws_server, "WebSocket", "ws://localhost:18789")
    Rel(web_ui, ws_server, "WebSocket", "ws://localhost:18789")
    Rel(macos_app, ws_server, "WebSocket", "ws://localhost:18789")
    Rel(ios_app, ws_server, "WebSocket", "ws://localhost:18789")
    
    Rel(ws_server, session_mgr, "Manages")
    Rel(ws_server, agent_runtime, "Invokes")
    Rel(ws_server, channel_mgr, "Routes")
    Rel(ws_server, plugin_mgr, "Uses")
    Rel(ws_server, cron, "Schedules")
    
    Rel(session_mgr, sessions, "Reads/Writes")
    Rel(ws_server, config, "Reads")
    Rel(ws_server, credentials, "Reads")
    
    Rel(channel_mgr, whatsapp_adapter, "Uses")
    Rel(channel_mgr, telegram_adapter, "Uses")
    Rel(channel_mgr, slack_adapter, "Uses")
    Rel(channel_mgr, discord_adapter, "Uses")
    
    Rel(agent_runtime, anthropic, "HTTPS/JSON")
    Rel(agent_runtime, openai, "HTTPS/JSON")
    
    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

---

## Component Diagram

Detailed view of the Gateway internal components.

```mermaid
graph TB
    subgraph "Gateway Core"
        WS[WebSocket Server<br/>Port 18789]
        HTTP[HTTP Server<br/>Control UI]
        CM[Connection Manager]
        DM[Device Manager]
    end
    
    subgraph "Session Management"
        SM[Session Manager]
        SS[Session Store<br/>~/.openclaw/sessions/]
        SP[Session Pruner]
        SC[Session Compactor]
    end
    
    subgraph "Agent Runtime"
        AR[Agent Runtime]
        PC[Pi Agent Core]
        TE[Tool Executor]
        TR[Tool Registry]
    end
    
    subgraph "Channel Management"
        ChM[Channel Manager]
        CR[Channel Router]
        WA[WhatsApp Adapter]
        TG[Telegram Adapter]
        SL[Slack Adapter]
        DS[Discord Adapter]
    end
    
    subgraph "Plugin System"
        PM[Plugin Manager]
        PR[Plugin Registry]
        PH[Hook System]
        SK[Skills Manager]
    end
    
    subgraph "Security"
        AUTH[Auth Manager]
        PAIR[Pairing System]
        SB[Sandbox Manager]
        AL[Allowlist Manager]
    end
    
    subgraph "External APIs"
        ANTH[Anthropic API]
        OAI[OpenAI API]
        BED[AWS Bedrock]
    end
    
    WS --> CM
    CM --> DM
    DM --> AUTH
    AUTH --> PAIR
    
    WS --> SM
    SM --> SS
    SM --> SP
    SM --> SC
    
    SM --> AR
    AR --> PC
    AR --> TE
    TE --> TR
    
    WS --> ChM
    ChM --> CR
    CR --> WA
    CR --> TG
    CR --> SL
    CR --> DS
    
    AR --> PM
    PM --> PR
    PM --> PH
    PM --> SK
    
    TE --> SB
    
    PC --> ANTH
    PC --> OAI
    PC --> BED
```

---

## Data Flow Diagrams

### Message Routing Flow

```mermaid
sequenceDiagram
    actor User
    participant Channel as Messaging Channel
    participant Gateway as OpenClaw Gateway
    participant Router as Channel Router
    participant Session as Session Manager
    participant Agent as Agent Runtime
    participant LLM as LLM Provider
    participant Tool as Tool Executor

    User->>Channel: Send message
    Channel->>Gateway: Webhook/WebSocket
    Gateway->>Router: Route message
    Router->>Router: Check allowlist
    Router->>Router: Resolve session key
    Router->>Session: Get/Create session
    Session->>Agent: Start agent run
    
    Agent->>Agent: Build context
    Agent->>Agent: Load tools/skills
    Agent->>LLM: Send prompt
    LLM-->>Agent: Stream response
    
    alt Tool Call Required
        Agent->>Tool: Execute tool
        Tool-->>Agent: Tool result
        Agent->>LLM: Send result
        LLM-->>Agent: Continue response
    end
    
    Agent-->>Session: Final response
    Session-->>Router: Route reply
    Router-->>Gateway: Send to channel
    Gateway-->>Channel: Deliver message
    Channel-->>User: Display response
```

### Agent Execution Flow

```mermaid
flowchart TD
    Start([Message Received]) --> Validate{Validate Input}
    Validate -->|Invalid| Error1[Return Error]
    Validate -->|Valid| Resolve[Resolve Session]
    
    Resolve --> LoadConfig[Load Agent Config]
    LoadConfig --> LoadSkills[Load Skills Snapshot]
    LoadSkills --> BuildContext[Build System Prompt]
    
    BuildContext --> CheckContext{Context Size OK?}
    CheckContext -->|Too Large| Compact[Compact Session]
    CheckContext -->|OK| CallModel
    Compact --> CallModel[Call LLM API]
    
    CallModel --> Stream{Stream Response}
    Stream -->|Assistant Token| Buffer[Buffer Tokens]
    Stream -->|Tool Call| Execute[Execute Tool]
    
    Execute --> ToolResult{Tool Result}
    ToolResult -->|Success| SendResult[Send to LLM]
    ToolResult -->|Error| HandleError[Handle Error]
    
    SendResult --> CallModel
    HandleError --> CallModel
    
    Stream -->|End| Assemble[Assemble Final Response]
    Buffer --> Assemble
    
    Assemble --> Persist[Persist to Session]
    Persist --> Deliver[Deliver to User]
    Deliver --> End([End])
    Error1 --> End
```

### WebSocket Connection Lifecycle

```mermaid
sequenceDiagram
    participant Client as Client (macOS/CLI/Web)
    participant GW as Gateway WebSocket
    participant Auth as Auth Manager
    participant Session as Session Manager

    Client->>GW: Connect (ws://host:18789)
    GW-->>Client: Connection Established
    
    Client->>GW: connect {deviceId, role, auth}
    GW->>Auth: Validate device/auth
    
    alt New Device
        Auth-->>GW: Device not paired
        GW-->>Client: Pairing required
        Client->>GW: Pairing request
        GW->>Auth: Generate pairing code
        Auth-->>GW: Pairing code
        GW-->>Client: Pairing code
        Note over Client: User approves pairing
        Client->>GW: Connect with token
    end
    
    Auth-->>GW: Auth OK
    GW->>Session: Load session state
    Session-->>GW: Session snapshot
    GW-->>Client: hello-ok + snapshot
    
    loop Heartbeat
        GW-->>Client: event:heartbeat
    end
    
    Client->>GW: req:agent {message}
    GW-->>Client: res:accepted {runId}
    
    loop Agent Streaming
        GW-->>Client: event:assistant {delta}
        GW-->>Client: event:tool {status}
    end
    
    GW-->>Client: event:lifecycle {end}
    
    Client->>GW: Disconnect
    GW->>Session: Save state
    GW-->>Client: Connection Closed
```

---

## Sequence Diagrams

### Tool Execution Sequence

```mermaid
sequenceDiagram
    participant User
    participant Gateway
    participant Agent as Agent Runtime
    participant Pi as Pi Agent Core
    participant Tool as Tool Handler
    
    User->>Gateway: Send message with tool request
    Gateway->>Agent: Start agent run
    Agent->>Pi: Initialize session
    
    loop Until completion
        Pi->>Agent: Request tool execution
        Agent->>Tool: Validate & execute
        
        alt Browser Tool
            Tool->>Tool: Launch Playwright
            Tool->>Tool: Navigate/interact
            Tool->>Tool: Capture screenshot
        else Exec Tool
            Tool->>Tool: Spawn process
            Tool->>Tool: Capture output
        else Read/Write Tool
            Tool->>Tool: File operations
        end
        
        Tool-->>Agent: Tool result
        Agent-->>Pi: Return result
    end
    
    Pi-->>Agent: Final response
    Agent-->>Gateway: Stream response
    Gateway-->>User: Deliver message
```

### Channel Message Handling

```mermaid
sequenceDiagram
    participant User
    participant WA as WhatsApp
    participant Gateway
    participant Router as Channel Router
    participant AL as Allowlist Manager
    participant Session
    
    User->>WA: Send message
    WA->>Gateway: Baileys message event
    
    Gateway->>Router: Route inbound message
    Router->>AL: Check allowlist
    
    alt Not Allowed
        AL-->>Router: Deny
        Router-->>Gateway: Drop message
    else Pairing Required
        AL-->>Router: Pairing needed
        Router->>Router: Generate pairing code
        Router-->>WA: Send pairing code
        WA-->>User: "Reply with: ABC123"
    else Allowed
        AL-->>Router: Allow
        Router->>Session: Get session
        Session-->>Router: Session context
        Router->>Gateway: Process message
        Gateway->>Gateway: Agent processing...
        Gateway-->>Router: Response
        Router-->>WA: Send reply
        WA-->>User: Deliver response
    end
```

### Multi-Agent Routing

```mermaid
sequenceDiagram
    participant User1 as User (Work)
    participant User2 as User (Personal)
    participant Slack
    participant Telegram
    participant Gateway
    participant Router
    participant Session1 as Work Session
    participant Session2 as Personal Session
    participant Agent
    
    User1->>Slack: Message to #work-channel
    Slack->>Gateway: Slack event
    Gateway->>Router: Route message
    Router->>Router: Check routing rules
    Router->>Session1: Work agent session
    Session1->>Agent: Process
    Agent-->>Session1: Response
    Session1-->>Router: Reply
    Router-->>Slack: Send to #work-channel
    Slack-->>User1: Display response
    
    User2->>Telegram: Direct message
    Telegram->>Gateway: Telegram update
    Gateway->>Router: Route message
    Router->>Router: Check routing rules
    Router->>Session2: Personal agent session
    Session2->>Agent: Process
    Agent-->>Session2: Response
    Session2-->>Router: Reply
    Router-->>Telegram: Send DM
    Telegram-->>User2: Display response
```

---

## Deployment Diagrams

### Local Development Deployment

```mermaid
graph TB
    subgraph "Development Machine"
        subgraph "OpenClaw Project"
            SRC[Source Code]
            DIST[Dist/Build]
            NODE[Node.js 22+]
        end
        
        subgraph "User Home"
            CONFIG[~/.openclaw/openclaw.json]
            SESSIONS[~/.openclaw/sessions/]
            WORKSPACE[~/.openclaw/workspace/]
            CREDS[~/.openclaw/credentials/]
        end
        
        subgraph "Running Services"
            GW[Gateway<br/>ws://127.0.0.1:18789]
            CANVAS[Canvas Host<br/>http://127.0.0.1:18793]
        end
    end
    
    subgraph "External"
        WA[WhatsApp Web]
        TG[Telegram API]
        CLAUDE[Anthropic API]
    end
    
    SRC -->|Build| DIST
    DIST --> NODE
    NODE --> GW
    NODE --> CANVAS
    
    CONFIG --> GW
    SESSIONS --> GW
    WORKSPACE --> GW
    CREDS --> GW
    
    GW <-->|WebSocket| WA
    GW <-->|HTTPS| TG
    GW <-->|HTTPS| CLAUDE
```

### Docker Deployment

```mermaid
graph TB
    subgraph "Docker Host"
        subgraph "OpenClaw Network"
            subgraph "openclaw-gateway Container"
                GW_APP[Gateway App]
                GW_DATA[/home/node/.openclaw]
            end
            
            subgraph "openclaw-cli Container"
                CLI[CLI Tool]
            end
        end
    end
    
    subgraph "Host Volumes"
        HOST_CONFIG[Config Dir]
        HOST_WORKSPACE[Workspace Dir]
    end
    
    subgraph "External Services"
        CHANNELS[Messaging Channels]
        LLMS[LLM Providers]
    end
    
    HOST_CONFIG -.->|Mount| GW_DATA
    HOST_WORKSPACE -.->|Mount| GW_DATA
    
    GW_APP -->|Port 18789| CHANNELS
    GW_APP -->|HTTPS| LLMS
    CLI -->|Internal| GW_APP
```

### Remote Gateway with Device Nodes

```mermaid
graph TB
    subgraph "Remote Server (VPS/Cloud)"
        subgraph "OpenClaw Gateway"
            GW[Gateway Daemon]
            SESSIONS[Session Store]
            TOOLS[Tool Executor]
        end
        
        subgraph "Network Exposure"
            TS[Tailscale]
            SSH[SSH Tunnel]
        end
    end
    
    subgraph "User's Home Network"
        subgraph "Local Devices"
            MAC[macOS Node]
            IOS[iOS Node]
            ANDROID[Android Node]
        end
        
        subgraph "Clients"
            CLI[CLI Tool]
            WEB[Web UI]
        end
    end
    
    subgraph "Channels"
        WA[WhatsApp]
        TG[Telegram]
        SL[Slack]
    end
    
    GW <-->|WebSocket| TS
    GW <-->|WebSocket| SSH
    TS <-->|Tailnet| MAC
    SSH <-->|Tunnel| CLI
    
    MAC -->|Device Actions| MAC_LOCAL[System/Camera/Screen]
    IOS -->|Device Actions| IOS_LOCAL[Camera/Screen]
    
    GW <-->|WebSocket| WA
    GW <-->|HTTPS| TG
    GW <-->|WebSocket| SL
```

---

## State Machine Diagrams

### Session State Machine

```mermaid
stateDiagram-v2
    [*] --> Idle: Create Session
    
    Idle --> Active: Receive Message
    
    Active --> Processing: Start Agent
    
    Processing --> Streaming: Model Response
    Processing --> ToolExecution: Tool Call
    
    ToolExecution --> Processing: Tool Result
    
    Streaming --> Completing: Response Complete
    
    Completing --> Idle: Save Transcript
    Completing --> Compacting: Context Full
    
    Compacting --> Idle: Compact Complete
    
    Idle --> Archived: Timeout
    Archived --> [*]
    
    Processing --> Error: Exception
    Error --> Idle: Error Handled
```

### Device Pairing State Machine

```mermaid
stateDiagram-v2
    [*] --> NewDevice: First Connect
    
    NewDevice --> PairingRequired: Not Paired
    
    PairingRequired --> PendingApproval: Send Pairing Code
    
    PendingApproval --> Approved: User Approves
    PendingApproval --> Rejected: Timeout/Deny
    
    Approved --> Connected: Issue Token
    
    Connected --> Authenticated: Valid Token
    
    Authenticated --> Connected: Session Active
    
    Connected --> Disconnected: Disconnect
    Rejected --> Disconnected
    
    Disconnected --> [*]
    
    Authenticated --> Revoked: Admin Action
    Revoked --> [*]
```

---

## Entity Relationship Diagram

### Core Data Model

```mermaid
erDiagram
    SESSION ||--o{ MESSAGE : contains
    SESSION ||--o{ TOOL_CALL : has
    SESSION }o--|| AGENT : uses
    SESSION }o--|| WORKSPACE : has
    
    SESSION {
        string sessionId PK
        string sessionKey
        string agentId
        string workspacePath
        datetime createdAt
        datetime updatedAt
        json context
        json metadata
    }
    
    MESSAGE {
        string messageId PK
        string sessionId FK
        string role
        string content
        json attachments
        datetime timestamp
        json metadata
    }
    
    TOOL_CALL {
        string callId PK
        string sessionId FK
        string toolName
        json parameters
        json result
        string status
        datetime startedAt
        datetime completedAt
    }
    
    AGENT {
        string agentId PK
        string model
        string thinkingLevel
        json tools
        json skills
        json config
    }
    
    WORKSPACE {
        string path PK
        string agentId
        json files
        datetime lastModified
    }
    
    CHANNEL ||--o{ SESSION : routes_to
    CHANNEL {
        string channelId PK
        string type
        string config
        json allowlist
        string dmPolicy
    }
    
    DEVICE ||--o{ SESSION : connects_to
    DEVICE {
        string deviceId PK
        string role
        string platform
        json capabilities
        boolean paired
        datetime pairedAt
    }
```

---

*Generated from source code analysis on 2026-01-31*
