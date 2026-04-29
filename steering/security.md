# Security — Steering Guide

Use this guide when working with module roles, entity access rules, and page/microflow access.

## Reading Security Configuration

Module roles are defined on the domain model and individual documents. To read entity access rules:
```
ped_read_document(documentName="MyFirstModule", documentType="DomainModels$DomainModel", paths=["/entities/0/accessRules"])
```

## Entity Access Rules

Get schema:
```
ped_get_schema(elementTypes=["DomainModels$AccessRule"])
```

Key properties:
- `moduleRoles` — list of module role references (e.g., `["MyFirstModule.User"]`)
- `allowCreate` — boolean
- `allowDelete` — boolean
- `memberAccesses` — per-attribute read/write permissions

## Page Access

Set `allowedRoles` on the page document:
```
ped_update_document(
  documentName="MyFirstModule.MyPage",
  documentType="Pages$Page",
  operations=[{
    "path": "/allowedRoles",
    "operation": { "type": "set", "value": ["MyFirstModule.User", "MyFirstModule.Administrator"] }
  }]
)
```

## Microflow Access

Set `allowedModuleRoles` on the microflow:
```
ped_update_document(
  documentName="MyFirstModule.ACT_DoSomething",
  documentType="Microflows$Microflow",
  operations=[{
    "path": "/allowedModuleRoles",
    "operation": { "type": "set", "value": ["MyFirstModule.User"] }
  }]
)
```

## Module Roles

Module roles are defined at the module level. Common roles in this project:
- `MyFirstModule.User` — standard user role
- `Administration.Administrator` — admin role from the Administration module

## Entity Access Best Practices

- Always define access rules for every entity
- Use the principle of least privilege — only grant what's needed
- `Administration.Account` entity access is managed by the Administration module — do not modify it directly

## After Every Change

```
ped_check_errors(documents=[{"documentType": "DomainModels$DomainModel", "documentName": "MyFirstModule"}])
```
