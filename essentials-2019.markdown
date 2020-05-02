Thinking that Windows Server 2019 Essentials might be good for test/repro. Seems it limits you to:

* one installation
* 64 GB RAM/2 CPUs
* 25 users
* 50 devices

[Link to evaluation for 180 days](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019-essentials), about a 4.6 download ``

[Get Started doc page](https://docs.microsoft.com/en-us/windows-server-essentials/get-started/get-started) Product Key: `NJ3X8-YTJRF-3R9J9-D78MF-4YBP4`

Install using Hyper-V on Windows 10: more or less following [this guide](https://www.moderndeployment.com/windows-server-2019-active-directory-installation-beginners-guide/)
`NJ3X8-YTJRF-3R9J9-D78MF-4YBP4` is product key
New VM Wizard

* Name: Windows
* Generation 2
* 4GB memory, dynamic memory is OK?
* Networking: Not Connected
* 127GB hard disk

New VM Settings (Windows VM)

* Disable Secure Boot and TPM
* Add a 2nd processor
* Enable Guest Services under Integration Services
* First network adapter to NATty virtual switch; this used for external access via NAT
* note: see Windows10 repo for IP address assignments
* Turn off checkpoints
* Set auto start and stop action as desired

Install Windows Server Essentials 2019

* Boot the ISO by starting the VM and hitting a key quickly to boot from DVD
* Go through basic install steps including product code
* Select Custom Install Windows Only
* Wait for Windows to install
* Fix up boot order and reboot
* Set Administrator password yourpassword
* Login for real (first login)
* Open Control Panel / System and Security / System
* Change computer name to WINAD
* set up static IP for first network adapter.
* Apply Windows updates.
* DEPRECATED DO NOT DO THIS Shut down, create second network adapter for internal only, put it static at 10.42.0.42. No gateway. 
* DNS is 192.168.0.45 and 127.0.0.1.
* Set local time zone, make sure "time is set automatically".
* Need to configure good time server at this point.

Install Active Directory

* DEPRECATED DON'T DO THIS Disable the first network adapter in Windows (the external one).
* Server Manager, Configure this Local Server, Add Roles and Features, next, next, next
* Add Active Directory Domain Services
* Add DNS (NO DHCP)
* Click through to Install, do not close window
* When you get to Results screen, click "Promote this server to a domain controller"
* Set Root domain name to MDBADDOM.local, leave server as WINAD, click Next
* Domain Controller Options: leave functional levels at Windows Server 2016, left DNS and GC checked, enter password for DSRM same as Administrator
* DNS Options: just click Next, do not check Create DNS Delegation despite warning
* Additional options: just Next, and Next until we get to Installation, click Install
* Machine will also restart and wait for a while. Seems to sit at "Please wait for the gpsvc". That lasted several minutes.
* You will now login to the Domain as Administrator
* Launch DNS tool from Windows Administrative Tools and create a new reverse DNS lookup zone, Primary zone, next, next, 192.168.0, click next through Finish.
* Now right-click DNS server WINAD, Set Aging/Scavenging for All Zones, check box, click through
* DEPRECATED DON'T DO Launch DHCP manager, right-click server winad.mdbaddmon.local, make sure it's authorized. All checkmarks should be green.
* DEPRECATED DON'T DO Right-click IPv4, New Scope, fill in the New Scope Wizard. Start IP 10.42.0.100, End IP 10.42.0.149, Length 24. Click through to Finish.
* DEPRECATED DON'T DO Re-enable the first network adapter. Now you have access to the outside.
* you have access on Windows to the MDBADDOM.local network through the "NATty" switch! 
* The DNS server auto-creates a forwarder for the previous DNS server (8.8.8.8)!

Install Certificate Services - Certification Authority

* Add role
* Configure Enterprise CA as Root CA
* Create new private key


Removing Active Directory

open Server Manager, manage
click through removing roles and features to remove everything
