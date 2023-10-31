# Application Inventory
These apps perform different application inventorying functions for their various envirnoments.  
The following indexes are required:
- oswinscript  
- osnixscript  

## _server_app_appfinder  
This app is for Windows based systems and uses both prefetch analysis (using PECmd and also captures Hard Drive serial numbers from the system. If you are running your Splunk Forwarders from a non-standard location, make sure to update the path in the script field for [powershell://run-pecmd]  

### inputs:
```
[powershell://run-pecmd]
disabled = 0
C:\Windows\Prefetch -q --csv C:\temp"
schedule = 0 */12 * * *
script = start-process -FilePath "C:\Program Files\SplunkUniversalForwarder\etc\apps\_server_app_appfinder\bin\PECmd.exe" -ArgumentList "-d C:\Windows\Prefetch -q --csv C:\temp\prefetch"
index = oswinscript

[monitor://c:\temp\prefetch\*.csv]
disabled = 0
index = oswinscript
sourcetype = csv
crcSalt = <SOURCE>

[powershell://run-hdserial]
disabled = 0
script = Get-WMIObject win32_physicalmedia | Select-Object -Property Tag,SerialNumber
schedule = 0 */12 * * *
index = oswinscript

```

## _server_app_installed_apps
This app uses the win_installed_app.bat script from the Splunk_TA app to query the uninstall key in Windows envirnoments. Need to update to also pull in user uninstall keys from NTUSER hives.

### inputs:
```
[script://.\bin\win_installed_apps.bat]
disabled = 0
## Run once
interval = -1
sourcetype = Script:InstalledApps
index = oswinscript
```

## _server_app_linuxapps
This app will query both installed applications (using rpm) and running processes (using ps).

### inputs:
```
[script://./bin/installed_apps.sh]
disabled = false
index = osnixscript
interval = 3600
source = installed_apps

[script://./bin/running_processes.sh]
disabled = false
index = osnixscript
interval = 3600
source = running_processes
```
