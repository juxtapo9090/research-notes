# 🔬 FAFO: LangChain MCP Adapters Integration

**Date:** 2026-01-18 03:57  
**Session:** Dynamic MCP tool discovery for cursor_bridge

---

## 🎯 Goal

Enable Cursor workers (OpenAI format) to properly access MCP tools through `cursor_bridge` using `langchain-mcp-adapters` for translation.

---

## ✅ Discoveries

### 1. langchain-mcp-adapters API

The `langchain-mcp-adapters` library (v0.2.1) provides:

```python
from langchain_mcp_adapters.client import MultiServerMCPClient

client = MultiServerMCPClient({
    "task": {
        "url": "http://localhost:3002/sse",
        "transport": "sse"
    }
})
tools = await client.get_tools()  # Returns LangChain tools
```

**Key insight:** `args_schema` is already a dict (not a Pydantic model), which simplifies conversion to OpenAI format.

### 2. Tool Schema Format

MCP tools come with full JSON Schema definitions:

```json
{
  "type": "function",
  "function": {
    "name": "submit_task",
    "description": "Submit a task for Cursor workers...",
    "parameters": {
      "type": "object",
      "properties": {
        "task_description": {"type": "string"},
        "priority": {"default": "normal", "type": "string"}
      },
      "required": ["task_description"]
    }
  }
}
```

### 3. CRITICAL FIX: Always Use Native Mode

**Problem**: Original auto-routing sent tool requests through CONVERT mode → OpenAI format → Antigravity routes to Gemini instead of Claude!

**Solution**: AUTO mode now ALWAYS uses NATIVE mode:
```python
# AUTO MODE: Always use NATIVE to get real Claude
if routing_mode == "auto":
    routing_mode = "native"  # Never use convert!
```

NATIVE mode sends to `/v1/messages` (Anthropic format) which Antigravity routes to **real Claude/Opus**.

### 4. Tool Response Extraction

Had to fix the non-streaming response handler to properly extract `tool_use` blocks:
```python
elif block.get("type") == "tool_use":
    tool_calls.append({
        "id": block.get("id"),
        "type": "function",
        "function": {
            "name": block.get("name"),
            "arguments": json.dumps(block.get("input", {}))
        }
    })
```

---

## 📁 Files Created/Modified

| File | Change |
|------|--------|
| `cursor_bridge/core/mcp_client.py` | **NEW** - MCPToolLoader class with dynamic discovery |
| `cursor_bridge/config.json` | Added `servers` and `dynamic_discovery` config |
| `cursor_bridge/handlers/chat.py` | Updated to use dynamic tools |
| `cursor_bridge/handlers/mcp.py` | Updated `is_mcp_tool()` function |

---

## 🔧 Configuration

```json
{
  "mcp": {
    "enabled": true,
    "inject_tools": true,
    "dynamic_discovery": true,
    "cache_ttl": 60,
    "servers": {
      "task": {
        "url": "http://localhost:3002/sse",
        "transport": "sse"
      }
    }
  }
}
```

---

## 🧪 Test Results

```
[MCP_CLIENT] Initialized with 1 servers
[MCP_CLIENT]   - task: {'url': 'http://localhost:3002/sse', 'transport': 'sse'}
[MCP_CLIENT] Discovered 6 tools from MCP servers
[MCP_CLIENT]   - submit_task
[MCP_CLIENT]   - get_pending_task
[MCP_CLIENT]   - complete_task
[MCP_CLIENT]   - list_tasks
[MCP_CLIENT]   - get_task_status
[MCP_CLIENT]   - cancel_task
```

---

## 🔮 Next Steps

1. **Tool Execution** - Execute MCP tools when Cursor invokes them
2. **Multiple Servers** - Add more MCP servers (mt5, etc.)
3. **Error Handling** - Graceful fallback when servers unavailable
4. **Cursor Testing** - Real end-to-end test with Cursor worker

---

## 💡 Key Takeaway

> "langchain-mcp-adapters bridges MCP ↔ OpenAI seamlessly. Dynamic tool discovery means cursor_bridge auto-syncs with any MCP server - no more static tool definitions!"

**Tags:** #FAFO #cursor_bridge #langchain #mcp #tool-discovery #openai-format
