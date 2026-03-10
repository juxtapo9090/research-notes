# Julia MCP Server - FAFO Discovery

**Date:** 2026-01-28
**Status:** ✅ Working

## What Is It

A Julia MCP HTTP server that exposes Julia tools over HTTP. Allows VPS Cursor to call Julia functions on PC over WireGuard.

```
VPS (Brain/Cursor)  ──MCP/HTTP──►  PC (Muscle/Julia)
                                         │
                                    tools/
                                    ├── echo
                                    ├── system_info
                                    └── julia_eval
```

## Project Location

```
C:\cee_v25_Pre_Final\brain\mcp_julia_server\
├── server.jl           # Main HTTP server (port 3000)
├── Project.toml        # Dependencies
└── tools/              # Drop .jl files here, auto-registered
```

## How To Run

```bash
cd C:\cee_v25_Pre_Final\brain\mcp_julia_server
julia server.jl
```

Server features:
- Auto-kills existing instance on port 3000 (no conflicts)
- Listens on `0.0.0.0:3000` (WireGuard ready)
- Auto-discovers tools from `tools/` directory

## VPS Config (Future)

```json
{
  "mcpServers": {
    "julia-pc": {
      "transport": "http",
      "url": "http://PC_WIREGUARD_IP:3000"
    }
  }
}
```

## Key Package

`ModelContextProtocol.jl` v0.4.0 at:
```
C:\cee_v25_Pre_Final\brain\ModelContextProtocol.jl-0.4.0\
```

Installed via: `Pkg.develop(path="...")`

## Test Command

```bash
curl -X POST http://localhost:3000/ \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "MCP-Protocol-Version: 2025-06-18" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":1}'
```

## Adding New Tools

Just drop a `.jl` file in `tools/` with this pattern:

```julia
tool_name = MCPTool(
    name = "tool_name",
    description = "What it does",
    parameters = [...],
    handler = function(params)
        return TextContent(text = "result")
    end
)
```

Restart server — tool appears automatically.
