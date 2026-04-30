---
name: "mendix-mcp"
displayName: "Mendix MCP Development"
description: "AI-assisted Mendix development using the MCP server (localhost:7782). Build domain models, microflows, pages, security, navigation, OQL queries, and JavaScript actions — all via MCP tools."
keywords: ["mendix", "mcp", "microflow", "nanoflow", "entity", "domain model", "page", "datagrid", "gallery", "module", "enumeration", "association", "xpath", "oql", "atlas", "mpr", "security", "navigation", "crud", "javascript action"]
author: "Workspace Power"
---

# Mendix MCP Development Power

This power gives Kiro deep knowledge of Mendix development via the MCP server running at `localhost:7782`. It covers the full development lifecycle: domain modelling, microflows, pages, security, navigation, OQL queries, and JavaScript actions — all through structured MCP tool calls against the live `.mpr` model.

> **Using Mendix via MxCLI instead?** If your project uses the MxCLI binary and MDL scripts rather than the Studio Pro MCP server, use the [Mendix MxCLI Power](https://github.com/Rvthof/awskiro-mxcli-power) instead. This power is specifically for MCP-based development.

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
| Entities, attributes, associations, enumerations | `domain-model.md` |

### Microflows & Logic
| Working on... | Load this steering file |
|---|---|
| Microflows, nanoflows, ACT_, SUB_, DS_, VAL_ | `microflows.md` |
| CRUD patterns | `patterns-crud.md` |
| XPath constraints | `xpath-constraints.md` |

### Pages & UI
| Working on... | Load this steering file |
|---|---|
| Pages, widgets, layouts, master-detail | `pages.md` |
| SCSS/CSS theme and styling | `theme-styling.md` |
| Custom pluggable widgets (React/TypeScript) | `create-custom-widget.md` |

### Security & Navigation
| Working on... | Load this steering file |
|---|---|
| Security roles, access rules | `security.md` |
| Navigation profiles, menus | `navigation.md` |

### OQL & View Entities
| Working on... | Load this steering file |
|---|---|
| OQL queries, VIEW entities | `oql-queries.md` |

### Data Sharing & Integrations
| Working on... | Load this steering file |
|---|---|
| OData inter-app data sharing | `odata-data-sharing.md` |
| REST API calls from microflows | `rest-integration.md` |
| Complex REST (SPARQL, special auth, nested JSON) | `rest-sparql-integration.md` |
| Event-driven messaging via Business Events / Kafka | `business-events.md` |

### Actions
| Working on... | Load this steering file |
|---|---|
| JavaScript actions | `javascript-actions.md` |
| Java actions | `java-actions.md` |

### AI Agents
| Working on... | Load this steering file |
|---|---|
| Mendix AI agents (Mendix 11.9+) | `agents.md` |

### System Reference
| Working on... | Load this steering file |
|---|---|
| System module entities (User, FileDocument, workflows) | `system-module.md` |

### Migration & Assessment
| Working on... | Load this steering file |
|---|---|
| Project quality audit | `assess-quality.md` |
| Migrating a non-Mendix app | `assess-migration.md` |

---

# Core Principles (Always Apply)

0. **Confirm the approach: MCP vs MxCLI** — This power is for MCP-based workflows only. If the user mentions MxCLI, MDL scripts, or wants to use the `./mxcli` binary, stop and point them to the [Mendix MxCLI Power](https://github.com/Rvthof/awskiro-mxcli-power) instead. Do not use MCP tools for MxCLI-based workflows.

1. **Read before writing** — always `ped_read_document` to understand current state before making changes
2. **Describe changes in plain language** — never show raw JSON document structures in chat unless the user explicitly asks
3. **Batch independent schema fetches** — call `ped_get_schema` with multiple types in one call to reduce round-trips
4. **Prefer parallel reads** — when reading multiple independent documents, read them in the same tool call batch
5. **Stop on persistent errors** — after one failed fix attempt, report the error and suggested solution to the user and stop
