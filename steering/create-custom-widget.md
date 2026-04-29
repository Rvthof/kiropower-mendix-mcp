---
inclusion: manual
---

# Creating Custom Pluggable Widgets

Use this when building a new Mendix pluggable widget using React and TypeScript.

## When to Use

Custom widgets are needed when Atlas UI and the built-in widget library can't cover a specific UX requirement — for example, a specialized chart, an embedded third-party component, or interactive behavior that microflows can't produce.

## Requirements

- Node.js 16+ and npm
- `@mendix/pluggable-widgets-tools` (installed via npm)
- Mendix Studio Pro (to deploy the `.mpk` file)

## Project Setup

```bash
npx @mendix/generator-widget MyWidgetName
cd my-widget-name
npm install
```

### Naming Conventions

| Thing | Convention | Example |
|-------|-----------|---------|
| npm package name | kebab-case | `my-widget-name` |
| Widget class name | PascalCase | `MyWidget` |
| Package path in `.mpk` | Reverse domain | `com.example.widgets` |

## Key Configuration Files

### `src/MyWidget.xml` — Property definitions

Defines what properties Studio Pro exposes for the widget. Supported types include:
- `string`, `boolean`, `integer` — primitive values
- `attribute` — bound to an entity attribute
- `datasource` — data source (microflow or XPath)
- `expression` — expression evaluated at runtime
- `action` — button/click action
- `widgets` — child widget slot

### `src/MyWidget.tsx` — Entry component

Maps Mendix-provided props to a pure React component:

```tsx
import { MyWidgetContainerProps } from "../typings/MyWidgetProps";

export function MyWidget({ label, value }: MyWidgetContainerProps) {
    return <MyWidgetDisplay label={label} value={value?.value} />;
}
```

### `src/components/MyWidgetDisplay.tsx` — Pure React component

No Mendix framework imports. Easier to test and reuse.

### CSS Isolation

Prefix all class names with the widget name to avoid style collisions:

```css
.my-widget { ... }
.my-widget__title { ... }
.my-widget--active { ... }
```

## Build and Deploy

```bash
npm run build          # produces dist/MyWidget.mpk
npm run dev            # watch mode for development
```

Copy the `.mpk` file to `<your-mendix-project>/widgets/` or install it via Studio Pro Marketplace. Redeploy the app to see changes.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `packagePath` in `package.json` doesn't match `xml` | Use the same reverse-domain path everywhere |
| Modifying code outside `BEGIN/END USER CODE` markers | Never — Studio Pro will overwrite it |
| Applying global CSS without widget prefix | Always prefix class names |
| Using Mendix runtime imports in the display component | Keep display components pure — no `mx` or `mendix` imports |

## Checklist

- [ ] `package.xml` path segments match `packagePath` in `package.json`
- [ ] All properties declared in `widget.xml` have generated TypeScript types
- [ ] CSS classes are prefixed with the widget name
- [ ] Build produces a `.mpk` file without errors
- [ ] Widget visible in Studio Pro toolbox after deployment
