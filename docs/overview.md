# Overview

Goal
- Staff request a ride between buildings.
- Available drivers receive a mobile push notification.
- Driver accepts; request is assigned and tracked.
- Admin can manage buildings, staff, vans, drivers.

Roles
- Staff: create requests, view their own request status.
- Driver: receive/accept requests, update status.
- Admin: manage master data and view all requests.

Key Behavior
- Request flow: Staff selects Pickup Building, Destination Building, optional notes, submits.
- Dispatch: Notify available drivers via Power Automate mobile push with accept/decline actions.
- Assignment: First driver to accept is assigned; others are informed request is taken.
- Status: Requested -> Assigned -> Enroute -> Completed or Cancelled.

Assumptions
- Excel tables are stored in OneDrive for Business or SharePoint.
- Power Apps and Power Automate have access to the same Excel file.
- Staff and drivers are uniquely identified by phone number and/or email.
- Admin access is managed via an optional `Admins` table (recommended).

Next steps
- First-time setup: `docs/build-guide.md`
- App screens + controls: `docs/msapp-outline.md` and `docs/powerapps-screens.md`
- Copy/paste Power Fx: `docs/powerapps-formulas.md`
- Flows: `docs/powerautomate-flows.md` and `docs/powerautomate-steps.md`
