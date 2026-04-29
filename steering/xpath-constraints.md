# XPath Constraints — Steering Guide

Use this guide when writing XPath constraints for retrieve actions, data sources, and access rules.

## Basic Syntax

XPath constraints filter which objects are retrieved. They go in `RetrieveAction` or `DatabaseSource` widgets.

```xpath
[AttributeName = 'Value']
[AttributeName != 'Value']
[AttributeName > 100]
[AttributeName <= 50]
[AttributeName = empty]
[AttributeName != empty]
```

## String Comparisons

```xpath
[Name = 'John']
[Name != 'John']
[contains(Name, 'John')]
[starts-with(Name, 'J')]
```

## Boolean

```xpath
[IsActive = true()]
[IsActive = false()]
```

## Enumeration

```xpath
[Status = 'MyFirstModule.Status.Active']
[Status != 'MyFirstModule.Status.Inactive']
```

Always use the fully qualified enum value: `Module.EnumName.Value`

## Date/Time

```xpath
[CreatedDate > '[%BeginOfCurrentDay%]']
[CreatedDate < '[%EndOfCurrentDay%]']
[CreatedDate > '[%CurrentDateTime%]']
```

Common tokens: `[%CurrentDateTime%]`, `[%BeginOfCurrentDay%]`, `[%EndOfCurrentDay%]`, `[%BeginOfCurrentWeek%]`, `[%BeginOfCurrentMonth%]`

## Association Traversal

```xpath
[MyFirstModule.Order_Customer/MyFirstModule.Customer/Name = 'Acme']
```

Format: `[Module.AssociationName/Module.TargetEntity/Attribute = value]`

## Current User

```xpath
[MyFirstModule.Customer_Account = '[%CurrentUser%]']
```

## Combining Conditions

```xpath
[Status = 'MyFirstModule.Status.Active' and CreatedDate > '[%BeginOfCurrentMonth%]']
[Status = 'MyFirstModule.Status.Active' or Status = 'MyFirstModule.Status.Pending']
```

## Nested Constraints

```xpath
[MyFirstModule.Order_Customer/MyFirstModule.Customer[Status = 'MyFirstModule.Status.Active']/ID != empty]
```

## In Retrieve Actions

When setting XPath on a `RetrieveAction`, the constraint goes in the `xpathConstraint` property as a string:
```json
"xpathConstraint": "[Status = 'MyFirstModule.Status.Active']"
```

## In Access Rules

Access rules use XPath to restrict which objects a role can see:
```json
"xpathConstraint": "[MyFirstModule.Customer_Account = '[%CurrentUser%]']"
```
This ensures users can only see their own records.
