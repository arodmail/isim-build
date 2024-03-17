# Install HTTP Server

#### Step 1

Downloaded IHS installation package from from IBM Fix Central

```
[g43pa00009854.az3.ash.cpc.ibm.com:/software/was_supplements] ls -o
total 6306112
-rw-------    1 a097089   482766367 Aug 29 09:01PM 8.5.5-WS-WASSupplements-FP023-part1.zip
-rw-------    1 a097089   778868291 Aug 29 09:21PM 8.5.5-WS-WASSupplements-FP023-part2.zip
-rw-------    1 a097089  1966668297 Aug 29 09:31PM 8.5.5-WS-WASSupplements-FP023-part3.zip
```

List available pakcages

```
[g43pa00009854.az3.ash.cpc.ibm.com:/opt/IBM/InstallationManager/eclipse/tools] ./imcl listAvailablePackages -repositories /software/was_supplements
com.ibm.websphere.APPCLIENT.v85_8.5.5023.20230116_1145
com.ibm.websphere.APPCLIENTILAN.v85_8.5.5023.20230116_1145
com.ibm.websphere.IHS.v85_8.5.5023.20230116_1145
com.ibm.websphere.IHSILAN.v85_8.5.5023.20230116_1145
com.ibm.websphere.PLG.v85_8.5.5023.20230116_1145
com.ibm.websphere.PLGILAN.v85_8.5.5023.20230116_1145
com.ibm.websphere.PLUGCLIENT.v85_8.5.5023.20230116_1145
com.ibm.websphere.PLUGCLIENTILAN.v85_8.5.5023.20230116_1145 {code}
Version of IHS running in prod:
```

```
g42pa00001765:/opt/IBM/HTTPServer/bin > ./versionInfo.sh
WVER0010I: Copyright (c) IBM Corporation 2002, 2012; All rights reserved.
WVER0012I: VersionInfo reporter version 1.15.1.50, dated 12/20/18
--------------------------------------------------------------------------------
IBM WebSphere Product Installation Status Report
--------------------------------------------------------------------------------
Report at date and time August 29, 2023 9:33:15 PM UTCInstallation
--------------------------------------------------------------------------------
Product Directory        /opt/IBM/HTTPServer
Version Directory        /opt/IBM/HTTPServer/properties/version
DTD Directory            /opt/IBM/HTTPServer/properties/version/dtd
Log Directory            /var/ibm/InstallationManager/logsProduct List
--------------------------------------------------------------------------------
IHS                      installedInstalled Product
--------------------------------------------------------------------------------
Name                  IBM HTTP Server for WebSphere Application Server
Version               8.5.5.23
ID                    IHS
Build Level           cf232303.01
Build Date            1/16/23
Package               com.ibm.websphere.IHS.v85_8.5.5023.20230116_1145
Java SE Version       8
Architecture          PPC64
Installed Features    IBM HTTP Server 64-bit with Java
                      Core runtime
--------------------------------------------------------------------------------
End Installation Status Report
--------------------------------------------------------------------------------
```

Found the matching package

```
com.ibm.websphere.IHS.v85_8.5.5023.20230116_1145
```
 
#### Step 2

Downloaded IHS RP (Refresh Pack) installation package from from IBM Fix Central

```
[g43pa00009853.az3.ash.cpc.ibm.com:/software/was_supplements_rp] ls -ltro *.zip
-rw-------    1 a097089  1077379866 Aug 31 02:01AM 8.5.5-WS-WASSupplements-RP-part1.zip
-rw-------    1 a097089  1498126405 Aug 31 02:03AM 8.5.5-WS-WASSupplements-RP-part2.zip
```

List available packages

```
[g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/InstallationManager/eclipse/tools] ./imcl listAvailablePackages -repositories /software/was_supplements_rp
com.ibm.websphere.APPCLIENT.v85_8.5.5000.20130514_1044
com.ibm.websphere.APPCLIENTILAN.v85_8.5.5000.20130514_1044
com.ibm.websphere.IHS.v85_8.5.5000.20130514_1044
com.ibm.websphere.IHSILAN.v85_8.5.5000.20130514_1044
com.ibm.websphere.PLG.v85_8.5.5000.20130514_1044
com.ibm.websphere.PLGILAN.v85_8.5.5000.20130514_1044
com.ibm.websphere.PLUGCLIENT.v85_8.5.5000.20130514_1044
com.ibm.websphere.PLUGCLIENTILAN.v85_8.5.5000.20130514_1044
```

#### Step 3

Ran install for RP

```
/opt/IBM/InstallationManager/eclipse/tools/imcl install com.ibm.websphere.IHS.v85_8.5.5000.20130514_1044 -repositories /software/was_supplements_rp -installationDirectory /opt/IBM/HTTPServer/ -acceptLicense -sP -showProgress -properties user.ihs.httpPort=80
```

```
                 25%                50%                75%                100%
------------------|------------------|------------------|------------------|
............................................................................
ERROR: Error preparing IBM HTTP Server for WebSphere Application Server  

ERROR:   'plug-in com.ibm.was.ihs.moreinfo.v85_8.5.0.20120118_0100' not found in /software/was_supplements_rp.
```

#### Step 4

Installing IBM HTTP Server by using response files

https://www.ibm.com/docs/en/ibm-http-server/8.5.5?topic=systems-installing-http-server-by-using-response-files

#### Step 5

Installed IBM HTTP Server using following command:

```
sudo ./imcl install com.ibm.websphere.IHS.v85_8.5.0.20120501_1108 -repositories /software/ihs -acceptLicense -showProgress -installationDirectory "/opt/IBM/HTTPServer" -sP -properties user.ihs.httpPort=80
```

Installed com.ibm.websphere.IHS.v85_8.5.0.20120501_1108 successfully.

Edited https.conf under /opt/IBM/HTTPServer/conf/httpd.conf to set User sroot.  Group system

```
chmod a+r /opt/IBM/HTTPServer/htdocs
```

```
chown -R sroot:system /opt/IBM/HTTPServer/htdocs
```

To get rid of forbidden message that was getting.

Now can access IBM HTTP Server using browser using url:

http://g43pa00009854.az3.ash.cpc.ibm.com/

http://g43pa00009853.az3.ash.cpc.ibm.com/
http://g43pa00009854.az3.ash.cpc.ibm.com/

