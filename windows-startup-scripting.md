Windows startup scripts are scripts that run automatically when your computer boots up or when a user logs in. These scripts can be used to perform various tasks, such as launching applications, setting environment variables, or running maintenance tasks. There are several methods to configure startup scripts on a Windows system:

### 1. **Using the Startup Folder**
   - **Location:** `C:\Users\<Your Username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`
   - **How to Use:**
     - Place a shortcut to the script or executable in the Startup folder. It will run automatically when you log in.
     - To access this folder quickly, you can press `Win + R`, type `shell:startup`, and press Enter.

### 2. **Using Task Scheduler**
   - **How to Use:**
     1. Open Task Scheduler (`taskschd.msc`).
     2. Create a new task by clicking "Create Task".
     3. Under the "General" tab, give the task a name.
     4. Under the "Triggers" tab, click "New" and set the trigger to "At startup" or "At log on".
     5. Under the "Actions" tab, click "New", set the action to "Start a program", and browse to your script or executable.
     6. Set any conditions or settings under the "Conditions" and "Settings" tabs as needed.
     7. Click "OK" to save the task.

### 3. **Using Group Policy Editor (for advanced users)**
   - **How to Use:**
     1. Open Group Policy Editor (`gpedit.msc`).
     2. Navigate to `User Configuration > Windows Settings > Scripts (Logon/Logoff)` or `Computer Configuration > Windows Settings > Scripts (Startup/Shutdown)`.
     3. Double-click on "Logon" or "Startup", then click "Add".
     4. Browse to your script and click "OK".
     5. The script will now run whenever the specified event (logon or startup) occurs.

### 4. **Using the Registry**
   - **How to Use:**
     1. Open Registry Editor (`regedit`).
     2. Navigate to one of the following keys:
        - For a user-specific startup script: `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run`
        - For a system-wide startup script: `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run`
     3. Right-click in the right pane and select `New > String Value`.
     4. Name the string value, then set its value to the path of your script or executable.
     5. The script will now run at startup.

### 5. **Using a Batch File or PowerShell Script**
   - **Batch File:**
     - Write a `.bat` file with the commands you want to run.
     - You can place this file in the Startup folder or set it up to run through Task Scheduler or Group Policy.
   - **PowerShell Script:**
     - Write a `.ps1` script.
     - You can execute it using Task Scheduler or through Group Policy.

### Example of a Batch File:
```batch
@echo off
rem Example of a startup script
cd /d C:\Path\To\Directory
start MyApplication.exe
```

### Example of a PowerShell Script:
```powershell
# Example PowerShell startup script
Set-Location -Path "C:\Path\To\Directory"
Start-Process "MyApplication.exe"
```

### Tips:
- Ensure your script or executable path is correct.
- If using Task Scheduler or Group Policy, test the setup to ensure it works as expected.
- You may need administrative privileges depending on the tasks performed by the script.

This setup will allow you to automate various tasks upon startup or login in Windows.
