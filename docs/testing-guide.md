# Testing Guide

This project is a specification for a **Power Apps + Power Automate + Excel** solution, so testing is done with a configured app/flow environment (not with `npm test` or `pytest`).

## 1) Test Environment Setup

1. Create one Excel workbook in **OneDrive for Business** or **SharePoint**.
2. Create the required tables with exact names and fields:
   - Buildings, Staff, Drivers, Vans, RideRequests, NotificationsLog.
3. Confirm Power Apps and Power Automate connect to the same workbook.
4. Import seed rows (at least):
   - 2 buildings (active)
   - 1 staff user (active)
   - 2 drivers (active; one set to `Available`)
   - 1 van (active)

## 2) Smoke Test Checklist

### A. Staff registration and login
1. Open the app and enter a phone number not yet in Staff.
2. Confirm registration screen is shown.
3. Register the user.
4. Confirm a new Staff row is created with `Active = true`.

**Expected:** User can proceed to Staff Home and appears in Staff table.

### B. Create ride request
1. From Staff Home, tap **New Ride Request**.
2. Select Pickup and Destination (both active buildings).
3. Add notes and submit.

**Expected:**
- A RideRequests row is created with `Status = Requested`.
- Notify Available Drivers flow is triggered.

### C. Driver dispatch and acceptance
1. Sign in as available driver (or use mobile push notification).
2. Accept the incoming request.

**Expected:**
- Request updates to `Status = Assigned`.
- `AssignedDriverId`, `AssignedVanId`, and `AssignedTime` are populated.
- Driver availability becomes `Busy`.
- Notification is logged in NotificationsLog with `Response = Accepted`.

### D. Race condition protection
1. Have two drivers attempt to accept the same `Requested` request.

**Expected:**
- Only first acceptance succeeds.
- Second driver receives “already taken” behavior.
- Request remains assigned to first driver only.

### E. Driver status progression
1. In Request Detail, set status to `Enroute`.
2. Then set status to `Completed`.

**Expected:**
- Request status transitions correctly.
- Completion timestamp is captured.
- Staff completion push notification is sent.

### F. Admin CRUD checks
1. Open Admin Home and update one record each in Buildings, Staff, Drivers, Vans.
2. Deactivate one building and confirm it is not selectable in new requests.

**Expected:** Admin updates persist and role-based visibility/filtering works.

## 3) Suggested UAT Test Cases

- Staff can only see their own request history.
- Drivers only see requests assigned to them.
- Cancel flow updates request to `Cancelled` and does not send completion push.
- If no drivers are `Available`, request remains `Requested` and logs timeouts/declines.
- Invalid phone number should not create duplicate Staff rows when an existing phone exists.

## 4) Debugging Tips

- If flows are not firing, verify trigger inputs from Power Apps (`RequestId`, `PickupBuildingId`, `DestinationBuildingId`, `Notes`).
- If push notifications are missing, verify driver accounts are signed in to Power Automate mobile app.
- If rows are not updating, verify Excel connector references the correct workbook and table names.
- Re-check status values exactly: `Requested`, `Assigned`, `Enroute`, `Completed`, `Cancelled`.

## 5) Definition of Done (Minimum)

A build is test-passed when all are true:
1. Staff can register/login and submit a request.
2. At least one available driver gets notified and can accept.
3. Request status goes `Requested -> Assigned -> Enroute -> Completed`.
4. Completion push reaches staff.
5. Admin can manage master data without breaking request creation.
