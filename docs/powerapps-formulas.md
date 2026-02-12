# Power Apps Screen Code (Power Fx)

Use these formulas as a copy/paste starter for each screen.

## Variable naming (single source of truth)

To avoid duplicate/unclear names, use this convention everywhere:

- `gbl*` = global app state (defined with `Set`)
- `ctx*` = screen-local context (defined with `UpdateContext`)

Recommended global variables:
- `gblRole`
- `gblCurrentStaff`
- `gblCurrentDriver`
- `gblIsAdmin`
- `gblCurrentRequest`
- `gblRequestId`

## App-level setup

### App.OnStart
```powerfx
Set(gblRole, "Guest");
Set(gblCurrentStaff, Blank());
Set(gblCurrentDriver, Blank());
Set(gblIsAdmin, false);
Set(gblCurrentRequest, Blank());
Set(gblRequestId, Blank());
```

### Optional: App.StartScreen
```powerfx
Login
```

## Shared helper snippets

### Generate IDs
```powerfx
// Request
"RR-" & Text(Now(), "yyyymmddhhmmss")

// Staff
"ST-" & Text(Now(), "yyyymmddhhmmss")

// Notifications log
"LOG-" & Text(Now(), "yyyymmddhhmmss")
```

---

## Screen: Login

### Controls
- `txtPhone` (TextInput)
- `btnStaffLogin` (Button)
- `btnDriverLogin` (Button)
- `lblError` (Label)

### `btnStaffLogin.OnSelect`
```powerfx
With(
    { phoneInput: Trim(txtPhone.Text) },
    Set(gblCurrentStaff, LookUp(Staff, Phone = phoneInput && Active = true));

    If(
        IsBlank(gblCurrentStaff),
        Navigate(StaffRegister, ScreenTransition.Fade),
        Set(gblRole, "Staff");
        Set(gblIsAdmin, !IsBlank(LookUp(Admins, StaffId = gblCurrentStaff.StaffId && Active = true)));
        Navigate(StaffHome, ScreenTransition.Fade)
    )
)
```

### `btnDriverLogin.OnSelect`
```powerfx
With(
    { phoneInput: Trim(txtPhone.Text) },
    Set(gblCurrentDriver, LookUp(Drivers, Phone = phoneInput && Active = true));

    If(
        IsBlank(gblCurrentDriver),
        Notify("Driver account not found or inactive.", NotificationType.Error),
        Set(gblRole, "Driver");
        Navigate(DriverHome, ScreenTransition.Fade)
    )
)
```

### `lblError.Text`
```powerfx
If(IsBlank(txtPhone.Text), "Enter your phone number", "")
```

---

## Screen: StaffRegister

### Key properties
- `ddHomeBuilding.Items`
```powerfx
Filter(Buildings, Active = true)
```

- `ddHomeBuilding.Value`
```powerfx
Name
```

### `btnRegister.OnSelect`
```powerfx
If(
    IsBlank(Trim(txtName.Text)) || IsBlank(Trim(txtPhone.Text)),
    Notify("Name and phone are required.", NotificationType.Error),
    If(
        !IsBlank(LookUp(Staff, Phone = Trim(txtPhone.Text))),
        Notify("This phone is already registered.", NotificationType.Warning),
        Patch(
            Staff,
            Defaults(Staff),
            {
                StaffId: "ST-" & Text(Now(), "yyyymmddhhmmss"),
                FullName: Trim(txtName.Text),
                Phone: Trim(txtPhone.Text),
                Email: Trim(txtEmail.Text),
                BuildingId: ddHomeBuilding.Selected.BuildingId,
                Active: true
            }
        );

        Set(gblCurrentStaff, LookUp(Staff, Phone = Trim(txtPhone.Text) && Active = true));
        Set(gblRole, "Staff");
        Set(gblIsAdmin, !IsBlank(LookUp(Admins, StaffId = gblCurrentStaff.StaffId && Active = true)));
        Notify("Registration successful", NotificationType.Success);
        Navigate(StaffHome, ScreenTransition.Fade)
    )
)
```

---

## Screen: StaffHome

### Key properties
- `galMyRequests.Items`
```powerfx
SortByColumns(
    Filter(RideRequests, StaffId = gblCurrentStaff.StaffId),
    "RequestTime",
    Descending
)
```

- `btnNewRequest.OnSelect`
```powerfx
Navigate(NewRequest, ScreenTransition.Fade)
```

- `galMyRequests.OnSelect`
```powerfx
Set(gblCurrentRequest, ThisItem);
Navigate(RequestDetail, ScreenTransition.Fade)
```

---

## Screen: NewRequest

### Key properties
- `ddPickup.Items`
```powerfx
Filter(Buildings, Active = true)
```

- `ddDestination.Items`
```powerfx
Filter(Buildings, Active = true)
```

### `btnSubmit.OnSelect`
```powerfx
If(
    IsBlank(ddPickup.Selected.BuildingId) || IsBlank(ddDestination.Selected.BuildingId),
    Notify("Select pickup and destination.", NotificationType.Error),

    Set(gblRequestId, "RR-" & Text(Now(), "yyyymmddhhmmss"));

    Patch(
        RideRequests,
        Defaults(RideRequests),
        {
            RequestId: gblRequestId,
            RequestTime: Now(),
            StaffId: gblCurrentStaff.StaffId,
            StaffPhone: gblCurrentStaff.Phone,
            PickupBuildingId: ddPickup.Selected.BuildingId,
            DestinationBuildingId: ddDestination.Selected.BuildingId,
            Notes: Trim(txtNotes.Text),
            Status: "Requested"
        }
    );

    // Flow must exist in app as NotifyDriversFlow
    NotifyDriversFlow.Run(
        gblRequestId,
        ddPickup.Selected.BuildingId,
        ddDestination.Selected.BuildingId,
        Trim(txtNotes.Text)
    );

    Notify("Ride request submitted", NotificationType.Success);
    Navigate(StaffHome, ScreenTransition.Fade)
)
```

---

## Screen: DriverHome

### Key properties
- `ddAvailability.Items`
```powerfx
["Available", "Busy", "Off"]
```

- `ddAvailability.Default`
```powerfx
gblCurrentDriver.Availability
```

- `ddAvailability.OnChange`
```powerfx
Patch(Drivers, gblCurrentDriver, { Availability: ddAvailability.Selected.Value });
Set(gblCurrentDriver, LookUp(Drivers, DriverId = gblCurrentDriver.DriverId));
```

- `galAssignedRequests.Items`
```powerfx
SortByColumns(
    Filter(RideRequests, AssignedDriverId = gblCurrentDriver.DriverId),
    "RequestTime",
    Descending
)
```

- `galAssignedRequests.OnSelect`
```powerfx
Set(gblCurrentRequest, ThisItem);
Navigate(RequestDetail, ScreenTransition.Fade)
```

- `btnOpenQueue.OnSelect`
```powerfx
Navigate(RequestQueue, ScreenTransition.Fade)
```

---

## Screen: RequestQueue (Driver/Admin)

### Key properties
- `galOpenRequests.Items`
```powerfx
SortByColumns(
    Filter(RideRequests, Status = "Requested"),
    "RequestTime",
    Ascending
)
```

- `ddAssignDriver.Items` (admin only)
```powerfx
Filter(Drivers, Active = true && Availability = "Available")
```

- `btnAssignToMe.Visible`
```powerfx
gblRole = "Driver"
```

- `btnAssignDriver.Visible`
```powerfx
gblIsAdmin
```

### `btnAssignToMe.OnSelect`
```powerfx
If(
    ThisItem.Status = "Requested" && !IsBlank(gblCurrentDriver),
    Patch(
        RideRequests,
        ThisItem,
        {
            Status: "Assigned",
            AssignedDriverId: gblCurrentDriver.DriverId,
            AssignedVanId: gblCurrentDriver.VanId,
            AssignedTime: Now()
        }
    );
    Patch(Drivers, gblCurrentDriver, { Availability: "Busy" });
    Notify("Request assigned to you", NotificationType.Success),
    Notify("Request already assigned", NotificationType.Warning)
)
```

### `btnAssignDriver.OnSelect` (admin)
```powerfx
If(
    gblIsAdmin && ThisItem.Status = "Requested" && !IsBlank(ddAssignDriver.Selected.DriverId),
    Patch(
        RideRequests,
        ThisItem,
        {
            Status: "Assigned",
            AssignedDriverId: ddAssignDriver.Selected.DriverId,
            AssignedVanId: ddAssignDriver.Selected.VanId,
            AssignedTime: Now()
        }
    );
    Patch(
        Drivers,
        LookUp(Drivers, DriverId = ddAssignDriver.Selected.DriverId),
        { Availability: "Busy" }
    );
    Notify("Driver assigned", NotificationType.Success)
)
```

---

## Screen: RequestDetail

### `RequestDetail.OnVisible`
```powerfx
If(IsBlank(gblCurrentRequest), Back())
```

### Labels
- `lblPickup.Text`
```powerfx
gblCurrentRequest.PickupBuildingId
```

- `lblDestination.Text`
```powerfx
gblCurrentRequest.DestinationBuildingId
```

### Buttons
- `btnEnroute.OnSelect`
```powerfx
Patch(RideRequests, gblCurrentRequest, { Status: "Enroute" });
Set(gblCurrentRequest, LookUp(RideRequests, RequestId = gblCurrentRequest.RequestId));
```

- `btnCompleted.OnSelect`
```powerfx
Patch(RideRequests, gblCurrentRequest, { Status: "Completed", CompletedTime: Now() });
If(!IsBlank(gblCurrentDriver), Patch(Drivers, gblCurrentDriver, { Availability: "Available" }));
Set(gblCurrentRequest, LookUp(RideRequests, RequestId = gblCurrentRequest.RequestId));
Notify("Request marked completed", NotificationType.Success)
```

- `btnCancel.OnSelect`
```powerfx
Patch(RideRequests, gblCurrentRequest, { Status: "Cancelled" });
If(!IsBlank(gblCurrentDriver), Patch(Drivers, gblCurrentDriver, { Availability: "Available" }));
Set(gblCurrentRequest, LookUp(RideRequests, RequestId = gblCurrentRequest.RequestId));
Notify("Request cancelled", NotificationType.Warning)
```

---

## Screen: AdminHome

### Access gating
- `AdminHome.Visible`
```powerfx
gblIsAdmin
```

- `AdminHome.OnVisible`
```powerfx
If(!gblIsAdmin, Notify("Admin access only", NotificationType.Error); Back())
```

### Admin-only buttons/forms
- `DisplayMode`
```powerfx
If(gblIsAdmin, DisplayMode.Edit, DisplayMode.Disabled)
```

---

## Logout buttons (recommended on StaffHome/DriverHome/AdminHome)

### `btnLogout.OnSelect`
```powerfx
Set(gblRole, "Guest");
Set(gblCurrentStaff, Blank());
Set(gblCurrentDriver, Blank());
Set(gblIsAdmin, false);
Set(gblCurrentRequest, Blank());
Set(gblRequestId, Blank());
Reset(txtPhone);
Navigate(Login, ScreenTransition.Fade)
```
