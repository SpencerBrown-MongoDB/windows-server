Windows Server 2019 Base is available on AWS as an AMI, Instances are pretty cheap. 
I set up a t3a.medium instance with the Windows Server 2019 Base AMI and it's less than 6 cents/hour.

* Create new VPC with one public subnet and launch instances within it.
* Do not let AWS auto-assign a public IP address.
* Create a security group, allow RDP inbound from your IP. 
* Assign a private IP for each instance above .4. For example 10.52.0.10 for first instance.
* Create an Elastic IP and associate it with the instance. That way, the public IP/AWS-assigned hostname will not change. 
* You will need a public FQDN for this machine and it will be the AD domain. You can use the AWS-assigned FQDN for the EIP, or you can use your own domain name. 
* Decrypt the administrator password, download the RDP script, login, change the password if you wish.

Other configuration

* Server Manager, Local Server
* Turn off all the virus protection.
* Turn off all the firewalls.
* Turn off IE Enhanced Security Configuration
* Apply Windows updates.
* Change the computer name if desired; Server Manager, Local Server, click Computer Name, change things, restart. We assume you've changed it to WINAD below.

Install Active Directory

* Server Manager, Dashboard, Configure this Local Server, Add Roles and Features, next, next, next
* Add Active Directory Domain Services
* Add DNS (NO DHCP)
* Click through to Install, skip past static IP warning (because the AWS private IP will not change), checking "restart automatically if required" do not close window
* When install complete, click "Promote this server to a domain controller", and select "Add a new forest"
* We are assuming your domain is ts-mongo.com, change this if otherwise.
* Set Root domain name to ts-mongo.com, leave server as WINAD, click Next
* Domain Controller Options: wait for window to finish installing. Leave functional levels at Windows Server 2016, leave DNS and GC checked, enter password for DSRM same as Administrator
* DNS Options: just click Next, do not check Create DNS Delegation despite warning
* Additional options: just Next, and Next until we get to Installation, click Install
* You will now login to the Domain as Administrator. You need to "use a different account" and specify username as ts-mongo\Administrator. There may be a long wait here while it does its thing.
* Launch DNS desktop app from Windows Administrative Tools and create a new reverse DNS lookup zone, Primary zone, next, next, Network ID (see next), click next through Finish.
* Your Network ID is basically your CIDR range for your VPC but as a cut-off IP address. For example if your VPC CIDR is 172.31.0.0/16, the Network ID is 172.31.
* Now right-click DNS server WINAD, Set Aging/Scavenging for All Zones, check box, click through
* The DNS server auto-creates a forwarder for the previous DNS server!

Create another instance and Join to Existing Domain

* Create another Windows 2019 Server following the above procedure up to "Install Active Directory". Set another private IP address. We call it WINMDB.
* Manually set the DNS server to the IP of WINAD.
* Join to domain WINAD. Server Manager, Local Server, click Workgroup, click Change, check Domain, fill in ts-mongo.com.
* Provide Administrator and password. 
* Restart Windows.
* Login as ts-mongo.com\Administrator.

Install Certificate Services - Certification Authority

* Add role
* Configure Enterprise CA as Root CA
* Create new private key

Removing Active Directory

* open Server Manager, click Manage link on right upper
* click through removing roles and features to remove AD Domain Services
* note you can keep the tools installed
* Validation Results: click 'Demote this domain controller'
* Credentials: "Last domain controller"
* Set new password for Administrator (workgroup)
* Server will restart; login as Administrator on local server (winad)
* Now remove DNS service, check to allow restart
