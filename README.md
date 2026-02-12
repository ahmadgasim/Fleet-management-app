# Fleet-management-app

Fleet Movement App (Power Apps + Power Automate + Excel)

This folder contains the build spec for a Power Apps canvas app backed by Excel and Power Automate push notifications.

Contents
- docs/overview.md: App scope, roles, and behavior
- docs/excel-schema.md: Excel tables and columns
- docs/build-guide.md: Start-to-finish build steps for first-time setup
- docs/powerapps-screens.md: Screen-by-screen UI spec
- docs/powerapps-formulas.md: Copy/paste Power Fx code for each screen
- docs/powerautomate-flows.md: Flows for dispatch and admin alerts
- docs/testing-guide.md: Step-by-step setup and manual testing checklist
- templates/: CSV headers you can paste into Excel to create tables


## GitHub Project Docs
- [Build Guide](docs/build-guide.md)
- [Testing Guide](docs/testing-guide.md)

Quick Start
1. Follow docs/build-guide.md for full first-time setup.
2. Create an Excel file in OneDrive or SharePoint (required for Power Apps/Automate).
3. Create tables using templates in templates/ and name them exactly as specified.
4. Build the Power Apps canvas app following docs/powerapps-screens.md and docs/powerapps-formulas.md.
5. Create the flows in docs/powerautomate-flows.md and docs/powerautomate-steps.md.

Testing
- Follow docs/testing-guide.md for end-to-end smoke tests and UAT checks.
