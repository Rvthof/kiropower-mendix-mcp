---
inclusion: manual
---

# Mendix AI Agents

Use this when setting up or modifying AI agents in Mendix (requires Mendix 11.9+ and the AgentEditorCommons marketplace module).

## Core Document Types

Four document types combine to define an agent:

| Document | Purpose |
|----------|---------|
| **Model** | Configures the GenAI provider (e.g., OpenAI, Azure OpenAI) |
| **Knowledge Base** | References external knowledge sources the agent can query |
| **Consumed MCP Service** | Connects to external MCP tool servers |
| **Agent** | Orchestrates model, tools, and system prompt |

## Prerequisites

- Install the `AgentEditorCommons` module from the Mendix Marketplace
- Mendix 11.9 or higher
- A GenAI provider configured (API key as constant or environment variable)

## Creating an Agent via MCP

Use `ped_get_schema` to get the schema for each document type, then `ped_create_document`:

```
ped_get_schema(["AgentEditorCommons$Model"])                  → model schema
ped_get_schema(["AgentEditorCommons$Agent"])                  → agent schema
ped_find_document(moduleName, "AgentEditorCommons$Agent")     → check if exists
ped_create_document([{...}])                                  → create
ped_check_errors([{documentType, documentName}])              → validate
```

## Dependency Order (Drop/Create)

When removing agents, drop in this order to avoid reference errors:
1. Agents (depend on models, knowledge bases, MCP services)
2. Knowledge Bases
3. Consumed MCP Services
4. Models

When creating, reverse the order.

## Multi-Line Prompts

System prompts often span multiple lines. Use dollar-quoting (`$...$`) rather than single quotes when writing prompts in MDL — single quotes cannot span lines.

## Calling Agents from Microflows

After creating the agent document, call it from a microflow using Java actions from `AgentEditorCommons`:

| Java Action | Use |
|-------------|-----|
| `Agent_Call_WithoutHistory` | Single-turn: send a prompt and get a response |
| `Agent_Call_WithHistory` | Multi-turn: maintain conversation context across calls |

## Auto-Populated Metadata

The portal automatically populates `DisplayName` and `DeepLinkURL` during synchronization. Do not set these manually — they will be overwritten.

## Checklist

- [ ] `AgentEditorCommons` module installed
- [ ] Model document created with valid provider credentials
- [ ] Agent references the correct model
- [ ] System prompt is set (use `$...$` for multi-line prompts)
- [ ] Microflow calls the agent via `Agent_Call_WithoutHistory` or `Agent_Call_WithHistory`
- [ ] Run `ped_check_errors` on all agent documents after creation
