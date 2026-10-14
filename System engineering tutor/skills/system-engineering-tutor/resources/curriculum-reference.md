# Master Engineering Curriculum

## Section 1 — Backend Security

### 2. Authorization — High Level

Authentication answered "Who are you?" Authorization answers a completely different question: **"What are you allowed to do?"**

These are separate systems, and conflating them is one of the most common security mistakes. A request can be fully authenticated (the server knows exactly who's asking) and still be denied — because being *someone* doesn't mean you're allowed to do *everything*.

Authorization checks happen in layers, and a real system usually needs all four:

- **Route level** — can this user reach this endpoint at all?
- **Permission level** — can this user perform this specific action (read vs. write vs. delete)?
- **Resource ownership** — do they own *this particular record*, or just records of this type?
- **Field level** — of the data they can see, which specific fields are they allowed to see or edit?

Now let's walk through each piece.

---

#### 1. Role-Based Access Control (RBAC)

The most common starting point. Users get one or more **roles** (`admin`, `editor`, `viewer`), and roles map to a fixed set of permissions.

```typescript
// Permissions are just action strings, scoped to a resource
type Permission = "task:read" | "task:write" | "task:delete" | "user:manage";

const ROLE_PERMISSIONS: Record<string, Permission[]> = {
  admin:  ["task:read", "task:write", "task:delete", "user:manage"],
  editor: ["task:read", "task:write"],
  viewer: ["task:read"],
};

function hasPermission(role: string, permission: Permission): boolean {
  return ROLE_PERMISSIONS[role]?.includes(permission) ?? false;
}
```

The real power of RBAC in Express is a **middleware factory** — a function that returns a middleware, configured for whatever permission that route needs:

```typescript
function requirePermission(permission: Permission) {
  return (req: Request, res: Response, next: NextFunction) => {
    const userRole = req.user?.role; // set earlier by your auth middleware

    if (!userRole || !hasPermission(userRole, permission)) {
      return res.status(403).json({ error: "Forbidden" });
    }
    next();
  };
}

// Usage — reads like a sentence at the route definition
router.delete("/tasks/:id", requireAuth, requirePermission("task:delete"), deleteTask);
```

⚠️ **Key gotcha:** RBAC breaks down when permissions depend on *context*, not just role — e.g. "editors can edit tasks, but only tasks in their own department." That's where ABAC comes in.

---

#### 2. Attribute-Based Access Control (ABAC)

Instead of a fixed role → permission map, ABAC evaluates a **rule** using attributes of the user, the resource, and sometimes the environment (time of day, IP range, etc.).

```typescript
interface AuthContext {
  user: { id: string; department: string; role: string };
  resource: { ownerId: string; department: string };
}

function canEditTask(ctx: AuthContext): boolean {
  // Admins can edit anything
  if (ctx.user.role === "admin") return true;

  // Editors can edit tasks in their own department
  if (ctx.user.role === "editor" && ctx.user.department === ctx.resource.department) {
    return true;
  }

  return false;
}
```

RBAC vs. ABAC:

| | RBAC | ABAC |
|---|---|---|
| Rule source | Fixed role → permission table | Dynamic evaluation of attributes |
| Simplicity | Easy to reason about, easy to audit | More flexible, harder to trace |
| Good for | "Can admins do X" style questions | "Can *this* user do X *to this* resource" |
| Common failure | Doesn't scale to per-resource context | Rules sprawl and become hard to test |

Most real systems use RBAC for the coarse route-level gate, then layer resource-ownership or ABAC-style checks on top for anything context-dependent.

---

#### 3. Resource Ownership Checks

Role and permission checks answer "can this *type* of user do this *type* of action." They don't answer "does this user own *this specific row*." That's a separate, mandatory check.

```typescript
async function requireOwnership(req: Request, res: Response, next: NextFunction) {
  const task = await db.task.findUnique({ where: { id: req.params.id } });

  if (!task) {
    // See the IDOR note below for why this is 404, not 403
    return res.status(404).json({ error: "Not found" });
  }

  if (task.ownerId !== req.user!.id) {
    return res.status(404).json({ error: "Not found" });
  }

  req.task = task; // attach so the handler doesn't re-fetch
  next();
}
```

⚠️ **Key gotcha — IDOR (Insecure Direct Object Reference):** this is what happens when a system checks that a resource *exists* but forgets to check that the requesting user *owns it*. An attacker just increments an ID in the URL (`/tasks/41`, `/tasks/42`, `/tasks/43`...) and reads other people's data.

The fix isn't just "check ownership" — it's also **what status code you return when the check fails**. If a resource exists but isn't theirs, return `404`, not `403`. A `403` confirms to the attacker that the resource exists and they're just not allowed to see it — that's information leakage. A `404` tells them nothing. From the outside, "doesn't exist" and "exists but isn't yours" should look identical.

---

#### 4. Field-Level Authorization

Even after route, permission, and ownership checks pass, not every field on a record should be visible to every authorized viewer. A patient's own record might be fully visible to them, partially visible to a nurse, and fully visible to a doctor.

```typescript
function serializeTaskForUser(task: Task, viewerRole: string) {
  const base = { id: task.id, title: task.title, status: task.status };

  if (viewerRole === "admin" || viewerRole === "owner") {
    return { ...base, internalNotes: task.internalNotes, costEstimate: task.costEstimate };
  }

  return base; // sensitive fields stripped for everyone else
}
```

⚠️ **Key gotcha:** never rely on the frontend to hide sensitive fields — that's a UI decision, not a security boundary. If the API response includes the field, anyone with dev tools can read it regardless of what the UI renders. Strip it server-side, before it ever leaves the backend.
