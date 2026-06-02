# Programmatic-Remediation-in-Windows-PowerShell-

I will demonstrate how to run an authenticated scan on a device through tenable.io and then pragmatically remediate all issues.

<h2>Requirements for this Lab</h2>

- <b> Microsoft Azure </b>
- <b> Windows 11 Pro Virtual Machine </b>
- <b> Tenable.IO </b>

<p align="center">
Today, I will run an authenticated scan against a device on the network and pragmatically apply all necessary remediations. The purpose of today's lab is to demonstrate how to create scans, run them, and then remediate the issues found in those scans. 

<h2> I will log in to Azure so I can create my Windows 11 virtual machine first. </h2>

<h2> After that, I am going to get my Public IP for my VM and connect to it from a remote Windows desktop. <h2>

<img src="https://i.imgur.com/yfP6Jjb.jpeg" height="80%" width="80%" alt="Agent Group created"/>
<br />

<h3> Now that I am logged into my VM, I will disable the firewall. </h3>

- WF.msc, and run as administrator
- Windows Defender Firewall Properties
- I'm going to set Domain, Private, and Public Profile Firewalls off.

<h2> I've successfully turned off the firewall! <h2>

<img src="https://i.imgur.com/eaNKWIF.jpeg" height="80%" width="80%" alt="Agent Group created"/>
<br />

<h3> I now have a script that I will run in PowerShell as an admin to enable remote administrative access. </h3>

> Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "LocalAccountTokenFilterPolicy" -Value 1 -Type DWord -Force

<img src="https://i.imgur.com/cGvHsix.jpeg" height="80%" width="80%" alt="Agent Group created"/>
<br />

<h2> Next, I'm going to run an authenticated scan from Tenable using the Win-11-DISA-STIG-v2-r6 Template
 </h2>

 - I log in to Tenable
 - Go to Scans
 - Select User Defined Scan Template

<h3> Now, within here, what I'm going to do is set my parameters for the scan, such as including the Private IP Address of my VM. Since I'm using the internal scanner, I will have to include the VM's private IP. </h3>

For this scan, the private IP address is
> 10.1.0.117

Now I'm going to include the credentials I used to create my VM, so it knows how to log in.

<h2> Now, typically I'd do a baseline scan to see where my VM is at now, but because I've already done so in the past, and nothing has been configured on this VM. I will skip that and move on to adding the vulnerabilities. </h2>

Now I'm going to add a bunch of vulnerabilities to my VM, such as:
- Old Version of Firefox
- Enable SMB V1
- Enable discouraged cryptographic protocols: SSL 2.0, SSL 3.0, TLS 1.0, TLS 1.1

Firefox

<img src="https://i.imgur.com/zwhPnCF.png" height="80%" width="80%" alt="Agent Group created"/>
<br />

SMB V1
- Right-click the Start menu, "appwiz.cpl"
- Turn Windows Features on or off
- Scroll down the SMB V1
- Select OK
- I won't click restart yet, because I have more vulnerabilities I want to upload.

<img src="https://i.imgur.com/2Rvxpwl.jpeg" height="80%" width="80%" alt="Agent Group created"/>
<br />

Next, I'm going to run a PowerShell script that will enable outdated versions of SSL and TLS while at the same time, disabling the current versions.

- Now I'm going to run PowerShell ISE
- I'm going to open the file I already have with the script

> #Variable to determine if we want to make the computer secure or insecure

> $makeSecure = $true 

Im going to change the $makeSecure from true to false. By doing this, it will make the VM insecure.

- Now I will run the file.

<h3> As you can see, the script ran successfully and all of the insecure protocols have been enabled. </h3>

<img src="https://i.imgur.com/JB59ESF.png" height="80%" width="80%" alt="Agent Group created"/>
<br />

<h2> At this point, it is enough for me to restart the computer so all the changes I have made can take effect. After the reboot is complete, I will then launch the DISA STIG Scan. </h2>

The scan is complete, and the results are down below. 

<img src="https://i.imgur.com/zqlpSys.png" height="80%" width="80%" alt="Agent Group created"/>
<br />

The results show that over 

- 48 Critical
- 15 High
- 10 Medium
- 3 low

Total Vulnerabilities found

<h3> Many of the vulnerabilities we created have been listed on the scan, such as Firefox and SMB V1. </h3>

<h2> Now that I have a copy of the previous scan results. I will now go back into the VM so that I can start the remediation process. </h2>

I have 3 scripts that I will use to remediate the following issues:
- Remediate Firefox
- Remidiate SMB V1
- Remediate the discouraged cryptographic protocols

<h3> The first script for Firefox is down below. </h3>

```powershell
 # Define the path to the uninstall helper
$uninstallHelperPath = 'C:\Program Files\Mozilla Firefox\uninstall\helper.exe'

# Check if the uninstall helper exists
if (Test-Path $uninstallHelperPath) {
    # If the file exists, execute it silently
    Invoke-Expression "& `"$uninstallHelperPath`" /S"
    Write-Host "Firefox uninstall command executed."
} else {
    Write-Host "Firefox uninstall helper does not exist at the specified path."
}
```
<h3> The second script for SMB V1 is down below. </h3>

```powershell
# Run PowerShell as Administrator
# Disable SMBv1 - CIFS File Sharing Support
Write-Output "Disabling SMBv1 Protocol..."
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart
# Disable the SMBv1 Client
$clientKeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters"
$clientDriverPath = "HKLM:\SYSTEM\CurrentControlSet\Services\mrxsmb10"
Write-Output "Disabling SMBv1 Client..."
if (Test-Path $clientKeyPath) {
    Set-ItemProperty -Path $clientKeyPath -Name "AllowInsecureGuestAuth" -Value 0
}
if (Test-Path $clientDriverPath) {
    Set-ItemProperty -Path $clientDriverPath -Name "Start" -Value 4
} else {
    Write-Output "SMBv1 Client driver registry path does not exist. It may not be necessary or supported on this system."
}
# Disable the SMBv1 Server
$serverKeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters"
Write-Output "Disabling SMBv1 Server..."
if (Test-Path $serverKeyPath) {
    Set-ItemProperty -Path $serverKeyPath -Name "SMB1" -Value 0
} else {
    Write-Output "SMBv1 Server registry path does not exist. Check if SMBv1 is supported on this system."
}
Write-Output "SMBv1 has been disabled on your system. Please review the output for any potential issues."
```
<h3> The third script will be the same one I used earlier, but I will now change the secure back to true. </h3>

```powershell
# Variable to determine if we want to make the computer secure or insecure
$makeSecure = $true

# Check if the script is run as Administrator
function Check-Admin {
    $identity = [System.Security.Principal.WindowsIdentity]::GetCurrent()
    $principal = New-Object System.Security.Principal.WindowsPrincipal($identity)
    $principal.IsInRole([System.Security.Principal.WindowsBuiltInRole]::Administrator)
}

# Main script
if (-not (Check-Admin)) {
    Write-Error "Access Denied. Please run with Administrator privileges."
    exit 1
}

# SSL 2.0 settings
$serverPathSSL2 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 2.0\Server"
$clientPathSSL2 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 2.0\Client"

if ($makeSecure) {
    New-Item -Path $serverPathSSL2 -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL2 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL2 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    
    New-Item -Path $clientPathSSL2 -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL2 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL2 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    
    Write-Host "SSL 2.0 has been disabled."
} else {
    New-Item -Path $serverPathSSL2 -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL2 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL2 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    
    New-Item -Path $clientPathSSL2 -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL2 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL2 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    
    Write-Host "SSL 2.0 has been enabled."
}

# SSL 3.0 settings
$serverPathSSL3 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Server"
$clientPathSSL3 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Client"

if ($makeSecure) {
    New-Item -Path $serverPathSSL3 -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL3 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL3 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    
    New-Item -Path $clientPathSSL3 -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL3 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL3 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    
    Write-Host "SSL 3.0 has been disabled."
} else {
    New-Item -Path $serverPathSSL3 -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL3 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathSSL3 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    
    New-Item -Path $clientPathSSL3 -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL3 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathSSL3 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    
    Write-Host "SSL 3.0 has been enabled."
}

# TLS 1.0 settings
$serverPathTLS10 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server"
$clientPathTLS10 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Client"

if ($makeSecure) {
    New-Item -Path $serverPathTLS10 -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS10 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS10 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    
    New-Item -Path $clientPathTLS10 -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS10 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS10 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    
    Write-Host "TLS 1.0 has been disabled."
} else {
    New-Item -Path $serverPathTLS10 -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS10 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS10 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    
    New-Item -Path $clientPathTLS10 -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS10 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS10 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    
    Write-Host "TLS 1.0 has been enabled."
}

# TLS 1.1 settings
$serverPathTLS11 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server"
$clientPathTLS11 = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Client"

if ($makeSecure) {
    New-Item -Path $serverPathTLS11 -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS11 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS11 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    
    New-Item -Path $clientPathTLS11 -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS11 -Name 'Enabled' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS11 -Name 'DisabledByDefault' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    
    Write-Host "TLS 1.1 has been disabled."
} else {
    New-Item -Path $serverPathTLS11 -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS11 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $serverPathTLS11 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    
    New-Item -Path $clientPathTLS11 -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS11 -Name 'Enabled' -Value 1 -PropertyType 'DWord' -Force | Out-Null
    New-ItemProperty -Path $clientPathTLS11 -Name 'DisabledByDefault' -Value 0 -PropertyType 'DWord' -Force | Out-Null
    
    Write-Host "TLS 1.1 has been enabled."
}

Write-Host "Please reboot for settings to take effect."
```

<h3> Now I'm going to open up PowerShell ISE again and run the files. </h3>

<img src="https://i.imgur.com/nZOixc2.png" height="80%" width="80%" alt="Agent Group created"/>
<br />

The Firefox script has successfully run, and Firefox was removed from the VM.

<h3> Now I'm going to run the SMB script. </h3>

<img src="https://i.imgur.com/weIOfCf.png" height="80%" width="80%" alt="Agent Group created"/>
<br />

The SMB script was successfully run.

<h3> Now I'm going to run the script that disables the insecure cryptographic protocols. </h3>

<img src="https://i.imgur.com/E9Py9ND.png" height="80%" width="80%" alt="Agent Group created"/>
<br />

This script was successfully run as well. As you can see 
- SSL 2.0 has been disabled.
- SSL 3.0 has been disabled.
- TLS 1.0 has been disabled.
- TLS 1.1 has been disabled. 

Alternatively, if any of these did not work, I also had a batch file on standby
```powershell
powershell -ExecutionPolicy Bypass -File ".\remediation-FireFox-uninstall.ps1"
powershell -ExecutionPolicy Bypass -File ".\remediation-SMBv1.ps1"
powershell -ExecutionPolicy Bypass -File ".\toggle-protocols.ps1"
```
<h3> Now, before I rerun the scan, I'm going to check and see if there are any manual updates in settings I should do.</h3>

There were 2 security updates, so I made sure to update the VM. Now I'm going to restart the computer so the changes I made can take effect, and then I will re-run the scan.

<H2> The scan is complete, and the results have shown a significant drop in vulnerabilities. </H2>

<img src="https://i.imgur.com/oH9Onu0.png" height="80%" width="80%" alt="Agent Group created"/>
<br />

The vulnerability findings dropped from:

- 48 to 0 Critical
- 15 to 3 High
- 10 to 6 Medium
- 3 to 2 low

<h3> All of the vulnerabilities I added have now been remediated. I will include an additional copy of the new scans. </h3>
