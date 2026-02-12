# Power Apps Screens

App name: `FleetMovement`

These screen specs match:
- Screen order in `docs/msapp-outline.md`
- Copy/paste code in `docs/powerapps-formulas.md`
- Status values in `docs/excel-schema.md`

## Data connections
- Single Excel workbook with tables: `Buildings`, `Staff`, `Drivers`, `Vans`, `RideRequests`, `NotificationsLog`
- Optional (recommended): `Admins` table for admin access

## Security & roles (recommended)
- Role is derived at runtime:
  - Staff login sets `gblRole = "Staff"`
  - Driver login sets `gblRole = "Driver"`
  - Admin access sets `gblIsAdmin = true` when the logged-in staff member exists in `Admins`
- Gate admin UI with `gblIsAdmin` and avoid direct navigation to admin screens for non-admins.

## Screens (build in this order)

1. **Login**
   - Input: phone number
   - Staff login button: looks up `Staff` and navigates to `StaffRegister` if not found
   - Driver login button: looks up `Drivers` and navigates to `DriverHome` if found

2. **StaffRegister**
   - Inputs: full name, phone, email (optional), home building (optional)
   - Creates a new `Staff` row with `Active = true`

3. **StaffHome**
   - Shows the logged-in staff member’s request history
   - Button: **New Ride Request**

4. **NewRequest**
   - Select pickup + destination buildings (active only)
   - Submit creates a new `RideRequests` row with `Status = "Requested"`
   - Calls the dispatch flow `NotifyDriversFlow.Run(...)`

5. **DriverHome**
   - Shows driver availability (Available/Busy/Off) and assigned requests
   - Button: **Open Queue** (for manual assignment / admin view)

6. **RequestQueue** (Driver/Admin)
   - Gallery: open requests (`Status = "Requested"`)
   - Driver action: **Assign to me** (sets request to `Assigned`, sets driver to Busy)
   - Admin action: **Assign to driver** (pick an available driver)

7. **RequestDetail**
   - Shows request details (pickup/destination/notes/status)
   - Buttons: Enroute, Completed, Cancelled

8. **AdminHome**
   - Admin-only management UI
   - Tabs (example): Buildings, Staff, Drivers, Vans, Requests, Admins

## Data logic hints
- Use `Patch()` to insert/update records.
- Always filter lists by the logged-in user (`StaffId` / `DriverId`) unless admin.
- Status values are case-sensitive; use: `Requested`, `Assigned`, `Enroute`, `Completed`, `Cancelled`.
