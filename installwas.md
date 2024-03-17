# Install WAS

#### Step 1

Uploaded the following media to WAS 1 - g43pa00009853:

```
g43pa00009853:/home/a097089 > ls -o
total 2352904
-rw-------    1 a097089   250368424 Aug 18 18:16 7.0.10.45-WS-IBMWASJAVA-AIX.zip
-rw-------    1 a097089   261808240 Aug 18 18:09 7.1.4.45-WS-IBMWASJAVA-AIX.zip
-rw-------    1 sroot     310745495 Aug 18 17:52 8.0.5.37-WS-IBMWASJAVA-AIX.zip
-rw-------    1 a097089   254873600 Aug 18 22:49 WS_SDK_JAVA_TEV7.0_1OF3_WAS_8.5.5.zip
-rw-------    1 sroot     126876604 Aug 18 17:36 agent.installer.aix.motif.ppc_1.6.0.20120831_1216.zip
```

Note: copied to `/software`

Extracted Installation Manager to /software/im
```
g43pa00009853:/software/im > unzip agent.installer.aix.motif.ppc_1.6.0.20120831_1216.zip
```

Ran the installation:
```
[g43pa00009853.az3.ash.cpc.ibm.com:/software/im] ./installc -acceptLicense
Installed com.ibm.cic.agent_1.6.0.20120831_1216 to the /opt/IBM/InstallationManager/eclipse directory.
```

Used Installation Manager to list packages available:

WebSphere 8.5.5
```
[g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/InstallationManager/eclipse/tools] ./imcl listAvailablePackages -repositories /software/IBM/WAS/WAS_ND_V8
com.ibm.websphere.ND.v85_8.5.5000.20130514_1044
```

JDK 8.0.5
```
[g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/InstallationManager/eclipse/tools] ./imcl listAvailablePackages -repositories /software/jdk8
com.ibm.websphere.IBMJAVA.v80_8.0.5037.20190801_0015
com.ibm.websphere.liberty.IBMJAVA.v80_8.0.5037.20190801_0015
```

JDK 7.0.1
```
[g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/InstallationManager/eclipse/tools]./imcl listAvailablePackages -repositories /software/jdk70
com.ibm.websphere.IBMJAVA.v70_7.0.10045.20190801_0005
com.ibm.websphere.liberty.IBMJAVA.v70_7.0.10045.20190801_0005
```

Then used Installation Manager to install from the available packages:
```
/opt/IBM/InstallationManager/eclipse/tools/imcl install com.ibm.websphere.ND.v85_8.5.5000.20130514_1044 com.ibm.websphere.IBMJAVA.v70_7.0.10045.20190801_0005,com.ibm.websphere.liberty.IBMJAVA.v70_7.0.10045.20190801_0005 -repositories /software/IBM/WAS/WAS_ND_V8,/software/jdk70 -installationDirectory /opt/IBM/WebSphere/AppServer -acceptLicense -sP 
```

Encountered the following error:
```
                 25%                50%                75%                100%
------------------|------------------|------------------|------------------|
............................................................................
ERROR: Error preparing IBM WebSphere SDK Java Technology Edition (Optional)  

CRIMC1020E ERROR:   plug-in com.ibm.was.jdk7.moreinfo_7.0.0.20120119_0100 not found    
ERROR:     'plug-in com.ibm.was.jdk7.moreinfo_7.0.0.20120119_0100' not found in /software/jdk70.    
ERROR:     'plug-in com.ibm.was.jdk7.moreinfo_7.0.0.20120119_0100' not found.   

CRIMC1020E ERROR:   plug-in com.ibm.ws.detect.running.process.jdk.v7_7.0.0.20120310_0100 not found    
ERROR:     'plug-in com.ibm.ws.detect.running.process.jdk.v7_7.0.0.20120310_0100' not found in /software/jdk70.    
ERROR:     'plug-in com.ibm.ws.detect.running.process.jdk.v7_7.0.0.20120310_0100' not found. 
```

Links

Installing the Installation Manager

https://www.ibm.com/docs/en/samfess/8.2.2?topic=855-installing-installation-manager

Listing available packages by using imcl commands

https://www.ibm.com/docs/en/installation-manager/1.8.5?topic=command-listing-available-packages-by-using-imcl-commands

Silent Installation of Websphere Application Server 8.5.5 on Linux

http://webspherepundit.com/silent-installation-of-websphere-application-server-8-5-5-on-linux

Installation Manager

https://www.ibm.com/support/pages/installation-manager-version-16


#### Step 2

Searched ExtremeLeverage for Part Numbers

```
CIUC7ML
CIUC8ML
CIUC9ML
```

Downloaded locally:
```
(base) ~/Downloads $ ls -ltr WS*.zip
-rw-r--r--@ 1 arod  staff  1031722332 Aug 18 11:51 WS_SDK_JAVA_TEV7.0_1OF3_WAS_8.5.5.zip
-rw-r--r--@ 1 arod  staff  1017226584 Aug 18 15:38 WS_SDK_JAVA_TEV7.0_2OF3_WAS_8.5.5.zip
-rw-r--r--@ 1 arod  staff   165472051 Aug 18 15:40 WS_SDK_JAVA_TEV7.0_3OF3_WAS_8.5.5.zip
```

#### Step 3

Transfered and unziped files from Xtreme leverage to the g43pa00009853 host:

```
g43pa00009853:/software/IBM/WAS/WAS_SDK > ls -o
total 4325056
-rwxr-xr-x    1 sroot    1031722332 Aug 21 17:29 WS_SDK_JAVA_TEV7.0_1OF3_WAS_8.5.5.zip
-rw-------    1 sroot    1017226584 Aug 21 17:34 WS_SDK_JAVA_TEV7.0_2OF3_WAS_8.5.5.zip
-rw-------    1 sroot     165472051 Aug 21 17:35 WS_SDK_JAVA_TEV7.0_3OF3_WAS_8.5.5.zip
```
 
```
g43pa00009853:/software/IBM/WAS/WAS_SDK > ls -o
total 4325080
-rw-r--r--    1 sroot           380 May 23 2013  Copyright.txt
-rwxr-xr-x    1 sroot    1031722332 Aug 21 17:29 WS_SDK_JAVA_TEV7.0_1OF3_WAS_8.5.5.zip
-rw-------    1 sroot    1017226584 Aug 21 17:34 WS_SDK_JAVA_TEV7.0_2OF3_WAS_8.5.5.zip
-rw-------    1 sroot     165472051 Aug 21 17:35 WS_SDK_JAVA_TEV7.0_3OF3_WAS_8.5.5.zip
drwxr-xr-x    5 sroot           256 May 23 2013  disk1
drwxr-xr-x    3 sroot           256 May 23 2013  disk2
drwxr-xr-x    3 sroot           256 May 23 2013  disk3
drwxr-xr-x   10 sroot          4096 May 23 2013  readme
-rw-r--r--    1 sroot            81 May 23 2013  repository.config
drwxr-xr-x    3 sroot           256 May 23 2013  responsefiles
```

Used imcl to list packages in `/software/IBM/WAS/WAS_SDK` as a repository:

```
g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/InstallationManager/eclipse/tools > ./imcl listAvailablePackages -repositories /software/IBM/WAS/WAS_SDK
```

```
com.ibm.websphere.IBMJAVA.v70_7.0.4001.20130510_2103
com.ibm.websphere.liberty.IBMJAVA.v70_7.0.4001.20130510_2103
```

#### Step 4

Ran installation on g43pa00009853:

```
/opt/IBM/InstallationManager/eclipse/tools/imcl install com.ibm.websphere.ND.v85_8.5.5000.20130514_1044 com.ibm.websphere.IBMJAVA.v70_7.0.4001.20130510_2103 -repositories /software/IBM/WAS/WAS_ND_V8,/software/jdk70 -installationDirectory /opt/IBM/WebSphere/AppServer -acceptLicense -sP
```

Installation status report:
```
[g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/WebSphere/AppServer/bin] ./versionInfo.sh
WVER0010I: Copyright (c) IBM Corporation 2002, 2012; All rights reserved.
WVER0012I: VersionInfo reporter version 1.15.1.48, dated 2/8/12
Aug 24, 2023 12:04:23 AM java.util.prefs.FileSystemPreferences$7 run
WARNING: Prefs file removed in background /home/root/.java/.userPrefs/com/ibm/crypto/pkcs11impl/provider/prefs.xml
--------------------------------------------------------------------------------
IBM WebSphere Product Installation Status Report
--------------------------------------------------------------------------------
Report at date and time August 24, 2023 12:04:23 AM UTCInstallation
--------------------------------------------------------------------------------
Product Directory        /opt/IBM/WebSphere/AppServer
Version Directory        /opt/IBM/WebSphere/AppServer/properties/version
DTD Directory            /opt/IBM/WebSphere/AppServer/properties/version/dtd
Log Directory            /var/ibm/InstallationManager/logsProduct List
--------------------------------------------------------------------------------
IBMJAVA7                 installed
ND                       installedInstalled Product
--------------------------------------------------------------------------------
Name                  IBM WebSphere SDK Java Technology Edition (Optional)
Version               7.0.4.1
ID                    IBMJAVA7
Build Level           gm1318.03
Build Date            5/10/13
Package               com.ibm.websphere.IBMJAVA.v70_7.0.4001.20130510_2103
Architecture          PPC64
Installed Features    IBM WebSphere SDK for Java Technology Edition 7Installed Product
--------------------------------------------------------------------------------
Name                  IBM WebSphere Application Server Network Deployment
Version               8.5.5.0
ID                    ND
Build Level           gm1319.01
Build Date            5/14/13
Package               com.ibm.websphere.ND.v85_8.5.5000.20130514_1044
Architecture          PPC64
Installed Features    IBM 64-bit WebSphere SDK for Java
                      WebSphere Application Server Full Profile
                      EJBDeploy tool for pre-EJB 3.0 modules
                      Embeddable EJB container
                      Stand-alone thin clients and resource adapters
--------------------------------------------------------------------------------
End Installation Status Report
-------------------------------------------------------------------------------- 
```

#### Step 5

A. Apply fix pack 23 for WebSphere Java SDK 7:

1. Downloaded FP23 zip files from Fix Central, extracted into /software/fp23:

```
[g43pa00009853.az3.ash.cpc.ibm.com:/software/fp23] ls -ltro *.zip
-rw-------    1 sroot     165472051 Aug 22 09:20PM WS_SDK_JAVA_TEV7.0_3OF3_WAS_8.5.5.zip
-rw-------    1 sroot    1017226584 Aug 22 09:42PM WS_SDK_JAVA_TEV7.0_2OF3_WAS_8.5.5.zip
-rw-------    1 sroot    1031722332 Aug 22 09:45PM WS_SDK_JAVA_TEV7.0_1OF3_WAS_8.5.5.zip
```

2. List available package(s):

```
/opt/IBM/InstallationManager/eclipse/tools/imcl listAvailablePackages -repositories /software/fp23
```

```
com.ibm.websphere.IBMJAVA.v70_7.0.4001.20130510_2103
com.ibm.websphere.liberty.IBMJAVA.v70_7.0.4001.20130510_2103
```

3. Apply fix pack for WebSphere Java SDK 7

```
/opt/IBM/InstallationManager/eclipse/tools/imcl install com.ibm.websphere.IBMJAVA.v70_7.0.4001.20130510_2103 -repositories /software/fp23 -installationDirectory /opt/IBM/WebSphere/AppServer -acceptLicense -sP
```

B. Apply fix pack 23 for WebSphere 8.5.5:

1. Downloaded FP23 zip files from Fix Central, extracted into /software/was_fp23:

```
g43pa00009853:/software/was_fp23 > ls -lro *.zip
-rw-------    1 sroot    1966668297 Aug 23 21:14 8.5.5-WS-WAS-FP023-part3.zip
-rw-------    1 sroot     198696280 Aug 23 20:57 8.5.5-WS-WAS-FP023-part2.zip
-rw-------    1 sroot    1043662686 Aug 23 20:54 8.5.5-WS-WAS-FP023-part1.zip
```

2. List available packages

```
/opt/IBM/InstallationManager/eclipse/tools/imcl listAvailablePackages -repositories /software/was_fp23
```

```
com.ibm.websphere.BASE.v85_8.5.5023.20230116_1145
com.ibm.websphere.BASETRIAL.v85_8.5.5023.20230116_1145
com.ibm.websphere.DEVELOPERS.v85_8.5.5023.20230116_1145
com.ibm.websphere.DEVELOPERSILAN.v85_8.5.5023.20230116_1145
com.ibm.websphere.EXPRESS.v85_8.5.5023.20230116_1145
com.ibm.websphere.EXPRESSTRIAL.v85_8.5.5023.20230116_1145
com.ibm.websphere.ND.v85_8.5.5023.20230116_1145
com.ibm.websphere.NDDMZ.v85_8.5.5023.20230116_1145
com.ibm.websphere.NDDMZTRIAL.v85_8.5.5023.20230116_1145
com.ibm.websphere.NDTRIAL.v85_8.5.5023.20230116_1145
```

3. Apply fix pack 23 packages for WebSphere 8.5.5:

```
/opt/IBM/InstallationManager/eclipse/tools/imcl install com.ibm.websphere.BASE.v85_8.5.5023.20230116_1145 com.ibm.websphere.BASETRIAL.v85_8.5.5023.20230116_1145 com.ibm.websphere.DEVELOPERS.v85_8.5.5023.20230116_1145 com.ibm.websphere.DEVELOPERSILAN.v85_8.5.5023.20230116_1145 com.ibm.websphere.EXPRESS.v85_8.5.5023.20230116_1145 com.ibm.websphere.EXPRESSTRIAL.v85_8.5.5023.20230116_1145 com.ibm.websphere.ND.v85_8.5.5023.20230116_1145 com.ibm.websphere.NDDMZ.v85_8.5.5023.20230116_1145 com.ibm.websphere.NDDMZTRIAL.v85_8.5.5023.20230116_1145 com.ibm.websphere.NDTRIAL.v85_8.5.5023.20230116_1145 -repositories /software/was_fp23 -installationDirectory /opt/IBM/WebSphere/AppServer -acceptLicense -sP
```
 
#### Step 6

Re-install WAS 8.5.5

```
/opt/IBM/InstallationManager/eclipse/tools/imcl install com.ibm.websphere.ND.v85_8.5.5000.20130514_1044  -repositories /software/IBM/WAS/WAS_ND_V8 -installationDirectory /opt/IBM/WebSphere/AppServer -acceptLicense -sP
```

```
                 25%                50%                75%                100%
------------------|------------------|------------------|------------------|
............................................................................
Installed com.ibm.websphere.ND.v85_8.5.5000.20130514_1044 to the /opt/IBM/WebSphere/AppServer directory.
```

IBM WebSphere Product Installation Status Report

```
[g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/WebSphere/AppServer/bin] ./versionInfo.sh
WVER0010I: Copyright (c) IBM Corporation 2002, 2012; All rights reserved.
WVER0012I: VersionInfo reporter version 1.15.1.48, dated 2/8/12Aug 25, 2023 1:35:36 AM java.util.prefs.FileSystemPreferences$7 run
WARNING: Prefs file removed in background /home/root/.java/.userPrefs/com/ibm/crypto/pkcs11impl/provider/prefs.xml
--------------------------------------------------------------------------------
IBM WebSphere Product Installation Status Report
--------------------------------------------------------------------------------
Report at date and time August 25, 2023 1:35:36 AM UTCInstallation
--------------------------------------------------------------------------------
Product Directory        /opt/IBM/WebSphere/AppServer
Version Directory        /opt/IBM/WebSphere/AppServer/properties/version
DTD Directory            /opt/IBM/WebSphere/AppServer/properties/version/dtd
Log Directory            /var/ibm/InstallationManager/logsProduct List
--------------------------------------------------------------------------------
ND                       installedInstalled Product
--------------------------------------------------------------------------------
Name                  IBM WebSphere Application Server Network Deployment
Version               8.5.5.0
ID                    ND
Build Level           gm1319.01
Build Date            5/14/13
Package               com.ibm.websphere.ND.v85_8.5.5000.20130514_1044
Architecture          PPC64
Installed Features    IBM 64-bit WebSphere SDK for Java
                      WebSphere Application Server Full Profile
                      EJBDeploy tool for pre-EJB 3.0 modules
                      Embeddable EJB container
                      Stand-alone thin clients and resource adapters
--------------------------------------------------------------------------------
End Installation Status Report
-------------------------------------------------------------------------------- 
```

#### Step 7

```
[g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/InstallationManager/eclipse/tools] chfs -a size=+200M /usr
Filesystem size changed to 11010048
```

#### Step 8 

Retry to apply WAS FP 23, success:

```
/opt/IBM/InstallationManager/eclipse/tools/imcl install com.ibm.websphere.ND.v85_8.5.5023.20230116_1145 -repositories /software/was_fp23 -installationDirectory /opt/IBM/WebSphere/AppServer -acceptLicense -sP
```

```
                 25%                50%                75%                100%
------------------|------------------|------------------|------------------|
............................................................................
Updated to com.ibm.websphere.ND.v85_8.5.5023.20230116_1145 in the /opt/IBM/WebSphere/AppServer directory.
```

#### Step 9

IBM WebSphere Product Installation Status Report:

```
[g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/WebSphere/AppServer/bin] ./versionInfo.sh
WVER0010I: Copyright (c) IBM Corporation 2002, 2012; All rights reserved.
WVER0012I: VersionInfo reporter version 1.15.1.50, dated 12/20/18
--------------------------------------------------------------------------------
IBM WebSphere Product Installation Status Report
--------------------------------------------------------------------------------
Report at date and time August 25, 2023 1:50:30 AM UTCInstallation
--------------------------------------------------------------------------------
Product Directory        /opt/IBM/WebSphere/AppServer
Version Directory        /opt/IBM/WebSphere/AppServer/properties/version
DTD Directory            /opt/IBM/WebSphere/AppServer/properties/version/dtd
Log Directory            /var/ibm/InstallationManager/logsProduct List
--------------------------------------------------------------------------------
ND                       installedInstalled Product
--------------------------------------------------------------------------------
Name                  IBM WebSphere Application Server Network Deployment
Version               8.5.5.23
ID                    ND
Build Level           cf232303.01
Build Date            1/16/23
Package               com.ibm.websphere.ND.v85_8.5.5023.20230116_1145
Java SE Version       8
Architecture          PPC64
Installed Features    IBM 64-bit WebSphere SDK for Java
                      WebSphere Application Server Full Profile
                      EJBDeploy tool for pre-EJB 3.0 modules
                      Embeddable EJB container
                      Stand-alone thin clients and resource adapters
--------------------------------------------------------------------------------
End Installation Status Report
--------------------------------------------------------------------------------
```

#### Step 10

Create profiles and federate nodes:

1. Create a Dmgr01 profile for Deployment Manager on pre-prod WAS Node 2, g43pa00009854:

```
/opt/IBM/WebSphere/AppServer/bin/manageprofiles.sh -create -templatePath /opt/IBM/WebSphere/AppServer/profileTemplates/management/ -profileName /opt/IBM/WebSphere/AppServer/profiles/Dmgr01 -profileName Dmgr01 -enableAdminSecurity true -adminUserName admin -adminPassword ****** 
```

Response

```
INSTCONFSUCCESS: Success: Profile Dmgr01 now exists. Please consult /opt/IBM/WebSphere/AppServer/profiles/Dmgr01/logs/AboutThisProfile.txt for more information about this profile.
```

2. Create a Custom01 profile for applications on pre-prod WAS Node 2, g43pa00009854 and WAS Node 1: g43pa00009853:
```
/opt/IBM/WebSphere/AppServer/bin/manageprofiles.sh -create -templatePath /opt/IBM/WebSphere/AppServer/profileTemplates/default/ -profileName /opt/IBM/WebSphere/AppServer/profiles/Custom01 -profileName Custom01
```

3. Start the Deployment Manager on WAS Node 2:
```
/opt/IBM/WebSphere/AppServer/profiles/Dmgr01/bin/startManager.sh
```

4. Add the WAS 2 node on g43pa00009854:
```
/opt/IBM/WebSphere/AppServer/bin/addNode.sh g43pa00009854.az3.ash.cpc.ibm.com 8879 -profileName Custom01 -username admin -password ****** 
```

5. Add the WAS 1 node on g43pa00009853:
```
/opt/IBM/WebSphere/AppServer/bin/addNode.sh g43pa00009854.az3.ash.cpc.ibm.com 8879 -profileName Custom01 -username admin -password ******
```

Reponse

```
ADMU0116I: Tool information is being logged in file
           /opt/IBM/WebSphere/AppServer/profiles/AppSrv01/logs/addNode.log
ADMU0128I: Starting tool with the Custom01 profile
CWPKI0308I: Adding signer alias "CN=g43pa00009853.az3.ash.cpc.ib" to local
           keystore "ClientDefaultTrustStore" with the following SHA digest:
           19:F8:66:EF:77:47:01:5C:88:14:85:42:62:70:85:6A:E6:2B:14:D7
CWPKI0309I: All signers from remote keystore already exist in local keystore.
ADMU0001I: Begin federation of node g43pa00009853Node01 with Deployment Manager
           at g43pa00009853.az3.ash.cpc.ibm.com:8879.
ADMU0009I: Successfully connected to Deployment Manager Server:
           g43pa00009853.az3.ash.cpc.ibm.com:8879
ADMU0505I: Servers found in configuration:
ADMU0506I: Server name: server1
ADMU2010I: Stopping all server processes for node g43pa00009853Node01
ADMU0512I: Server server1 cannot be reached. It appears to be stopped.
ADMU0024I: Deleting the old backup directory.
ADMU0015I: Backing up the original cell repository.
ADMU0012I: Creating Node Agent configuration for node: g43pa00009853Node01
ADMU0014I: Adding node g43pa00009853Node01 configuration to cell:
           g43pa00009853Cell01
ADMU0016I: Synchronizing configuration between node and cell.
ADMU0018I: Launching Node Agent process for node: g43pa00009853Node01
ADMU0020I: Reading configuration for Node Agent process: nodeagent
ADMU0022I: Node Agent launched. Waiting for initialization status.
ADMU0030I: Node Agent initialization completed successfully. Process id is:
           17826244
ADMU0300I: The node g43pa00009853Node01 was successfully added to the
           g43pa00009853Cell01 cell.
ADMU0306I: Note:
ADMU0302I: Any cell-level documents from the standalone g43pa00009853Cell01
           configuration have not been migrated to the new cell.
ADMU0307I: You might want to:
ADMU0303I: Update the configuration on the g43pa00009853Cell01 Deployment
           Manager with values from the old cell-level documents.
ADMU0306I: Note:
ADMU0304I: Because -includeapps was not specified, applications installed on
           the standalone node were not installed on the new cell.
ADMU0307I: You might want to:
ADMU0305I: Install applications onto the g43pa00009853Cell01 cell using wsadmin
           $AdminApp or the Administrative Console.
ADMU0003I: Node g43pa00009853Node01 has been successfully federated.
```

