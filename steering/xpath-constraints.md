---
inclusion: manual
---

# XPath Constraints — Steering Guide

Use this when writing XPath constraints for retrieve actions, data sources, and access rules.

> **Important:** Load the `microflow-xpath` MCP skill for the full reference before writing constraints.

## XPath vs Mendix Expressions

| Feature | XPath `[...]` | Mendix Expression |
|---|---|---|
| Use | Database filtering in retrieve activities | Calculations and logic in non-retrieve activities |
| Boolean ops | lowercase: `and`, `or`, `not()` | `and`, `or`, `not()` |
| Token quoting | `'[%CurrentUser%]'` (quoted) | `[%CurrentUser%]` (unquoted) |

## Basic Syntax

```xpath
[AttributeName = 'Value']
[AttributeName != 'Value']
[AttributeName > 100]
[AttributeName = empty]
[AttributeName != empty]
[IsActive = true()]
```

Operators: `=`, `!=`, `<`, `>`, `<=`, `>=`

## String Functions

```xpath
[contains(Name, 'John')]
[starts-with(Name, 'J')]
[ends-with(Email, '@company.com')]
```

## Enumeration Comparisons

Always use the fully qualified enum value:
```xpath
[Status = 'MyFirstModule.Status.Active']
[Status != 'MyFirstModule.Status.Inactive']
```

## Date/Time

```xpath
[CreatedDate > '[%BeginOfCurrentDay%]']
[CreatedDate < '[%EndOfCurrentDay%]']
[OrderDate >= $StartDate and OrderDate <= $EndDate]
```

Common tokens: `[%CurrentDateTime%]`, `[%BeginOfCurrentDay%]`, `[%EndOfCurrentDay%]`, `[%BeginOfCurrentWeek%]`, `[%BeginOfCurrentMonth%]`

## Association Traversal

```xpath
[MyFirstModule.Order_Customer/MyFirstModule.Customer/Name = 'Acme']
[MyFirstModule.Order_Customer/MyFirstModule.Customer]          -- existence check
[not(MyFirstModule.Order_Customer/MyFirstModule.Customer)]     -- no associated object
```

Format: `[Module.AssociationName/Module.TargetEntity/Attribute = value]`

## Current User

```xpath
[MyFirstModule.Customer_Account = '[%CurrentUser%]']
[System.owner = '[%CurrentUser%]']
```

## Combining Conditions

```xpath
[Status = 'MyFirstModule.Status.Active' and CreatedDate > '[%BeginOfCurrentMonth%]']
[Status = 'MyFirstModule.Status.Active' or Status = 'MyFirstModule.Status.Pending']
[(Status = 'MyFirstModule.Status.Pending' or Status = 'MyFirstModule.Status.Processing') and TotalAmount >= 100]
```

## Optional Filters (empty = skip)

```xpath
[($Category = empty or MyModule.Order_Category = $Category)]
[($ActiveOnly = false or IsActive = true) and contains(Name, $Query)]
```

## Usage in MCP Server

When setting XPath on a `RetrieveAction`, the constraint goes in `xpathConstraint` as a string:
```json
"xpathConstraint": "[Status = 'MyFirstModule.Status.Active']"
```

In security access rules, escape single quotes by doubling:
```
'[System.owner = ''[%CurrentUser%]'']'
```
