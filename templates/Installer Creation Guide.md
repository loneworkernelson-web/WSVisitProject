# Optional Guide: Creating a Desktop Installer for the Monitor App

This is an **optional but recommended** step for your safety monitor.

This guide will walk you through turning the **Monitoring Dashboard URL** into a standalone desktop application (like an `.exe` file on Windows or an `.app` file on Mac).

**Why do this?**
* It puts a dedicated "LWS Monitor" icon on the monitor's desktop.
* It runs in its own window, so it won't get lost in browser tabs.
* It feels more like a permanent, professional application.

We will use a free, trusted tool called `Nativefier` to do this. This process involves using the command line, but this guide will walk you through every single step, assuming no prior experience.

---

## **Part 1: Install the Necessary Tools**

Before you can build the app, you need to install a program called **Node.js**. Node.js is a very common, safe, and free software package that includes `npm` (Node Package Manager), a tool we need to install Nativefier.

1.  **Go to the Node.js Website:**
    * Open your web browser and go to: [https://nodejs.org/](https://nodejs.org/)
2.  **Download the Installer:**
    * You will see two buttons. Click the one for **"LTS"** (Long Term Support). This is the most stable version.
    * Download the installer for your operating system (Windows or macOS).
3.  **Run the Installer:**
    * Find the file you just downloaded (usually in your "Downloads" folder) and double-click it.
    * Follow the on-screen prompts. You can safely accept all the default settings. Just click "Next," "Agree," "Install," etc., until it says it has finished.

You have now installed the tools we need.

---

## **Part 2: Open Your Command Line Tool**

This is the part that looks like a "hacker" screen, but it's just a way to type commands directly to your computer.

* **On Windows (Command Prompt):**
    1.  Press the **Windows Key** on your keyboard (or click the Start button).
    2.  Type the letters: `cmd`
    3.  You will see **"Command Prompt"** appear in the list. Click on it.
    4.  A black window will open.

* **On macOS (Terminal):**
    1.  Click the **Spotlight icon** (the magnifying glass in the top-right corner of your screen).
    2.  Type: `Terminal`
    3.  You will see the **"Terminal.app"** icon. Click on it.
    4.  A white or black window will open.

You are now in your command-line tool.

---

## **Part 3: Install Nativefier**

Now, you'll tell your computer to download and install the Nativefier tool using the `npm` program that came with Node.js.

1.  **Type the Command:**
    * Carefully type (or copy and paste) the following command into the black/white window you just opened.
    * `npm install -g nativefier`
2.  **Press Enter:**
    * Hit the **Enter** key.
    * Your computer will connect to the internet and download the tool. You'll see text scrolling by. This may take a minute or two.
    * Once it's finished, it will return you to a new line, ready for your next command. (You can ignore any "warnings" (`WARN`) it might show).

You have now installed Nativefier. You only need to do this part (Parts 1 & 3) once.

---

## **Part 4: Build Your Desktop App**

Now, we'll tell Nativefier *where* to save your new app and *what* website to turn into an app.

### **4.1 Go to Your Desktop**

It's easiest to save the new app directly to your Desktop.

* In your command-line window (Command Prompt or Terminal), type the following command and press **Enter**:
    `cd Desktop`
* This command means "Change Directory" (or "move") into your Desktop folder.

### **4.2 Run the Nativefier Command**

This is the final command. It tells Nativefier all the details it needs.

* **Get Your URL:** First, find the **Monitoring App URL** you got from GitHub Pages in the main Deployment Guide (e.g., `https://your-username.github.io/my-wsa-apps/MonitoringDashboardApp/`).
* **Type the Command:** Carefully type (or copy and paste) the following command into your command-line window, including the quotation marks. **Replace the placeholder URL** with your actual Monitor App URL.

    `nativefier --name "LWS Monitor" --internal-urls ".*" "https://your-username.github.io/my-wsa-apps/MonitoringDashboardApp/"`

* **Press Enter:**
    * Hit the **Enter** key.
    * Nativefier will start working. It will download some things and build your app. This will take a few minutes.
    * When it's finished, you'll see a message like "App built to..."

### **4.3 Find Your App**

* Look on your computer's **Desktop**.
* You will find a new folder named `LWS Monitor-win32-x64` (on Windows) or `LWS Monitor-darwin-x64` (on Mac).
* Open this folder. Inside, you will find your new application:
    * **On Windows:** `LWS Monitor.exe` (with the app icon)
    * **On Mac:** `LWS Monitor.app` (with the app icon)

You can double-click this file to run your Monitoring Dashboard as a standalone desktop app!

---

## **Part 5: Share the App with Your Monitor**

To send this to your safety monitor, you can't just send the `.exe` or `.app` file; you must send the **entire folder** Nativefier created.

1.  Find the folder on your Desktop (e.g., `LWS Monitor-win32-x64`).
2.  **Right-click** on the folder.
3.  Select **"Send to" > "Compressed (zipped) folder"** (on Windows) or **"Compress..."** (on Mac).
4.  This will create a new `.zip` file (e.g., `LWS Monitor-win32-x64.zip`).
5.  You can now send this single `.zip` file to your monitor via email or a file-sharing service.

**Instructions for your Monitor:**
Tell them to download the `.zip` file, unzip it, open the folder, and find and run the `LWS Monitor` application inside.

### **Part 6: (Optional): How to Make the Monitor App Start Automatically**

To make the `LWS Monitor` app launch every time you log in to your computer, follow these steps.

#### **On Windows (using the Startup Folder):**

1.  First, find your `LWS Monitor.exe` application. It's inside the folder Nativefier created on your Desktop (e.g., `LWS Monitor-win32-x64`).
2.  **Right-click** on the `LWS Monitor.exe` file and select **"Create shortcut"**. This will make a new file called `LWS Monitor - Shortcut`.
3.  On your keyboard, press the **Windows Key + R** at the same time. This will open a small "Run" window.
4.  In the "Run" window, type `shell:startup` and press **Enter**.
5.  A new folder window named "Startup" will open.
6.  **Drag and drop** (or cut and paste) the `LWS Monitor - Shortcut` file you created in step 2 into this "Startup" folder.

That's it! The next time you log in to Windows, the monitor app will start automatically.

#### **On MacOS (using Login Items):**

1.  Open **System Settings** (from the Apple menu in the top-left corner). On older macOS versions, this is called **System Preferences**.
2.  Click on **"General"** in the sidebar, then click **"Login Items"**.
3.  You will see a list called "Open at Login". Click the **plus icon (`+`)** below this list.
4.  A Finder window will open. Navigate to your Desktop (or wherever you saved the app) and open the `LWS Monitor-darwin-x64` folder.
5.  Select the `LWS Monitor.app` file and click **"Open"** (or "Add").

The app is now added to the list and will launch automatically when you log in. You can also tick the **"Hide"** checkbox next to it in the list if you want it to start minimized.
