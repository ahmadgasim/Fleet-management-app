# Power Apps Formulas (Canvas App)

App assumptions
- Excel tables: Buildings, Staff, Drivers, Vans, RideRequests, NotificationsLog
- Global variables: gStaff, gRole, gDriver
- Admins table for gating admin access
- OnStart sets app defaults

App.OnStart
```
Set(gRole, "Guest");
Set(gStaff, Blank());
Set(gDriver, Blank());
Set(gIsAdmin, false);
```

Screen: Login
Controls
- txtPhone (TextInput)
- btnLookup (Button)
- lblError (Label)

btnLookup.OnSelect
```
Set(gStaff, LookUp(Staff, Phone = txtPhone.Text && Active = true));
If(
    !IsBlank(gStaff),
    Set(gRole, "Staff");
    Set(gIsAdmin, !IsBlank(LookUp(Admins, StaffId = gStaff.StaffId && Active = true)));
    Navigate(StaffHome, ScreenTransition.Fade),
    Navigate(StaffRegister, ScreenTransition.Fade)
);
```

Screen: StaffRegister
Controls
- txtName, txtPhone, txtEmail
- ddHomeBuilding (Dropdown) Items = Filter(Buildings, Active = true)
- btnRegister

btnRegister.OnSelect
```
Patch(
    Staff,
    Defaults(Staff),
    {
        StaffId: "ST-" & Text(Now(), "yyyymmddhhmmss"),
        FullName: txtName.Text,
        Phone: txtPhone.Text,
        Email: txtEmail.Text,
        BuildingId: ddHomeBuilding.Selected.BuildingId,
        Active: true
    }
);
Set(gStaff, LookUp(Staff, Phone = txtPhone.Text && Active = true));
Set(gRole, "Staff");
Set(gIsAdmin, !IsBlank(LookUp(Admins, StaffId = gStaff.StaffId && Active = true)));
Navigate(StaffHome, ScreenTransition.Fade);
```

Screen: StaffHome
Controls
- btnNewRequest
- galMyRequests (Gallery) Items = Filter(RideRequests, StaffId = gStaff.StaffId)

btnNewRequest.OnSelect
```
Navigate(NewRequest, ScreenTransition.Fade);
```

Screen: NewRequest
Controls
- ddPickup (Dropdown) Items = Filter(Buildings, Active = true)
- ddDestination (Dropdown) Items = Filter(Buildings, Active = true)
- txtNotes (TextInput)
- btnSubmit

btnSubmit.OnSelect
```
Set(
    varReqId,
    "RR-" & Text(Now(), "yyyymmddhhmmss")
);
Patch(
    RideRequests,
    Defaults(RideRequests),
    {
        RequestId: varReqId,
        RequestTime: Now(),
        StaffId: gStaff.StaffId,
        StaffPhone: gStaff.Phone,
        PickupBuildingId: ddPickup.Selected.BuildingId,
        DestinationBuildingId: ddDestination.Selected.BuildingId,
        Notes: txtNotes.Text,
        Status: "Requested"
    }
);
// Trigger Power Automate flow
NotifyDriversFlow.Run(
    varReqId,
    ddPickup.Selected.BuildingId,
    ddDestination.Selected.BuildingId,
    txtNotes.Text
);
Navigate(StaffHome, ScreenTransition.Fade);
```

Screen: DriverHome
Controls
- lblDriverName
- ddAvailability (Dropdown) Items = ["Available","Busy","Off"]
- galAssignedRequests Items = Filter(RideRequests, AssignedDriverId = gDriver.DriverId)

OnVisible
```
Set(gDriver, LookUp(Drivers, Phone = txtPhone.Text && Active = true));
```

ddAvailability.OnChange
```
Patch(
    Drivers,
    gDriver,
    { Availability: ddAvailability.Selected.Value }
);
```

Screen: RequestQueue
Controls
- galOpenRequests Items = Filter(RideRequests, Status = "Requested")
- btnAssignToMe (inside gallery)
- ddAssignDriver (admin-only dropdown) Items = Filter(Drivers, Active = true && Availability = "Available")
- btnAssignDriver (admin-only)

btnAssignToMe.OnSelect
```
If(
    ThisItem.Status = "Requested",
    Patch(
        RideRequests,
        ThisItem,
        {
            Status: "Assigned",
            AssignedDriverId: gDriver.DriverId,
            AssignedVanId: gDriver.VanId,
            AssignedTime: Now()
        }
    );
    Patch(Drivers, gDriver, { Availability: "Busy" })
);
```

btnAssignDriver.OnSelect
```
If(
    gIsAdmin && ThisItem.Status = "Requested",
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
    Patch(Drivers, LookUp(Drivers, DriverId = ddAssignDriver.Selected.DriverId), { Availability: "Busy" })
);
```

Screen: RequestDetail
Controls
- btnEnroute
- btnCompleted
- btnCancel

btnEnroute.OnSelect
```
Patch(
    RideRequests,
    ThisItem,
    { Status: "Enroute" }
);
```

btnCompleted.OnSelect
```
Patch(
    RideRequests,
    ThisItem,
    { Status: "Completed", CompletedTime: Now() }
);
```

btnCancel.OnSelect
```
Patch(
    RideRequests,
    ThisItem,
    { Status: "Cancelled" }
);
```

Screen: AdminHome
Controls
- Tabs for Buildings, Staff, Drivers, Vans, Requests
- Use EditForm + DataCards bound to each table

Admin gating
- AdminHome.Visible
```
gIsAdmin
```
- Any admin-only button DisplayMode
```
If(gIsAdmin, DisplayMode.Edit, DisplayMode.Disabled)
```
- Optional: block access on screen OnVisible
```
If(!gIsAdmin, Notify("Admin access only", NotificationType.Error); Back())
```

Admin visibility
- If you add an Admins table (StaffId, Active) use:
```
If(!IsBlank(LookUp(Admins, StaffId = gStaff.StaffId && Active = true)), true, false)
```
