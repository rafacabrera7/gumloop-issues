# Gumloop — Issues & Improvement Proposals

**Author:** Rafael Cabrera Jiménez
**Contact:** rafaelcabrerajimenez7@gmail.com | +1 (412) 844-0926
**Context:** Compiled while building a production-grade grant discovery automation pipeline for a nonprofit (Propel) using Gumloop as the core orchestration layer.

---

## Why This Document Exists

I've been using Gumloop extensively to automate workflows for Propel, a nonprofit supporting Latin American organizations. Over weeks of real-world usage, I've logged the bugs I hit and the features I wished existed — not as complaints, but as someone who genuinely wants this product to be better.

I'm applying to join the Gumloop team as a Full Stack Software Engineer. This document is my way of showing I'm serious: I know the product, I think in systems, and I can ship fixes — not just file tickets.

---

## Summary

| ID | Type | Title | Priority | Area |
|----|------|-------|----------|------|
| [BUG-001](./bugs/001-if-else-edges-intertwine-after-save/ISSUE.md) | Bug | If-Else branch edges intertwine after saving a flow | Medium | Canvas / Edge Routing |
| [BUG-002](./bugs/002-flows-page-slow-initial-load/ISSUE.md) | Bug / Performance | Workflows page takes 10+ seconds to load | Low | Frontend / `/flows` |
| [FEAT-001](./features/001-if-else-pass-through-expose-raw-variables/ISSUE.md) | Feature | If-Else "Pass Inputs Through" should expose raw variable values | Medium | If-Else Node |
| [FEAT-002](./features/002-extract-data-mandatory-fields/ISSUE.md) | Feature | Extract Data node should support mandatory fields with LLM retry | Medium | Extract Data Node |
| [FEAT-003](./features/003-ai-nodes-per-node-credential-selector/ISSUE.md) | Feature | AI nodes should support per-node credential selection (own key vs. credits) | Medium | Ask AI / Extract Data / All AI Nodes |
| [FEAT-004](./features/004-agent-native-workflow-authoring/ISSUE.md) | Feature / Strategic | Agent-native workflow authoring — let external AI agents create and run workflows | Medium | API / Developer Experience |

---

## Bugs

### BUG-001 — If-Else branch edges intertwine after saving a flow

**Priority:** Medium | **Area:** Canvas / Edge Routing

After saving a flow with an If-Else node, the outgoing edges for the `true` and `false` branches cross and intertwine instead of routing cleanly to their targets. Purely visual — flow execution is unaffected — but makes branch logic hard to follow in complex flows.

**Proposed fix:** Persist computed edge routes in the saved flow state, or add a crossing-avoidance pass to the edge routing logic on re-render.

→ [Full issue with screenshots](./bugs/001-if-else-edges-intertwine-after-save/ISSUE.md)

### BUG-002 — Workflows page takes 10+ seconds to load

**Priority:** Low *(client-side not ruled out)* | **Area:** Frontend / `/flows`

The `/flows` page shows a full skeleton loader for 10+ seconds on initial load, tested consistently across Comet and Edge browsers on fast hardware. Suggests the frontend is blocked on a single API call fetching all workflows before rendering anything.

**Proposed investigation:** Profile the `/flows` API endpoint; add pagination or cursor-based loading; implement stale-while-revalidate to render cached data immediately while refreshing in the background.

→ [Full issue with screenshot](./bugs/002-flows-page-slow-initial-load/ISSUE.md)

---

## Features

### FEAT-001 — If-Else "Pass Inputs Through" should expose raw variable values

**Priority:** Medium | **Area:** If-Else Node / Pass Inputs Through

When wrapping an Ask AI node, **Pass Inputs Through** outputs the fully-rendered prompt (template + all substituted variables) instead of each input variable individually. Users are forced to add an extra extraction node downstream just to recover a value that was already in scope — adds unnecessary friction and node bloat to a very common pattern.

**Proposed improvement:** Expose each input variable as a separate named output. Alternatively, expose both the rendered prompt and the individual raw variable values.

→ [Full issue with screenshot](./features/001-if-else-pass-through-expose-raw-variables/ISSUE.md)

### FEAT-002 — Extract Data node should support mandatory fields with LLM retry

**Priority:** Medium | **Area:** Extract Data Node / Field Schema

All extraction fields are implicitly optional — if the LLM omits a field, the node silently passes an empty/null value downstream with no warning or retry. For production pipelines depending on specific fields, this silently corrupts downstream nodes (Sheets writers, Salesforce updaters, etc.).

**Proposed improvement:** Add a "Required" checkbox per field. If a required field comes back empty, trigger a targeted LLM correction prompt and retry (up to N times) before raising a structured error.

→ [Full issue with screenshot](./features/002-extract-data-mandatory-fields/ISSUE.md)

### FEAT-003 — AI nodes should support per-node credential selection

**Priority:** Medium | **Area:** Ask AI / Extract Data / All AI Nodes

AI nodes have no per-node credential selector — the only way to use a personal API key is an account-wide setting. The Firecrawl node already implements exactly this: a "Credentials to use" dropdown in the node panel with personal default, saved credentials, and inline "Add New Credential". AI nodes need the same feature parity.

**Proposed improvement:** Add a "Credentials to use" selector to the More Options section of all AI nodes, matching the existing Firecrawl node UX. At runtime, use the selected credential's API key (billed at 1 credit) or fall back to Gumloop managed credits.

→ [Full issue with screenshot](./features/003-ai-nodes-per-node-credential-selector/ISSUE.md)

### FEAT-004 — Agent-native workflow authoring

**Priority:** Medium | **Area:** API / Developer Experience / Agent Integration

The current API only supports running pre-built flows — there is no way to create or edit workflows programmatically. External AI agents (Claude Code, Codex, ChatGPT) cannot build Gumloop workflows on a user's behalf. This proposal adds a workflow authoring API, an official MCP server, and a CLI so that any agent can create, configure, and run workflows — with results immediately visible and editable in the web UI. The built-in Gummie assistant is already proof the internal primitives exist; this is about opening them to the outside.

**Proposed implementation:** Workflow CRUD REST API + Gumloop MCP server (`@gumloop/mcp`) + CLI. Agents create, humans refine — web UI stays the canonical surface.

→ [Full proposal](./features/004-agent-native-workflow-authoring/ISSUE.md)

---

## UX Improvements

*More coming soon.*

---

---

*Last updated: 2026-03-05 — 2 bugs, 4 features*
