Thinking that Windows Server 2016 Essentials might be good for test/repro. Seems it limits you to:

* one installation
* 64 GB RAM/2 CPUs
* 25 users
* 50 devices

[Link to evaluation for 180 days](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016-essentials), about a 5GB download `14393.0.161119-1705.RS1_REFRESH_SERVERESSENTIALS_OEM_X64FRE_EN-US.ISO`

[Get Started doc page](https://docs.microsoft.com/en-us/windows-server-essentials/get-started/get-started) Product Key: `NCPR7-K6YH2-BRXYM-QMPPQ-3PF6X`

* Set up a Host Network non-DHCP
* configure VirtualBox VM 8GB/2vCPU/60GBVDD
* Give it adapter 1 on the Host Only Network, adapter 2 either bridged or NAT. Note, do not use virtio adapters, the drivers are *not* included in the Guest Additions.
* boot the ISO
* login Administrator/yourpassword
* install VirtualBox Guest Additions
* Set to host time (Central)
* make sure Windows is using NTP for time sync
* Company MongoDB, Domain MDBADTEST, Server mdbadtestserver
* leave full DNS name at MDBADTEST.local
* network admin: admin/yourpassword

[here's a guide](http://workendtech.com/how-to-set-up-windows-domain-server-2016-essentials/)

* change adapter settings in Windows for static IP

