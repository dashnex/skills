# Authentication & Authorization

`@dashnex/auth` provides access control for routes and pages.

## Server

- **Route Guards** — `AuthGuard` and `CheckRoleGuard`
- **`handleAuthenticatedRequest`** — handles routes with roles check and custom guards
- **`getUser(request)`** — get current user from request
- **`hasRole(user, role)`** — check if user has a specific role

## Client

- **`useAuth()`** — React hook for current user, login/logout, flags and actions
- **`RoleGuard`** — limit access to page components by role
- **`OAuthButton`** — universal OAuth provider button
- **`DashnexLoginButton`** — Login with DashNex button

## Role Hierarchy

`owner` > `admin` > `user` — each level inherits permissions from the level below. Avoid redundant role checks.
