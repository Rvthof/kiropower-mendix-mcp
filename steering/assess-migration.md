---
inclusion: manual
---

# Migration Assessment: Investigating Non-Mendix Projects

Use this when analyzing an existing non-Mendix application to produce a structured migration assessment.

## Investigation Process

### Step 1: Identify the Technology Stack

Examine build files, config files, framework indicators, database config, and frontend framework.

### Step 2: Map the Data Model

| Technology | Data Model Location |
|------------|-------------------|
| Java/Spring | `@Entity` classes, JPA annotations |
| .NET/EF | `DbContext`, entity classes, EF migrations |
| Django | `models.py` files |
| Rails | `app/models/`, `db/schema.rb` |
| Node.js | Sequelize/TypeORM/Prisma models |
| Database-first | SQL schema, stored procedures, views |

Document entities, attributes, associations, and enumerations with their Mendix mapping.

### Step 3: Catalog Business Logic

This is the most critical part. Identify:
- Validation rules → Mendix validation microflows
- Business rules → Mendix microflows
- Workflows/multi-step processes → Mendix Workflow or microflow chains
- Calculated fields → Mendix calculated attributes or microflows

### Step 4: Inventory Pages and UI

Document all screens, their purpose, data they display, and key actions.

### Step 5: Map Integrations

Document REST/SOAP clients, message queues, file imports/exports, email, external auth.

### Step 6: Document Security Model

Document authentication methods, user roles, and data access rules.

## Migration Phases

1. **Domain model** — Entities, associations, enumerations
2. **Core business logic** — Validation rules, business rules, calculations
3. **Pages** — Overview pages, edit forms, dashboards
4. **Integrations** — REST clients, file handling, external systems
5. **Security** — Roles, access rules, authentication
6. **Testing & cutover** — Data migration, parallel running, go-live

## Tips

- Be thorough with business logic — missing validation rules create hard-to-trace bugs
- Check the database — stored procedures, triggers, views often contain hidden business logic
- Document what you DON'T migrate — legacy reports, dead code, deprecated features
- Ask about undocumented behavior — users know about special cases not in the code
