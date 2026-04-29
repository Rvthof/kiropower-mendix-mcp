---
inclusion: manual
---

# OQL Queries & View Entities — Steering Guide

Use this when creating or updating view entities with OQL queries.

> **Important:** Load the `view-entities` MCP skill for the full reference before working with OQL.

## MCP Tools

```
oql_generate(moduleName, userIntent)               → generate OQL from natural language
oql_generate(moduleName, userIntent, documentName) → generate and write directly
oql_read(documentName)                             → read existing OQL
ped_find_document(moduleName, "DomainModels$ViewEntitySourceDocument") → check if exists
```

## Creating a View Entity

1. Create the `ViewEntitySourceDocument` first:
```
ped_create_document(documentType="DomainModels$ViewEntitySourceDocument", moduleName="MyFirstModule",
  documentName="CustomerSummaryView", documentContent={"$Type": "DomainModels$ViewEntitySourceDocument", "name": "CustomerSummaryView"})
```

2. Add the entity to the domain model referencing it:
```json
{
  "$Type": "DomainModels$Entity",
  "name": "CustomerSummaryView",
  "source": {
    "$Type": "DomainModels$OqlViewEntitySource",
    "sourceDocument": "MyFirstModule.CustomerSummaryView"
  },
  "attributes": []
}
```

3. Generate and write OQL:
```
oql_generate(moduleName="MyFirstModule", userIntent="...", documentName="MyFirstModule.CustomerSummaryView")
```

## OQL Syntax Rules

### All SELECT columns MUST have explicit AS aliases
```sql
-- WRONG
SELECT fl.ForecastDate, fl.ProjectedIncome FROM Finance.ForecastLine AS fl

-- CORRECT
SELECT fl.ForecastDate AS ForecastDate, fl.ProjectedIncome AS ProjectedIncome
FROM Finance.ForecastLine AS fl
```

### No ORDER BY or LIMIT at the view level
ORDER BY and LIMIT are only valid inside correlated subqueries.

### Aggregate functions must be lowercase
```sql
sum(o.Amount)   avg(o.Amount)   count(t.ID)   max(o.Date)   min(o.Price)
```

### Use count(entity.ID) not count(*)
```sql
count(t.ID)   -- CORRECT
count(*)      -- NOT SUPPORTED
```

### Division uses colon, not slash
```sql
amount : quantity AS price    -- CORRECT
amount / quantity AS price    -- WRONG
```

### Enumeration comparisons use string literals
```sql
WHERE t.Status = 'Active'               -- CORRECT
WHERE t.Status = Finance.Status.Active  -- WRONG
```

### Use != not <>
```sql
WHERE t.Status != 'Voided'    -- CORRECT
WHERE t.Status <> 'Voided'    -- WRONG
```

## Common Patterns

**Date aggregation:**
```sql
SELECT datepart(YEAR, t.Date) AS Year, sum(t.Amount) AS Total
FROM Finance.Transaction AS t
GROUP BY datepart(YEAR, t.Date)
```

**Association navigation:**
```sql
SELECT o.OrderId AS OrderId, c.Name AS CustomerName
FROM Shop.Order AS o
INNER JOIN o/Shop.Order_Customer/Shop.Customer AS c
```

**Correlated subquery (latest price):**
```sql
SELECT p.Name AS Name,
  (SELECT pr.Price FROM Shop.Price AS pr
   WHERE pr/Shop.Price_Product = p.ID
   ORDER BY pr.StartDate DESC LIMIT 1) AS LatestPrice
FROM Shop.Product AS p
```

## After Every Change

```
ped_check_errors(documents=[
  {"documentType": "DomainModels$DomainModel", "documentName": "MyFirstModule"},
  {"documentType": "DomainModels$ViewEntitySourceDocument", "documentName": "MyFirstModule.CustomerSummaryView"}
])
```
