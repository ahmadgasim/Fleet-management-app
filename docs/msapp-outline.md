# Power Apps .msapp Starter Layout Outline

Note
- A true .msapp file must be exported from Power Apps Studio. This file provides a build outline you can follow to assemble an importable app quickly.

Suggested Canvas Layout
- App name: FleetMovement
- Screen order:
  1) Login
  2) StaffRegister
  3) StaffHome
  4) NewRequest
  5) DriverHome
  6) RequestQueue
  7) RequestDetail
  8) AdminHome

Control Map
Login
- txtPhone
- btnLookup
- lblError

StaffRegister
- txtName, txtPhone, txtEmail
- ddHomeBuilding
- btnRegister

StaffHome
- btnNewRequest
- galMyRequests

NewRequest
- ddPickup
- ddDestination
- txtNotes
- btnSubmit

DriverHome
- ddAvailability
- galAssignedRequests
- btnOpenQueue

RequestQueue
- galOpenRequests
- btnAssignToMe
- ddAssignDriver
- btnAssignDriver

RequestDetail
- lblPickup
- lblDestination
- btnEnroute
- btnCompleted
- btnCancel

AdminHome
- Tabs: Buildings, Staff, Drivers, Vans, Requests, Admins
- EditForm per table

How to produce the .msapp
1. Create a new Canvas app from blank.
2. Add the data connections to Excel tables in `fleet.xlsx`.
3. Create the screens and controls listed above.
4. Copy formulas from `docs/powerapps-formulas.md` and `docs/admin-role-model.md`.
5. File > Save > Export package to get .msapp.