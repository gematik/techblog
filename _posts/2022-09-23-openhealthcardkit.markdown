---
layout: post
title:  "OpenHealthCardKit"
date:   2022-09-23 11:51:16 +0200
author: Gerald Bartz, Martin Fiebig
categories: tech
tags: OpenHealthCardKit OHCKit 
---


The digitalization of the German Health System is progressing. One essential part for the users is having a digital identity on a health card that authenticates oneself. OpenHealthCardKit is the link to provide access to this card via a mobile device. It abstracts the technical communication and provides a semantic interface for your use case. With OpenHealthCardKit, accessing a card's data and applications in your health-related app is now easy.


## What is OpenHealthCardKit?

OpenHealthCardKit (OHC) started as a proof of concept for using smart card hardware interfaces with mobile devices with no NFC interface to connect them with German health cards (elektronische Gesundheitskarte → eGK). As Apple added NFC capabilities to their mobile devices and finally opened them for general purposes, we added just another interface implementation, and OHCKit gained the ability to talk to eGKs on iPhones contactless. The OHCKit repository also comes with a demo application, "OpenHealthCard connect" (see Fig. 1), also available on AppStore [^1].

<p align="center">
<a href="{{ site.baseurl }}/assets/img/220923-ohckit/ohckit_screenshot1.png"><img src="{{ site.baseurl }}/assets/img/220923-ohckit/ohckit_screenshot1.png" alt="drawing" width="200"/></a>
<a href="{{ site.baseurl }}/assets/img/220923-ohckit/ohckit_screenshot2.png"><img src="{{ site.baseurl }}/assets/img/220923-ohckit/ohckit_screenshot2.png" alt="drawing" width="200"/></a>
</p>
*Fig. 1: Screenshots "OpenHealthCard connect" app*

Later on, gematik got the mandate to implement the official E-Prescription App[^3]. The E-Rezept-App employs the eGK to authenticate the user against an identity provider (IDP) and uses OHCKit for this task.

**Fun Fact:** As far as we know, the "OpenHealthCard connect" App is the first mobile application that let's you reset your eGK's PIN regardless of your insurance company.


## What does OHCKit do?

The bread and butter of communicating with a German health card (or any smart card thereof) is to be able to

1. create a **Command** (that is, a byte sequence) that a smart card can understand,
2. transmit it to the card,
3. interpret the **Response** that the card sends back, and finally

repeat these steps in a meaningful way.

So OHCKit does just that (and a little more).


## How is OHCKit structured?

### Specifications

The groundwork of the framework is specified by three documents.

1. *ETSI TS 102 226* is a base document for Smart Cards communication structure in general
2. gematik's *gemSpec_COS* (meaning gematik specification Card Operating system) extends the ETSI document in the German health card-specific context
3. Various more *gemSpec_ObjSys_xyz* (Object System) documents that describe the file system and applications for the various health card types and generations

All this technical depth is abstracted by OHCKit and its sub-modules.

[![gemSpec_COS example]({{ site.baseurl }}/assets/img/220923-ohckit/gemspec-cos-example.png)]({{ site.baseurl }}/assets/img/220923-ohckit/gemspec-cos-example.png)
*Fig. 2: Example of a "Select"-command specification in gemSpec_COS*

Keep in mind that, while these documents describe the content and behavior of smart cards, one can derive the desired behavior of the coupling device, i.e. how this framework can achieve it.

**Fun Fact:** The specification *gemSpec_COS* was created over 10 years ago and has about 500-550 pages - depending on the revision

### Architectural insights

The architectural structure of the OpenHealthCardKit framework mainly consists of three layers that build on one another. These layers also resemble the specifications discussed above.


| Modules<br>(Layers low to high) | Summary                                                           | implements Spec                |
|---------------------------------|-------------------------------------------------------------------|--------------------------------|
| CardReaderProviderAPI           | (Smart)CardReader protocols for interacting with HealthCardAccess | ETSI                           |
| HealthCardAccess                | Abstraction of cards, commands, responses, and file systems       | gemSpec_COS,<br>gemSpec_ObjSys |
| HealthCardControl               | High level API-gateway                                            | -                              |

In a nutshell: 
 - *HealthCardAccess* carries the weight of creating commands to be sent and abstracting the responses to be received.
 - *HealthCardControl* is a convenience "flow controller" that aligns multiple command-response cycles in a meaningful way (e.g. go to a subfolder, then read file xy) and exposes this flow as a callable function.
 - One must implement *CardReaderProviderAPI* for one's card-reading hardware interface (e.g. card terminal, NFC, ..) in order to gain access to *HealthCardAccess'* abstractions.

For far more detailed explanations and examples, visit OHCKit's GitHub repository[^2].


## Self-managed dependencies

Some technologies and protocols employed in the German/European health sector PKI environment are uncommon in the mobile world (and some are almost unique to it). As a result, we had to learn that neither the standard SDK-provided tooling nor the open-source-based projects accessible quite match our needs.

To overcome this issue,  we developed and maintain yet another two frameworks that we'd like to draw your attention to:

### ASN1Kit

**Abstract Syntax Notation One** (ASN.1) is a standard for putting data in a structure that can be serialized and deserialized. Albeit being broadly used in networking and cryptography, out-of-the-box support for inspection and creation of custom ASN.1-serialized data is unavailable on iOS. Smart cards often send and expect data that is ASN.1-encoded, so having a good handle on this undertaking was unavoidable. For lack of viable alternatives, we created our own Swift-native framework **ASN1Kit**[^4] to aid our needs.

We also extensively use the power of this framework in the official E-Prescription App.

**Fun Fact:** ASN1Kit was the first Swift-native (almost complete) ASN.1-handling framework available on GitHub to our knowledge. 

### OpenSSL-Swift

Whenever you work with cryptographic algorithms on the administrative side of the German health sector context, it is inevitable to sooner or later cross paths with

1. the concept of **Elliptic Curve Cryptography**  (ECC) as a base for Key Exchange- and Signature-related algorithms and
2. so-called **Brainpool**-curves, which represent collections of parameters that exactly describe how (more exact: which seed values to use) to perform arithmetical calculations on Elliptic Curves, and
3. when establishing smart card communication via NFC, a sophisticated Brainpool-curve backed **Password Authenticated Key Exchange** algorithm (PACE),

all required and specified by the Federal Office for Information Security (BSI).


While ECC, in general, is supported by iOS, namely the Brainpool-curves are not. Even if they were, the PACE protocol uses a peculiar arithmetic operation that high-level key-exchange interfaces usually don't expose.

Initially, there was no solution to these problems available to us, besides implementing the Elliptic Curve cryptography by ourselves (what we actually did in the proof of concept stage).

Later we created **OpenSSL-Swift**[^5], a wrapper framework for gematik- and BSI-specific crypto operations with an embedded OpenSSL library. It mainly serves two purposes:

1. Check out a current version of the OpenSSL source code and build libraries from it for different combinations of platforms (iPhone, mac, simulator) and architectures (arm64, x86_64).
2. Wrap parts of the OpenSSL interface, which is written in C, into Swift functions that can be accessed more easily

That is a lot of bases to cover from cross-compiling C code, bridging it to the Swift world, and then aiding some of the quirks that performing cryptographic operations in our context demands from us. Seeing all this work out makes us very proud and amazes us to this day.

**Fun Fact:** Whenever a German ID card or passport is addressed via the NFC interface, the very same PACE protocol with underlying Brainpool Elliptic Curve cryptography we just outlined is employed.


## Where to go from here?

OpenHealthCardKit is available as open source software on GitHub[^2]. You can find some use cases in the Readme and the demo app that is included within the repository. The demo app is also available for free on the AppStore[^1].

# About The Authors

Gerald Bartz has been a software developer for five years, four years on iOS. At gematik he enjoys porting not-so-standard cryptographic algorithms to the mobile world.

Martin Fiebig has worked as a software engineer for 18 years, 10 years on iOS and Apple Platforms. His focus topics include large scale mobile applications and developer experience. Martin leads the iOS Development Chapter at gematik since 2020.

[^1]: <https://apps.apple.com/de/app/openhealthcard-connect/id1450490405>
[^2]: <https://github.com/gematik/ref-OpenHealthCardKit/>
[^3]: <https://apps.apple.com/de/app/das-e-rezept/id1511792179>
[^4]: <https://github.com/gematik/ASN1Kit>
[^5]: <https://github.com/gematik/OpenSSL-Swift>
