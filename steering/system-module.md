---
inclusion: manual
---

# Mendix System Module Reference

The `System` module is built into every Mendix app. Reference these entities when creating associations, extending for file/image storage, or working with workflows and HTTP services.

## User Management

### System.User

| Attribute | Type | Description |
|-----------|------|-------------|
| Name | String | Username (login name) |
| LastLogin | DateTime | Last successful login |
| Blocked | Boolean | Account is blocked |
| Active | Boolean | Account is active |
| IsAnonymous | Boolean | Anonymous user |

Key associations: `User_UserRoles` (→ System.UserRole), `User_Language`, `User_TimeZone`

**Common patterns:**
- Associate entities to `System.User` for audit trails (CreatedBy, ModifiedBy)
- Use `[%CurrentUser%]` token in XPath to filter by logged-in user

> ⚠️ **Prefer `Administration.Account` over `System.User` in microflows.** `System.User` is read-only and lacks `FullName`, `Email`, and writable associations. See below.

---

### Administration.Account (specialization of System.User)

`Administration.Account` is the **concrete, writable user entity** used in almost all application logic. It specializes `System.User`, meaning every `Administration.Account` IS a `System.User`, but with additional attributes and full read/write access.

| Attribute | Type | Description |
|-----------|------|-------------|
| FullName | String | Display name of the user |
| Email | String | Email address |
| (inherits) | — | All `System.User` attributes (Name, Active, Blocked, etc.) |

**Key differences from System.User:**

| | System.User | Administration.Account |
|---|---|---|
| Writable in microflows | ❌ No | ✅ Yes |
| Has FullName / Email | ❌ No | ✅ Yes |
| Can be associated to domain entities | ⚠️ Limited | ✅ Full bidirectional |
| Use in `$currentUser` context | ✅ (token type) | ✅ (cast or retrieve by association) |

**When to use `Administration.Account`:**
- Whenever you need to read `FullName` or `Email` (e.g., for notifications, audit log entries)
- Whenever you need to associate a user to a domain entity (e.g., `MaintenanceRequest_AssignedEngineer`)
- Whenever you need to create or update user objects
- Whenever you retrieve users filtered by module role

**Associations on Administration.Account are bidirectional.** Because it is a specialization, domain associations that point to `Administration.Account` can be traversed from both sides:
- From the request: `$MaintenanceRequest/ProductionFloorMonitor.MaintenanceRequest_AssignedEngineer` → gives the `Administration.Account`
- From the account: `$Account/ProductionFloorMonitor.MaintenanceRequest_AssignedEngineer_Administration.Account` → gives all requests assigned to that account

**Retrieving accounts by module role:**

There is no direct XPath attribute for module role on `Administration.Account`. The correct approach is to retrieve accounts via their `User_UserRoles` association and filter by the role name:

```
[System.User_UserRoles/System.UserRole/Name = 'ProductionFloorMonitor.Maintenance_Engineer']
```

Use this XPath on a retrieve of `Administration.Account` to get all users with a specific module role. The role name must be the **fully qualified** module role name (e.g., `ProductionFloorMonitor.Maintenance_Engineer`, not just `Maintenance_Engineer`).

**Getting `Administration.Account` from `$currentUser`:**

`$currentUser` is typed as `System.User`. To get the `Administration.Account` for the current user, retrieve by association:

```
Retrieve Administration.Account
XPath: [System.User_UserRoles = $currentUser] 
```

Or more directly, retrieve `Administration.Account` where `Name = $currentUser/Name` — but the cleanest approach is to use a retrieve with XPath `[id = $currentUser]` which works because Account specializes User and shares the same object ID.

### System.UserRole

| Attribute | Type | Description |
|-----------|------|-------------|
| Name | String | Role name as defined in the model |
| Description | String | Role description |

---

## File Management

### System.FileDocument

Base entity for all file storage. Specialize this to create custom file types.

| Attribute | Type | Description |
|-----------|------|-------------|
| Name | String | File name (including extension) |
| HasContents | Boolean | Whether file content has been uploaded |
| Size | Long | File size in bytes |
| DeleteAfterDownload | Boolean | Auto-delete after first download |

**Usage:** Create a specialization via `ped_update_document` on the domain model:
- Entity with `generalization: "System.FileDocument"`
- Add custom attributes like Description, Category

### System.Image

Extends `System.FileDocument` with image-specific features (thumbnail, caching).
Specialize for application images with `generalization: "System.Image"`.

---

## HTTP / Web Services

### System.HttpRequest (non-persistent)

| Attribute | Type | Description |
|-----------|------|-------------|
| Uri | String | Request URI |
| Content | String | Message body content |

### System.HttpResponse (non-persistent)

| Attribute | Type | Description |
|-----------|------|-------------|
| StatusCode | Integer | HTTP status code (200, 404, 500, etc.) |
| ReasonPhrase | String | Status reason phrase |
| Content | String | Response body |

### System.HttpHeader (non-persistent)

| Attribute | Type | Description |
|-----------|------|-------------|
| Key | String | Header name |
| Value | String | Header value |

---

## Workflow Engine

### System.Workflow

| Attribute | Type | Description |
|-----------|------|-------------|
| Name | String | Workflow instance name |
| StartTime | DateTime | When the workflow started |
| State | Enum (WorkflowState) | Current state |
| DueDate | DateTime | Workflow deadline |

**WorkflowState values:** `InProgress`, `Paused`, `Completed`, `Aborted`, `Failed`

### System.WorkflowUserTask

| Attribute | Type | Description |
|-----------|------|-------------|
| Name | String | Task name |
| DueDate | DateTime | Task deadline |
| State | Enum (WorkflowUserTaskState) | Task state |
| Outcome | String | Selected outcome |

Key associations: `WorkflowUserTask_TargetUsers`, `WorkflowUserTask_Assignees`, `WorkflowUserTask_Workflow`

---

## Inheritance Hierarchies

```
System.User → Administration.Account (adds FullName, Email; fully writable; use this in microflows)
System.FileDocument → System.Image, System.SynchronizationErrorFile
System.HttpMessage → System.HttpRequest, System.HttpResponse
```

## Common Patterns

**Audit Trail:** Add associations to `Administration.Account` (not `System.User`) on your entities so you can read `FullName` and write the association. Example: `Order_CreatedBy` → `Administration.Account`.

**Retrieving users by module role:** Use XPath on `Administration.Account`:
```
[System.User_UserRoles/System.UserRole/Name = 'ModuleName.RoleName']
```
The role name must be fully qualified with the module prefix.

**Getting current user as Administration.Account:** `$currentUser` is `System.User`. Retrieve `Administration.Account` with XPath `[id = $currentUser]` to get the writable Account object with `FullName` and `Email`.

**File Attachments:** Create an entity with `generalization: "System.FileDocument"` and associate it to the parent entity.

**XPath Tokens:**
- `[%CurrentUser%]` — The logged-in System.User (use `[id = $currentUser]` to retrieve the Administration.Account equivalent)
- `[%CurrentDateTime%]` — Current date and time
- `[%CurrentObject%]` — The current context object

## Lesson Learned: Administration.Account in Microflows

When a task says "retrieve accounts with module role X" or "associate an engineer to a request", always use `Administration.Account` as the entity type — not `System.User`. Associations defined in the domain model that reference users (e.g., `MaintenanceRequest_AssignedEngineer`) are typed to `Administration.Account`. Attempting to use `System.User` in these contexts causes type mismatch errors that are hard to diagnose.

The retrieve XPath for filtering by module role is:
```
[System.User_UserRoles/System.UserRole/Name = 'ProductionFloorMonitor.Maintenance_Engineer']
```

After retrieving the list of `Administration.Account` objects, iterate and count their associated active requests using a separate retrieve per account, then compare counts to find the minimum.
