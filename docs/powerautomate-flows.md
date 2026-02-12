# Power Automate Flows

Notes
- Table names and status values are case-sensitive. Use: `Requested`, `Assigned`, `Enroute`, `Completed`, `Cancelled`.

Flow 1: Notify Available Drivers
Trigger
- Power Apps (V2) with inputs: RequestId, PickupBuildingId, DestinationBuildingId, Notes.

Steps
1. Get row from RideRequests by RequestId.
2. List rows from Drivers where Active = true and Availability = "Available".
3. For each driver:
- Send mobile notification (Power Automate mobile push)
- Title: "New ride request"
- Message: "Pickup: {PickupBuildingId} -> {DestinationBuildingId}"
- Actions: Accept, Decline
4. If Accept:
- Update RideRequests: Status = Assigned, AssignedDriverId, AssignedVanId, AssignedTime = utcNow()
- Update Drivers: Availability = Busy
- Log in NotificationsLog: Response = Accepted
- Send push to Staff: "Driver assigned"
5. If Decline or Timeout:
- Log response in NotificationsLog

Race Condition Handling
- Before assigning, check RideRequests Status is still Requested.
- If not Requested, respond "Already taken" and do not update.

Flow 2: Staff Push on Completion
Trigger
- When RideRequests row is modified and Status changes to Completed.

Steps
1. Get Staff by StaffId.
2. Send mobile push: "Ride completed. Thank you."

Flow 3: Admin Alert (Optional)
Trigger
- When RideRequests is Created.
Steps
- Send Teams or Email to admin group.
