# Build Guide (Power Apps + Power Automate + Excel)

If you're starting from zero, follow this guide in order.

## What you are building

- A Canvas Power App named **FleetMovement**.
- An Excel workbook as the data source.
- Power Automate flows for dispatch and notifications.

Core flow:
1. Staff creates ride request.
2. Available drivers are notified.
3. First driver to accept gets assigned.
4. Driver updates request to completed.

## Prerequisites

- Microsoft 365 account with access to:
  - Power Apps
  - Power Automate
  - OneDrive for Business or SharePoint
- Power Automate mobile app (for push notifications).

## Step 1: Create the Excel backend

1. In OneDrive for Business or SharePoint, create an Excel file named `fleet.xlsx`.
2. Create these tables in that workbook (exact names):
   - `Buildings`
   - `Staff`
   - `Drivers`
   - `Vans`
   - `RideRequests`
   - `NotificationsLog`
3. Use CSV headers in `templates/` to create each table quickly.
4. (Optional but recommended) Add an `Admins` table for admin role gating:
   - `AdminId`, `StaffId`, `Active`

Tip: Keep all tables in the **same workbook** to simplify Power Apps/Automate connectors.

## Step 2: Seed minimum data

Before building screens, insert sample rows:

- `Buildings`: at least 2 active buildings.
- `Vans`: at least 1 active van.
- `Drivers`: at least 2 active drivers, one with `Availability = Available`.
- `Staff`: at least 1 active staff user.
- `Admins` (optional): map one StaffId to admin access.

## Step 3: Create the Canvas app

1. Go to Power Apps -> **Create** -> **Canvas app from blank**.
2. Name the app `FleetMovement`.
3. Add Excel data connections to the workbook tables.
4. Add global app variables in `App.OnStart`:
   - `gRole`, `gStaff`, `gDriver`, `gIsAdmin`.

Use these screen specs and formulas while building:
- Screen layout and behavior: `docs/powerapps-screens.md`
- Copy/paste formulas: `docs/powerapps-formulas.md`

## Step 4: Build screens in this order

1. **Login/Identify**
   - Phone lookup in `Staff`.
   - Navigate to registration if not found.
2. **Staff Registration**
   - Create staff row with `Active = true`.
3. **Staff Home**
   - Show request history for logged-in staff.
4. **New Ride Request**
   - Patch `RideRequests` with `Status = Requested`.
   - Trigger flow `NotifyDriversFlow.Run(...)`.
5. **Driver Home**
   - Show assigned requests.
   - Update availability (`Available/Busy/Off`).
6. **Request Queue**
   - Driver `Assign to me`.
   - Admin `Assign to driver`.
7. **Request Detail**
   - Buttons: `Enroute`, `Completed`, `Cancel`.
8. **Admin Home**
   - CRUD for Buildings, Staff, Drivers, Vans, Requests.

## Step 5: Create Power Automate flows

Create flows in this order:

1. **Notify Available Drivers** (Power Apps trigger)
   - Inputs: `RequestId`, `PickupBuildingId`, `DestinationBuildingId`, `Notes`.
   - Notify available drivers.
   - On accept, re-check request is still `Requested` before assigning.
2. **Staff Push on Completion**
   - Trigger when `RideRequests` row is modified and `Status = Completed`.
3. **Admin Alert (optional)**
   - Trigger on request creation and send Teams/Email alert.

Use these docs while building:
- High-level flow logic: `docs/powerautomate-flows.md`
- Step-by-step actions/expressions: `docs/powerautomate-steps.md`

## Step 6: Wire app to flows

In Power Apps, add your dispatch flow to the app and call it when submitting a new request:

`NotifyDriversFlow.Run(varReqId, ddPickup.Selected.BuildingId, ddDestination.Selected.BuildingId, txtNotes.Text)`

Confirm the flow name in Power Apps exactly matches your created flow connection.

## Step 7: Add admin role gating (recommended)

If using the `Admins` table:

- Set `gIsAdmin` after login by checking `Admins` membership.
- Hide/disable admin-only UI when `gIsAdmin = false`.
- Optionally block direct navigation to admin screens.

Reference: `docs/admin-role-model.md`.

## Step 8: End-to-end validation

After building, run the manual checklist in `docs/testing-guide.md`.

Minimum success path:
1. Staff submits request.
2. Driver receives push and accepts.
3. Request updates to Assigned/Enroute/Completed.
4. Staff receives completion push.

## Common build blockers

- Table name mismatch between Excel and app/flow connectors.
- Using personal OneDrive instead of OneDrive for Business/SharePoint.
- Missing phone number matches for Staff/Drivers.
- Driver not marked `Active = true` and `Availability = Available`.
- Race condition not handled in flow (must re-check `Status = Requested`).
