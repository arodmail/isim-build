# Configure LDAPS

#### Step 1

Enabled port 636 in pre-prod:

1) Steps to create self-signed SSL certificate and Key Store using gskit are in:

```
root@g43pa00009852:/db2/idsinst/idsslapd-idsinst/etc/ssl/create-ssl.sh
```

Output from steps:

```
root@g43pa00009852:/db2/idsinst/idsslapd-idsinst/etc/ssl$ ll
total 80
drwx------    2 idsinst  idsldap         256 Oct 12 04:11PM ./
drwxrwx---    3 idsinst  idsldap        4096 Oct 12 04:14PM ../
-rwxr-x---    1 idsinst  idsldap         854 Oct 12 04:11PM create-ssl.sh
-rw-------    1 idsinst  idsldap         814 Oct 12 04:11PM isim_ldap_2048.arm
-rw-------    1 idsinst  idsldap        1326 Oct 12 04:11PM isim_ldap_4096.arm
-rw-------    1 idsinst  idsldap          88 Oct 12 04:11PM key.crl
-rw-------    1 idsinst  idsldap       10088 Oct 12 04:11PM key.kdb
-rw-------    1 idsinst  idsldap          88 Oct 12 04:11PM key.rdb
-rw-------    1 idsinst  idsldap         193 Oct 12 04:11PM key.sth 
```

2) Updated /db2/idsinst/idsslapd-idsinst/etc/ibmslapd.conf, and restarted /opt/IBM/ldap/V6.4/sbin/64/ibmslapd:


```
dn: cn=SSL, cn=Configuration
cn: SSL
ibm-slapdSecurePort: 636
#ibm-slapdSecurity must be one of none/SSL/SSLOnly/TLS/SSLTLS
ibm-slapdSecurity: SSLTLS
#ibm-slapdSecurityProtocol must be one or more protocols from SSLV3,TLS10,TLS11,TLS12
ibm-slapdSecurityProtocol: TLS12
#ibm-slapdSslAuth must be one of serverAuth/serverClientAuth
ibm-slapdSslAuth: serverauth
ibm-slapdSslCertificate: ISIM_LDAP_4096
ibm-slapdSslCipherSpec: AES
ibm-slapdSslCipherSpec: AES-128
ibm-slapdSslFIPSProcessingMode: false
ibm-slapdSslKeyDatabase: /db2/idsinst/idsslapd-idsinst/etc/ssl/key.kdb
ibm-slapdSslKeyDatabasePW: {AES256}<redacted>
ibm-slapdSslPKCS11AcceleratorMode: none
ibm-slapdSslPKCS11Enabled: false
ibm-slapdSslPKCS11Keystorage: false
ibm-slapdSslPKCS11Lib: libcknfast.so
ibm-slapdSslPKCS11TokenLabel: none
objectclass: top
objectclass: ibm-slapdConfigEntry
objectclass: ibm-slapdSSL
 ```

#### Step 2

HR Feed Solution Updates

1. Configure stunnel

```
root@g43pa00009852:/$ find / -name stunnel.conf
/opt/freeware/etc/stunnel/stunnel.conf
```

Comment out blueldap section

```
; Service-level configuration
;[blueldap]
;accept  = localhost:636
;connect = bluepages.ibm.com:636

[JNDI]
accept = localhost:9080
connect = 9.214.56.189:443

[bluepages]
client = yes
sslVersion = all
accept  = 12389
connect = 9.15.48.48:636
connect = 9.15.0.48:636
failover = rr[itim_ldap]
client = yes
sslVersion = all
accept = 127.0.0.1:11389
connect = 9.214.57.92:636
```

Restart stunnel

```
root@g43pa00009852:/$ ps -ef | grep stunnel 
    root 39518580        1   0 04:00:52 PM      -  0:00 stunnel
root@g43pa00009852:/$ kill -9 39518580
root@g43pa00009852:/$ stunnel
```

2. Set LDAPS url in TDI solution properties

```
root@g43pa00009852:/opt/tdihr$ ls -ltro *.properties
-rw-r--r--    1 root            115 Oct 17 02:30PM bp2isimtmp.properties
-rw-r--r--    1 root          18078 Oct 26 02:59PM solution.properties
-rw-r-----    1 rushim         1947 Nov 07 01:56AM bp2isim.properties
```

```
root@g43pa00009852:/opt/tdihr$ vi bp2isim.properties
```

```
 LDAP_URL=ldaps://127.0.0.1:636
 LDAP_CN:cn=root
 #{protect}-LDAP_PW={protect}-LDAP_pw=**********
 LDAP_USERBASE:ou=people,erglobalid=00000000000000000000,ou=IBM,dc=itim
 LDAP_PW=****** 
```

3. Update TDI JRE security jars (to expand supported cipher suites)

```
root@g43pa00009852:/opt/IBM/TDI/V7.2/jvm/jre/lib/security$ ls -ltro *.jar
-rwxr-xr-x    1 root           3726 Nov 07 03:29PM local_policy.jar
-rwxr-xr-x    1 root           3715 Nov 07 03:29PM US_export_policy.jar
```

4. Test run HR Feed and check logs

```
root@g43pa00009852:/opt/tdihr$ ./start_bp2isim_hrLoad
```

```
root@g43pa00009852:/opt/tdihr$ vi Bluepages2ITIM.log 
```
 
Diagnostics/Test

A. Check basic LDAPS connection

```
root@g43pa00009852:/opt/tdihr$ openssl s_client -connect localhost:636 -showcerts
```

```
CONNECTED(00000004)
Can't use SSL_get_servername
depth=0 C = US, O = IBM, CN = g43pa00009852
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, O = IBM, CN = g43pa00009852
verify return:1
---
Certificate chain
 0 s:C = US, O = IBM, CN = g43pa00009852
   i:C = US, O = IBM, CN = g43pa00009852
-----BEGIN CERTIFICATE-----
MIIFKjCCAxKgAwIBAgIIKaHh+PPsCakwDQYJKoZIhvcNAQELBQAwMzELMAkGA1UE
BhMCVVMxDDAKBgNVBAoTA0lCTTEWMBQGA1UEAxMNZzQzcGEwMDAwOTg1MjAeFw0y
MzEwMTExNjExNTJaFw00MzEwMTIxNjExNTJaMDMxCzAJBgNVBAYTAlVTMQwwCgYD
VQQKEwNJQk0xFjAUBgNVBAMTDWc0M3BhMDAwMDk4NTIwggIiMA0GCSqGSIb3DQEB
AQUAA4ICDwAwggIKAoICAQD0r1sm2pvSOxGAx8SX7Sl3WxEdHw0N2sc2Od2DDpF0
5SGQfDz9Aob6SuETv4sIFbQ2M4lU9eF1fd+OMYc3rjvSUZXOOXfLn6/eE40+jQY3
dephyEsq/7tx63n+bMusNQVuyEK6MPcKyvcYGDSWLi7IR7JOTTf0F6Rq5QuYlYLf
AmODPjkwuWVA4uWuruNxnxfvMPeVnZYw7e7Oc85xNxGL9sUcOgqdQW9EP+w9zOwP
VE1Equ7ECoSTZmkDaWysNvqMhkV3dZ3+JwEZpQnK9Roc3heWX00zd3Rm2/59v0Du
G7H3f/aRyY2veMkszD4zcAB4pvhIxACZc8aIA966nd5jaKDJEEVFqBPDs1x8LcAd
1VJE6hEYpITRZDriWaPkP2T3nVyIuhugw+JWGRT6JGsLTCXZ3mp8etX+dkPiho8g
w4z8zbp9JvttfRaFEHYbUNmfc3vDyaLAM4dyFCVWEQ/M++haO80EJ7m1c/O28r2G
zOpKKt6mny4YmKrYmyjKo177scT2DHDygBLVMVoBM7PDVoDjC169RWB38wS22ua2
VSOX8AUq+lG6nxnRTzqk1YCsywsn25hip9jXaX5qc6QlZ6SKDbprae4OWZMj5NM2
r50l24ho0wpZri9Zqwg4/jh4nLLZ/QWlnh4mRGeb1nodk+m/LMc4plVkYe4LqgqL
dwIDAQABo0IwQDAdBgNVHQ4EFgQUAN0lyBgMi7Ce+7MI0syeX88ZgMMwHwYDVR0j
BBgwFoAUAN0lyBgMi7Ce+7MI0syeX88ZgMMwDQYJKoZIhvcNAQELBQADggIBAIKp
QopYi0wjUIbQyuOdkXbKtoHTQrabWG9nVR78y2mYy3THjtGJpLl9niGJ0YLAE5SS
O/QmtEKvQBFHDyZnyrtMC37dhJEKglUmHU+QY/ih/ZP1qcyNlJ3jlbH2tEsX+kGF
f73OMLQFvnlnbQ4FYwwfVfIXKM0gV7V6SsIvd5NWEm414xuGBKBtOGdxki1mJ6Sl
7iqUmCKAR+z9qpejr9A0zqSkbUjyTaLTxAx/YsAJfO2cZMn8THG+ZnR6IsKVjL88
GsKQI1anTZlosjCfEN4feUec99C1EHvAJ97D+1IHSkRu3ldouVf86s9pG4GiI4zO
D2JMHxXaubm8qf5Szzv+MgrDMmhD++ZVDoUxN6VZz7idAg6ycco/w9MlXzKMJ3Gc
C9NX9785Y9oIyFDwONNAYIAJAp1yx/XVIInpgKWWdxSD4aj/595a+G9kQHXUtTLE
F3Y5RhYKA8jCVTZNaTDj+k5x7wGpmP1mIAVjQ/+pvaUkx/vSDChVIAZTibqZ6tGR
uy4505Rgai+K7diihgJQO1VpbwofFstrITa8zwVtxye3fdTMM9w1IOYbQkCdHUqz
pp4hBacu89c/1udnzRuNH9tTJ55XdG2vdCdeX5Fk289+Cx6fRd2oqaWoTVLI62JO
hAvVtCqh9oSszcAhEx6+xXgpAKM5v583RxEfrRo5
-----END CERTIFICATE-----
---
Server certificate
subject=C = US, O = IBM, CN = g43pa00009852issuer=C = US, O = IBM, CN = g43pa00009852---
No client certificate CA names sent
---
SSL handshake has read 1513 bytes and written 895 bytes
Verification error: self signed certificate
---
New, SSLv3, Cipher is AES128-SHA
Server public key is 4096 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : AES128-SHA
    Session-ID: F27D10004866955486101B2DFC7A5AA152327C878E74B4C7B48CF4589FDA55B7
    Session-ID-ctx: 
    Master-Key: 10F1F556C52E875B8B0C28DD18D389D418145CFA1B2269B1C7D08D58568E11356388415F76E3F3FD1AB8ADA813C2BBEA
    Start Time: 1699381769
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
    Extended master secret: yes
---
```

B. Debug Connect through TDI

```
root@g43pa00009852:/opt/tdihr$ /opt/IBM/TDI/V7.2/ibmdisrv -s /opt/tdihr -c bp2isim.xml -r debugConnect 
```

```
IBMJSSEProvider2 Build-Level: -20130318
IBMJSSEProvider2 Build-Level: -20130318
Installed Providers = 
        IBMJSSE2
        IBMJCE
        IBMJGSSProvider
        IBMCertPath
        IBMSASL
        IBMXMLCRYPTO
        IBMXMLEnc
        IBMSPNEGO
        SUN
keyStore is: /opt/IBM/TDI/V7.2/jvm/jre/lib/security/cacerts
keyStore type is: 
keyStore provider is: 
init keymanager of type IbmX509
keyStore is: /opt/IBM/TDI/V7.2/jvm/jre/lib/security/cacerts
keyStore type is: 
keyStore provider is: 
trustStore is: bp2isim.jks
trustStore type is: jks
trustStore provider is: 
init truststore
adding as trusted cert:
  Subject: CN=IBM INTERNAL INTERMEDIATE CA, O=International Business Machines Corporation, C=US
  Issuer:  CN=IBM Internal Root CA, O=International Business Machines Corporation, C=US
  Algorithm: RSA; Serial number: 0x11
  Valid from Wed Feb 25 05:00:00 UTC 2015 until Sun Dec 31 04:59:59 UTC 2023adding as trusted cert:
  Subject: O=Internet Widgits Pty Ltd, ST=Some-State, C=AU
  Issuer:  O=Internet Widgits Pty Ltd, ST=Some-State, C=AU
  Algorithm: RSA; Serial number: 0xbcac74e099a222d8
  Valid from Thu Aug 15 11:43:42 UTC 2019 until Sun Aug 12 11:43:42 UTC 2029adding as trusted cert:
  Subject: CN=g43pa00009852, O=IBM, C=US
  Issuer:  CN=g43pa00009852, O=IBM, C=US
  Algorithm: RSA; Serial number: 0x29a1e1f8f3ec09a9
  Valid from Wed Oct 11 16:11:52 UTC 2023 until Mon Oct 12 16:11:52 UTC 2043adding as trusted cert:
  Subject: CN=IBM Internal Root CA, O=International Business Machines Corporation, C=US
  Issuer:  CN=IBM Internal Root CA, O=International Business Machines Corporation, C=US
  Algorithm: RSA; Serial number: 0x0
  Valid from Tue May 31 04:00:00 UTC 2011 until Tue Jan 02 03:59:59 UTC 2035adding as trusted cert:
  Subject: CN=DigiCert Global Root CA, OU=www.digicert.com, O=DigiCert Inc, C=US
  Issuer:  CN=DigiCert Global Root CA, OU=www.digicert.com, O=DigiCert Inc, C=US
  Algorithm: RSA; Serial number: 0x83be056904246b1a1756ac95991c74a
  Valid from Fri Nov 10 00:00:00 UTC 2006 until Mon Nov 10 00:00:00 UTC 2031SSLContextImpl:  Using X509ExtendedKeyManager com.ibm.jsse2.uc
SSLContextImpl:  Using X509TrustManager com.ibm.jsse2.yc
JsseJCE:  Using SecureRandom IBMSecureRandom from provider IBMJCE version 1.7
trigger seeding of SecureRandom
done seeding SecureRandom
JsseJCE:  Using SecureRandom IBMSecureRandom from provider IBMJCE version 1.7
JsseJCE:  Using KeyAgreement ECDH from provider IBMJCE version 1.7
JsseJCE:  Using signature SHA1withECDSA from provider TBD via init 
JsseJCE:  Using signature NONEwithECDSA from provider TBD via init 
JsseJCE:  Using KeyFactory EC from provider IBMJCE version 1.7
JsseJCE:  Using KeyPairGenerator EC from provider TBD via init 
JsseJce:  EC is available
JsseJCE:  Using cipher AES/CBC/NoPadding from provider TBD via init 
CipherBox:  Using cipher AES/CBC/NoPadding from provider from init IBMJCE version 1.7
JsseJCE:  Using cipher AES/GCM/NoPadding from provider TBD via init 
IBMJSSE2 will not enable CBC protection
IBMJSSE2 will allow RFC 5746 renegotiation per com.ibm.jsse2.renegotiate set to none or default
IBMJSSE2 will not require renegotiation indicator during initial handshake per com.ibm.jsse2.renegotiation.indicator set to OPTIONAL or default taken
IBMJSSE2 will not perform identity checking against the peer cert check during renegotiation per com.ibm.jsse2.renegotiation.peer.cert.check set to OFF or defaultIs initial handshake: true
Ignoring unsupported cipher suite: SSL_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
Ignoring unsupported cipher suite: SSL_ECDHE_RSA_WITH_AES_256_CBC_SHA384
Ignoring unsupported cipher suite: SSL_RSA_WITH_AES_256_CBC_SHA256
Ignoring unsupported cipher suite: SSL_ECDH_ECDSA_WITH_AES_256_CBC_SHA384
Ignoring unsupported cipher suite: SSL_ECDH_RSA_WITH_AES_256_CBC_SHA384
Ignoring unsupported cipher suite: SSL_DHE_RSA_WITH_AES_256_CBC_SHA256
Ignoring unsupported cipher suite: SSL_DHE_DSS_WITH_AES_256_CBC_SHA256
Ignoring unsupported cipher suite: SSL_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
Ignoring unsupported cipher suite: SSL_ECDHE_RSA_WITH_AES_128_CBC_SHA256
Ignoring unsupported cipher suite: SSL_RSA_WITH_AES_128_CBC_SHA256
Ignoring unsupported cipher suite: SSL_ECDH_ECDSA_WITH_AES_128_CBC_SHA256
Ignoring unsupported cipher suite: SSL_ECDH_RSA_WITH_AES_128_CBC_SHA256
Ignoring unsupported cipher suite: SSL_DHE_RSA_WITH_AES_128_CBC_SHA256
Ignoring unsupported cipher suite: SSL_DHE_DSS_WITH_AES_128_CBC_SHA256
%% No cached client session
*** ClientHello, TLSv1
RandomCookie:  GMT: 1699316329 bytes = { 68, 215, 52, 200, 43, 84, 110, 167, 172, 252, 68, 182, 224, 124, 23, 228, 41, 74, 123, 138, 139, 51, 156, 149, 108, 222, 121, 159 }
Session ID:  {}
Cipher Suites: [TLS_EMPTY_RENEGOTIATION_INFO_SCSV, SSL_ECDHE_ECDSA_WITH_AES_256_CBC_SHA, SSL_ECDHE_RSA_WITH_AES_256_CBC_SHA, SSL_RSA_WITH_AES_256_CBC_SHA, SSL_ECDH_ECDSA_WITH_AES_256_CBC_SHA, SSL_ECDH_RSA_WITH_AES_256_CBC_SHA, SSL_DHE_RSA_WITH_AES_256_CBC_SHA, SSL_DHE_DSS_WITH_AES_256_CBC_SHA, SSL_ECDHE_ECDSA_WITH_AES_128_CBC_SHA, SSL_ECDHE_RSA_WITH_AES_128_CBC_SHA, SSL_RSA_WITH_AES_128_CBC_SHA, SSL_ECDH_ECDSA_WITH_AES_128_CBC_SHA, SSL_ECDH_RSA_WITH_AES_128_CBC_SHA, SSL_DHE_RSA_WITH_AES_128_CBC_SHA, SSL_DHE_DSS_WITH_AES_128_CBC_SHA, SSL_ECDHE_ECDSA_WITH_RC4_128_SHA, SSL_ECDHE_RSA_WITH_RC4_128_SHA, SSL_RSA_WITH_RC4_128_SHA, SSL_ECDH_ECDSA_WITH_RC4_128_SHA, SSL_ECDH_RSA_WITH_RC4_128_SHA, SSL_ECDHE_ECDSA_WITH_3DES_EDE_CBC_SHA, SSL_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA, SSL_RSA_WITH_3DES_EDE_CBC_SHA, SSL_ECDH_ECDSA_WITH_3DES_EDE_CBC_SHA, SSL_ECDH_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_RSA_WITH_3DES_EDE_CBC_SHA, SSL_DHE_DSS_WITH_3DES_EDE_CBC_SHA, SSL_RSA_WITH_RC4_128_MD5]
Compression Methods:  { 0 }
Extension elliptic_curves, curve names: {secp256r1, secp192r1, secp224r1, secp384r1, secp521r1, secp160k1, secp160r1, secp160r2, secp192k1, secp224k1, secp256k1}
Extension ec_point_formats, formats: [uncompressed]
***
[write] MD5 and SHA1 hashes:  len = 135
0000: 01 00 00 83 03 01 65 4a  82 69 44 d7 34 c8 2b 54  ......eJ.iD.4..T
0010: 6e a7 ac fc 44 b6 e0 7c  17 e4 29 4a 7b 8a 8b 33  n...D......J...3
0020: 9c 95 6c de 79 9f 00 00  38 00 ff c0 0a c0 14 00  ..l.y...8.......
0030: 35 c0 05 c0 0f 00 39 00  38 c0 09 c0 13 00 2f c0  5.....9.8.......
0040: 04 c0 0e 00 33 00 32 c0  07 c0 11 00 05 c0 02 c0  ....3.2.........
0050: 0c c0 08 c0 12 00 0a c0  03 c0 0d 00 16 00 13 00  ................
0060: 04 01 00 00 22 00 0a 00  18 00 16 00 17 00 13 00  ................
0070: 15 00 18 00 19 00 0f 00  10 00 11 00 12 00 14 00  ................
0080: 16 00 0b 00 02 01 00                               .......Thread-17, WRITE: TLSv1 Handshake, length = 135
[Raw write]: length = 140
0000: 16 03 01 00 87 01 00 00  83 03 01 65 4a 82 69 44  ...........eJ.iD
0010: d7 34 c8 2b 54 6e a7 ac  fc 44 b6 e0 7c 17 e4 29  .4..Tn...D......
0020: 4a 7b 8a 8b 33 9c 95 6c  de 79 9f 00 00 38 00 ff  J...3..l.y...8..
0030: c0 0a c0 14 00 35 c0 05  c0 0f 00 39 00 38 c0 09  .....5.....9.8..
0040: c0 13 00 2f c0 04 c0 0e  00 33 00 32 c0 07 c0 11  .........3.2....
0050: 00 05 c0 02 c0 0c c0 08  c0 12 00 0a c0 03 c0 0d  ................
0060: 00 16 00 13 00 04 01 00  00 22 00 0a 00 18 00 16  ................
0070: 00 17 00 13 00 15 00 18  00 19 00 0f 00 10 00 11  ................
0080: 00 12 00 14 00 16 00 0b  00 02 01 00              ............[Raw read]: length = 5
0000: 15 03 03 00 02                                     .....[Raw read]: length = 2
0000: 02 28                                              ..Thread-17, READ: TLSv1.2 Alert, length = 2
Thread-17, RECV TLSv1 ALERT:  fatal, handshake_failure
Thread-17, called closeSocket()
Thread-17, handling exception: javax.net.ssl.SSLHandshakeException: Received fatal alert: handshake_failure
```

#### Step 3

Basic connect/search tests 

A. LDAP Search (over LDAPS port 636)

```
sroot@g43pa00009852:/home/sroot$ alias
```

```
lds='ldapsearch -Z -h localhost -p 636 -K /home/sroot/.ssl/clientkey.kdb' 
```

```
sroot@g43pa00009852:/home/sroot$ lds -b dc=itim -s sub "(mail=arodrigu@us.ibm.com)" 
```
Note: Returns a full `arodrigu@us.ibm.com` BluePages record


B. LDAP Search (over LDAP port 389):

```
sroot@g43pa00009852:/home/sroot$ ldapsearch -b dc=itim -s sub "(mail=arodrigu@us.ibm.com)" 
```
Note: Returns a full `arodrigu@us.ibm.com` BluePages record

#### Step 4

To restart ibmslapd

```
root@g43pa00009852:/opt/tdihr$ ps -ef | grep slapd
 idsinst 23658994        1   0 03:58:59 PM      -  0:01 /opt/IBM/ldap/V6.4/sbin/64/ibmslapd -I idsinst 
```

Remove /64 directory and add -k (for kill)

```
root@g43pa00009852:/opt/tdihr$ /opt/IBM/ldap/V6.4/sbin/ibmslapd -I idsinst -k
```

Restart

```
root@g43pa00009852:/opt/tdihr$ /opt/IBM/ldap/V6.4/sbin/ibmslapd -I idsinst
```

#### Step 5

Add a certificate to the TDI Java Key Store:

Launch IKey Manager

```
sroot@g43pa00009852:/home/sroot$ export DISPLAY=localhost:10.0 
```

```
sroot@g43pa00009852:/home/sroot$ echo $DISPLAY         
localhost:10.0 
```

```
sroot@g43pa00009852:/home/sroot$ xauth list 
g43pa00009852/unix:10  MIT-MAGIC-COOKIE-1  360384582dacfc6756e9091c7c490c77 
```

```
sroot@g43pa00009852:/home/sroot$ xauth add g43pa00009852/unix:10  MIT-MAGIC-COOKIE-1  360384582dacfc6756e9091c7c490c77 
```

```
ssh -X -i ~/.ssh/ldap-host-ssh-key sroot@g43pa00009852.az3.ash.cpc.ibm.com 
```

Launch the GUI:
```
/opt/IBM/TDI/V7.2/jvm/jre/bin/ikeyman 
```

Open the keystore:

```
/opt/tdihr/bp2isim.jks 
```

#### Step 6

Create a .cer file to add to the keystore

Test the secure connection using openssl, -showcerts option

```
root@g43pa00009852:/opt/tdihr$ openssl s_client -connect localhost:636 -showcerts
CONNECTED(00000004)
Can't use SSL_get_servername
depth=0 C = US, O = IBM, CN = g43pa00009852
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, O = IBM, CN = g43pa00009852
verify return:1
---
Certificate chain
 0 s:C = US, O = IBM, CN = g43pa00009852
   i:C = US, O = IBM, CN = g43pa00009852
-----BEGIN CERTIFICATE-----
MIIDKjCCAhKgAwIBAgIIV7ZrEwQWp8wwDQYJKoZIhvcNAQELBQAwMzELMAkGA1UE
BhMCVVMxDDAKBgNVBAoTA0lCTTEWMBQGA1UEAxMNZzQzcGEwMDAwOTg1MjAeFw0y
MzExMDgxNTQ4MThaFw00MzExMDkxNTQ4MThaMDMxCzAJBgNVBAYTAlVTMQwwCgYD
VQQKEwNJQk0xFjAUBgNVBAMTDWc0M3BhMDAwMDk4NTIwggEiMA0GCSqGSIb3DQEB
AQUAA4IBDwAwggEKAoIBAQC1KtCAlLhrLai2sKG2+Pj4sLXKa9cxgyjZjDYXf1y9
Nv01LIMjfORv/TJJfv7Snnq4VKZklgQy3pxu3lZd8qFSPUhPmA5wg93SgTicTTKJ
dZOkCxHYZdDZ1qq/CJpzOTwpTeOVu7VyPvjnPi84p1Ka7QDUFPk7tF0vl6PVX6lg
ZJ9NrB33HscZmP/bzUQfjEJc07KBHUl2jMLT9aHmPSlDNb/8gy8Yb7knDGHkakec
ji3pU504OJQjwbUMoK7ISjAwVMRtHlnR4lVQWn0yeGOMsxN/wuijSmlaMrXuAdLS
Cx/E3t4i7jb0uKNXM3qGUPsvGE5cbIrpzd/Yxs6h4tHNAgMBAAGjQjBAMB0GA1Ud
DgQWBBT99HFf96p9TSJukXT2ojjcFekWQjAfBgNVHSMEGDAWgBT99HFf96p9TSJu
kXT2ojjcFekWQjANBgkqhkiG9w0BAQsFAAOCAQEAqJFyac2VoXE6FQ97UVRCtmAr
+4eenRhrTw03gDsGG3hCcnQBupBIWz7kluzrb/oKk6QCM4PZGNJG0cT17NFDurOn
fI8Giwjgz21FxrbB8sRR7fNm2WJbpqfHS/4EcgmYIha5aQ8nk8OZWKvgad3NxB9j
2KIA0NtUzg/tFezLCkKrKCxQOZLjOdz/tCU7M+YrAf4g2pYw4onCT9TXoGBtFH/q
+WvBPiqIAUlSiUqEWPKtFnjFejGJWUPuO84kEaXLUs3wW32DcsObRPoa4zZna4iA
liDnlwXErqtpvBZY5JvoU5pNE4DW2W2FV8WRorHA4cykK5iE5k/NDSnLbOfFQA==
-----END CERTIFICATE-----
---
Server certificate
subject=C = US, O = IBM, CN = g43pa00009852issuer=C = US, O = IBM, CN = g43pa00009852---
No client certificate CA names sent
---
SSL handshake has read 1001 bytes and written 639 bytes
Verification error: self signed certificate
---
New, SSLv3, Cipher is AES128-SHA
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : AES128-SHA
    Session-ID: 659FD0B4F48BDF76E9304ED219693EF8082E17F12E108A99CD0BEEC2544B2143
    Session-ID-ctx: 
    Master-Key: 164617DBE5A7D71CC3F2C223A0A990F1521417F394F1C60C6DF4706D5499183192629D0E68D42B2D2E4BF90327C4A572
    Start Time: 1699577735
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
    Extended master secret: yes
---
```

Create a .cer file from BEGIN/END:

```
root@g43pa00009852:/opt/tdihr$ vi itimldap-1.cer 
```

#### Step 7

Options for the gsk8capicmd_64 tool/cmd:

```
/opt/IBM/db2/V11.1/gskit/bin/gsk8capicmd_64 -cert -create  -?
```

Output:

```
-Command usage-

-db | -crypto         Required
-tokenlabel           Optional if -crypto present
-label                Optional
-pw | -stashed        Optional
-dn | -template       Optional
-type                 Optional if -db present <cms | kdb>
-expire               Optional
-size                 Optional
-x509version          Optional <1 | 2 | 3>
-default_cert         Optional <yes | no> [Deprecated]
-ca                   Optional <true | false>
-sig_alg | -sigalg    Optional < | md5 | MD5_WITH_RSA | MD5WithRSA | sha1 | SHA_WITH_RSA | SHAWithRSA | SHA1WithRSA | sha224 | SHA224_WITH_RSA | SHA224WithRSA | sha256 | SHA256_WITH_RSA | SHA256WithRSA | sha3_256 | SHA3_256WithRSA | sha384 | SHA384_WITH_RSA | SHA384WithRSA | sha3_384 | SHA3_384WithRSA | sha512 | SHA512_WITH_RSA | SHA512WithRSA | sha3_512 | SHA3_512WithRSA | RSASSAPSS | RSASSAPSSPSS | SHA224_WITH_RSASSAPSS | SHA224WithRSASSAPSS | SHA256_WITH_RSASSAPSS | SHA256WithRSASSAPSS | SHA384_WITH_RSASSAPSS | SHA384WithRSASSAPSS | SHA512_WITH_RSASSAPSS | SHA512WithRSASSAPSS | SHA3_256WithRSASSAPSS | SHA3_384WithRSASSAPSS | SHA3_512WithRSASSAPSS | SHA_WITH_DSA | SHA1WithDSA | SHAWithDSA | SHA256WithDSA | SHA1WithECDSA | EC_ecdsa_with_SHA1 | SHA224WithECDSA | EC_ecdsa_with_SHA224 | SHA256WithECDSA | EC_ecdsa_with_SHA256 | SHA384WithECDSA | EC_ecdsa_with_SHA384 | SHA512WithECDSA | EC_ecdsa_with_SHA512 | SHA3_256WithECDSA | SHA3_384WithECDSA | SHA3_512WithECDSA | DH | Kyber | Dilithium | SHA256WithDilithium | SHA384WithDilithium | SHA512WithDilithium>
-ca_label             Optional
-san_dnsname          Optional
-san_emailaddr        Optional
-san_ipaddr           Optional
-certpolicy           Optional
-eku                  Optional <ocspSigning, timeStamping, emailProtection, codeSigning, clientAuth, serverAuth, SSLStepUpApproval, any>
-ku                   Optional <digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment, keyAgreement, keyCertSign, cRLSign, encipherOnly, decipherOnly>
-template             Optional
-secondarydb          Optional if -crypto present
-secondarydbpw        Optional if -secondarydb present
-secondarydbtype      Optional if -secondarydb present
-pqcdemo              Optional
-pqcdemor             Optional 
```

Added -sigalg SHA256_WITH_RSA option to the .sh script:

```
root@g43pa00009852:/db2/idsinst/idsslapd-idsinst/etc/ssl/create-ssl.sh
```

```
/opt/IBM/db2/V11.1/gskit/bin/gsk8capicmd_64 -cert -create -db key-2.kdb -pw ******** -label ISIM_LDAP_2048 -dn "CN=g43pa00009852,O=IBM,C=US" -default_cert yes -expire 7305 -size 2048 -sigalg SHA256_WITH_RSA 
```

#### Step 8

Use OpenSSL:

1. Generate private key

```
root@g43pa00009852:/home/root/ssl_certs$ openssl genrsa -out mykey.key 2048
```

```
Generating RSA private key, 2048 bit long modulus (2 primes)
................................................................+++++
...................................................+++++
e is 65537 (0x010001)
```

2. Generate public key

```
root@g43pa00009852:/home/root/ssl_certs$ openssl req -new -key "mykey.key" -out "2048_key.csr"  
```

```
Can't load /home/root/.rnd into RNG
1:error:2406F079:random number generator:RAND_load_file:Cannot open file:crypto/rand/randfile.c:98:Filename=/home/root/.rnd
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:CA
Locality Name (eg, city) []:Los Angeles
Organization Name (eg, company) [Internet Widgits Pty Ltd]:IBM
Organizational Unit Name (eg, section) []:CIO
Common Name (e.g. server FQDN or YOUR name) []:g43pa00009852.az3.ash.cpc.ibm.com
Email Address []:arodrigu@us.ibm.com
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []: 
```

3. Files generated

```
root@g43pa00009852:/home/root/ssl_certs$ ls -o
total 16
-rw-------    1 root           1078 Nov 14 07:24PM 2048_key.csr
-rw-------    1 root           1679 Nov 14 07:21PM mykey.key 
```

4. Generate X509 certificate:

```
root@g43pa00009852:ssl_certs$ openssl x509 -req -days 365 -in "2048_key.csr" -signkey "mykey.key"  -out "X509.crt"
```

```
Signature ok
subject=C = US, ST = CA, L = Los Angeles, O = IBM, OU = CIO, CN = g43pa00009852.az3.ash.cpc.ibm.com, emailAddress = arodrigu@us.ibm.com
Getting Private key 
```

#### Step 9

Create a pkcs12 keystore with the public/private keys:

```
root@g43pa00009852:/home/root/ssl_certs$ openssl pkcs12 -export -in X509.crt -inkey mykey.key -out keystore.p12 
Enter Export Password: 
Verifying - Enter Export Password:
```

```
root@g43pa00009852:/home/root/ssl_certs$ ls -o
total 32
-rw-------    1 root           1078 Nov 14 07:24PM 2048_key.csr
-rw-------    1 root           2565 Nov 14 07:50PM keystore.p12 
-rw-------    1 root           1679 Nov 14 07:21PM mykey.key
-rw-------    1 root           1363 Nov 14 07:25PM X509.crt 
```

Verify contents:

```
root@g43pa00009852:/home/root/ssl_certs$ keytool -v -list -storetype pkcs12 -keystore keystore.p12  
```

```
Enter keystore password:  

Keystore type: PKCS12
Keystore provider: IBMJCE

Your keystore contains 1 entryAlias name: 1

Creation date: Nov 14, 2023
Entry type: keyEntry
Certificate chain length: 1
Certificate[1]:
Owner: EMAILADDRESS=arodrigu@us.ibm.com, CN=g43pa00009852.az3.ash.cpc.ibm.com, OU=CIO, O=IBM, L=Los Angeles, ST=CA, C=US
Issuer: EMAILADDRESS=arodrigu@us.ibm.com, CN=g43pa00009852.az3.ash.cpc.ibm.com, OU=CIO, O=IBM, L=Los Angeles, ST=CA, C=US
Serial number: 29ad0a872709438552eea3e3ea9308ac39bc9e19
Valid from: 11/14/23 7:25 PM until: 11/13/24 7:25 PM
Certificate fingerprints:
         MD5:  B7:F7:0C:AE:62:A8:06:08:41:3F:17:4B:97:B2:F8:68
         SHA1: 1D:7E:60:35:28:D4:AC:E5:53:5D:B2:08:3C:D9:54:03:BB:A9:02:E4
         SHA256: 66:51:AC:97:EC:92:57:45:07:C4:33:21:3A:92:E6:43:C0:23:FA:71:5A:67:34:54:AC:53:3D:E4:10:86:12:04
         Signature algorithm name: SHA256withRSA
         Version: 1
*******************************************
*******************************************
```

#### Step 10

Upgrade SDI JRE from 7 to 8

1 - Download 7.2.0-ISS-SDI-LA0023-JAVA-8SR6FP25

[https://www.ibm.com/support/fixcentral/swg/downloadFixes?parent=IBM%20Security&product=ibm/Tivoli/Security+Directory+Integrator&release=7.2.0&platform=All&function=fixId&fixids=7.2.0-ISS-SDI-LA0023-JAVA-8SR6FP25&includeRequisites=1&includeSupersedes=0&downloadMethod=http&source=fc]

2 - Unzip

```
root@g43pa00009852:/software/SDI-72-LA0023$ gunzip jre864redist.tar.gz 
```

3 - Extract

```
root@g43pa00009852:/software/SDI-72-LA0023$ tar -xvf jre864redist.tar 
```

4 - Backup the Java 7 directory

```
root@g43pa00009852:/opt/IBM/TDI/V7.2/jvm$ mv jre jre-7   
```

5 - Create a new jre directory

```
root@g43pa00009852:/opt/IBM/TDI/V7.2/jvm$ mkdir jre 
```

6 - Copy extracted JRE 8 contents

```
root@g43pa00009852:/opt/IBM/TDI/V7.2/jvm/jre$ cp -R /software/SDI-72-LA0023/jre/* . 
```

7 - Verify

```
root@g43pa00009852:/opt/IBM/TDI/V7.2/jvm/jre/bin$ ./java -version
java version "1.8.0_281"
Java(TM) SE Runtime Environment (build 8.0.6.25 - pap6480sr6fp25-20210115_01(SR6 FP25))
IBM J9 VM (build 2.9, JRE 1.8.0 AIX ppc64-64-Bit Compressed References 20201218_462060 (JIT enabled, AOT enabled)
OpenJ9   - 4c03b71
OMR      - 86a8e1a
IBM      - 8c30c56)
JCL - 20210108_01 based on Oracle jdk8u281-b09
```

#### Step 11

Apply SDI Fix Pack 9:

Download IBM Security Directory Integrator Version 7.2.0 Fix Pack 9 from

http://www.ibm.com/support/fixcentral/

Search for:  7.2.0-ISS-SDI-FP0009

Readme

https://ak-delivery04-mul.dhe.ibm.com/sar/CMA/TIA/0b4uw/1/7.2.0-ISS-SDI-FP0009_README.html

1 - Backup UpdateInstaller.jar

```
[g43pa00009853.az3.ash.cpc.ibm.com:/opt/IBM/TDI/V7.2/maintenance] mv UpdateInstaller.jar UpdateInstaller.jar.bak
```

2 - Download and extract FP 9 zip file to /maintenance directory:

```
root@g43pa00009852:/opt/IBM/TDI/V7.2/maintenance$ unzip 7.2.0-ISS-SDI-FP0009.zip 
```

```
root@g43pa00009852:/opt/IBM/TDI/V7.2/maintenance$ ls -o         
total 450328
-rwxr-xr-x    1 root          71980 Feb 17 2023    7.2.0-ISS-SDI-FP0009_README.html
-rwxr-xr-x    1 root      115198748 Nov 16 03:56PM 7.2.0-ISS-SDI-FP0009.zip
drwxr-xr-x    3 root            256 Sep 15 12:55PM BACKUP
-rwxr-xr-x    1 root           3544 Jan 25 2023    SAP_JCo_V3_SUPPORT_README.TXT
-rwxr-xr-x    1 root      115094253 Jan 25 2023    SDI-7.2-FP0009.zip
-rwxr-xr-x    1 root          95322 Dec 19 2022    UpdateInstaller.jar
-rwxr-xr-x    1 root          92385 Sep 15 12:52PM UpdateInstaller.jar.bak
```

3 - Copy unzipped .zip to /bin directory

```
root@g43pa00009852:/opt/IBM/TDI/V7.2/bin$ cp ../maintenance/SDI-7.2-FP0009.zip . 
```

4 - Apply the FP

```
root@g43pa00009852:/opt/IBM/TDI/V7.2/bin$ ./applyUpdates.sh -update SDI-7.2-FP0009.zip
CTGDKO023I Applying fix 'SDI-7.2-FP0009' using backup directory '/opt/IBM/TDI/V7.2/maintenance/BACKUP/SDI-7.2-FP0009'.
CTGDKO027I Updating BASE.
CTGDKO027I Updating SERVER.
CTGDKO027I Updating CE.
CTGDKO027I Updating EXAMPLES.
CTGDKO027I Updating JAVADOCS. 
```

5 - Update Log4J

```
root@g43pa00009852:/opt/IBM/TDI/V7.2/bin$ chmod 755 updateLog4j.sh
root@g43pa00009852:/opt/IBM/TDI/V7.2/bin$ ./updateLog4j.sh        
Successfully created new ./applyUpdates.sh
Success! 
```

5 - Verify FP applied

```
root@g43pa00009852:/opt/IBM/TDI/V7.2/bin$ ./applyUpdates.sh -queryreg 
```

```
Information from .registry file in: /opt/IBM/TDI/V7.2
Edition: Identity
Level: 7.2.0.9
License: FullFixes Applied
=-=-=-=-=-=-=
SDI-7.2-FP0009(7.2.0.6)SDI-7.2-FP0006(7.2.0.0) Components Installed
=-=-=-=-=-=-=-=-=-=
BASE
   -SDI-7.2-FP0009
   -SDI-7.2-FP0006
SERVER
   -SDI-7.2-FP0009
   -SDI-7.2-FP0006
CE
   -SDI-7.2-FP0009
   -SDI-7.2-FP0006
JAVADOCS
   -SDI-7.2-FP0009
EXAMPLES
   -SDI-7.2-FP0009
   -SDI-7.2-FP0006
EMBEDDED WEB PLATFORM
AMC
   Deferred: false 
```

#### Step 12

Re-gen key db and cert for ldapsearch (as an ldaps client):

1. Backup script

```
root@g43pa00009852:/db2/idsinst/idsslapd-idsinst/etc/ssl/client$ cp create-client-ssl.sh create-client.ssl.sh.bak 
```

2. Update script:

```
root@g43pa00009852:/db2/idsinst/idsslapd-idsinst/etc/ssl/client$ cat create-client-ssl.sh
#!/usr/bin/ksh
/opt/IBM/db2/V11.1/gskit/bin/gsk8capicmd_64 -keydb -create -db clientkey.kdb -pw ****** -type cms -stash
/opt/IBM/db2/V11.1/gskit/bin/gsk8capicmd_64 -cert -add -db clientkey.kdb -pw ****** -label ISIM_LDAP_2048 -file isim_ldap_2048.arm -format binary 
```

3. Remove previous clientkey files

```
root@g43pa00009852:/db2/idsinst/idsslapd-idsinst/etc/ssl/client$ rm clientkey*
rm: Removing clientkey.crl
rm: Removing clientkey.rdb
rm: Removing clientkey.sth 
```

4. Copy the 2048 key to client directory

```
root@g43pa00009852:/db2/idsinst/idsslapd-idsinst/etc/ssl/client$ cp ../isim_ldap_2048.arm . 
```

5. Run script:

```
root@g43pa00009852:/db2/idsinst/idsslapd-idsinst/etc/ssl/client$ ./create-client-ssl.sh     
```

6. Copy to sroot home dir ssl

```
root@g43pa00009852:/home/root$ cp /db2/idsinst/idsslapd-idsinst/etc/ssl/client/clientkey* /home/sroot/.ssl/ 
```

7. Confirm

```
root@g43pa00009852:/home/root$ ls -ltr /home/sroot/.ssl/                                                       
total 56
-rw-------    1 sroot    igachef        5088 Nov 28 12:26AM clientkey.kdb
-rw-------    1 sroot    igachef          88 Nov 28 12:26AM clientkey.crl
-rw-------    1 sroot    igachef          88 Nov 28 12:26AM clientkey.rdb
-rw-------    1 sroot    igachef         193 Nov 28 12:26AM clientkey.sth 
```

8. Run ldapsearch

```
sroot@g43pa00009852:/home/sroot$ alias | grep ldapsearch
lds='ldapsearch -Z -h localhost -p 636 -K /home/sroot/.ssl/clientkey.kdb' 
```

```
sroot@g43pa00009852:/home/sroot$ lds -b dc=itim -s sub "(mail=arodrigu@us.ibm.com)"  
```

Reponse

```
erglobalid=2694969388316574572,ou=0,ou=people,erglobalid=00000000000000000000,ou=IBM,dc=itim
passwordmodifytimestamp=20230821
mail=arodrigu@us.ibm.com
uid=9a0970897
hrcountrycode=us
workplaceindicator=M
emailaddress=arodrigu@us.ibm.com
emailaddress=us9a0970@us.ibm.com
... 
```

#### Step 13

1. Generate new cert using SHA384WithECDSA:

```
root@g43pa00009852:/db2/idsinst/idsslapd-idsinst/etc/ssl$ cat create-ssl.sh 
```

```
#!/usr/bin/ksh
/opt/IBM/db2/V11.1/gskit/bin/gsk8capicmd_64 -keydb -create -db key-1.kdb -pw ****** -type cms -stash
/opt/IBM/db2/V11.1/gskit/bin/gsk8capicmd_64 -cert -create -db key-1.kdb -pw ****** -label ISIM_LDAP_sha384-ecdsa-fqdn -dn CN=g43pa00009852.az3.ash.cpc.ibm.com,O=IBM,C=US -default_cert yes -expire 7305 -sigalg SHA384WithECDSA
/opt/IBM/db2/V11.1/gskit/bin/gsk8capicmd_64 -cert -extract -db key-1.kdb -pw ****** -label ISIM_LDAP_sha384-ecdsa-fqdn -target isim_ldap.arm -format binary 
/opt/IBM/db2/V11.1/gskit/bin/gsk8capicmd_64 -cert -list -db key-1.kdb -pw ******  
```

2. Run .sh

```
 /db2/idsinst/idsslapd-idsinst/etc/ssl$ ./create-ssl.sh 
```

3. Confirm certificate and keystore database are created:

```
root@g43pa00009852:/db2/idsinst/idsslapd-idsinst/etc/ssl$ ls -ltr key-1* *.arm
-rw-------    1 idsinst  system           88 Nov 28 03:28PM key-1.rdb
-rw-------    1 idsinst  system           88 Nov 28 03:28PM key-1.crl
-rw-------    1 idsinst  system          193 Nov 28 03:28PM key-1.sth
-rw-------    1 idsinst  system         5088 Nov 28 03:28PM key-1.kdb
-rw-------    1 root     system          524 Nov 28 03:28PM isim_ldap.arm 
```

4. Update /db2/idsinst/idsslapd-idsinst/etc/ibmslapd.conf

```
ibm-slapdSslAuth: serverauth
ibm-slapdSslCertificate: ISIM_LDAP_sha384-ecdsa-fqdn
ibm-slapdSslCipherSpec: AES
ibm-slapdSslCipherSpec: AES-128
ibm-slapdSslCipherSpec: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
ibm-slapdSslCipherSpec: TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
ibm-slapdSslCipherSpec: TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
ibm-slapdSslFIPSProcessingMode: false
ibm-slapdSslKeyDatabase: /db2/idsinst/idsslapd-idsinst/etc/ssl/key-1.kdb
```

5. Resart slapd

```
root@g43pa00009852:/opt/tdihr$ /opt/IBM/ldap/V6.4/sbin/ibmslapd -I idsinst -k 
```

```
root@g43pa00009852:/opt/tdihr$ /opt/IBM/ldap/V6.4/sbin/ibmslapd -I idsinst 
```

#### Step 14

For /home/sroot/ldap-connect Java client (test):

1. Add the SHA384WithECDSA certificate to Java 8 trustore

```
/usr/java8_64/jre/bin/keytool -import -alias ca -file /db2/idsinst/idsslapd-idsinst/etc/ssl/client/isim_ldap.arm -keystore cacerts -storepass changeit  

```
Output

```
Owner: CN=g43pa00009852.az3.ash.cpc.ibm.com, O=IBM, C=US
Issuer: CN=g43pa00009852.az3.ash.cpc.ibm.com, O=IBM, C=US
Serial number: 6db2c003129d156b
Valid from: 11/27/23 3:28 PM until: 11/28/43 3:28 PM
Certificate fingerprints:
         MD5:  EA:B9:03:95:14:3C:E5:77:73:34:66:CA:EF:36:A2:AD
         SHA1: 04:DD:37:D7:99:A8:1D:96:6A:6C:DD:55:F2:9B:05:9C:76:46:FA:5F
         SHA256: 9D:E9:53:AA:EB:52:F5:20:75:1C:41:23:32:94:B3:66:86:B6:A3:15:F1:B3:4E:74:BC:E4:BF:F8:82:4A:E8:24
         Signature algorithm name: SHA384withECDSA
         Version: 3Extensions: #1: ObjectId: 2.5.29.35 Criticality=false
AuthorityKeyIdentifier [
KeyIdentifier [
0000: 93 5a ab 83 9b 87 fc b6  61 f9 eb fd d4 4a 1b 60  .Z......a....J..
0010: af 4a 23 68                                        .J.h
]
]#2: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 93 5a ab 83 9b 87 fc b6  61 f9 eb fd d4 4a 1b 60  .Z......a....J..
0010: af 4a 23 68                                        .J.h
]
]
```

```
Trust this certificate? [no]:  yes
Certificate was added to keystore 
```

2. Run ldap-connect

```
root@g43pa00009852:/home/sroot/ldap-connect/bin$ ./run-ldap-connect.sh 
```

3. Optional add Java param

```
-Dcom.sun.jndi.ldap.object.disableEndpointIdentification=true
```
 
#### Links

GSKCapiCmd_64

https://www.ibm.com/docs/en/sdsu/8.0.1?topic=commands-gsk8capicmd-64

Directory server instance with the SSL and TLS protocols

https://www.ibm.com/docs/he/sdse/6.4.0?topic=131a-directory-server-instance-ssl-tls-protocols

Protocols and ciphers in version 6.3, Fix Pack 17 or later

https://www.ibm.com/docs/he/sdse/6.4.0?topic=dsistp-protocols-ciphers-in-version-63-fix-pack-17-later

Configuring a directory server with security protocols and ciphers

https://www.ibm.com/docs/da/sdse/6.4.0?topic=protocols-configuring-directory-server-security-ciphers

Enabling TLS 1.2 in IBM Java

https://www.ibm.com/docs/en/license-metric-tool?topic=certificate-enabling-tls-12-in-java

