# TeamForge — Project Blueprint
### The reference spec you build against for the rest of the roadmap

This document is the single source of truth for what TeamForge *is*. Every module from here on (Database Engineering → Production API → Testing → Caching → DevOps → System Design) builds against this spec instead of you re-deciding the domain each time. When in doubt about "what should this feature do," the answer is in here.

---

## 1. Product Vision

TeamForge is a multi-tenant backend for team-based project and task management — the API layer behind a Jira/Linear/Asana-style product. Multiple organizations use the same deployment; each organization's data is completely isolated from every other's. Inside an organization, work is organized as **Teams → Projects → Tasks**, with commenting, role-based permissions, notifications, and an audit trail.

---

## 2. Domain Glossary

Precise definitions matter here — half of real-world backend bugs come from ambiguous domain terms.

| Term | Definition |
|---|---|
| **Organization** | The tenancy boundary. All data lives inside exactly one Organization. Nothing is ever shared across two Organizations. |
| **Team** | A subgroup within an Organization (e.g. "Backend," "Design"). Owns Projects. |
| **Project** | A body of work within a Team (e.g. "Q3 Mobile Redesign"). Owns Tasks. |
| **Task** | A unit of work within a Project. Has a status, priority, optional assignee, optional due date. |
| **Membership** | The record that says "this User belongs to this Organization/Team," carrying the permission role. |
| **Role** | Owner / Admin / Member / Viewer — determines what a User can do *within an Organization*. |

---

## 3. Entity Model

### 3.1 Core hierarchy (strict one-to-many — no orphans allowed)

```
Organization
    └── Team
          └── Project
                └── Task
                      ├── Comment
                      └── Attachment   (stretch goal)
```

A Task cannot exist without a Project. A Project cannot exist without a Team. A Team cannot exist without an Organization. This strictness is deliberate — it's what makes the Module 2 foreign-key/normalization work meaningful instead of arbitrary.

### 3.2 Entities and their key attributes

| Entity | Key attributes | Notes |
|---|---|---|
| **User** | id, email (globally unique), password_hash, name, created_at | Not scoped to one org — a User can belong to many Organizations |
| **Organization** | id, name, slug (unique), created_at | The tenancy boundary |
| **OrgMembership** | id, user_id, org_id, role, joined_at | Join table — this is where RBAC lives |
| **Team** | id, org_id, name, created_at | |
| **TeamMembership** | id, user_id, team_id, joined_at | Join table — determines who's assignable within that team |
| **Project** | id, team_id, name, description, status (active/archived), created_at | |
| **Task** | id, project_id, title, description, status, priority, assignee_id (nullable), created_by, due_date (nullable), created_at, updated_at | The busiest entity — most workflows touch this |
| **Comment** | id, task_id, author_id, body, created_at | body is parsed for @mentions |
| **Attachment** 🔹 | id, task_id, uploaded_by, file_url, file_name, created_at | Stretch goal — not MVP |
| **ActivityLog** | id, org_id, actor_id, action_type, target_type, target_id, metadata (jsonb), created_at | Append-only audit trail |
| **Notification** | id, recipient_id, type, payload (jsonb), read_at (nullable), created_at | Generated as a side-effect of other actions |
| **Invitation** | id, org_id, email, invited_by, role, token, status, created_at, expires_at | Handles inviting people who don't have accounts yet |

### 3.3 Cross-cutting relationships (the many-to-many and reference links)

| Relationship | Type | Carries |
|---|---|---|
| User ↔ Organization | many-to-many via OrgMembership | role |
| User ↔ Team | many-to-many via TeamMembership | — |
| Task → User (assignee) | many-to-one, nullable | must be an OrgMembership holder |
| Task → User (created_by) | many-to-one | |
| ActivityLog → Organization, User | many-to-one each | |
| Notification → User (recipient) | many-to-one | |
| Invitation → Organization | many-to-one | |

---

## 4. Roles & Permissions (RBAC Matrix)

| Action | Owner | Admin | Member | Viewer |
|---|:---:|:---:|:---:|:---:|
| Delete organization | ✅ | ❌ | ❌ | ❌ |
| Invite / remove members | ✅ | ✅ | ❌ | ❌ |
| Create / archive teams & projects | ✅ | ✅ | ❌ | ❌ |
| Create / edit tasks | ✅ | ✅ | ✅ | ❌ |
| Change status of own/assigned tasks | ✅ | ✅ | ✅ | ❌ |
| Comment | ✅ | ✅ | ✅ | ❌ |
| View everything in the org | ✅ | ✅ | ✅ | ✅ |

**Business rule:** a Task's `assignee_id` must reference a User who holds an active `OrgMembership` in that Task's Organization. This single rule is what your Module 3 authorization middleware and Module 4 "broken object level authorization" test will both be built around — it's your best interview story later, so build it deliberately.

---

## 5. Core Workflows

Each workflow below is a real user journey. Together they *are* your API surface — Module 3 is largely "turn each of these into endpoints."

### Workflow A — Organization Onboarding
1. User signs up → `User` created.
2. User creates an Organization → `Organization` created, User becomes **Owner** via `OrgMembership`.
3. Owner creates initial Teams.
4. Owner invites teammates by email → `Invitation` created, email queued (Module 3's background job work).
5. Invitee accepts → `User` created if new, `OrgMembership` created with the invited role, `ActivityLog` entry written.

### Workflow B — Project & Task Creation
1. Admin/Owner creates a Project under a Team.
2. Member creates a Task under a Project (default status: Todo, default priority: Medium).
3. Task is assigned to a User → **validated** against that user's `OrgMembership`.
4. Assignment triggers a `Notification` (in-app + queued email).

### Workflow C — Task Lifecycle (state machine)

```
        ┌────────┐   blocked   ┌─────────┐
        │  Todo  │────────────▶│ Blocked │
        └───┬────┘◀────────────└─────────┘
            │ start                 unblock
            ▼
     ┌───────────────┐  submit   ┌────────────┐  approve   ┌──────┐
     │  In Progress   │─────────▶│ In Review  │───────────▶│ Done │
     └───────┬────────┘          └─────┬──────┘             └──────┘
             ▲                          │ changes requested
             └──────────────────────────┘
```

- Only the assignee (or Admin/Owner) can transition a task's status.
- Every transition writes an `ActivityLog` entry.
- Transitioning **to Done** notifies the task's `created_by` if different from the assignee.

### Workflow D — Commenting & Mentions
1. User adds a `Comment` to a Task.
2. If the comment body contains `@username`, parse the mention and create a `Notification` for that user — **only if they're a member of the org** (never let a mention leak a notification to someone outside the tenant).

### Workflow E — Due-Date Reminders (background job)
1. A scheduled job (BullMQ repeatable job, runs daily) queries all Tasks with `due_date` within the next 24 hours and `status != Done`.
2. For each, create a `Notification` (type: `due_soon`) and queue a reminder email.
3. This is your concrete Module 3 "background jobs & queues" deliverable — it's not a toy example, it's a real feature.

### Workflow F — Dashboard / Analytics
1. Client requests org dashboard → aggregation query: task count grouped by status per project, overdue task count, etc.
2. This query is expensive at scale → in Module 5, this becomes your Redis cache-aside target, invalidated on every task write that touches the aggregate.

### Workflow G — Removing a Member (data-integrity edge case)
- When a member is removed from an Org, what happens to Tasks assigned to them?
- **Decision to make and document (do this in Module 2):** reassign to `null` (unassigned) rather than cascading a delete. This is exactly the kind of foreign-key `ON DELETE` decision interviewers ask about — you'll have a real answer because you made a real decision.

---

## 6. Constraints & Business Rules (explicit list)

- **Tenant isolation is absolute.** Every query must be scoped by `org_id`. A User must never be able to fetch data belonging to an org they're not a member of — this is OWASP API1 (Broken Object Level Authorization) and it is your single most important thing to get right and test.
- Email is globally unique per `User`, but one email can hold membership in multiple Organizations.
- Task → Project → Team → Organization is a strict, non-optional chain. No orphaned Tasks.
- Only Org members can be assigned Tasks in that Org.
- Organization deletion is Owner-only and should be a **soft delete / archive**, not an immediate hard delete — more realistic, and safer.
- Task lists must always be paginated — never return an unbounded list (this becomes literal in Module 3).
- Invite and comment endpoints are rate-limit candidates (spam prevention).

---

## 7. System Component Map

```
┌──────────┐      ┌────────────────────────────────────────────┐
│  Client  │─────▶│            Express API (TypeScript)          │
└──────────┘      │  Auth MW → RBAC MW → Validation MW (Zod)     │
                   │                    │                        │
                   │             Controller Layer                │
                   │                    │                        │
                   │              Service Layer                  │
                   └───────┬────────────────────────┬────────────┘
                           │                         │
                 ┌─────────▼─────────┐     ┌─────────▼─────────┐
                 │    PostgreSQL      │     │       Redis        │
                 │  (primary data)    │     │ (cache, rate-limit, │
                 │                    │     │   queue backing)    │
                 └────────────────────┘     └─────────┬───────────┘
                                                        │
                                              ┌─────────▼─────────┐
                                              │  BullMQ Workers    │
                                              │ (notifications,    │
                                              │  due-date digests) │
                                              └─────────┬──────────┘
                                                         │
                                              ┌─────────▼──────────┐
                                              │   Nodemailer/Email  │
                                              └──────────────────────┘
```

The `ActivityLog` write happens inside the Service Layer, on every mutating action — treat it as a cross-cutting concern (like logging), not a separate feature.

---

## 8. MVP vs. Stretch Scope

**MVP (build this — everything above assumes this scope):** User, Organization, OrgMembership, Team, TeamMembership, Project, Task, Comment, ActivityLog, Notification, Invitation. Workflows A–G as written.

**Stretch 🔹 (only if a module goes faster than expected — never a reason to fall behind):**
- Attachments (file upload to Task)
- Real-time updates via WebSockets/SSE
- Outbound Webhooks for external integrations
- Full-text search across tasks
- Configurable (non-default) workflow statuses

---

## 9. Module → TeamForge Mapping

So the roadmap and the project never drift apart:

| Module | What you build on TeamForge |
|---|---|
| 2 — Database Engineering | Implement this exact schema in PostgreSQL (normalized to 3NF). Your "atomic transaction" milestone = the org-deletion-with-task-reassignment logic from Workflow G. |
| 3 — Production API | Turn Workflows A–F into endpoints. RBAC middleware enforces Section 4's matrix. The due-date reminder (Workflow E) is your background-job deliverable. |
| 4 — Testing | Your highest-value test: prove tenant isolation (Section 6) actually holds — write a test where User A tries to fetch User B's org data and gets denied. |
| 5 — Caching | Cache Workflow F's dashboard aggregation with event-based invalidation on task writes. |
| 6 — DevOps | Containerize the whole stack in Section 7 (API + Postgres + Redis) via Docker Compose. |
| 7 — System Design | Your system design doc = "how does this exact architecture handle 100k orgs" — you're not designing something new, you're stress-testing what you already built. |

---

*Reference this document whenever a module's checklist says "build this" — it always means "build the relevant piece of this spec."*
