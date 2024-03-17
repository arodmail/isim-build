# Install TDI

#### Step 1

Version check in prod:

```
/opt/IBM/TDI/V7.0/bin/applyUpdates.sh -queryreg
```

```
Information from .registry file in: /opt/IBM/TDI/V7.2
Edition: Identity
Level: 7.2.0.6
License: FullFixes Applied
=-=-=-=-=-=-=
SDI-7.2-FP0006(7.2.0.0) Components Installed
=-=-=-=-=-=-=-=-=-=
BASE
   -SDI-7.2-FP0006
SERVER
   -SDI-7.2-FP0006
CE
   -SDI-7.2-FP0006
JAVADOCS
EXAMPLES
   -SDI-7.2-FP0006
EMBEDDED WEB PLATFORM
AMC
   Deferred: false
```

#### Step 2

Downloaded TDI 7.2 from xTreme Leverage

```
Part Number: CIS7SML
Description: IBM Security Directory Integrator V7.2 for AIX Multilingual
File Name: SDI_7.2_AIX_ML.tar
```

Downloaded FP 6

```
Fix Pack Central: 7.2.0-ISS-SDI-FP0006
File Name: 7.2.0-ISS-SDI-FP0006.zip
```

#### Step 3

1. Unzip installers from

```
/software/SDI_72
/software/SDI_72FP6
```

2. Run TDI Base Installer

```
[g43pa00009854.az3.ash.cpc.ibm.com:/software/SDI_72/aix_ppc] ./install_sdiv72_aix_ppc.bin
```

(See install_sdiv72_aix_ppc.bin.txt)

3. Move UpdateInstaller.jar

```
[g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/TDI/V7.2/maintenance] mv UpdateInstaller.jar UpdateInstaller.jar.bak
[g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/TDI/V7.2/maintenance] cp /software/SDI_72FP6/7.2.0-ISS-SDI-FP0006/UpdateInstaller.jar . 
```

4. Update TDI to FixPack 6

```
[g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/TDI/V7.2/bin] ./applyUpdates.sh -update /software/SDI_72FP6/7.2.0-ISS-SDI-FP0006/SDI-7.2-FP0006.zip
CTGDKO023I Applying fix 'SDI-7.2-FP0006' using backup directory '/opt/IBM/TDI/V7.2/maintenance/BACKUP/SDI-7.2-FP0006'.
CTGDKO027I Updating BASE.
CTGDKO027I Updating SERVER.
CTGDKO027I Updating CE.
CTGDKO027I Updating EXAMPLES.
```

5. Verify Version + FixPack level

```
[g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/TDI/V7.2/bin] ./applyUpdates.sh -queryreg
Information from .registry file in: /opt/IBM/TDI/V7.2
Edition: Identity
Level: 7.2.0.6
License: FullFixes Applied
=-=-=-=-=-=-=
SDI-7.2-FP0006(7.2.0.0) Components Installed
=-=-=-=-=-=-=-=-=-=
BASE
   -SDI-7.2-FP0006
SERVER
   -SDI-7.2-FP0006
CE
   -SDI-7.2-FP0006
JAVADOCS
EXAMPLES
   -SDI-7.2-FP0006
EMBEDDED WEB PLATFORM
AMC
   Deferred: false 
```
 