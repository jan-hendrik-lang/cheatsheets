# Windows Registry Analysis for Forensics

## 1. User Activity and Evidence of Execution

### Recently Accessed Files and Programs
- `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`  
  Stores recently opened documents.
- `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU`  
  Stores recently executed commands from the Run dialog.
- `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist`  
  Tracks user activity, including executed programs.

### Program Execution Evidence
- `HKEY_CURRENT_USER\Software\Microsoft\Windows\Shell\Bags\1\Desktop`  
  Contains information about folder and file views.
- `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`  
  Stores programs set to run at startup.
- `HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services`  
  Contains details of installed services and drivers.
- `HKEY_CURRENT_USER\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\MuiCache`  
  Stores execution history of recently run applications.

## 2. Persistence and Malware Indicators

### Auto-Start Locations
- `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run`  
  Programs that start automatically with Windows.
- `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run`  
  Same as above but for the current user.
- `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce`  
  Programs executed once at startup.
- `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunServices`  
  Auto-start services.
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\BootExecute`  
  Lists programs executed at boot.
- `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders`  
  Can be modified by malware for persistence.

## 3. Network and Remote Access
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters`  
  Stores IP configurations.
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\StandardProfile\AuthorizedApplications\List`  
  List of applications allowed through the firewall.
- `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Internet Settings`  
  Stores proxy settings and connections.
- `HKEY_CURRENT_USER\Software\Microsoft\Terminal Server Client\Default`  
  Tracks remote desktop connections.

## 4. USB Devices and External Storage
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`  
  Lists USB devices connected to the system.
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\DeviceClasses`  
  Tracks connected hardware.
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\UsbStor`  
  Can be used to enable/disable USB storage.

## 5. Deleted Files and User Profiles
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\PrefetchParameters`  
  Information on prefetch files, which can indicate program execution.
- `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders`  
  Can store paths to user profile folders.
- `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList`  
  Tracks user profiles on the system.

## 6. Browser and Internet Artifacts
- `HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\TypedURLs`  
  List of URLs typed in Internet Explorer.
- `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall`  
  Lists installed programs, useful for identifying suspicious software.

## 7. Time Stamps and System Information
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName`  
  Stores the system's hostname.
- `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\InstallDate`  
  Windows installation timestamp.
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation`  
  Time zone settings, useful for timeline analysis.

## 8. Security and Audit Logs
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog`  
  Stores Windows event logs.
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa`  
  Security settings, including authentication policies.
