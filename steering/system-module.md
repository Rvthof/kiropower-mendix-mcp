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
System.User → Administration.Account (adds FullName, Email, etc.)
System.FileDocument → System.Image, System.SynchronizationErrorFile
System.HttpMessage → System.HttpRequest, System.HttpResponse
```

## Common Patterns

**Audit Trail:** Add associations to `System.User` on your entities (e.g., `Order_CreatedBy`, `Order_ModifiedBy`).

**File Attachments:** Create an entity with `generalization: "System.FileDocument"` and associate it to the parent entity.

**XPath Tokens:**
- `[%CurrentUser%]` — The logged-in System.User
- `[%CurrentDateTime%]` — Current date and time
- `[%CurrentObject%]` — The current context object
