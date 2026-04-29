# Domain Model — Steering Guide

Use this guide when creating or modifying entities, attributes, associations, and enumerations.

## Reading the Domain Model

```
ped_read_document(documentName="MyFirstModule", documentType="DomainModels$DomainModel")
```

The result contains:
- `entities[]` — list of entity stubs; re-read individual paths to expand
- `associations[]` — associations within the module
- `crossAssociations[]` — associations to entities in other modules

## Creating Entities

Always get the schema first:
```
ped_get_schema(elementTypes=["DomainModels$Entity"])
```

Then add via `ped_update_document` with an `add` operation on `/entities`.

### Attribute Types
`Binary`, `Boolean`, `DateTime`, `HashedString`, `String`, `Decimal`, `Integer`, `Long`, `AutoNumber`, `Enumeration`

For `Enumeration` type, set `enumerationName` to the fully qualified enum name (e.g., `"MyFirstModule.Status"`). The enumeration must already exist.

### Entity Locations
Set `location` as a point `{"x": 100, "y": 100}` to position entities on the canvas. Space entities ~200px apart to avoid overlap.

## Creating Enumerations

Check first:
```
ped_find_document(moduleName="MyFirstModule", documentType="Enumerations$Enumeration")
```

Get schema:
```
ped_get_schema(elementTypes=["Enumerations$Enumeration"])
```

Create:
```
ped_create_document(documentType="Enumerations$Enumeration", moduleName="MyFirstModule", documentName="Status", documentContent={...})
```

## Creating Associations

Get schema:
```
ped_get_schema(elementTypes=["DomainModels$Association"])
```

Add via `ped_update_document` on `/associations`. Key properties:
- `parent` — reference to the "one" side entity (by name, e.g., `"MyFirstModule.Customer"`)
- `child` — reference to the "many" side entity
- `type` — `"Reference"` (many-to-one) or `"ReferenceSet"` (many-to-many)
- `owner` — `"Default"` (child owns) or `"Both"` (both own, for ReferenceSet)

**Always add entities before associations** — associations reference entities by ID.

## User Associations

Never associate to `System.User`. Always use `Administration.Account`:
```json
{ "parent": "Administration.Account", "child": "MyFirstModule.MyEntity" }
```

## Validation Rules

Add validation rules after creating the entity. Get schema:
```
ped_get_schema(elementTypes=["DomainModels$ValidationRule"])
```

## After Every Change

```
ped_check_errors(documents=[{"documentType": "DomainModels$DomainModel", "documentName": "MyFirstModule"}])
```
