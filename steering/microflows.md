# Microflows — Steering Guide

Use this guide when creating or modifying microflows.

## Naming Conventions

| Prefix | Purpose |
|---|---|
| `ACT_` | User-triggered actions |
| `SUB_` | Sub-microflows called by other microflows |
| `DS_` | Data source microflows (return object or list) |
| `VAL_` | Validation microflows (return Boolean) |
| `IVK_` | Integration/REST call microflows |

## Reading an Existing Microflow

```
ped_read_document(documentName="MyFirstModule.MyMicroflow", documentType="Microflows$Microflow")
```

Expand objects and flows:
```
ped_read_document(documentName="MyFirstModule.MyMicroflow", documentType="Microflows$Microflow", paths=["/objectCollection/objects", "/flows"])
```

## Creating a Microflow

Check first:
```
ped_find_document(moduleName="MyFirstModule", documentType="Microflows$Microflow")
```

Get schema:
```
ped_get_schema(elementTypes=["Microflows$Microflow"])
```

## Canvas Layout Rules

- Happy path flows left → right
- Error/exception flows top → bottom
- Connection points: `0=top`, `1=right`, `2=bottom`, `3=left`
- Connect right side of origin (`originConnectionIndex=1`) to left side of destination (`destinationConnectionIndex=3`)
- Space objects ~150px apart horizontally

## Object Types

| Type | Use |
|---|---|
| `Microflows$StartEvent` | Every microflow needs exactly one. Never connect parameters to it. |
| `Microflows$EndEvent` | Only ONE incoming flow allowed. Use ExclusiveMerge to merge paths first. |
| `Microflows$ActionActivity` | Wraps any action (retrieve, create, change, call microflow, etc.) |
| `Microflows$ExclusiveSplit` | IF/SWITCH branching. Must cover ALL cases (true+false for boolean; all enum values + `(empty)` for enums). |
| `Microflows$ExclusiveMerge` | Merges multiple paths back to one. Required before EndEvent when multiple paths exist. |
| `Microflows$LoopedActivity` | FOR-EACH loop. Child objects and flows are nested inside. Never connect outer flows to inner objects. |
| `Microflows$MicroflowParameterObject` | Input parameter. Never a flow destination. |
| `Microflows$InheritanceSplit` | Routes by entity subtype. Requires N subtypes + base type + empty case flows. |
| `Microflows$BreakEvent` | Breaks out of a loop. |
| `Microflows$ContinueEvent` | Continues to next loop iteration. |
| `Microflows$ErrorEvent` | Only on custom error handling flows. |
| `Microflows$Annotation` | Documentation. Connect with `Microflows$AnnotationFlow`, never SequenceFlow. |

## ExclusiveSplit Rules (Critical)

**Boolean split:** MUST have exactly two flows with `caseValue.enumerationCase` = `"true"` and `"false"`.

**Enumeration split:** MUST have one flow per enum value PLUS one mandatory `"(empty)"` flow. Missing the empty case causes a validation error.

## Flow References

Use `$id(/objectCollection/objects/N)` to reference objects by position (zero-based index).

Example flow connecting object 0 → object 1:
```json
{
  "$Type": "Microflows$SequenceFlow",
  "originId": "$id(/objectCollection/objects/0)",
  "destinationId": "$id(/objectCollection/objects/1)",
  "originConnectionIndex": 1,
  "destinationConnectionIndex": 3
}
```

## Common Actions (get schema for each before use)

- `Microflows$RetrieveAction` — retrieve objects from DB or association
- `Microflows$CreateObjectAction` — create a new object
- `Microflows$ChangeObjectAction` — change object attributes
- `Microflows$DeleteAction` — delete an object
- `Microflows$CommitAction` — commit object to DB
- `Microflows$MicroflowCallAction` — call another microflow
- `Microflows$ShowMessageAction` — show a message to the user
- `Microflows$ShowPageAction` — open a page
- `Microflows$ClosePageAction` — close the current page
- `Microflows$LogMessageAction` — write to the Mendix log

## After Every Change

```
ped_check_errors(documents=[{"documentType": "Microflows$Microflow", "documentName": "MyFirstModule.MyMicroflow"}])
```
