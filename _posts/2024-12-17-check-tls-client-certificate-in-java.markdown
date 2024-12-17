---
layout: post
title: "Check TLS Client Certificate in Java"
date: 2025-01-08 08:00:00 +0200
author: Robert Staeber
categories: tech
tags: TLS handshake client's certificate check TUC_PKI_018 specification Java
excerpt: "<br/>This article explains how to interrupt a TLS handshake on the server side (using Java and Spring Boot) to validate the client's certificate and, if necessary, abort the handshake. The certificate validation process adheres to the specifications provided by Gematik. The Java library introduced in this article implements this validation and is already in use by several software companies. <br/><br/>"
---

## Motivation

The gematik document "Übergreifende Spezifikation
PKI" [gemSpec_PKI](https://gemspec.gematik.de/docs/gemSpec/gemSpec_PKI/latest/) specifies the
certificate validation process within the TI (Telematik Infrastruktur): TUC_PKI_018 "
Zertifikatsprüfung in der TI".<br/>
Among other things, the document defines that the TLS handshake must be interrupted to validate the
client's certificate.
This results in 2 main tasks:

1. Implementing a certificate validation process.
2. Integrating this validation into the TLS handshake.

## Certificate Validation Process

The certificate validation process is already implemented and well-documented in the `gemLibPki`
library.
Simply use this library (introduced below) and follow the instructions in its `README.md`.

### Library `gemLibPki`

The `gemLibPki` implements all the checks defined in TUC_PKI_018 and is already used by several
software companies.<br/>
[source code -> gitHub](https://github.com/gematik/ref-GemLibPki)<br/>
[binaries -> maven central](https://mvnrepository.com/artifact/de.gematik.pki/gemLibPki)<br/>

## Interrupting the TLS Handshake on the Server Side to Validate the Client Certificate

The following code snippets demonstrate how to interrupt the TLS handshake in a Spring Boot
application. <br/>
You need to implement a Spring Boot component that extends `X509TrustManager` and use it as
a `HandshakeInterceptor` in the `TomcatServletCustomizer`.<br/>

<p align="center">
<img src="{{ site.baseurl }}/assets/img/241217-checkTLScert/tls-TLS_Handshake_with_Validation_of_Client_s_Certificate.png" alt="drawing" width="800"/>
</p>
_TLS handshake with validation of client's certificate_

The overridden method `checkClientTrusted` serves as the entry point for invoking the certificate
validation process:

```java
/**
 * This class is not managed by Spring, it is managed by TomcatServletCustomizer...
 */
@Slf4j
@Component("HandshakeInterceptor")
@RequiredArgsConstructor
public final class HandshakeInterceptor implements X509TrustManager { 
  ...

  @Override
  public void checkClientTrusted(final X509Certificate[] chain, final String authType)
      throws CertificateException {
    ...
    final TucPki018Verifier tucPki18Verifier;
    ...
    tucPki18Verifier.performTucPki018Checks(chain[0]);
    ...
  }
}
```

Set your HandshakeInterceptor in the TomcatServletCustomizer:

```java

@Component
public class TomcatServletCustomizer
    implements WebServerFactoryCustomizer<TomcatServletWebServerFactory> {
  ...

  @Override
  public void customize(final TomcatServletWebServerFactory factory) {
    ...
    factory.addConnectorCustomizers(
        connector -> {
          ...
          sslHostConfig.setTrustManagerClassName(HandshakeInterceptor.class.getCanonicalName());
          ...
        });

  }
}
```

### Example Implementation on GitHub

The PKI testsuite published by gematik, includes a "System Under Test Server
Simulator" ([gitHub -> pkits-sut-server-sim](https://github.com/gematik/app-PkiTestsuite/tree/main/pkits-sut-server-sim))
that interrupts the TLS handshake to validate the client's certificate. The simulator uses
the `gemLibPki` library to
perform the required validation.<br/>


---

## About the Author

Robert Stäber is a software engineer for more than 20 years. He joined the gematik in 2016 and is
member of the product team `IDM (Identity Management)` and the `Chapter Identity & Security` as
well.

---
