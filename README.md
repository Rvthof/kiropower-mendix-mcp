# Mendix MCP Power

This Kiro Power enables AI-assisted Mendix development by connecting directly to the **Studio Pro MCP Server** — Mendix's built-in MCP server that exposes the same capabilities as Maia, Mendix's own AI assistant.

Instead of using a CLI binary or scripting language, all model changes (entities, microflows, pages, security, OQL, and more) are made through structured MCP tool calls against the live `.mpr` project file, with changes reflected in real time inside Studio Pro.

---

## How It Works

```
Kiro ──MCP tools──▶ Studio Pro MCP Server (localhost:7782) ──▶ .mpr project file
                                                               ◀── live updates in Studio Pro
```

1. Studio Pro runs an MCP server locally on a configurable port (default `7782`)
2. Kiro connects to it via the MCP configuration in `.kiro/settings/mcp.json`
3. Kiro uses the MCP tools to read and write directly to the Mendix model
4. Every change appears live in Studio Pro — no restart or sync needed

---

## Prerequisites

### 1. Enable the MCP Server in Studio Pro

Navigate to **Preferences → Maia → MCP Server** and check **Enable MCP Server**.

You can also configure the port here (default is `7782`). Make sure the port in `.kiro/settings/mcp.json` matches.

For full details, see the official docs: [Studio Pro MCP Server](https://docs.mendix.com/refguide/studio-pro-mcp-server/)

### 2. Open your project in Studio Pro

The MCP server only runs while Studio Pro is open with your project loaded. Kiro cannot connect if Studio Pro is closed.

### 3. Verify the MCP config in this workspace

The connection is pre-configured at `.kiro/settings/mcp.json`:

```json
{
  "mcpServers": {
    "localhost-7782": {
      "url": "http://localhost:7782/mcp",
      "disabled": false,
      "autoApprove": [
        "ped_read_document",
        "ped_find_document",
        "ped_get_schema",
        "oql_generate",
        "ped_list_folder",
        "glob",
        "search_mendix_knowledge_base"
      ]
    }
  }
}
```

If you changed the port in Studio Pro, update the `url` here to match.

---

## What You Can Build

| Capability | Tools Used |
|---|---|
| Domain model — entities, attributes, associations, enumerations | `ped_update_document`, `ped_create_document` |
| Microflows and nanoflows | `ped_create_document`, `ped_update_document` |
| Pages and widgets | `ped_create_document`, `ped_update_document` |
| Security — access rules, module roles | `ped_update_document` |
| Navigation — menus, home pages | `ped_update_document` |
| OQL queries and view entities | `oql_generate`, `oql_read` |
| REST API integration | `ped_create_document`, `ped_update_document` |
| OData inter-app data sharing | `ped_create_document`, `oql_generate` |
| Business Events (Kafka pub/sub) | `ped_create_document`, `ped_update_document` |
| JavaScript actions | `glob`, `read_file`, `write_file` |
| Java actions | Kiro file tools (`Read`, `Edit`, `Write`) |
| Mendix AI agents | `ped_create_document`, `ped_get_schema` |
| Custom pluggable widgets | npm / widget build tools |
| Mendix knowledge lookup | `search_mendix_knowledge_base` |

---

## Steering Guides

This power includes steering files that Kiro loads based on what you're working on:

| File | When it's used |
|---|---|
| `steering/domain-model.md` | Creating entities, attributes, associations, enumerations |
| `steering/microflows.md` | Building microflows and nanoflows |
| `steering/patterns-crud.md` | CRUD microflow and page patterns |
| `steering/xpath-constraints.md` | XPath syntax for retrieves and access rules |
| `steering/pages.md` | Creating and modifying pages, widgets, and layouts |
| `steering/theme-styling.md` | SCSS/CSS theme customization |
| `steering/create-custom-widget.md` | Building custom pluggable widgets (React/TypeScript) |
| `steering/security.md` | Configuring access rules and module roles |
| `steering/navigation.md` | Managing navigation profiles and menus |
| `steering/oql-queries.md` | Writing OQL queries for view entities |
| `steering/odata-data-sharing.md` | OData inter-app data sharing |
| `steering/rest-integration.md` | Calling external REST APIs from microflows |
| `steering/rest-sparql-integration.md` | Complex REST integrations (SPARQL, special auth, nested JSON) |
| `steering/business-events.md` | Event-driven messaging via Business Events / Kafka |
| `steering/javascript-actions.md` | Creating and editing JavaScript actions |
| `steering/java-actions.md` | Creating and editing Java actions |
| `steering/agents.md` | Setting up Mendix AI agents (Mendix 11.9+) |
| `steering/system-module.md` | System module entity reference |
| `steering/assess-quality.md` | Auditing Mendix project quality |
| `steering/assess-migration.md` | Assessing a non-Mendix app for migration |

---

## Key Rules Kiro Follows

- **Always checks before creating** — runs `ped_find_document` to avoid duplicates
- **Always gets schemas** — runs `ped_get_schema` before constructing any new element
- **Always validates** — runs `ped_check_errors` after every change; stops and reports if errors persist
- **Never uses `System.User`** — always targets `Administration.Account` for user associations
- **Reads before writing** — understands current model state before making changes

---

## Troubleshooting

**Kiro can't connect to the MCP server**
- Make sure Studio Pro is open with your project loaded
- Check that the MCP server is enabled: Preferences → Maia → MCP Server
- Verify the port matches between Studio Pro and `.kiro/settings/mcp.json`

**Changes aren't appearing in Studio Pro**
- The MCP server reflects changes in real time — if you don't see them, try clicking into the affected document in Studio Pro to refresh the view

**"Module not found" errors**
- Module names are case-sensitive. Use `ped_list_folder` to confirm the exact module name

---

## Further Reading

- [Studio Pro MCP Server — Official Docs](https://docs.mendix.com/refguide/studio-pro-mcp-server/)
- [Maia Make Capabilities](https://docs.mendix.com/refguide/maia-make-capabilities/)
- [Mendix AI Assistance (Maia)](https://docs.mendix.com/refguide/mendix-ai-assistance/)
