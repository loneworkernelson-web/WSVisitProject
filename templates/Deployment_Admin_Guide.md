
# Deployment Administrator Guide: Workplace Support App

Welcome! This guide explains how to use the online Setup Tool to create your customized Workplace Support App (WSA) and deploy them using GitHub Pages.

**Goal:** To generate and host your own private versions of the Worker App and Monitoring Dashboard.

**Estimated Time:** 20-30 minutes.

---

## **Part 1: Use the Online Setup Tool**

Your system administrator has provided a link to the WSA Setup Tool website.

1.  **Open the Setup Tool Link** in your web browser. You should see the "Welcome to the WSA Setup Wizard".
2.  Follow the steps presented in the tool:

    ### **Step 1: Welcome**
    * Read the introduction.
    * Click **"Start Setup"**.

    ### **Step 2: Sheet Setup**
    * **Create Sheet:** Click the **`sheets.new`** link to open a blank Google Sheet. Give it a name (e.g., "My Team WSA Data").
    * **Set Up `Visits` Sheet:** The first tab (default "Sheet1") will be your `Visits` log. **Rename this tab to `Visits`**.
    * Click **once** on cell **A1** in the `Visits` sheet.
    * Click the **"Copy Headers"** button in the setup tool.
    * **Paste** the copied text. The 21 headers should automatically fill row 1 (A1 to U1).
    * **Create `Checklists` Sheet:** Click the **`+`** icon at the bottom of the sheet to add a new tab. **Rename this new tab to `Checklists`** (must be exact).
    * In cell **A1** of the `Checklists` sheet, type the header `Company Name`.
    * In **B1**, type `Question 1`. In **C1**, type `Question 2`, and so on, for any custom questions you want to define for specific companies.
    * Click **"Next Step"** in the setup tool.

    ### **Step 3: Deploy Script**
    * **Open Script Editor:** In your Google Sheet, click **`Extensions` > `Apps Script`**. Delete any existing code in `Code.gs`.
    * **Create Secret Key:** In the setup tool, type a **strong, private password** into the "Create a Secret Key" box. **Remember this key!**
    * **Copy Script:** Click the **"Copy Script"** button in the setup tool.
    * **Paste Script:** In the Apps Script editor, paste the copied code.
    * **Deploy:** Click **`Deploy` > `New deployment`**. Select type **`Web app`**, Execute as **`Me`**, Who has access **`Anyone`**. Click **`Deploy`**.
    * **Authorize:** Allow the script access to your Google Account (click "Advanced", "Go to...", "Allow").
    * **Copy URL:** After deployment, **copy the `Web app URL`**.
    * **Set Trigger:** In Apps Script, click "Triggers" (alarm clock icon), click **`Add Trigger`**, set it to run **`checkOverdueWorkers`** on a **`Time-driven`** trigger, **`Minutes timer`**, **`Every 5 minutes`**. Click **`Save`**.
    * Click **"Next Step"** in the setup tool.

    ### **Step 4: Configure Apps**
    * **Web App URL:** Paste the **`Web app URL`** you copied.
    * **Secret Key:** Verify this matches the key you created.
    * **Configure:** Set the First Alert Time, Check-in options, and optional Logo URL.
    * **Test:** Click **"Test Connection & Proceed"**. If successful, you'll move to the last step.

    ### **Step 5: Download**
    * Click the **"Generate & Download App Package (.zip)"** button.
    * Save the `LWS_App_Package.zip` file to your computer.

    ### **Step 6: Finish**
    * Click the link to download the **Full Documentation PDF** for your records.

---

## **Part 2: Host Your Generated Apps using GitHub Pages**

Now you'll upload the apps from the `.zip` file to your own GitHub repository to make them live.

### **2.1 Prepare Your Files**

* **Unzip:** Find the `LWS_App_Package.zip` file you downloaded and unzip it.
* **Locate Folders:** Inside, find the `LoneWorkerApp` folder and the `MonitoringDashboardApp` folder.

### **2.2 Create a GitHub Account (If you don't have one)**

* Go to [https://github.com/](https://github.com/).
* Click **"Sign up"** and create a free account. Verify your email.

### **2.3 Create a New Repository**

* Log in to GitHub. Click **"+"** > **"New repository"**.
* **Repository name:** Choose a name, e.g., `my-wsa-apps`.
* Select **"Public"**.
* Click **"Create repository"**.

### **2.4 Upload App Folders**

* On your new repository page, click **"Add file"** > **"Upload files"**.
* **Important:** Drag the **`LoneWorkerApp` folder** AND the **`MonitoringDashboardApp` folder** from your unzipped package onto the GitHub upload area.
* Scroll down and click **"Commit changes"**.

### **2.5 Enable GitHub Pages**

* Go to your repository's **"Settings"** tab.
* Click **"Pages"** in the left sidebar.
* Under "Build and deployment", set "Source" to **"Deploy from a branch"**.
* Under "Branch", select `main`, folder `/root`, and click **"Save"**.
* **Wait:** GitHub will provide a public URL (e.g., `https://your-username.github.io/my-wsa-apps/`). It may take a minute or two.

### **2.6 Get Your App URLs**

Your apps are now live! Your final URLs are:

* **Worker App URL:** `[Your GitHub Pages URL]` + `/LoneWorkerApp/`
    * e.g., `https://your-username.github.io/my-wsa-apps/LoneWorkerApp/`
* **Monitor App URL:** `[Your GitHub Pages URL]` + `/MonitoringDashboardApp/`
    * e.g., `https://your-username.github.io/my-wsa-apps/MonitoringDashboardApp/`

Save these two URLs.

---

## **Part 3: Distribute to Your Team**

### **For Your Safety Monitor:**

* Send them the **Monitoring App URL**.
* **IMPORTANT:** On their first visit, they must enter the **`Web app URL`** and **`Secret Key`** from Part 1, Step 3. This is a one-time setup.

### **For Your Lone Workers:**

* Send them the **Worker App URL**.
* **Instruct them to:**
    1. Open the link on their smartphone.
    2. Use their browser's menu to **"Add to Home Screen"** or **"Install app"**.
    3. Open the app from their home screen.
    4. Go to **Settings** (gear icon) and fill in **all fields** (Name, Phone, Contacts, PINs). The URL is pre-filled.
    5. Go to the **Main Screen** and add their visit locations, making sure to assign the correct **Company Name** and check the **"Use custom form"** box if needed.
    6. Tap **"Save Settings"**.

---

**Setup Complete!** Your system is operational.