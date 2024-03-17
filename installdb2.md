# Install DB2

Display user account attributes
```
lsuser -f db2inst1 
```

View file permissions
```
cd /opt/IBM/db2/V11.1/instance
ls -la /db2 
```

Remount the file system 
```
unmount /home/db2inst1
smitty storage
File Systems > Add / Change / Show > Change Characteristics Journaled File System > 
File System Name = /db2/itimdb
New Mount Point = /db2/itimdb
Exit smitty 
```

Create the DB user
```
smitty users
Add a User
User NAME = itimdb
Primary Group = db2iadm1
Home Directory = /db2/itimdb 
```

Change the file ownership
```
cd /db2
chown root:system itimdb
chmod 751 itimdb 
```

Mount the filesystem
```
mount /db2/itimdb  
```

Change the ownership of itimdb to db2iadm1
```
chown -R itimdb:db2iadm1 itimdb 
```

Remove the previous database instance
```
cd /db2/itimdb
rm -Rf db2inst1 
```

Edit the users profile
```
vi .profile 
```
(Remove the lines added by instance creation tool)

Apply the latest FixPack
```
ssh -i ldap-host-ssh-key sroot@g43pa00009852.az3.ash.cpc.ibm.com

cd /software

scp v11.1.4fp7_aix64_universal_fixpack.tar  > db2 host 

tar -xvf v11.1.4fp7_aix64_universal_fixpack.tar 

cd /software/fp/universal/

./installFixPack 
Installation Directory: /opt/IBM/db2/V11.1
```

Complete FP installation

Create the ITIM DB instance
```
cd /opt/IBM/db2/V11.1/instance
./db2icrt -a server -s ese -p 50000 -u db2fenc1 itimdb 
```

Start the DB
```
su - itimdb
cd sqllib
. ./db2profile
dbstart 

SQL1063N DB2START processing was successful. 
```

Check the DB port listening
```
netstat -an | grep -i "50000" 
```

Check the schema user
```
lsuser -f itimuser 
```

Check applied licenses
```
sudo su -
cd /opt/IBM/db2/V11.1/adm
./dblicm -l 
Expiry date: Permanent
Number of licensed authorized users: 25
```
(There is only 1 user = itimdb) 

WebSphere

Encryption Settings
```
cd /opt/IBM/WebSphere/AppServer/profiles/Custom01/properties
grep -i xor *.props
ssl.client.props:com.ibm.ssl.keyStorePassword={xor}CDo9Hgw=
ssl.client.props:com.ibm.ssl.trustStorePassword={xor}CDo9Hgw=
ssl.client.props:#com.ibm.ssl.keyStorePassword={xor}CDo9Hgw=
ssl.client.props:#com.ibm.ssl.trustStorePassword={xor}CDo9Hgw= 
```

Use AES encryption (recommended for PenTest or Security Audit)

WebSphere runs a root user

Change permissions on profiles directory
```
chmod 750 profiles 
```
 

 