# CRUD Patterns — Steering Guide

Use this guide when implementing standard Create, Read, Update, Delete patterns in Mendix via MCP.

## Standard CRUD Microflow Set

For an entity `MyFirstModule.Customer`, the standard set is:

| Microflow | Purpose | Returns |
|---|---|---|
| `DS_Customer_GetList` | Retrieve all customers | List of Customer |
| `DS_Customer_GetById` | Retrieve single customer by ID | Customer object |
| `ACT_Customer_New` | Open new customer form | Void |
| `ACT_Customer_Edit` | Open edit form for existing customer | Void |
| `ACT_Customer_Save` | Validate and commit customer | Void |
| `ACT_Customer_Delete` | Delete customer with confirmation | Void |

## DS_ Data Source Pattern

```
Microflow: DS_Customer_GetList
- StartEvent
- RetrieveAction (from DB, XPath: all Customer objects, sorted by Name)
- EndEvent (returns $CustomerList)
```

## ACT_Save Pattern

```
Microflow: ACT_Customer_Save
- StartEvent
- Parameter: Customer (MyFirstModule.Customer)
- [Optional] VAL_ validation microflow call
  - ExclusiveSplit on validation result (true/false)
    - false → ShowMessageAction (validation error) → EndEvent
    - true → continue
- CommitAction (commit Customer)
- ClosePageAction
- EndEvent
```

## ACT_Delete Pattern

```
Microflow: ACT_Customer_Delete
- StartEvent
- Parameter: Customer (MyFirstModule.Customer)
- DeleteAction (delete Customer)
- EndEvent
```

## Overview Page Pattern

1. Create `Customer_Overview` page with `Atlas_Core.Atlas_Default` layout
2. Add a `DataGrid2` widget with `DatabaseSource` pointing to `MyFirstModule.Customer`
3. Add columns for key attributes
4. Add "New" button → calls `ACT_Customer_New`
5. Add row-level "Edit" button → calls `ACT_Customer_Edit` with selected object
6. Add row-level "Delete" button → calls `ACT_Customer_Delete` with selected object

## Detail Page Pattern

1. Create `Customer_Detail` page with `Atlas_Core.PopupLayout` layout
2. Add a `DataView` with the Customer object as context
3. Add `TextBox` widgets for each editable attribute
4. Add "Save" button → calls `ACT_Customer_Save`
5. Add "Cancel" button → `ClosePageAction`

## Validation Pattern

```
Microflow: VAL_Customer_Validate
- StartEvent
- Parameter: Customer (MyFirstModule.Customer)
- ExclusiveSplit: $Customer/Name = empty
  - true → EndEvent (returns false)
  - false → continue
- [Additional checks...]
- EndEvent (returns true)
```
