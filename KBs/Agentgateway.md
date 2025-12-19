What is Agentgateway?


https://github.com/agentgateway/agentgateway#


  An open-source data plane for Agentic AI connectivity - written in Rust. It provides:

- Agent-to-Agent (A2A) communication (Google's protocol)
- Model Context Protocol (MCP) support
- Security, observability, and governance for AI agent communication
- Legacy API transformation (OpenAPI → MCP)

  Prerequisites

| Requirement | Version        |
| ----------- | -------------- |
| Rust        | 1.86+          |
| npm         | 10+            |
| Node.js     | (for UI build) |

  Setup Options

  Option 1: Build from Source (Local)

  cd /Users/rupeshpanwar/Documents/AI-Projects/AI-pack/agentgateway

# 1. Build the UI

  cd ui
  npm install
  npm run build

# 2. Build the binary

  cd ..
  export CARGO_NET_GIT_FETCH_WITH_CLI=true
  make build

# 3. Run

  ./target/release/agentgateway

  UI available at: http://localhost:15000/ui

  Option 2: Docker

  make docker

  Option 3: GitHub Codespaces

  Click "Code → Create codespace on main" on GitHub.

  Running with a Config

# Basic example - proxies to MCP "everything" tool

  cargo run -- -f examples/basic/config.yaml

  Configuration Structure

  Example config.yaml:  binds:

- port: 3000              # Port to listen onlisteners:
  - routes:
    - policies:           # Traffic policies (CORS, auth, etc.)cors:allowOrigins: ["*"]backends:
      - mcp:              # MCP backend typetargets:
        - name: everything
          stdio:        # Connect via stdio
          cmd: npx
          args: ["@modelcontextprotocol/server-everything"]

  Key Concepts

| Concept   | Description                            |
| --------- | -------------------------------------- |
| binds     | Ports the server listens on            |
| listeners | Groups of resources                    |
| routes    | Routing rules + traffic policies       |
| backends  | Where traffic goes (MCP, A2A, HTTP)    |
| targets   | Individual MCP servers (stdio or HTTP) |

  Available Examples

| Example             | Purpose                                 |
| ------------------- | --------------------------------------- |
| basic/              | Simple MCP proxy                        |
| a2a/                | Agent-to-Agent protocol                 |
| authorization/      | RBAC setup                              |
| mcp-authentication/ | OAuth/Keycloak auth                     |
| ratelimiting/       | Rate limiting                           |
| telemetry/          | Observability (Langfuse, OpenTelemetry) |
| tls/                | TLS configuration                       |
| openapi/            | Legacy API → MCP transformation        |

  Quick Test

  After running, use MCP Inspector to test:
  npx @modelcontextprotocol/inspector

  Connect to http://localhost:3000/mcp or /sse.
