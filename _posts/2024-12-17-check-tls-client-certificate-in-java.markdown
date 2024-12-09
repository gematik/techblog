---
layout: post
title: "Check TLS client certificate in Java"
date: 2024-12-17 08:00:00 +0200
author: Robert Staeber
categories: tech
tags: TLS handshake client certificate check TUC_PKI_018 specification Java
excerpt: "<br/>This article explains how to interrupt a TLS handshake on the server side (using Java, Spring Boot) to validate the client's certificate and potentially abort the handshake if necessary. The certificate validation process follows the specifications provided by gematik. The Java library introduced in this article implements this validation and is already utilized by several software companies. <br/><br/>"
---

## Motivation

The gematik document "Übergreifende Spezifikation
PKI" [gemSpec_PKI](https://gemspec.gematik.de/docs/gemSpec/gemSpec_PKI/latest/) specifies the
certificate validation process in the TI: TUC_PKI_018 "Zertifikatsprüfung in der TI"
Among other things the document defines that the TLS handshake must be interrupted to validate the
client's certificate.
Hence, there are two things to do:

1. implement a certificate validation process
2. place the code from above into the TLS handshake

## Certificate validation process

This step is already implemented and well documented in the gemLibPki library.
Just use that library (described below) and follow the instructions in its README.md.

### gemLibPki

The library contains all checks defined in TUC_PKI_018 mentioned above. It is already used by
several software companies.<br/>
[source code -> gitHub](https://github.com/gematik/ref-GemLibPki)<br/>
[binaries -> maven central](https://search.maven.org/artifact/de.gematik.pki/gemlibpki)<br/>

## Interruption of the TLS handshake on server side, for a validation of the client certificate

The following code snippets show how to interrupt the TLS handshake in a Spring Boot application.
Implement a Spring boot component that extends `X509TrustManager` and use it as
a `HandshakeInterceptor` in the `TomcatServletCustomizer`.<br/>
Overridden method `checkClientTrusted` is your entry point. Here you can call the certificate
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

### Example implementation on gitHub

The PKI testsuite, published by gematik, contains a "System Under Test Server
Simulator" ([gitHub -> pkits-sut-server-sim](https://github.com/gematik/app-PkiTestsuite/tree/main/pkits-sut-server-sim))
that interrupts
the TLS handshake to validate the client certificate. The simulator uses the gemLibPki library to
perform the validation.<br/>


---

## About the author

Robert Stäber is a software engineer for more than 20 years. He joined the gematik in 2016 and is
member of the product team `IDM (Identity Management)` and the `Chapter Identity & Security` as
well.

---

# TODO

- 
