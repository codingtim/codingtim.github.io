---
title: "Java 8/11 disabled TLS 1.0/1.1 by default."
tags:
  - java
---

OpenJDK has backported disabling TLS 1.0 and 1.1 by default.

The relevant OpenJDK tickets can be found: [CSR](https://bugs.openjdk.java.net/browse/JDK-8257122), 
[Enhancement](https://bugs.openjdk.java.net/browse/JDK-8202343)

Some counter points to backporting this were made on the [mailing list]( https://mail.openjdk.java.net/pipermail/jdk-updates-dev/2021-April/005577.html).

Not all distributions kept the behaviour. 

## Amazon Corretto 

Amazon decided to re-enable TLS 1.0 and 1.1 by default. This can be found in the release notes:

- [Java 11](https://github.com/corretto/corretto-11/blob/release-11.0.11.9.1/CHANGELOG.md#corretto-version-1101191)
- [Java 8](https://github.com/corretto/corretto-8/blob/release-8.292.10.1/CHANGELOG.md#corretto-version-8292101)

## Azul Community 

Azul decided to keep the default disable.
 
- [Java 11](https://docs.azul.com/core/zulu-openjdk/release-notes/april-2021/pdf/zulu11.48-release-notes-ca-linux-aarch64-rev1.0.pdf)
- [Java 8](https://docs.azul.com/core/zulu-openjdk/release-notes/april-2021/pdf/zulu8.54-release-notes-ca-linux-aarch64-rev1.0.pdf)

## AdoptOpenJDK

AdoptOpenJDK keeps the default disable.

## Enable 

To enable TLS 1.0 or 1.1 again you can alter the `java.security` file and remove TLSv1 or TLSv1.1 from the security property `jdk.tls.disabledAlgorithms`.

