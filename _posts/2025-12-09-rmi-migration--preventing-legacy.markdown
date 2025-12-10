---
layout: post
title:  "RMI Migration - Preventing Legacy"
date:   2025-12-09 08:00:00 +0200
author: Zhenwu Duan
categories: tech
tags: RMI migration prevent legacy Spring REST
excerpt: "<br/>This article examines how to prevent a project from becoming legacy by migrating RMI(Remote Method Invocation) as a case study. When an outdated technology becomes a blocking factor, replacement is often more effective than continued maintenance. RMI was once a prominent component of the Java ecosystem, but Spring has discontinued its support due to evolving industry standards and modern alternatives. This article discusses: <br/>Why RMI has become problematic in contemporary projects <br/>Strategies for migrating away from RMI<br/>How to ensure long-term project maintainability and updatability<br/>While this demonstration focuses on RMI, the principles and migration strategies presented can be applied to similar scenarios involving deprecated technologies. Proactively replacing outdated dependencies prevents technical debt and keeps projects aligned with current frameworks and security standards.<br/><br/>"
---

**1.  Overview**

Remote Method Invocation (RMI), which has existed since the introduction
of Java, is the invocation of a method of a remote Java object and
implements Java\'s own version of Remote Procedure Call.

Java RMI (in java since 1.1) was developed as part of the Java ecosystem
to facilitate the creation of distributed systems where objects can
interact across network boundaries.

**2.  Problem**

A problem was discovered during the update of the project "ZTDV" that is
preventing the Spring Framework update.

Project ZTDV uses RMI to access various services of library "gem-cards".

The class \'org.springframework.remoting.rmi.RmiServiceExporter\' from
\'spring-context\' establishes the RMI connection between "ZTDV" and
"gem-cards" until now, as shown in Figure below (rmi-config.xml in
"ZTDV").

![RMI Configuration in ZTDV](../assets/img/20251209-rmi-migration-preventing-legacy/ztdv-rmi-config.png)

The current Spring release in "ZTDV" is 5.3.39. Update of Spring is
approached.

But Spring Framework releases 6.x.x and later no longer support RMI.

All RMI classes no longer exist in 6.x.x. This is shown in Figure by
comparison of R5.3.39 and R6.2.6 of spring-context. As a result, the
Spring Framework update in the "ZTDV" project is blocked.

<img src="../assets/img/20251209-rmi-migration-preventing-legacy/spring-context-5.x.x-vs-6.x.x.png" width=450 height=400>


**3.  What is RMI\'s Current Position in the IT World?**

- **Java\'s Official Stance on RMI**

Java officially discourages using RMI (Remote Method Invocation) for new
projects. While RMI remains in maintenance mode and is technically
supported, it is no longer under active development. The Java
documentation clearly states that RMI is considered obsolete and rarely
used in modern applications.

Recommended alternatives for distributed systems include:

\- RESTful services

\- gRPC

\- Message-based architectures (e.g., RabbitMQ, Kafka)

- **Spring Framework\'s Position on RMI**

Spring currently recommends no longer using RMI for new applications.
The Spring documentation indicates that RMI is obsolete and that
modern, HTTP-based protocols such as REST (Spring Web, Spring WebFlux)
or gRPC should be used instead. While RMI is still supported,
development has ceased, and there are security and compatibility
issues. For new projects, Spring recommends using REST APIs with JSON
or other formats.

- **Impact on Legacy Projects**

Projects still using RMI face increasing technical debt. Migration to
modern protocols is recommended to:

\- Ensure long-term maintainability

\- Improve security posture

\- Enable cloud and microservices architectures

\- Maintain compatibility with current frameworks and tooling

**4.  Implementing the problem solution in Project "ZTDV"**

"Carddownloader" for example is a useful tool in "ZTDV". Following
figure shows the current implementation with RMI. The implementation is
primarily located on the "ZTDV" side. The "gem-card" services are
accessed remotely from "ZTDV".

<img src="../assets/img/20251209-rmi-migration-preventing-legacy/impl-with-rmi.png" width=450 height=400>

After removal of RMI, a clear client-server relationship exists. The
implementation of the "Carddownloader" then resides on the "gem-cards"
side, where the services are. The "gem-cards" services are called
directly on-site through the endpoints like below

\`[https://host/gem-cards/send-apdu]{.underline}\` is preferred for a
single query

\`[https://host/gem-cards/cardDownloader]{.underline}\` is preferred for
multiple queries

The communication is secured by SSL/TLS.

The results are returned back to "ZTDV", where they are saved for user

The lesson learned from migrating from RMI to REST is the need to define
the calls cleanly. It\'s a shift in implementation.

<img src="../assets/img/20251209-rmi-migration-preventing-legacy/impl-with-rest.png" width=450 height=400>

**5.  A Non-Disruptive Migration Strategy**

The migration from RMI to REST can be executed as a non-disruptive,
localized change. This approach offers significant advantages:

- Zero Service Interruption: The migration requires no downtime or
  interruption of existing functionality. End users experience seamless
  operation throughout the transition period.

- Minimal Scope and Cost: By isolating the migration to the
  communication layer (RMI → REST), the core business logic and primary
  workflows remain unchanged. The refactoring is confined to:

\- The remote invocation mechanism

\- SSL/TLS configuration

\- Client-server communication protocol

This containment strategy significantly reduces development effort,
testing complexity, and deployment risk.

Key Benefits:

\- Existing features continue operating without modification

\- Changes are localized to the single layer (as shown above)

\- Business logic remains untouched, reducing regression risk

\- Gradual rollout is possible without affecting production systems

\- Low risk of introducing new defects in unrelated components

The Example from Implementation demonstrates this principle---the REST
endpoint replaces RMI communication while maintaining identical
functionality for the card terminal operations. The "gem-cards"
interface and business logic remain consistent.

This surgical, low-impact approach makes RMI migration an ideal template
for modernizing other legacy components in Java applications.

**6.  Conclusion**

Since REST replaces RMI, the Spring Framework can be easily updated to
the latest release.

A good idea is to start the Spring Boot Server as a separate service or
via a process management tool (such as Docker, Kubernetes, or a Windows
service). This way, the server is always running and accessible to
clients.

According to current development plans, RMI should no longer be used in
new projects. Projects that currently use RMI should be gradually
replaced with REST.

Migrating to Spring 6.x.x and replacing RMI with REST ensures your
application is modern, secure, and maintainable. REST is the simplest
and most flexible solution for most use cases.

\- Use REST if you need a simple, platform-independent solution.

\- Use gRPC if you need high-performance, binary communication.

\- Use Message Brokers (e.g., RabbitMQ, Kafka) for asynchronous
communication.

**About the author**

Zhenwu Duan is responsible in the Smartcard team for the development and maintenance of test tools. His interests include various programming languages such as ​​Java, Go, C#, Typescript, as well as system testing and PKI technology.