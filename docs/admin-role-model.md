# Admin Role Model

This adds an `Admins` table and gates admin-only screens/controls by membership.

## Excel table: Admins (recommended)
- AdminId (text, unique) e.g. ADM-001
- StaffId (text, unique)
- Active (yes/no)

If you don’t want a separate table, you can alternatively store an `IsAdmin` yes/no column on `Staff` — but the docs in this repo assume the **Admins table** approach.

## Power Apps gating (Power Fx)

These snippets match the variable naming convention in `docs/powerapps-formulas.md`.

### Set `gblIsAdmin` after staff login/registration
```powerfx
Set(
    gblIsAdmin,
    !IsBlank(
        LookUp(
            Admins,
            StaffId = gblCurrentStaff.StaffId && Active = true
        )
    )
);
```

Admin-only navigation (recommended)
- On the button/icon that navigates to `AdminHome`, set `Visible` to:
```powerfx
gblIsAdmin
```

- On `AdminHome.OnVisible`, block access for non-admins:
```powerfx
If(!gblIsAdmin, Notify("Admin access only", NotificationType.Error); Back())
```

Admin tab buttons
- Disable if not admin:
```powerfx
If(gblIsAdmin, DisplayMode.Edit, DisplayMode.Disabled)
```

Admin data restrictions (optional)
- Block edits by non-admins:
```powerfx
If(!gblIsAdmin, Notify("Admin access only", NotificationType.Error); Back())
```

Audit columns (optional)
- Add CreatedBy, CreatedTime, UpdatedBy, UpdatedTime columns to each table.
- Use Patch() to set them on create/update.
