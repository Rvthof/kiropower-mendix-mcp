---
inclusion: manual
---

# CRUD Patterns — Steering Guide

Use this when implementing standard Create, Read, Update, Delete patterns in Mendix.

## Naming Conventions

| Prefix | Purpose | Example |
|---|---|---|
| `ACT_` | User-triggered action microflow | `ACT_Customer_Save` |
| `VAL_` | Validation microflow (returns Boolean) | `VAL_Customer_Save` |
| `DS_` | Data source microflow (returns object or list) | `DS_Customer_GetAll` |
| `SUB_` | Sub-microflow (internal, called by other microflows) | `SUB_SendNotification` |

## Standard CRUD Microflow Set

For an entity `MyFirstModule.Customer`:

| Microflow | Purpose | Returns |
|---|---|---|
| `DS_Customer_GetList` | Retrieve all customers | List of Customer |
| `DS_Customer_GetById` | Retrieve single customer by ID | Customer object |
| `ACT_Customer_New` | Open new customer form | Void |
| `ACT_Customer_Save` | Validate and commit customer | Void |
| `ACT_Customer_Delete` | Delete customer | Void |

## Save Pattern

```
ACT_Customer_Save
1. Call VAL_Customer_Save
2. ExclusiveSplit on result:
   - false → ShowMessage (validation error) → End
   - true → CommitAction → ClosePageAction → End
```

## Validation Pattern

```
VAL_Customer_Save
1. $IsValid = true
2. Check each required field:
   IF field empty → $IsValid = false → show validation feedback
3. Return $IsValid
```

Enumeration comparisons use fully qualified values:
```
CORRECT: $Task/Status = Module.TaskStatus.Completed
WRONG:   $Task/Status = 'Completed'
```

## Delete Pattern

```
ACT_Customer_Delete
1. DeleteAction (delete Customer)
2. End
```

## Overview Page Pattern

1. Create `Customer_Overview` with `Atlas_Core.Atlas_Default` layout
2. DataGrid2 with DatabaseSource pointing to `MyFirstModule.Customer`
3. Columns for key attributes
4. "New" button → `ACT_Customer_New`
5. Row "Edit" button → opens `Customer_NewEdit` with selected object
6. Row "Delete" button → `ACT_Customer_Delete` with selected object

## Detail/Form Page Pattern

1. Create `Customer_NewEdit` with `Atlas_Core.PopupLayout` layout
2. DataView with PageParameterSource bound to the Customer page parameter
3. Input widgets per attribute type
4. "Save" button → `ACT_Customer_Save`
5. "Cancel" button → ClosePageAction

## Best Practices

- Always validate before commit — call VAL_ microflow in ACT_Save
- Commit WITH EVENTS — triggers event handlers
- Close page on success
- Initialize defaults in ACT_New microflow
- All action microflows should return Boolean success status
