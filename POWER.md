---
name: "mendix-mcp"
displayName: "Mendix MCP Development"
description: "AI-assisted Mendix development using the MCP server (localhost:7782). Build domain models, microflows, pages, security, navigation, OQL queries, and JavaScript actions — all via MCP tools."
keywords: ["mendix", "mcp", "microflow", "nanoflow", "entity", "domain model", "page", "datagrid", "gallery", "module", "enumeration", "association", "xpath", "oql", "atlas", "mpr", "security", "navigation", "crud", "javascript action"]
author: "Workspace Power"
---

# ⚙️ Setup Required

After installing this Power, set up the following hook so that file-writing rules are automatically enforced before any write operation. Create `.kiro/hooks/file-writing-rules.json` with this content:

```json
{
  "name": "File Writing Rules",
  "version": "1.0.0",
  "description": "Remind the agent to follow file-writing rules before any write operation",
  "when": {
    "type": "preToolUse",
    "toolTypes": ["write"]
  },
  "then": {
    "type": "askAgent",
    "prompt": "Before writing this file, read and follow the rules in .kiro/steering/file-writing.md. Key points: use fsWrite for the first ~40 lines, then fsAppend for each subsequent ~40-line chunk. Never put more than ~45 lines in a single call."
  }
}
```

> If this hook is not in place, large file writes may be silently truncated.

---

# Mendix MCP Development Power

This power gives Kiro deep knowledge of Mendix development via the MCP server running at `localhost:7782`. It covers the full development lifecycle: domain modelling, microflows, pages, security, navigation, OQL queries, and JavaScript actions — all through structured MCP tool calls against the live `.mpr` model.

---

# MCP Server Connection

The MCP server is pre-configured at `localhost:7782` via `.kiro/settings/mcp.json`. No binary or CLI setup is required. All model changes are made directly through MCP tool calls.

**Available modules in this project:**
`MyFirstModule`, `Administration`, `FeedbackModule`, `Atlas_Core`, `Atlas_Web_Content`, `DataWidgets`, `NanoflowCommons`, `WebActions`, `System`

---

# MCP Tool Reference

## Project Editor (PED) Tools — Core Model Manipulation

| Tool | Purpose | Key Parameters |
|---|---|---|
| `ped_list_folder` | Browse module contents (documents + subfolders) | `moduleName`, `folderPath?` |
| `ped_find_document` | Search for documents by type in a module | `moduleName`, `documentType` |
| `ped_read_document` | Read a document's full structure via JSON Pointer paths | `documentName`, `documentType`, `paths?` |
| `ped_get_schema` | Get the schema for element types | `elementTypes[]` |
| `ped_create_document` | Create new documents (microflows, pages, enumerations, etc.) | `documentType`, `moduleName`, `documentName`, `documentContent`, `folderPath?` |
| `ped_update_document` | Atomically update an existing document with set/add/remove ops | `documentName`, `documentType`, `operations[]` |
| `ped_check_errors` | Validate documents for errors | `documents[]` |
| `ped_create_module` | Create a new module (only when explicitly requested) | `moduleName` |

### Supported Document Types
- `DomainModels$DomainModel` — entities, attributes, associations (use module name only as documentName)
- `Microflows$Microflow` — server-side logic flows
- `Pages$Page` — UI pages
- `Workflows$Workflow` — workflow documents
- `Enumerations$Enumeration` — enum types

## OQL Tools — View Entity Queries

| Tool | Purpose | Key Parameters |
|---|---|---|
| `oql_generate` | Generate OQL for a view entity from natural language | `moduleName`, `userIntent`, `documentName?`, `currentOql?` |
| `oql_read` | Read the current OQL from a ViewEntitySourceDocument | `documentName` |

## File System Tools — JavaScript Actions & Themes

| Tool | Purpose | Key Parameters |
|---|---|---|
| `glob` | List files matching a pattern in `/jsactions` or `/themes` | `pattern` |
| `read_file` | Read a JS action or theme file | `path`, `startLine?`, `endLine?` |
| `write_file` | Create or update a JS action or theme file | `path`, `newContent`, `span?` |

**Accessible file domains:**
- `/jsactions/<module>/actions/<action>.js` — JavaScript actions
- `/themes/` — SCSS/CSS theme files

## Knowledge Base Tool

| Tool | Purpose | Key Parameters |
|---|---|---|
| `search_mendix_knowledge_base` | Search Mendix docs and marketplace knowledge | `query` |

---

# Mandatory Workflow Rules

These rules MUST be followed on every operation to avoid errors and duplicates.

### 1. Always check before creating
Run `ped_find_document` before `ped_create_document`. If a matching document exists, read and update it instead.

### 2. Always get schemas before adding elements
Run `ped_get_schema` with all needed element types before constructing `documentContent` or `operations`. Never guess schema structure from prior reads.

### 3. Always validate after every change
Run `ped_check_errors` after every `ped_create_document` or `ped_update_document`. If errors are found, attempt one fix then re-check. If errors persist, report to user and stop.

### 4. Domain model uses module name only
`documentName` for domain models is just the module name: `"MyFirstModule"` — not `"MyFirstModule.DomainModel"`.

### 5. Use `$id(/path)` for cross-references
When referencing elements within the same document (e.g., linking microflow flows to objects), use `"$id(/objects/0)"` syntax. Indices are zero-based.

### 6. User associations always target Administration.Account
Never use `System.User` as an association target. Always use `Administration.Account`.

### 7. Order of operations matters in updates
When adding elements that reference each other (e.g., entities then associations), add the referenced elements first. Re-read arrays after any remove operation before further add/remove on the same path.

### 8. Never batch add and remove on the same path
Re-read the document after every remove before issuing further operations on the same array path.

---

# Steering Instructions

## When to Load Steering Files

Load the appropriate steering file(s) based on what you're working on:

### Domain Model
| Working on... | Load this steering file |
|---|---|
| Domain model, entities, associations, enumerations | `domain-model.md` |

### Microflows & Logic
| Working on... | Load this steering file |
|---|---|
| Microflows, nanoflows, ACT_, SUB_, DS_, VAL_ | `microflows.md` |
| CRUD patterns | `patterns-crud.md` |
| XPath constraints | `xpath-constraints.md` |

### Pages & UI
| Working on... | Load this steering file |
|---|---|
| Pages, widgets, DATAGRID, GALLERY, DATAVIEW | `pages.md` |

### Security & Navigation
| Working on... | Load this steering file |
|---|---|
| Security roles, access rules | `security.md` |
| Navigation profiles, menus | `navigation.md` |

### OQL & View Entities
| Working on... | Load this steering file |
|---|---|
| OQL queries, VIEW entities | `oql-queries.md` |

### JavaScript Actions
| Working on... | Load this steering file |
|---|---|
| JavaScript actions | `javascript-actions.md` |

### General
| Working on... | Load this steering file |
|---|---|
| Writing any file | `file-writing.md` |

> **Hook setup required:** The `file-writing.md` steering file should be loaded automatically before any file write operation. Set it up as a `preToolUse` hook targeting `write` tools:
>
> ```json
> {
>   "name": "File Writing Rules",
>   "version": "1.0.0",
>   "description": "Remind the agent to follow file-writing rules before any write operation",
>   "when": {
>     "type": "preToolUse",
>     "toolTypes": ["write"]
>   },
>   "then": {
>     "type": "askAgent",
>     "prompt": "Before writing this file, read and follow the rules in .kiro/steering/file-writing.md. Key points: use fsWrite for the first ~40 lines, then fsAppend for each subsequent ~40-line chunk. Never put more than ~45 lines in a single call."
>   }
> }
> ```
>
> Save this as `.kiro/hooks/file-writing-rules.json` in your workspace.

---

# Core Principles (Always Apply)

1. **Read before writing** — always `ped_read_document` to understand current state before making changes
2. **Describe changes in plain language** — never show raw JSON document structures in chat unless the user explicitly asks
3. **Batch independent schema fetches** — call `ped_get_schema` with multiple types in one call to reduce round-trips
4. **Prefer parallel reads** — when reading multiple independent documents, read them in the same tool call batch
5. **Stop on persistent errors** — after one failed fix attempt, report the error and suggested solution to the user and stop
