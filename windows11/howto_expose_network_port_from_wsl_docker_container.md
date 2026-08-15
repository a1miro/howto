Here is a PowerShell script that automates the process. It automatically grabs your current WSL2 IP address, removes any outdated port forwarding rules, and sets up the new mapping.
## Auto-Update Script

# 1. Get the current WSL2 IP Address
```PowerShell
$wslIp = (wsl -d Ubuntu hostname -I).Trim().Split(" ")[0]
if (-not $wslIp) {
    Write-Host "Error: Could not find WSL2 IP address. Is WSL running?" -ForegroundColor Red
    exit
}

Write-Host "Found WSL2 IP Address: $wslIp" -ForegroundColor Cyan
# 2. Define the port to forward
$port = 2224
# 3. Remove any old portproxy rules for this port
netsh interface portproxy delete v4tov4 listenport=$port listenaddress=0.0.0.0 | Out-Null
# 4. Add the new portproxy rule
netsh interface portproxy add v4tov4 listenport=$port listenaddress=0.0.0.0 connectport=$port connectaddress=$wslIp
# 5. Check if Firewall rule exists, if not, create it
$fwRule = Get-NetFirewallRule -DisplayName "WSL2-Port-$port" -ErrorAction SilentlyContinueif (-not $fwRule) {
    New-NetFirewallRule -DisplayName "WSL2-Port-$port" -Direction Inbound -Protocol TCP -LocalPort $port -Action Allow | Out-Null
    Write-Host "Created new Windows Firewall rule for port $port." -ForegroundColor Green
}

Write-Host "Successfully forwarded external port $port to WSL2 ($wslIp:$port)" -ForegroundColor Green
```

## How to use this script

* Open Notepad.
* Copy and paste the script above.
* Note: If your WSL distro is not named "Ubuntu", change -d Ubuntu in line 2 to your distro name (e.g., -d Debian).
* Save the file as wsl_port_forward.ps1.
* Right-click the file and select Run with PowerShell (ensure you run it with Administrator privileges).

Would you like me to show you how to configure Task Scheduler so this script runs automatically every time your Windows 11 computer boots up?

To run the script automatically at startup, you must configure a task in Windows Task Scheduler that triggers at system boot and executes with the highest administrative privileges.
## How to Configure Task Scheduler

   1. Open Task Scheduler
   * Press the Windows Key.
      * Type Task Scheduler and press Enter.
   2. Create a New Task
   * Click Create Task... in the right-hand Actions pane.
      * In the General tab:
      * Name it Update WSL Port Forwarding.
         * Check Run whether user is logged on or not.
         * Check Run with highest privileges (required for netsh and Firewall commands).
      3. Set the Trigger
   * Go to the Triggers tab.
      * Click New....
      * Change Begin the task to At startup.
      * Click OK.
   4. Set the Action
   * Go to the Actions tab.
      * Click New....
      * Set Action to Start a program.
      * In Program/script, type: powershell.exe
      * In Add arguments (optional), paste the following line (replace the path with your actual script path):
      
      -ExecutionPolicy Bypass -File "C:\Path\To\Your\wsl_port_forward.ps1"
      
      * Click OK.
   5. Adjust Conditions (Crucial for Laptops)
   * Go to the Conditions tab.
      * Uncheck Start the task only if the computer is on AC power so it runs while on battery.
   6. Save the Task
   * Click OK.
      * Enter your Windows user password when prompted to finalize the administrative task.
   
Would you like to know how to test the script immediately to confirm the Task Scheduler setup is working properly?

