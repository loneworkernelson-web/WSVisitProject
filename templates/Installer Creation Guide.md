# Optional Guide: Creating a Desktop Installer

The Monitoring App is a web page, but it works best as a "standalone" app on your monitor's desktop. This prevents it from being lost in browser tabs and allows for better notifications.

The easiest way to do this is using a free tool called **Nativefier**. This tool wraps any website into a simple .exe (Windows) or .dmg (Mac) file.

**This is an optional, advanced step.**

### **Prerequisites**

You (the administrator) must have Node.js installed on your computer to use Nativefier. You can get it from [nodejs.org](https://nodejs.org/).

### **How to Create the Installer**

1. **Open your computer's command prompt or terminal.**

  * **Windows:** Press Start, type cmd, and press Enter.

  * **Mac:** Open Terminal.app from your Applications/Utilities folder.

2. **Install Nativefier:**

  * Type the following command and press Enter. This installs it globally on your system.

  * npm install -g nativefier

3. **Get Your Monitoring App URL:**

  * Find the link you created in the Admin Guide (Part 3, Step 2).

  * **Example:** https://my-monitor-dash.netlify.app

4. **Run the Nativefier Command:**

  * In your terminal, cd to your Desktop or a folder where you want the app to be created.

  * **Example:** cd Desktop

  * Run the following command. This will create the app in a new folder on your Desktop.

  ```

  nativefier --name "LWS Monitor" --internal-urls ".*" "[https://YOUR-MONITOR-APP-URL.netlify.app](https://YOUR-MONITOR-APP-URL.netlify.app)"

  ```

  * **Breakdown of the command:**

  * --name "LWS Monitor": Sets the application's name.

  * --internal-urls ".\*": (Important!) Ensures that links to Google Maps open in the user's *real* browser, not inside the app.

  * "https://...": Your Monitoring App URL.

5. **Distribute the App:**

  * Nativefier will create a folder (e.g., LWS Monitor-win32-x64).

  * Inside this folder is the .exe (or on Mac, the .app) file.

  * You can zip this folder and send it to your Safety Monitor to install like a normal program.

When they run this app, it will be the exact same as the website, but in its own dedicated window. All login and data fetching works identically.
