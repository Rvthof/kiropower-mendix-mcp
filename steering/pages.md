# Pages — Steering Guide

Use this guide when creating or modifying pages and their widgets.

## Reading an Existing Page

```
ped_read_document(documentName="MyFirstModule.Home_Web", documentType="Pages$Page")
```

Expand the layout and widgets:
```
ped_read_document(documentName="MyFirstModule.Home_Web", documentType="Pages$Page", paths=["/layoutCall", "/layoutCall/widgets"])
```

## Creating a Page

Check first:
```
ped_find_document(moduleName="MyFirstModule", documentType="Pages$Page")
```

Get schema:
```
ped_get_schema(elementTypes=["Pages$Page"])
```

Key properties:
- `name` — page name (e.g., `"Customer_Overview"`)
- `layout` — layout reference (e.g., `"Atlas_Core.PopupLayout"` or `"Atlas_Core.Atlas_Default"`)
- `allowedRoles` — list of module roles that can access this page (e.g., `["MyFirstModule.User"]`)
- `title` — display title (use `Texts$Text` element)
- `canvasWidth` / `canvasHeight` — default `1200` / `600`

## Common Layout Names

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

Widgets that display data need a data source. Common types:
- `Pages$MicroflowSource` — calls a DS_ microflow
- `Pages$DatabaseSource` — retrieves directly from DB with optional XPath constraint
- `Pages$AssociationSource` — follows an association from the enclosing context

## Page Roles

Always set `allowedRoles` to restrict access. Reference module roles as `"ModuleName.RoleName"`.

## After Every Change

```
ped_check_errors(documents=[{"documentType": "Pages$Page", "documentName": "MyFirstModule.MyPage"}])
```
