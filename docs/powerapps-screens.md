# Power Apps Screens

App Name: FleetMovement

Data Connections
- Excel workbook with tables: Buildings, Staff, Drivers, Vans, RideRequests, NotificationsLog.

Security
- Simple role handling: Admin flag in Staff table or separate Admins table.
- Filter visibility by role in app logic.

Screens

1. Login or Identify
- Input: Phone number.
- Lookup Staff by Phone.
- If not found, show registration screen.

2. Staff Registration
- Inputs: FullName, Phone, Email (optional), Home Building (optional).
- Creates row in Staff table with Active = true.

3. Staff Home
- Button: New Ride Request.
- List: My recent requests with status.

4. New Ride Request
- Dropdown: PickupBuilding (Buildings where Active = true).
- Dropdown: DestinationBuilding (Buildings where Active = true).
- Text input: Notes.
- Submit button: Creates row in RideRequests with Status = Requested.
- On submit: Trigger Power Automate flow to notify drivers.

5. Driver Home
- Shows driver status (Available/Busy/Off).
- Toggle or dropdown to set Availability in Drivers table.
- List of assigned requests.

5a. Request Queue (Driver/Admin)
- Gallery: open requests where Status = "Requested"
- Button: "Assign to me" (driver)
- Admin-only button: "Assign to driver" (dropdown of available drivers)

6. Request Detail (Driver)
- Shows Request info.
- Buttons: Enroute, Completed, Cancel (update Status and time fields).

7. Admin Home
- Tabs: Buildings, Staff, Drivers, Vans, Requests.
- CRUD forms for each table.

Data Logic Hints
- Use Patch() to insert/update.
- Use Filter() with StaffId or DriverId to show relevant records.
- For RequestId, use: "RR-" & Text(Now(), "yyyymmddhhmmss")
