---
inclusion: manual
---

# Pages — Steering Guide

Use this when creating or modifying pages and their widgets via the MCP server.

> **Important:** Before creating or modifying any page, load the `page-gen-common` MCP skill.
> Also call `ped_list_folder` on the module root to confirm placement before creating.

## MCP Tools

```
ped_find_document(moduleName, "Pages$Page")               → check if exists first
ped_get_schema(["Pages$Page"])                            → get schema
ped_create_document([{...}])                              → create page
ped_read_document("Module.PageName", "Pages$Page")        → read page
ped_update_document("Pages$Page", "Module.PageName", ops) → update
ped_check_errors([{documentType, documentName}])          → validate after every change
```

Always call `ped_find_document` first — if a match exists, read and update instead of creating.

## Naming Conventions

| Pattern | Example |
|---|---|
| Overview page | `Customer_Overview` |
| Create/edit page | `Customer_NewEdit` |
| Detail page | `Customer_Detail` |
| Popup dialog | `Customer_Confirm` |

## Common Layouts

| Layout | Use |
|---|---|
| `Atlas_Core.Atlas_Default` | Standard full-page layout |
| `Atlas_Core.Atlas_TopBar` | Page with top navigation bar |
| `Atlas_Core.PopupLayout` | Modal/popup dialog |
| `Atlas_Core.Atlas_Default_Sidebar` | Page with sidebar |

## Common Widget Types (get schema before use)

| Widget | Use |
|---|---|
| `Pages$DataView` | Single-object form. Requires a data source. |
| `Pages$ListView` | Scrollable list of objects. |
| `Pages$DataGrid2` | Tabular data grid with sorting/filtering. |
| `Pages$Gallery` | Card-based grid layout. |
| `Pages$TextBox` | Input field bound to a string attribute. |
| `Pages$TextArea` | Multi-line text input. |
| `Pages$DatePicker` | Date/time input. |
| `Pages$DropDown` | Enumeration selector. |
| `Pages$CheckBox` | Boolean toggle. |
| `Pages$Button` | Action button (calls microflow, opens page, etc.). |
| `Pages$Label` | Static text label. |
| `Pages$DynamicText` | Text that displays an attribute value. |
| `Pages$Container` | Layout container for grouping widgets. |
| `Pages$GroupBox` | Collapsible group with a title. |
| `Pages$TabContainer` | Tabbed layout. |
| `Pages$NavigationTree` | Navigation menu widget. |
| `Pages$FileManager` | File upload/download widget. |
| `Pages$ImageViewer` | Displays an image attribute. |

## Data Sources

Widgets that display data need a data source:
- `Pages$MicroflowSource` — calls a DS_ microflow
- `Pages$DatabaseSource` — retrieves directly from DB with optional XPath constraint
- `Pages$AssociationSource` — follows an association from the enclosing context
- `Pages$PageParameterSource` — uses the page parameter (for NewEdit/Detail pages)
- `Pages$WidgetSelectionSource` — listens to selection in another widget (master-detail)

## Page Roles

Always set `allowedRoles` to restrict access. Reference module roles as `"ModuleName.RoleName"`.

## Common Page Patterns

### Overview (List) Page
- Layout: `Atlas_Core.Atlas_Default`
- DataGrid2 with DatabaseSource, columns per attribute, search filters
- "New" button → opens NewEdit page; row "Edit" → opens NewEdit with object; row "Delete" → delete microflow

### NewEdit (Form) Page
- Layout: `Atlas_Core.PopupLayout`
- DataView with PageParameterSource bound to the page parameter
- Input widgets per attribute type; Save/Cancel buttons in footer

### Master-Detail Page
- LayoutGrid with two columns (~35% / ~65%)
- Left: Gallery with `selectionMode = Single` and a unique `widgetName`
- Right: DataView with `Pages$WidgetSelectionSource` pointing to the gallery's `widgetName`
- Set `noEntityMessage` on the DataView for the empty state

## Checklist

- [ ] `ped_find_document` called before creating to avoid duplicates
- [ ] Schemas fetched with `ped_get_schema` before adding widgets
- [ ] `allowedRoles` set — never leave a page open to all roles unless intended
- [ ] DataView widgets have a data source
- [ ] All targets (microflows, pages) are fully qualified (`Module.Name`)
- [ ] `ped_check_errors` run after every create/update
