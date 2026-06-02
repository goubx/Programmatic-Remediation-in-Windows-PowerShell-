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
