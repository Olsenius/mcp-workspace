# Architecture

## Runtime

Clients reach every MCP server through mcp-gateway. The servers sit on an
internal Docker network with no public ports, logins or OAuth of their own.

```mermaid
flowchart LR
    subgraph clients [MCP clients]
        claude[claude.ai]
        chatgpt[ChatGPT]
        code[Claude Code<br/>personal token]
    end

    tunnel[Tailscale Funnel<br/>or Cloudflare Tunnel]

    subgraph internal [Internal Docker network]
        gateway[mcp-gateway<br/>OAuth 2.1 · admin page]
        bibliofil[bibliofil-mcp]
        vigilo[vigilo-connector]
        fribo[fribo-mcp]
        tibi[tibi-mcp]
    end

    subgraph services [External services]
        bf[(Bibliofil and<br/>Biblioteksøk)]
        vg[(Vigilo)]
        fr[(Fribo portal)]
        tb[(tibi.no)]
    end

    otel[(OTLP collector<br/>optional)]

    claude & chatgpt & code -->|HTTPS| tunnel --> gateway
    gateway -->|/bibliofil/mcp| bibliofil --> bf
    gateway -->|/vigilo/mcp| vigilo --> vg
    gateway -->|/fribo/mcp| fribo --> fr
    gateway -->|/tibi/mcp| tibi --> tb
    gateway -.->|traces| otel
```

## A tool call

The gateway swaps the client's OAuth token for the server's own token, so
servers never see client credentials.

```mermaid
sequenceDiagram
    participant C as MCP client
    participant G as mcp-gateway
    participant S as MCP server
    participant X as External service

    C->>G: POST /tibi/mcp<br/>Bearer (OAuth access token)
    G->>G: Validate token for this server
    G->>S: POST /mcp<br/>Bearer MCP_AUTH_TOKEN
    S->>X: API request with stored account settings
    X-->>S: Response
    S-->>G: MCP result
    G-->>C: MCP result
```

## The upstream contract

Each server exposes the same endpoints, so the gateway can show a settings
form and a **Test** button for it.

| Endpoint | Auth | Purpose |
|---|---|---|
| `POST /mcp` | Bearer | Streamable HTTP MCP |
| `GET /healthz` | none | Liveness |
| `GET /healthz?check=1` | Bearer | Real checks with current settings |
| `GET /config`, `POST /config` | Bearer | Settings schema and saved values |

Settings are stored in `DATA_DIR` (default `/data`). Environment variables
always win and show as locked in the form.

## Workspace

This repository ties the separate repositories together for development.

```mermaid
flowchart TB
    mise[mise.toml<br/>pins Go, Python, Task, mani]
    task[Taskfile.yml<br/>all commands]
    mani[mani.yaml<br/>repositories and tags]
    compose[compose.yaml<br/>local stack from source]
    repos[mcp-gateway · bibliofil-mcp · vigilo-connector<br/>fribo-mcp · tibi-mcp]
    ci[GitHub Actions<br/>task check, docs build]

    mise --> task
    task -->|clone, sync, test| mani --> repos
    task -->|dev:up| compose -->|builds| repos
    ci --> task
```

The local stack (`task dev:up`) runs as the Compose project
`mcp-workspace-dev`. Only the gateway is published, on
`127.0.0.1:18080`; servers start with `MCP_READ_ONLY=true`.
