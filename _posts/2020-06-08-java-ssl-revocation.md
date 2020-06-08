---
title: "How to enable SSL revocation checking."
tags:
  - java
  - ssl
  - security
---

Revocation checking is disabled by default in Java but it can be enabled by setting the correct properties.

To better understand what Java is doing under the covers during the SSL handshake, debug logging can be enabled with the following system properties:

```
-Djavax.net.debug=all -Djava.security.debug=certpath
```

The following test tries to connect with a server that uses a revoked certifcate. 

{% gist codingtim/9b3ba67625ae0c8b28df15aa8f173a99 %}

Running this test with the debug logging enabled will show a test failure. 
No exception is thrown as the default config does not do revocation checking.
The debug logging of the certpath shows 7 checkers but no instance of RevocationChecker.
Enabling the revocation checking with the following properties will make the test succeed: 

```
System.setProperty("com.sun.net.ssl.checkRevocation", "true");
Security.setProperty("ocsp.enable", "true");
```
    
The debug logging now shows an additional checker of type RevocationChecker.
This checker uses the configured ocsp to find out the certificate has been revoked.

Note that the ocsp.enable property is a Security property. 
It is only possible to set this programmatically or through the java.security file that is by default located in the java installation directory.
    
Documentation on the Oracle site can be found online:
- Java 11: [Client-Driven OCSP and OCSP Stapling](https://docs.oracle.com/en/java/javase/11/security/java-secure-socket-extension-jsse-reference-guide.html#GUID-E1A3A7C3-309A-4415-903B-B31C96F68C86)
- Java 8: [PKIX TrustManager Support](https://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html#CERTPATH) and [On-Line Certificate Status Protocol (OCSP) Support](https://docs.oracle.com/javase/8/docs/technotes/guides/security/certpath/CertPathProgGuide.html#AppC)

The complete test in a maven project can be found [here](https://github.com/codingtim/ssl-revocation/blob/master/src/test/java/AwsSslTest.java).
