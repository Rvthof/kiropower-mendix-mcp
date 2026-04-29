# Navigation — Steering Guide

Use this guide when managing navigation profiles, menu items, and home pages.

## Reading Navigation

Navigation is stored as a document in the project. Find it:
```
ped_find_document(moduleName="MyFirstModule", documentType="Navigation$NavigationProfile")
```

Or search in the System module:
```
ped_find_document(moduleName="System", documentType="Navigation$NavigationDocument")
```

## Navigation Document Structure

The navigation document contains profiles (Web, Responsive, Phone, Tablet). Each profile has:
- `homePage` — the default page shown after login
- `menuItemCollection` — the navigation menu items

## Adding a Menu Item

Get schema:
```
ped_get_schema(elementTypes=["Navigation$MenuItem"])
```

Key properties:
- `caption` — display text (use `Texts$Text` element)
- `action` — the action when clicked (typically `Pages$PageClientAction` to open a page)
- `icon` — optional icon reference

## Setting the Home Page

Update the profile's `homePage` to reference a page:
```
ped_update_document(
  documentName="...",
  documentType="Navigation$NavigationDocument",
  operations=[{
    "path": "/profiles/0/homePage",
    "operation": { "type": "set", "value": { "$Type": "Navigation$PageSettings", "page": "MyFirstModule.Home_Web" } }
  }]
)
```

## Role-Based Navigation

Menu items can be restricted by role using `allowedRoles`. Only users with matching roles will see the item.

## After Every Change

```
ped_check_errors(documents=[{"documentType": "Navigation$NavigationDocument", "documentName": "..."}])
```
