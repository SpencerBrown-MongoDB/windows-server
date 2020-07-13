# Installing MongoDB on Windows Server and enabling Kerberos Auth

## Create SPN for MongoDB nodes

For each MongoDB node, an SPN in AD is needed. *mongod* runs as the SPN. (In Linux, we use a keytab, which is generated from the SPN.)

First, create a service account in AD. Server Manager - Tools - Active Directory Users and Computers. click domain (ts-mongo.com) and expand Users. New - User, create user with login name `svc_mongodb` and password that never expires.

From elevated command prompt, run `setspn -S mongodb/winmdb.ts-mongo.com svc_mongodb` to create SPN for the host `winmdb.ts-mongo.com`. 

Now, start the *mongod* service as the service account, For example, account `ts-mongo.com\svc_mongodb`. This can be done during installation, if you install from the MSI download and specify the service account and password. 

## Install MongoDB

* Download MongoDB .msi from [Download Center](https://www.mongodb.com/download-center/enterprise/releases)
* Run the MSI and choose to install MongoDB as a service. Be sure to set the correct login name/password as per above.

## Configure and test Kerberos

* Modify the server config file `mongodb.cfg` with GSSAPI and PLAIN as authentication mechanisms, bindIP 0.0.0.0. It will use the SPN `mongodb/hostname` by default.   
* Create a user in $external. Be sure to CAPITALIZE the Kerberos realm and the username is case sensitive. e.g. `Adminstrator@TS-MONGO.COM`. 
* Make sure you are logged in as a domain user. Use `whoami` to make sure. `whoami /all` gives you more info.
* MongoDB executables are at `C:\Program Files\MongoDB\Server\4.2\bin`. `mongod.cfg` is also there.
* Mongo shell login for Kerberos:

```text
.\mongo.exe --host winmdb.ts-mongo.com --authenticationMechanism GSSAPI --authenticationDatabase '$external' --username Administrator@TS-MONGO.COM
```
* The hostname must be the FQDN. 
* The username must be specified, it will not default to the logged-in user. 
* You do not have to give a password if you are logged into the Windows domain as that user.
* You can authenticate as a different domain user by giving a password and username.
* You can also use db.auth() on $external with a document `{user: 'Administrator@TS-MONGO.COM', mechanism: 'GSSAPI'}`

## Configure and test LDAP

* Add ldap.servers pointing to the Windows AD domain server, and ldap.transportSecurity: none. This lets LDAP users authenticate. 
* Add userToDNMapping so they can use their login shortnames, e.g.  '[{match:"(.*)",ldapQuery: "CN=Users,DC=ts-mongo,DC=com??sub?sAMAccountName={0}"}]'. If you want you can use `substitute` instead of `ldapQuery`.
* Add ldap.bind.queryUser with the FQDN of the admin user e.g.  "CN=Administrator,CN=users,DC=ts-mongo,DC=com", and ldap.bind.queryPassword. This sets up the LDAP session to enumerate groups for LDAP authorization.
* Add ldap.authz.queryTemplate with the template for the LDAP query to get group membershis, e.g.  '{USER}?memberOf?base'
* Add a role on admin for one of the group memberships of the user.
* Try LDAP authn/authz: (password will be prompted for)

```text
.\mongo.exe --host winmdb.ts-mongo.com --authenticationMechanism PLAIN --authenticationDatabase '$external' --username Administrator
```

