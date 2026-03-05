# [Feature] Agent-native workflow authoring — let external AI agents create and run Gumloop workflows

**ID:** `FEAT-004`
**Type:** `Feature Request / Strategic`
**Priority:** `Medium`
**Area:** API / Developer Experience / Agent Integration
**Reported by:** Rafael Cabrera (power user — Funding Hub automation)
**Authored with:** Claude Code (AI-assisted writeup, verified by Rafael Cabrera)
**Date:** 2026-03-05

---

## Description

Gumloop's existing API and Python SDK are powerful for **running** pre-built workflows — but there is currently no programmatic interface for **creating or editing** them. The only way to build a workflow is through the web UI.

At the same time, external AI coding agents — Claude Code, Codex, Cursor, ChatGPT — are becoming the primary way developers interact with their tools. These agents can already call APIs, write code, and orchestrate complex tasks. What they can't do is reach into a user's Gumloop account and build a workflow on their behalf.

This proposal is to make Gumloop **agent-native**: expose a first-class interface (MCP server, CLI, or extended SDK) that lets external agents create, edit, and run workflows — while the Gumloop web UI remains the canonical view where humans can inspect, refine, and approve what the agent built.

This is not a niche power-user request. As AI coding agents become mainstream, the tools that integrate with them natively will win. N8N has tried this but the experience is still low-level and not widely adopted. Gumloop is better positioned to do it right.

---

## What Already Exists (Foundation)

The current API and SDK cover workflow *execution*:

| Capability | Available Today |
|---|---|
| Start a flow run | ✅ REST API + Python SDK |
| Kill a flow run | ✅ REST API |
| Retrieve run details / history | ✅ REST API |
| List saved flows | ✅ REST API |
| Upload / download files | ✅ REST API |
| **Create a new workflow** | ❌ Not available |
| **Add / edit / delete nodes** | ❌ Not available |
| **Connect nodes programmatically** | ❌ Not available |
| **Configure node parameters via API** | ❌ Not available |

The Gummie assistant already exists as internal proof-of-concept that Gumloop can generate workflows from natural language. This feature is about opening that capability to external agents.

---

## Proposed: The Gumloop Agent Interface

### Layer 1 — Workflow Authoring API

Extend the REST API with workflow CRUD operations:

```
POST   /api/v1/workflows                    # Create a new workflow
GET    /api/v1/workflows/{id}               # Get workflow definition (nodes + edges)
PATCH  /api/v1/workflows/{id}               # Update workflow (add/remove/edit nodes)
DELETE /api/v1/workflows/{id}               # Delete workflow

POST   /api/v1/workflows/{id}/nodes         # Add a node
PATCH  /api/v1/workflows/{id}/nodes/{node}  # Configure a node's parameters
DELETE /api/v1/workflows/{id}/nodes/{node}  # Remove a node
POST   /api/v1/workflows/{id}/edges         # Connect two nodes
```

The workflow definition should be serializable as JSON — the same format used internally to save/load flows — making it the lingua franca between the API and the UI.

### Layer 2 — MCP Server for Gumloop

Publish an official **Gumloop MCP server** so that any MCP-compatible agent (Claude Code, Cursor, etc.) can add Gumloop as a tool with a single config line:

```json
{
  "mcpServers": {
    "gumloop": {
      "command": "npx",
      "args": ["-y", "@gumloop/mcp"],
      "env": { "GUMLOOP_API_KEY": "..." }
    }
  }
}
```

This exposes tools like:
- `gumloop_create_workflow(name, description)` — scaffold a new workflow
- `gumloop_add_node(workflow_id, node_type, config)` — add a configured node
- `gumloop_connect_nodes(workflow_id, source, target)` — wire nodes together
- `gumloop_run_workflow(workflow_id, inputs)` — execute and return results
- `gumloop_list_workflows()` — browse existing workflows
- `gumloop_get_node_schema(node_type)` — get the input/output schema for any node type (critical for agents to know what's available)

### Layer 3 — CLI

A `gumloop` CLI for terminal-native agents (like Claude Code):

```bash
gumloop workflows list
gumloop workflows create --name "My Flow" --description "..."
gumloop node add <workflow_id> ask_ai --prompt "Summarize: {text}"
gumloop node connect <workflow_id> <source_node> <target_node>
gumloop run <workflow_id> --input text="Hello world"
```

---

## The Key UX Principle: Agents Create, Humans Refine

The web UI stays as the canonical surface. When an agent creates or modifies a workflow via the API/MCP/CLI, the result immediately appears in the user's Gumloop account — visible, editable, and runnable from the web UI exactly like a manually-built flow. The agent's output is a starting point, not a black box.

This mirrors how Claude Code generates code that you then review in your editor — not a replacement for the human, but a massive acceleration.

---

## Why This Matters for Gumloop

1. **Distribution:** Every Claude Code user who can say `"build me a Gumloop workflow that..."` is a potential Gumloop user. Agent adoption drives human adoption.
2. **Differentiation:** N8N has an API but it's low-level, undiscoverable, and not designed for agent consumption. Being the first to do this well is a real competitive moat.
3. **Stickiness:** Workflows created by agents in a user's account are workflows the user now owns, edits, and runs — deepening platform engagement.
4. **The Gummie precedent:** Gumloop already built an internal agent that generates workflows from natural language. The hard part (workflow IR, generation logic) is done. This is about opening the interface.

---

## Competitive Reference

N8N offers a [REST API](https://docs.n8n.io/api/) that includes workflow creation endpoints. However, it is verbose, low-level (raw JSON graph), and has no MCP server or agent-friendly abstractions. Usage is limited to developers writing integration code — not to AI agents operating autonomously. Gumloop has the opportunity to leapfrog by designing for agents from the start, with rich tool descriptions, node schemas as discoverable capabilities, and a CLI optimized for agent invocation patterns.

---

*Proposed after extensive use of both Gumloop (workflow automation) and Claude Code (AI agent) for production data pipelines — the absence of a bridge between the two is the single most limiting factor in building truly autonomous automation systems.*
