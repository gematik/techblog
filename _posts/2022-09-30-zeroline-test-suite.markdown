---
layout: post
title:  "Zero-line test suite"
date:   2022-10-13 06:51:16 +0200
author: Julian Peters
categories: testing
tags: testing test-suite tiger
---

Writing test suites that don't age, don't require software developers to set up, and still have significant technical depth is an impossible goal we at Gematik are constantly working towards. Zero-line test suites are a concept for minimizing required code while maximizing technical depth.

## What seems to be the problem?

At gematik, we test a lot. A LOT! The core of what our company does is to specify products and then test, verify and validate implementations. We have about a hundred product types, hundreds of vendors, and far above a thousand different products that need our attention. The upkeep alone to maintain all those test suites is enough to engage a small army of developers and testers. But our developers much rather want to support the specification by writing POCs, and of course, we constantly need to add new test suites to our portfolio.

In this post, we propose a novel way of writing test suites, the zero-line test suite. These test suites minimize the amount of java code written and aim to keep all the domain knowledge in Gherkin. Flexible Gherkin test steps allow expressive statements that offer deep technical verification. “Zero lines” is not meant religiously. The goal is to create expressive test suites that require minimal customization but offer great flexibility and specificity. As long as the Java code is very minimal it can easily be maintained by a tester, has to meet very minimal quality standards, and can easily be replaced when the need arises.

It should be noted that a zero-line test suite is not a band-aid solution for every scenario. When looking for alternatives, you should consider the screenplay pattern[^1], which is also used inside the gematik. Screenplay is tailored to more complex interactions, which require a great amount of custom code, and helps the developer and the tester to organize the code.

Zero-line test suites, on the other hand, are especially helpful in very technical tests, protocol tests, for example, where the test suite itself is ripe with technical details about the specified interaction.

This violates the BRIEF-Principles for BDD test suites[^2]. This is okay since this is not a BDD test suite. We merely use serenity and cucumber. A zero-line test suite is not meant to communicate with the business side. We want to convey technical details, not hide them.

## How is it done?

The framework at the heart of the zero-line test suite is the Tiger framework[^3]. It is developed by gematik, is open source, and is available to the public. It is based on serenity enabling the fast and easy creation of powerful test suites using Cucumber/Gherkin.

## Setup: The test environment

So, what do we need to write a test suite? First, we need a test environment. We use a YAML file to describe the test objects, how they are started, where they are found, and which interfaces they offer. This YAML file replaces the concrete with the abstract, allowing us to easily swap test environments. Want to code on a train? Want to do a regression test? Test a product? All of that and much more is possible.

```yaml
servers:
  identityServer:
    type: externalUrl
    source:
      - http://localhost:${free.port.1}
  myTestClient:
    type: externalJar
    source:
      - local:../octopus-identity-service/target/octopus-identity-service.jar
    healthcheckUrl: http://localhost:${free.port.2}/status
    dependsUpon: identityServer
    externalJarOptions:
      arguments:
        - --identityServer=http://localhost:${free.port.1}
        - --server.port=${free.port.2}
```
_Short example of a tiger.yaml to start a client which will connect to an already running server_

You always send your requests to “http://myTestClient” and let the framework take care of the details. This myTestClient could be a local application, a docker image, a JAR to be downloaded, or a remote server. The important point is that the test suite should not know or care about the details. They are abstracted away.

<p align="center">
<img src="{{ site.baseurl }}/assets/img/220930-zlts/testEnv.png" alt="drawing" width="800"/>
</p>
_The resulting test setup from the above YAML_
<!-- 
@startuml
!include https://raw.githubusercontent.com/bschwarz/puml-themes/master/themes/cerulean/puml-theme-cerulean.puml
skinparam classFontColor red
skinparam backgroundColor white
skinparam defaultFontSize 18
skinparam defaultFontName "Verdana Bold"
skinparam BoxPadding 80
rectangle "**<color:white>Tiger Proxy</color>**" as tigerProxy $WARNING
rectangle "**<color:white>Tiger test suite</color>**" as tigerTestSuite $WARNING
rectangle "**<color:white>myTestClient</color>**" as myTestClient $PRIMARY
note top
  http://localhost:${free.port.2}
end note
rectangle "**<color:white>identityServer</color>**" as identityServer $PRIMARY
note top
  http://localhost:${free.port.1}
end note
myTestClient <-> tigerProxy :HTTP
tigerProxy <-> identityServer :HTTP
tigerProxy .down.> tigerTestSuite :WebSocket
tigerTestSuite -up-> myTestClient :HTTP
@enduml
-->

Another important function is the integrated management of test data: By using placeholders later on in the test suite, the tester can abstract away from actual data as well, making the transition between different environments (or different test runs, different test users…) seamless.

## Let's get it moving: Actuation

Now we come to actuation. We provide multiple plugins for the Tiger framework to enable easy handling of various standard components that are found in our domain. We thought about writing Gherkin-based HTTP client steps, to truly achieve the zero-line goal here as well, but, as of yet, the need has not arisen.

Here we come again to what I said above about the zero lines not being an actual goal. In our experience so far the resulting test suites are very expressive if we do the requests in Java. You can use an arbitrary HTTP Library for this purpose.

## Are you for real? Verification

Next up is the verification. Here we rely heavily on the Tiger proxy. This is the component that enables us to use generic URLs in the test environment: We simply put a proxy in between the components (forward or reverse, the choice is yours). Inside the proxy we not only forward the data, but we also log the communication.

The communication data is then parsed using RbelLogger, another in-house development. We can parse JSON, XML, JWT, X.509, and many more data formats. What is more, we can also parse when they are nested inside of each other. This is done without prior knowledge or assumptions. So when you have a SOAP message with a certificate in the header, we can parse that seamlessly. This is important for the gematik since we often have additional end-to-end encryption in place, rendering normal parsing tools useless.

The Tiger proxy can also be run remotely. Do you need to verify traffic inside the internal network communication of more complex systems? Roll out the Tiger proxy to the deployment in question. The Tiger proxy running on your local machine as part of the test suite can then connect to the remote Tiger proxy, which will in turn forward any incoming traffic.

The traffic can now be queried using RbelPath, a query language very similar to JsonPath and XPath. With these, you can ask questions “Is the value at $..username” equal to myConfiguredUsername”? Placeholders will be resolved using the test data mechanism described above. This enables exactly what we want to achieve: Very expressive, easy-to-understand test suites. Testers are no longer held back by the lack of Java code underneath but rather enabled by the ability to directly express their questions in Gherkin.

```gherkin
Feature: Login

  Scenario: Login as unregistered user, then register and finally login again
    Given I try to log in as "${octopus.user.name}" with password "${octopus.user.password}"
    And TGR finds the last request to path "/testdriver/performLogin"
    And TGR current response at "$.responseCode" matches "200"

    And TGR current response at "$.body.header.alg" matches "RS256"
    And TGR current response at "$..name" matches "${octopus.user.name}"
    And TGR current response at "$.body.header.x5c.0.content.issuer" matches "${octopus.idService.certificate.dn}"
```
_Example scenario from our Tiger workshop[^4]. The "I try to log in as..." step is implemented in glue code. The octopus data points are defined in a YAML and are read from there. The verification steps prefixed with TGR are all directly available in Tiger. Note that they verify directly into HTTP (responseCode), JWT (body.header.alg), wildcard into JSON (..name) and into a X.509 certificate (content.issuer). This is all achieved with only one line of code (The actuation step)_

## What took you so long? Shift-left

This brings us to the final ingredient: Shift-left. An important part of any good testing strategy, there are two major avenues in which shift-left is enabled through the Tiger framework.

On the one hand, as stated above, the abstraction from the concrete test environment enables the team to start development of the test suite sooner. Maybe you use POCs, maybe some old hardware. The point is to give the team confidence that the initial setup will not metastate into the test suite itself, giving a clean separation between the test suite and the System-under-test.

On the other hand, due to the tiger framework being open-source, we can now start to ship test suites to the vendors and enable them to test in-house. Before every vendor had to develop their own test suite themselves, leading to a huge overhead. Now we can support them directly.

## Can this work?

We had examples of sharing the test suite with external vendors even before finalizing the specification. We have introduced new test cases on the fly, releasing them to the vendor during development. This massively reduces the cost of bugs. In the past, we had failed test runs causing a delay in projects for months or even years. With this approach, we increase confidence in the products and tests while massively reducing overhead for the vendor.

## Now what do I tell my boss?

The Zero-Line test suite is a great way to hit the ground running when writing a test suite. Testers can immediately start in their domain (test specification, test reporting) while developers can concentrate on the low level problems (and potentially write code that transfers well into other teams and test suites). Less code means less overhead, less aging, and fewer problems, but not less technical depths or fewer details.

# About the author

Julian Peters is a programmer at heart, writing code for 20 years. He joined gematik 9 years ago and held positions ranging from tech lead to software architect. He is the leader of the chapter on testing technologies.

### Linked information
[^1]: <https://serenity-js.org/handbook/design/screenplay-pattern.html>
[^2]: <https://cucumber.io/blog/bdd/keep-your-scenarios-brief/>
[^3]: <https://github.com/gematik/app-Tiger>
[^4]: <https://github.com/Arkinator/octopus>