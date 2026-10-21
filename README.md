# EECE 503M - Lab 3 - Broken Access Control and Command Injection

**Mohammad Kourani - American University of Beirut**

This repository contains the remediated code for Lab 3. All vulnerabilities live
in `staff.py` (the application's staff views), included here at the repository root.

## Fixes applied

1. **IDOR / broken object-level access control (A1 and the finale).**
   Added an access check (`ticket_perm_check`) to both `view_ticket` and
   `export_ticket`, so an agent can only view or export a ticket that belongs to a
   queue they are allowed. A support agent requesting a Billing ticket now gets a
   403 instead of the record.

2. **Vertical privilege escalation (A2).**
   `promote_user` now requires administrator rights (`@helpdesk_superuser_required`)
   instead of merely being staff, so a plain agent can no longer grant superuser.

3. **Cross-site request forgery (A3).**
   Removed `@csrf_exempt` from `change_email` so Django's CSRF token protection
   applies again. A state-changing POST with no CSRF token is now rejected with 403.

4. **OS command injection (Part B).**
   The ticket export no longer builds a shell command from the user-supplied label.
   The label is restricted to a safe character set and the file is written with plain
   Python (no `subprocess`, no shell), so the label can never be parsed and executed
   as a command.

## Verification

After the fixes, every attack from the lab fails:
- `/tickets/4/` returns 403 for the support agent.
- `/admin-tools/promote/` refuses a non-admin.
- the tokenless POST to `/profile/email/` returns 403.
- `/tickets/<id>/export/?label=x; id #` returns the normal export text, never `uid=`.
