# AI RECRUITER — SECURITY SPECIFICATION & THREAT MATRIX (PHASE 0)

**Project**: AI RECRUITER  
**Platform**: Google AI Studio Build — Web Full-Stack  
**Approved Blueprint**: P0-BLUEPRINT-v1.0-APPROVED-HARDENED  
**Threat Registry Version**: v1.3  
**Governance Version**: v1.3  

---

## 1. Data Invariants & Security Boundaries

1. **Deny-by-Default Architecture**: All client-side direct mutations to sensitive collections (`roles`, `permissions`, `role_permissions`, `user_roles`, `audit_logs`, `system_settings`) are strictly DENIED via Firestore Security Rules.
2. **Server-Side Authorization Boundary**: Privileged actions (User Creation, Role Assignment, Status Change, Organization Mutation, Settings Mutation) MUST pass server token verification and role/permission checks on Node.js server routes.
3. **Canonical 6 Roles**:
   - `SYSTEM_ADMIN`: Platform administration, system users, system settings, governance artifacts.
   - `HR_ADMIN`: HR operations, business users administration (cannot assign/grant `SYSTEM_ADMIN`).
   - `RECRUITER`: Recruitment workflow operations.
   - `HIRING_MANAGER`: Departmental hiring requests & team evaluation.
   - `INTERVIEWER`: Interview evaluations.
   - `VIEWER`: Read-only reporting access.
4. **Append-Only Audit Integrity**: Audit logs are generated strictly on the server and appended. Client write, update, and delete operations on `audit_logs` return `PERMISSION_DENIED`.
5. **SYSTEM_ADMIN Governance & Maker-Checker**:
   - Initial `SYSTEM_ADMIN` is bootstrapped via trusted server setup.
   - No user can self-promote or assign themselves roles.
   - `HR_ADMIN` cannot assign `SYSTEM_ADMIN`.
   - Granting `SYSTEM_ADMIN` requires an active `SYSTEM_ADMIN` actor, target != actor, audit reason, and maker/reviewer metadata if multiple admins exist.
6. **Public Signup = OFF**: Unauthenticated users cannot register or bootstrap profiles directly. Unknown Google Auth users are denied application entry without an active profile and role assignment.

---

## 2. Dirty Dozen Adversarial Payload Test Plan

| ID | Attack Vector | Payload / Action | Expected Result | Mitigation |
|---|---|---|---|---|
| ADV-01 | Public Signup Escalation | Anonymous `POST /api/auth/register` or direct Firestore `setDoc(profiles)` with `role="SYSTEM_ADMIN"` | DENIED | Public signup disabled; Firestore Rules reject write; server requires active admin token |
| ADV-02 | Self-Role Assignment | Viewer sends `POST /api/admin/user-roles` targeting own `user_id` with `role_key="HR_ADMIN"` | DENIED | Server verifies actor permission `users.roles.manage` and checks `actor_uid !== target_uid` |
| ADV-03 | HR_ADMIN Privilege Escalation | HR_ADMIN sends `POST /api/admin/user-roles` with `role_key="SYSTEM_ADMIN"` | DENIED | Server explicitly checks `if (role_key === 'SYSTEM_ADMIN' && !actor.isSystemAdmin) throw DENIED` |
| ADV-04 | Client Audit Log Tampering | Client executes `deleteDoc(doc(db, 'audit_logs', logId))` or `updateDoc(...)` | DENIED | Firestore Rules `match /audit_logs/{id} { allow write: if false; }` |
| ADV-05 | Direct URL Bypass | Unauthenticated user navigates directly to `/admin/users` or `/admin/audit-logs` | DENIED | ProtectedRoute redirects to `/login`; server API checks token + permission |
| ADV-06 | Mass Assignment Attack | User sends profile update containing `{ "status": "ACTIVE", "role": "SYSTEM_ADMIN", "unauthorized_field": "1" }` | DENIED | Server uses strict allowlist schema validation; Firestore rules block status/role key mutations |
| ADV-07 | IDOR / Object Poisoning | Viewer sends `PUT /api/organization/departments/dept-99` without `org.manage` permission | DENIED | Server checks `hasPermission(actor, 'org.manage')` |
| ADV-08 | Script Injection (XSS) | User submits Department Name `<script>alert("xss")</script>` | CLEAN / SANITIZED | Text input stored as sanitized text; React auto-escapes DOM rendering |
| ADV-09 | Secret Leak in Error Response | API error response includes `process.env.GEMINI_API_KEY` or stack trace | CLEAN | Error middleware strips internal details and secrets |
| ADV-10 | Private Storage Public URL Bypass | Client attempts direct public access to private candidate CV file | DENIED | Private storage enforces tokenized time-bound signed URLs; direct public access denied |
| ADV-11 | Audit Log Deletion Attempt | Authorized Admin sends `DELETE /api/admin/audit-logs/log-123` | DENIED | Server provides NO delete endpoint for `audit_logs`; collection is strictly append-only |
| ADV-12 | System Admin Self-Grant Duplicate | Single System Admin attempts to grant System Admin without audit reason | DENIED | Server requires mandatory `reason` string min 10 chars and logs governance review |

---

## 3. Attack Surface Inventory Mapping

- **AS-01 Login / Auth Token**: Protected by Firebase Auth + Server session token verification.
- **AS-02 Protected Routes**: Guarded by `ProtectedRoute` wrapper & client permission checks.
- **AS-03 Firestore Client SDK**: Deny-by-default rules applied in `firestore.rules`.
- **AS-04 Express Server API**: Token verify middleware + role/permission matrix evaluation.
- **AS-05 User Administration Endpoint**: Guarded by `users.manage` permission + HR_ADMIN restriction.
- **AS-06 Role Assignment Endpoint**: Dual actor check, self-assignment block, maker-checker rule.
- **AS-07 Organization CRUD Endpoints**: Guarded by `org.manage` permission + audit log trigger.
- **AS-08 System Settings Endpoint**: Guarded by `settings.manage` permission + audit log trigger.
- **AS-09 Audit Log Viewer**: Read-only endpoint guarded by `audit.read` permission.
- **AS-10 Private Storage Manager**: Time-bound token authorization endpoint.
