# Project Specification: Workplace Support App (WSA) v1.3

**Version:** 1.3 (Reflecting discussions up to Oct 25, 2025 - Added AI grammar/spelling correction to reporting)

## 1. Overview

This document specifies a complete, free, self-hosted Workplace Support App (WSA) system. The system integrates Lone Worker Safety (LWS) features with a robust Visit Reporting module.

The system is comprised of four main components:

1. **The Worker App (PWA):** A mobile-first web app for workers, combining safety timers with offline-capable visit reporting.

2. **The Monitoring Dashboard:** A web-based dashboard for safety monitors to view worker status in real-time.

3. **The Google Apps Script Backend:** A Google Sheet-based backend that acts as the database and logic controller for both safety alerts and visit report data.

4. **The Deployment Tool (`setup.html`):** A web-based wizard that allows a "Deployment Administrator" to generate the customized Worker and Monitor apps.

This specification also assumes a "Setup Tool Administrator" (the project owner) is responsible for hosting the Deployment Tool and its templates (as described in a separate `Setup_Tool_Admin_Guide.md`).

## 2. Component 1: The Database (Google Sheet)

The system requires a single Google Sheet file containing two specific tabs (sheets).

### 2.1. `Visits` Sheet (Data Log)

* **Tab Name:** Must be exactly `Visits`.

* **Headers (A1:U1):**
  `Date,Worker Name,Worker Phone Number,Emergency Contact Name,Emergency Contact Number,Emergency Contact Email,Escalation Contact Name,Escalation Contact Number,Escalation Contact Email,Location Name,Location Address,Company Name,Arrival Time,Anticipated Departure Time,Actual Departure Time,Alarm Status,Visit Report Data,Notes,Last Known GPS,GPS Timestamp,Battery Level`

### 2.2. `Checklists` Sheet (Custom Form Database)

* **Tab Name:** Must be exactly `Checklists`.

* **Headers:** `A1`: `Company Name`, `B1`: `Question 1`, `C1`: `Question 2`, etc.

* **Syntax:**

  * **Header:** `"# Header Text"` (accepts with or without a space after `#`).

  * **Question:** `Did you check the...`

### 2.3. `Reports` / `Master Report` Sheets (Control Panels)

* The user must manually create tabs named `Reports` and `Master Report` to use the reporting functions.

* These sheets are control panels, with `B1` and `B2` used for input (Company Name, Month).

## 3. Component 2: The Backend (`apps_script_template.txt`)

This script is deployed as a Web App from the Google Sheet.

### 3.1. Configuration Variables

* `SECRET_KEY`: Placeholder.

* **`GEMINI_API_KEY`**: Placeholder. **(Optional)** If provided, enables AI grammar/spelling correction on reports.

* `SPREADSHEET_ID`: The ID of the Google Sheet file.

* `VISITS_SHEET_NAME`: `"Visits"`

* `CHECKLISTS_SHEET_NAME`: `"Checklists"`

* `REPORT_TAB_PREFIX`: `"Report` - "

### 3.2. `doGet(e)` Function (Handles GET requests)

* **Monitor Request (JSONP):** Validates `token` and returns all data from `Visits` sheet.

* **Worker App Request (JSON):**

  * **Trigger:** `action=getForms` and `companyName`.

  * **Security:** Keyless.

  * **Action:** Reads `Checklists` sheet. If `companyName` matches, it reads questions.

  * **Parsing:** Correctly parses `"# Header"` (with or without space) as `{"type": "header", ...}` and all other non-empty cells as `{"type": "checkbox", ...}`.

  * **Output:** Returns a standard JSON array of these objects.

### 3.3. `doPost(e)` Function (Handles POST requests)

* **Security:** Uses `LockService`.

* **Phone Number Fix:** Prepends a single quote (`'`) to phone numbers to preserve leading zeros.

* **Row Identification:** Finds row by `Worker Name` and `Arrival Time`.

* **Save/Update Logic:** Saves/updates all data, including `Company Name`, `Visit Report Data` (as JSON string), and parses/saves `Battery Level` from `Notes`.

* **Alert Trigger:** Calls `sendAlertEmail` for `PANIC`, `DURESS`, or `MISSED_CHECKIN`.

### 3.4. `checkOverdueWorkers()` Function (Time-Triggered)

* Runs on a 5-minute trigger.

* Ignores resolved statuses.

* Finds overdue workers and sends escalating alerts via `sendAlertEmail`.

### 3.5. `sendAlertEmail()` Function

* Sends formatted HTML emails.

* Strips leading quote (`'`) from phone numbers for the email body.

### 3.6. Reporting Functions

* **`generateMonthlyReport()`:** Helper function to generate a single-company report on the "Reports" tab.

* **`generateMasterMonthlyReport()`:**

  * Reads `Month (YYYY-MM)` from "Master Report" cell `B1`.

  * **Clears Old Reports:** Deletes all existing sheets that start with `REPORT_TAB_PREFIX`.

  * **Aggregates Data:** Gathers all `Visits` data for the specified month, aggregating it by `Company Name`.

  * **Caches Checklists:** Fetches all custom checklist definitions from the `Checklists` sheet and a hardcoded `STANDARD_CHECKLIST`.

  * **(Optional) AI Correction:** If `GEMINI_API_KEY` is provided, it sends the aggregated notes for each company to the Gemini API (`gemini-2.5-flash-preview-09-2025`) with a system prompt to correct spelling/grammar to "professional New Zealand English" and return a JSON array. If the call fails, it logs the error and proceeds with the original notes.

  * **Creates New Tabs:** For each company, it creates a new, formatted sheet (e.g., `"Report - Smith & Co."`).

  * **Outputs Data:** Prints the company's stats (Total Visits, Total Hours), Checklist Totals (in correct order with `0`s), and a detailed table of `Date`, `Worker`, `Note`, and `Duration (Mins)`, with a "Total Hours" sum at the bottom.

## 4. Component 3: The Worker App (`worker_app_template.txt`)

*(No changes from v1.2 / LWS v2.12)*

### 4.1. Core Features

* Location Management (with `Company Name` and `Use custom form` fields)

* Visit Timer ("Start" button, 1.5s hold)

* Locked Screen, Panic Button, PINs

* **Duress on Extend/Safe:** Duress PIN triggers silent alert but appears to work.

* "Are You OK?" Check-ins (Escalating, Confirmation vibration)

* GPS Tracking (Foreground dependent)

* Low Battery Reporting

* **"Depart & File Report" Workflow:**

  * Triggers `showReportModal()` on 1.5s hold.

  * `buildReportForm()` correctly renders headers and checkboxes from *either* `STANDARD_CHECKLIST` (object format) or cached custom form (object format).

  * `submitReport()` gathers *all* checkbox values (`true`/`false`) and notes into a JSON string.

* **Offline & Syncing Logic:**

  * `syncCustomForms()`: Proactively fetches and caches custom forms from backend.

  * `processUploadQueue()`: Sends any pending offline reports.

  * `sendToGoogleSheet()`: Queues "Departed" reports on network failure.

* Start Timer Reminder (5 min)

### 4.2. Technical Specifications

* PWA, Tailwind, Vanilla JS

* `Tone.js`, `navigator.vibrate`

* `localStorage` (for `state` object including `cachedForms` and `pendingUploads`)

* `fetch` POST (`no-cors`) for data, `fetch` GET (keyless) for forms.

* `manifest.json`, `sw.js`

* Long Press Buttons (1.5s no-PIN, 1s PIN)

## 5. Component 4: The Monitor App (`monitor_app_template.txt`)

*(No changes from v2.13 specification)*

## 6. Component 5: The Deployment Tool (`setup.html`)

*(No changes from v1.0 / LWS v2.8)*

### 6.1. Functionality

* **Wizard Interface:** 6 steps.

* **Template Fetching:** Fetches *code* templates (`.txt`, `.js`, `.json`) on load.

* **Step 2:** "Copy Headers" button copies the 21-column tab-delimited string. Includes instructions to create both `Visits` and `Checklists` tabs.

* **Step 4:** Includes all config inputs (`checkinInterval`, etc.).

* **Zip Generation:**

  * Injects config into `worker_app_template.txt`, `apps_script_template.txt`, and `manifest.json`.

  * Only injects `%%LOGO_URL%%` into `monitor_app_template.txt`.

  * Packages `LoneWorkerApp/`, `MonitoringDashboardApp/`, and `01-...txt` into `.zip`.

  * Documentation is NOT included in the zip.

* **Step 6:** Links to the external `LWS_Documentation.pdf`.

## 7. Documentation (Separate PDF)

* A comprehensive PDF document must be created and hosted by the Setup Tool Admin.

* The `setup.html` tool links to this PDF.

* The documentation (based on `Deployment_Admin_Guide.md`) must instruct the Deployment Admin on:

  * How to set up *both* the `Visits` (paste headers) and `Checklists` (add headers/questions) tabs.

  * How to host the apps (e.g., on GitHub Pages).

  * That their Monitor will need to log in once with the URL and Secret Key.

  * **(New)** How to optionally create and add a `GEMINI_API_KEY` to the `Code.gs` file to enable AI-powered report correction.

  * How to create and use the `Reports` and `Master Report` tabs and assign
