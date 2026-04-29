# OQL Queries & View Entities — Steering Guide

Use this guide when working with OQL queries and view entities.

## What is a View Entity?

A view entity is a read-only entity backed by an OQL query. It behaves like a database view — you can use it in data grids, microflows, and pages just like a regular entity, but its data comes from the OQL query rather than a direct table.

## Generating OQL

Use the `oql_generate` tool with a natural language description:
```
oql_generate(
  moduleName="MyFirstModule",
  userIntent="Get all customers with their total order count and latest order date"
)
```

To write the result directly to a ViewEntitySourceDocument:
```
oql_generate(
  moduleName="MyFirstModule",
  userIntent="...",
  documentName="MyFirstModule.CustomerSummaryView"
)
```

To refine existing OQL:
```
oql_generate(
  moduleName="MyFirstModule",
  userIntent="Also filter to only active customers",
  currentOql="SELECT ..."
)
```

## Reading Existing OQL

```
oql_read(documentName="MyFirstModule.CustomerSummaryView")
```

## Creating a View Entity

1. First create the `ViewEntitySourceDocument`:
```
ped_find_document(moduleName="MyFirstModule", documentType="DomainModels$ViewEntitySourceDocument")
ped_get_schema(elementTypes=["DomainModels$ViewEntitySourceDocument"])
ped_create_document(documentType="DomainModels$ViewEntitySourceDocument", moduleName="MyFirstModule", documentName="CustomerSummaryView", documentContent={...})
```

2. Then add a view entity to the domain model that references it:
```
ped_get_schema(elementTypes=["DomainModels$Entity"])
ped_update_document(documentName="MyFirstModule", documentType="DomainModels$DomainModel", operations=[{
  "path": "/entities",
  "operation": {
    "type": "add",
    "value": {
      "$Type": "DomainModels$Entity",
      "name": "CustomerSummary",
      "source": {
        "$Type": "DomainModels$OqlViewEntitySource",
        "sourceDocument": "MyFirstModule.CustomerSummaryView"
      },
      "attributes": [...]
    }
  }
}])
```

## OQL Syntax Basics

```sql
SELECT
  c/ID AS CustomerID,
  c/Name AS CustomerName,
  COUNT(o/ID) AS OrderCount
FROM MyFirstModule.Customer AS c
LEFT JOIN MyFirstModule.Order AS o ON o/Customer = c/ID
WHERE c/Status = 'Active'
GROUP BY c/ID, c/Name
ORDER BY c/Name ASC
```

Key rules:
- Use `/` as the path separator (not `.`)
- Entity names are fully qualified: `Module.EntityName`
- Attribute access: `alias/AttributeName`
- Association traversal: `alias/AssociationName`

## After Every Change

```
ped_check_errors(documents=[
  {"documentType": "DomainModels$DomainModel", "documentName": "MyFirstModule"},
  {"documentType": "DomainModels$ViewEntitySourceDocument", "documentName": "MyFirstModule.CustomerSummaryView"}
])
```
