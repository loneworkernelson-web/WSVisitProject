# Project Specification: On-the-Go AppSuite v1.0

## 1. Overview

This document specifies a complete, free, self-hosted On-the-Go AppSuite (OTG AppSuite). The system integrates Lone Worker Safety (LWS) features with a robust Visit Reporting module.

The system is comprised of four main components:

1. **The Worker App (PWA):** A mobile-first web app for workers, combining safety timers with offline-capable visit reporting.

2. **The** Monitoring **Dashboard:** A web-based dashboard for safety monitors to view worker status in real-time.

3. **The Google Apps Script Backend:** A Google Sheet-based backend that acts as the database and logic controller for both safety alerts and visit report data.

4. **The Deployment Tool (`setup.html`):** A web-based wizard that allows a "Deployment Administrator" to generate the customized Worker and Monitor apps.

This specification also assumes a "Setup Tool Administrator" (the project owner) is responsible for hosting the Deployment Tool and its templates (as described in a separate `Setup_Tool_Admin_Guide.md`).

## 2. Component 1: The Database (Google Sheet)

The system requires a single Google Sheet file containing *at least* three tabs (sheets).

### 2.1. `Visits` Sheet (Data Log)

* **Tab Name:** Must be exactly `Visits`.

* **Headers:** The first row (Row 1) must contain these 21 headers in this exact order (A1 to U1):
  `Date`
  `Worker Name`
  `Worker Phone Number`
  `Emergency Contact Name`
  `Emergency Contact Number`
  `Emergency Contact Email`
  `Escalation Contact Name`
  `Escalation Contact Number`
  `Escalation Contact Email`
  `Location Name`
  `Location` Address
  `Company Name`
  `Arrival Time`
  `Anticipated Departure Time`
  `Actual Departure Time`
  `Alarm Status`
  `Visit Report Data`
  `Notes`
  `Last Known GPS`
  `GPS Timestamp`
  `Battery Level`

### 2.2. `Checklists` Sheet (Custom Form Database)

* **Tab Name:** Must be exactly `Checklists`.

* **Headers:** `A1`: `Company Name`, `B1`: `Question 1`, `C1`: `Question 2`, etc.

* **Syntax:**

  * **Header:** `"# Header Text"` (accepts with or without a space after `#`).

  * **Note Field:** `"% Note Field"` (accepts with or without a space after `%`).

  * **Question:** `Did you check the...`

* **New Required Entry:**

  * A row **must** be created with the `Company Name` set to **`(Standard)`**.

  * The questions in this row will serve as the default, standard checklist for all workers.

### 2.3. `Reports` / `Master Report` Sheets (Control Panels)

* The user must manually create tabs named `Reports` and `Master` Report to use the reporting functions.

* These sheets are control panels, with `B1` and `B2` used for input (Company Name, Month).

## 3. Component 2: The Backend (`apps_script_template.txt`)

This script is deployed as a Web App from the Google Sheet.

### 3.1. Configuration Variables

* `SECRET_KEY`: Placeholder.

* `GEMINI_API_KEY`: Placeholder (Optional).

* `SPREADSHEET_ID`: The ID of the Google Sheet file.

* `VISITS_SHEET_NAME`: `"Visits"`

* `CHECKLISTS_SHEET_NAME`: `"Checklists"`

* `REPORT_TAB_PREFIX`: `"Report - "`

### 3.2. `doGet(e)` Function (Handles GET requests)

This function must handle two distinct request types:

* **Monitor Request (JSONP):**

  * **Trigger:** Detects the presence of a `callback` parameter.

  * **Security:** Validates the `token` parameter against the `SECRET_KEY`. Returns "Access Denied" if invalid.

  * **Action:** Reads all data from the `Visits` sheet.

  * **Output:** Returns all data as a JSONP response. Date objects are serialized to ISO strings.

* **Worker App Request (JSON):**

  * **Trigger:** Detects an `action=getForms` parameter and a `companyName` parameter.

  * **Security:** This request is **keyless**.

  * **Action:** Reads `Checklists` sheet. If `companyName` matches (e.g., `(Standard)` or `Smith & Co.`), it reads questions.

  * **Parsing:** Correctly parses `"# Header"` (with/without space) as `{"type": "header", ...}`, `"% Note Field"` (with/without space) as `{"type": "textarea", ...}` and all other cells as `{"type": "checkbox", ...}`.

  * **Output:** Returns a standard JSON array of these objects.

### 3.3. `doPost(e)` Function (Handles POST requests)

* **Security:** Uses `LockService`.

* **Phone Number Fix:** Prepends a single quote (`'`) to phone numbers to preserve leading zeros.

* **Row Identification:** Finds row by `Worker Name` and `Arrival Time`.

* **Save/Update Logic:** Saves/updates all data for all 21 columns, including `Company Name`, `Visit Report Data` (as JSON string), and parses/saves `Battery Level` from `Notes`.

* **Alert Trigger:** Calls `sendAlertEmail` for `PANIC`, `DURESS`, or `MISSED_CHECKIN`.

### 3.4. `checkOverdueWorkers()` Function (Time-Triggered)

* Runs on a 5-minute trigger.

* Ignores resolved statuses.

* Finds overdue workers and sends escalating emails via `sendAlertEmail`.

### 3.5. `sendAlertEmail()` Function

* Sends formatted HTML emails.

* Strips leading quote (`'`) from phone numbers for the email body.

### 3.6. Reporting Functions

* **`generateMonthlyReport()`:** Helper function to generate a single-company report on the "Reports" tab.

* **`generateMasterMonthlyReport()`:**

  * Reads `Month (YYYY-MM)` from "Master Report" cell `B1`.

  * **Clears Old Reports:** Deletes all existing sheets that start with `REPORT_TAB_PREFIX`.

  * **Aggregates Data:** Gathers all `Visits` data for the specified month, aggregating it by `Company Name`.

  * **Caches Checklists:** Fetches all custom checklist definitions from the `Checklists` sheet and a hardcoded `STANDARD_CHECKLIST` (for fallback logic).

  * **(Optional) AI Correction:** If `GEMINI_API_KEY` is provided, it sends the aggregated notes (including custom notes) for each company to the Gemini API for correction (NZ English).

  * **Failure Warning:** If the AI call fails, it logs the error and places a prominent *warning* on the corresponding report tab.

  * **Creates New Tabs:** For each company, it **creates a new, formatted sheet** (e.g., `"Report - Smith & Co."`).

  * **Outputs Data:** Prints the company's stats (Total Visits, Total Hours), Checklist Totals (in correct order with `0`s), and a detailed table of `Date`, `Worker`, `Note` (with custom notes appended), and `Duration (Mins)`, with a "Total Hours" sum at the bottom.

## 4. Component 3: The Worker App (`worker_app_template.txt`)

### 4.1. `localStorage` State

* `settings`: Stores user-entered settings (Name, Phone, Contacts, PINs, URL).

* `locations`: An array of location objects. Each object must have: `id`, `name`, `address`, `companyName` (string), and `useCustomForm` (boolean).

* `activeVisit`: Stores details of the current visit.

* `cachedForms`: An object caching form objects (e.g., `{"(Standard)": [...], "Smith & Co.": [...]}`).

* `pendingUploads`: An array used as an offline queue for visit reports.

### 4.2. Core Features & Logic

* **`STANDARD_CHECKLIST_FALLBACK`:** A hardcoded array of question *objects* (e.g., `{"type":"header", ...}`) is included as a last-resort fallback.

* **Safety Features (LWS):**

  * Visit Timer ("Start" 1.5s, "Depart" 1.5s), Locked Screen, Panic (Triple-tap), PINs.

  * **Duress on Extend/Safe:** "Extend" (1s) and "I Am Safe" (1s) prompt for PIN. Duress PIN sends silent alert.

  * **Check-ins:** Optional (via `%%CHECKIN_ENABLED%%`), configurable (via `%%CHECKIN_INTERVAL%%`). Features escalating sound/vibration and a confirmation vibration.

  * **Reminder Banner:** Shows a "Remember to start..." banner if on main page for 5+ minutes.

* **Reporting Features (WSA):**

  * **Location Modal:** Includes text input for `Company Name` and a checkbox for `Use custom form`.

  * **"Depart" Workflow:** Triggers `showReportModal()` on 1.5s hold.

  * **`showReportModal()`:**

    * **Logic:**

      1. Check location's `useCustomForm` flag.

      2. If `true`, try to load `state.cachedForms[location.companyName]`.

      3. If `false` (or if custom form is not found), load `state.cachedForms["(Standard)"]`.

      4. If `(Standard)` is *also* not found (e.g., first-time offline), use `STANDARD_CHECKLIST_FALLBACK`.

      5. If no forms exist at all, show an error.

  * **`buildReportForm(items)`:** Dynamically builds the report modal's HTML. It must read `item.type` and render `<h3>` for "header", `<textarea class="wsa-custom-note">` for "textarea", and `<input type="checkbox">` for "checkbox".

  * **`submitReport()`:** Gathers all checkbox values (`true`/`false`), the main note (`#reportNotes`), and all custom textarea values (`.wsa-custom-note`), and saves them into a single `Visit Report Data` JSON string.

* **Offline & Syncing Logic:**

  * **`syncAllForms()`:** Runs on app load/navigate to main.

    1. *Always* calls `fetchForm("(Standard)")` to ensure the default form is up-to-date.

    2. Fetches any *other* custom forms that are not yet cached.

  * **`fetchForm(companyName)`:** Performs a **keyless** `GET` request to `[googleSheetUrl]?action=getForms&companyName=[name]` and saves the result to `state.cachedForms`.

* **`processUploadQueue()`:** Runs on app load/navigate to main. Sends any pending offline reports.

* **`sendToGoogleSheet()`:** Queues "Departed" reports on network failure.

* **Troubleshooting:** A "Clear Cached Forms & Data" button is included in the Settings page to clear `localStorage`.

### 4.3. Placeholders

* `%%FIRST_ALERT_MINUTES%%`, `%%CHECKIN_ENABLED%%`, `%%CHECKIN_INTERVAL%%`, `%%GOOGLE_SHEET_URL%%`

* `https://i.postimg.cc/dVmVg3Pn/favicon-logo-for-LWSApp.png` (Default logo URL)

## 5. Component 4: The Monitor App (`monitor_app_template.txt`)

* **Login:** Shows `setupPage` on first use, saves URL/Key to `localStorage`.

* **Header:** `%%LOGO_URL%%` placeholder, "Reset" button.

* **Dashboard:** Session Log, Worker Grid.

* **Card Features:** Pre-Alert Highlighting, Battery %, GPS Age, Stale Warning, Call Button.

* **Alerts:** Audible alarms (`Tone.js`), Desktop Notifications (`Notification` API).

## 6. Component 5: The Deployment Tool (`setup.html`)

A single HTML file to be hosted online.

* **Functionality:**

  * Loads all *code* templates (`.txt`, `.js`, `.json`) from a `./templates/` folder on `window.onload`.

  * **Step 2:** Provides a "Copy Headers" button that copies the **21-column, tab-delimited** header string. Includes instructions to create and name both the `Visits` and `Checklists` tabs, and to add the `(Standard)` row.

  * **Step 3:** Includes inputs for **`Secret Key`** and **`Gemini API Key (Optional)`**. The "Copy Script" button injects *both* keys into the `apps_script_template.txt`.

  * **Step 4:** Includes inputs for `firstAlert`, `enableCheckin`, `checkinInterval`, and `logoUrl`.

  * **`generateZip()` Function:**

    * Validates all inputs.

    * Replaces placeholders in `worker_app_template.txt` (all 4 `%%...%%` placeholders + logo).

    * Replaces placeholders in `apps_script_template.txt` (`YOUR_SECRET_KEY_HERE` and `YOUR_GEMINI_API_KEY_HERE`).

    * Replaces placeholders in `manifest.json` (logo URL, app names).

    * Replaces `%%LOGO_URL%%` in `monitor_app_template.txt`.

    * Packages the modified files into a `.zip` (`OTG_AppSuite_Package.zip`) containing `WorkerApp/`, `MonitorApp/`, and `01-PASTE_THIS_INTO_APPS_SCRIPT.txt`.

  * **Step 6:** Provides a hardcoded link to a separately hosted `OTG_AppSuite_Documentation.pdf`.

## 7. Documentation (Separate PDF & Source Files)

The Setup Tool Administrator is responsible for creating a single `OTG_AppSuite_Documentation.pdf` file. This file is linked to from `setup.html`. This PDF should be compiled from the following three source `.md` files (which are also kept in the `templates` folder):

* **`Deployment_Admin_Guide.md`:** The end-user guide. Explains how to *use* the hosted setup tool, set up the 2-tab Google Sheet (including the `(Standard)` row), deploy the script (and update it), and host the *generated* app folders on GitHub Pages. It must explain the Monitor App's one-time login.

* **`Installer Creation Guide.md`:** The verbose, beginner-friendly guide to using `Nativefier`.

* **`Promotional Blurb.md`:** A description of the OTG AppSuite, mentioning both safety and reporting features.

The final PDF must also be updated to instruct the Deployment Admin on:

1. How to get and add the *optional* `GEMINI_API_KEY` to the `Code.gs` file (either via Step 3 of the setup tool or manually) and the required steps to **enable the API and billing** in the Google Cloud Console.

2. How to create and use the `Reports` and `Master Report` tabs and assign the `generate...Report` scripts to buttons.

3. How to troubleshoot common issues (e.g., PWA install failures, Monitor app connection errors, and AI grammar checking issues.)
