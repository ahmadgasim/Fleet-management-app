# Excel Schema

Create a single Excel workbook and add tables with these exact names.

Table: Buildings
- BuildingId (text, unique) e.g. BLD-001
- Name (text)
- Zone (text)
- Active (yes/no)

Table: Staff
- StaffId (text, unique) e.g. ST-001
- FullName (text)
- Phone (text, unique)
- Email (text, optional)
- BuildingId (text, home building, optional)
- Active (yes/no)

Table: Drivers
- DriverId (text, unique) e.g. DR-001
- FullName (text)
- Phone (text, unique)
- Email (text, optional)
- VanId (text)
- Active (yes/no)
- Availability (choice: Available, Busy, Off)

Table: Vans
- VanId (text, unique) e.g. VAN-001
- PlateNumber (text)
- Capacity (number)
- Active (yes/no)

Table: RideRequests
- RequestId (text, unique) e.g. RR-000001
- RequestTime (datetime)
- StaffId (text)
- StaffPhone (text)
- PickupBuildingId (text)
- DestinationBuildingId (text)
- Notes (text, optional)
- Status (choice: Requested, Assigned, Enroute, Completed, Cancelled)
- AssignedDriverId (text, optional)
- AssignedVanId (text, optional)
- AssignedTime (datetime, optional)
- CompletedTime (datetime, optional)

Table: NotificationsLog
- LogId (text, unique)
- RequestId (text)
- DriverId (text)
- SentTime (datetime)
- Response (text: Accepted, Declined, Timeout)
- ResponseTime (datetime, optional)

Table: Admins (optional but recommended)
- AdminId (text, unique) e.g. ADM-001
- StaffId (text, unique)
- Active (yes/no)

Notes
- Use Data Validation in Excel for choice fields where possible.
- Keep all tables in a single workbook to simplify connectors.
- Power Apps can auto-generate RequestId; use a simple formula or flow.
