# Installing MongoDB on Windows Server and enabling Kerberos Auth

## Create SPN for MongoDB nodes

For each MongoDB node, an SPN in AD is needed. *mongod* runs as the SPN. (In Linux, we use a keytab, which is generated from the SPN.)

First, create a service account in AD. Server Manager - Tools - Active Directory Users and Computers. click domain (ts-mongo.com) and expand Users. New - User, create user with login name `svc_mongodb` and password that never expires.

From elevated command prompt, run `setspn -S mongodb/winmdb.ts-mongo.com svc_mongodb` to create SPN for the host `winmdb.ts-mongo.com`. 

Now, start the *mongod* service as the service account, For example, account `ts-mongo.com\svc_mongodb`. This can be done during installation, if you install from the MSI download and specify the service account and password. 

## Install MongoDB

* Download MongoDB .msi from [Download Center](https://www.mongodb.com/download-center/enterprise/releases)
* Run the MSI and choose to install MongoDB as a service. Be sure to set the correct login name/password as per above.
* Configure and run the server with GSSAPI as an authentication mechanism, bindIP 0.0.0.0. It will use the SPN `mongodb/hostname` by default.
* Create a user in $external. Be sure to CAPITALIZE the Kerberos realm and the username is case sensitive. e.g. `Adminstrator@TS-MONGO.COM`. 
* Make sure you are logged in as a domain user. Use `whoami` to make sure. `whoami /all` gives you more info.
* Mongo shell login:

```text
.\mongo.exe --host winmdb.ts-mongo.com --authenticationMechanism GSSAPI --authenticationDatabase '$external' --username Administrator@TS-MONGO.COM
```

* The hostname must be the FQDN. 
* The username must be specified, it will not default to the logged-in user. 
* You do not have to give a password if you are logged into the Windows domain as that user.
* You can authenticate as a different domain user by giving a password and username.
* You can also use db.auth() on $external with a document `{user: 'Administrator@TS-MONGO.COM', mechanism: 'GSSAPI'}`