# Mass Assignment Checklist

The endpoint accepts a JSON object and updates a model with it. The attacker includes fields the user shouldn't be able to set.

## Classic example

```
POST /api/users/profile
{ "displayName": "Alice", "is_admin": true }
```

If the controller does `user.update(request.body)`, Alice is now admin.

## Variations

- Setting `password_hash` directly to bypass password reset flow.
- Setting `tenant_id` or `org_id` to move a record across tenants.
- Setting `created_at` / `updated_at` to forge timestamps.
- Setting `email_verified` to skip verification.
- Setting `role` or `permissions`.
- Setting foreign keys: `owner_id`, `manager_id`, `customer_id`.
- Internal flags: `is_test_account`, `feature_flags`, `trial_extended`.

## Audit checklist

- Does any endpoint deserialize untrusted JSON directly into a model? Look for `update_attributes`, `assign`, `BodyParser` to model.
- For each accepting endpoint, is there an explicit allowlist of fields the user can set?
- Are sensitive fields marked at the model level (e.g., `attr_protected`, `attr_readonly`) AND at the controller level? Defense in depth.
- Are nested objects handled the same way? `user.address = request.body.address` carries the same risk one level down.
- For partial updates (PATCH): does the implementation iterate user-supplied keys and update them, or use a fixed schema?

## How to find these

- Search for: `assign(`, `update_attributes`, `update_columns`, `BodyToObject`, `MapTo`, `Object.assign`, `{...request.body}`, `**request.json`.
- Grep for ORM update calls that take a hash/dict directly from request body.
- Review every DTO; if there isn't a DTO, that's the finding.

## Tests

- For each editable resource: try setting fields that should be read-only (admin flags, ownership, timestamps).
- Use the API's own response body as a baseline of "fields this object has", then send all of them back in an update — see which got accepted.
- Test PATCH with extra keys not in the documented schema.

## Mitigations

- Explicit DTO / schema validation on every endpoint. Reject unknown fields.
- Allowlist fields for each endpoint, not blocklist.
- Database-level constraints where possible (e.g., trigger that prevents `is_admin` from being changed except by another admin).
- Field-level audit logging on sensitive flags.
