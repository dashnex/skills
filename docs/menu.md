# Menu

Modules can register menu items via config or via listeners on `system.menu_get`.

## Configuration

```ts
export interface MenuItem {
  name: string;
  icon?: React.ReactElement | React.ForwardRefExoticComponent<any>;
  path?: string;
  position?: string;
  priority?: number;
  items?: MenuItem[];
  onClick?: (item: MenuItem) => Promise<void>;
}

export interface AuthMenuItem extends MenuItem {
  roles?: string[];
  items?: AuthMenuItem[];
}
```

## Properties

- **name** — display text.
- **icon** — icon from `@dashnex/ui` or a custom React component.
- **path** — navigation target via `router.push(path)`.
- **position** — `"sidebar"` (left panel, 2 levels), `"dropdown"` (username menu, 1 level), or `"navbar"` (mobile top-right, 1 level).
- **priority** — higher values appear first. Items with different priorities are separated by a divider.
- **items** — child items (sidebar only).
- **roles** — controls visibility by user role.
- **onClick** — optional handler. If set and returns `false` with `path` set, navigation won't trigger.

## Example

```ts
// Via listener
export async function onMenuGet(context: MenuGetContext) {
  if (hasRole(context.user, "admin")) {
    context.items.push({
      name: "Contacts",
      position: "sidebar",
      path: "/contacts",
      priority: 50,
      icon: UsersIcon,
      roles: ["admin"],
    });
  }
}
```
