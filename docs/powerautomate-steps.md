# Power Automate Step-by-Step (with expressions)

Flow 1: Notify Available Drivers
Trigger
- Power Apps (V2)
- Inputs: RequestId (text), PickupBuildingId (text), DestinationBuildingId (text), Notes (text)

Steps
1. Initialize variable `vRequestId` (String) = triggerBody()['RequestId']
2. Get a row (Excel Online (Business))
   - Location: OneDrive for Business / SharePoint
   - File: fleet.xlsx
   - Table: RideRequests
   - Key Column: RequestId
   - Key Value: `vRequestId`
3. List rows present in a table (Excel Online (Business))
   - Table: Drivers
   - Filter Query: `Active eq true and Availability eq 'Available'`
4. Apply to each (Drivers)
   - Compose `vDriverId` = items('Apply_to_each')?['DriverId']
   - Send mobile notification (Power Automate Notifications)
     - Title: New ride request
     - Message: Pickup: @{triggerBody()['PickupBuildingId']} -> @{triggerBody()['DestinationBuildingId']}
     - Actions: Accept, Decline
   - Condition: if notification response = Accept
     - Get row (RideRequests) by RequestId (same as step 2)
     - Condition: Status = Requested
       - Update row (RideRequests)
         - Status: Assigned
         - AssignedDriverId: vDriverId
         - AssignedVanId: items('Apply_to_each')?['VanId']
         - AssignedTime: utcNow()
       - Update row (Drivers)
         - DriverId: vDriverId
         - Availability: Busy
       - Add a row (NotificationsLog)
         - LogId: concat('LOG-', formatDateTime(utcNow(),'yyyyMMddHHmmss'))
         - RequestId: vRequestId
         - DriverId: vDriverId
         - SentTime: utcNow()
         - Response: Accepted
         - ResponseTime: utcNow()
      - Send mobile notification to staff
        - Message: Driver assigned for request @{vRequestId}
     - Else (Status not Requested)
       - Add a row (NotificationsLog) Response = Timeout
   - Else (Decline)
     - Add a row (NotificationsLog) Response = Declined

Staff notification for cancellation (driver or staff)
Option A: Add to Flow 1
- If driver declines and no drivers accept after loop, update RideRequests Status = Cancelled
- Send mobile notification to staff: "No drivers available. Request cancelled."

Option B: Separate flow (recommended for clarity)
Flow 2b: Staff Push on Cancellation
Trigger
- When a row is modified (Excel Online (Business)) on RideRequests
Steps
1. Condition: Status equals Cancelled
2. Get row (Staff) by StaffId
3. Send mobile notification: "Your ride request was cancelled."

Expression note
- Notification response output varies by connector. Use dynamic content from the send notification step.

Flow 2: Staff Push on Completion
Trigger
- When a row is modified (Excel Online (Business))
- Table: RideRequests

Steps
1. Condition: Status equals Completed
2. Get row (Staff) by StaffId
3. Send mobile notification: "Ride completed. Thank you."

Flow 3: Admin Alert (Optional)
Trigger
- When a row is added (RideRequests)
Steps
- Send Teams message or Email

Race condition guidance
- Always re-fetch RideRequests before assignment and check Status.
