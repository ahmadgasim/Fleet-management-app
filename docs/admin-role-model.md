# Admin Role Model

This adds an Admins table and gates admin screens by membership.

Excel Table: Admins
- AdminId (text, unique) e.g. ADM-001
- StaffId (text, unique)
- Active (yes/no)

Power Apps gating
- Add to App.OnStart or Login flow:
```
Set(
    gIsAdmin,
    !IsBlank(LookUp(Admins, StaffId = gStaff.StaffId && Active = true))
);
```

Admin Home screen
- Visible property:
```
gIsAdmin
```

Admin tab buttons
- Disable if not admin:
```
DisplayMode = If(gIsAdmin, DisplayMode.Edit, DisplayMode.Disabled)
```

Admin data restrictions (optional)
- Block edits by non-admins:
```
If(!gIsAdmin, Notify("Admin access only", NotificationType.Error); Back())
```

Audit columns (optional)
- Add CreatedBy, CreatedTime, UpdatedBy, UpdatedTime columns to each table.
- Use Patch() to set them on create/update.