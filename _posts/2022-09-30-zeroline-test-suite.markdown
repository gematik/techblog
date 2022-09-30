---
layout: post
title:  "OpenHealthCardKit"
date:   2022-09-23 11:51:16 +0200
author: Julian Peters
categories: testing
tags: testing testsuite tiger
---

## What seems to be the problem?

At gematik, we test a lot. A LOT! The core of what our company does is to specify products and then to test, verify and validate implementations. We have about a hundred product types, hundreds of vendors and far above a thousand different products that need our attention. The upkeep alone to maintain all those testsuites is enough to engage a small army of developers and testers. But our developers much rather want to support the specification by writing POCs and of course we constantly need to add new test suites to our portfolio.

In this post we propose a novel way of writing test suites, the zero-line testsuite. These test suites use a minimal amount of java code and make all the magic happen in Gherkin. Flexible BDD steps allow expressive statements that offer deep technical verification. “Zero lines” is not meant religiously. The goal here is to create expressive BDD test suites that require minimal customization but offer great flexibility and specificity. As long as the Java code is very minimal it can easily be maintained by a tester, has to meet very minimal quality standards and can easily be replaced when the need arises.

It should be noted that a zero-line testsuite is not a band aid solution for every scenario. When you are looking for alternatives you should consider the screenplay pattern (https://serenity-js.org/handbook/design/screenplay-pattern.html), which is also used inside the gematik. Screenplay is tailored to more complex interactions which require a great amount of custom code and helps the developer and the tester to organize the code.

Zero-line test suites on the other hand are especially helpful in very technical tests, protocol tests for example, where the test suite itself is ripe with technical details about the specified interaction.

## How is it done?

The framework at the heart of the zero-line testsuite is the Tiger framework (https://github.com/gematik/app-Tiger). It is developed by gematik, is open source and available to the public. It is based on serenity enabling the fast and easy creation of powerful test suites using Cucumber/Gherkin, meaning BDD.

## Set-Up: The test environment

So, what do we need to write a testsuite? First we need a test environment. We use a YAML-file to describe the test objects, how they are started, where they are found and which interfaces they offer. This YAML-file replaces the concrete with the abstract, allowing to easily swap test environments. Want to code on a train? Want to do a regression test? Test a product? All of that and much more is possible.

You always send your requests to “http://myTestClient” and let the framework take care of the details. This myTestClient could be a local application, a docker image, a JAR to be downloaded or a remote server. The important point being that the test suite should not know or care about the details, they are abstracted away.

Another important function is the integrated management of testdata: By using placeholders later on in the testsuite the tester can abstract away from concrete data as well, making the transition between different environments (or different testruns, different test users…) seamless.

## Let's get it moving: Actuation

Now we come to actuation. We provide multiple plugins for the Tiger framework to enable an easy handling of various standard components that are found in our domain. We thought about writing Gherkin based http client steps, to truly achieve the zero-line goal here as well, but as of yet the need has not arisen.

Here we come again to what I said above about the zero lines not being an actual goal. In our experience so far the resulting test suites are very expressive if we do the requests in Java. You can use an arbitrary HTTP-Library for this purpose.

## Are you for real? Verification

Next up is the verification. Here we rely heavily on the tiger-proxy. This is the component which enables us to use generic URLs in the test environment: We simply put a proxy in between the components (forward or reverse, the choice is yours). Inside the proxy we not only forward the data, we also log the communication.

The communication data is then parsed using the RbelLogger, another in-house development. We can parse JSON, XML, JWT, X.509 and many more data formats. What is more, we can also parse when they are nested inside of each other. This is done without prior knowledge or assumptions. So when you have a SOAP-message with a certificate in the header, we can parse that seamlessly. This is important for the gematik since we often have additional end-to-end encryption in place, rendering normal parsing tools useless.

The tiger-proxy can also be run remotely. Do you need to verify traffic inside the internal network communication of more complex systems? Roll out the tiger-proxy to the deployment in question. The tiger-proxy running on your local machine as part of the test suite can then connect to the remote tiger-proxy, which will in turn forward any incoming traffic.

The traffic can now be queried using RbelPath, a query language very similar to JsonPath and XPath. With these you can ask questions “Is the value at $..username” equal to myConfiguredUsername”? Placeholders will be resolved using the test data mechanism described above. This enables exactly what we want to achieve: Very expressive, easy to understand test suites. Testers are no longer held back by the lack of Java code underneath but rather enabled by the ability to directly express their questions in BDD.

## What took you so long? Shift-left

This brings us to the final ingredient: Shift-left. An important part of any good testing strategy, there are two major avenues in which shift-left is enabled through the Tiger framework.

On the one hand, as stated above, the abstraction from the concrete test environment enables the team to start development of the test suite sooner. Maybe you use POCs, maybe some old hardware. The point is to give the team confidence that the initial setup will not metastate into the testsuite itself, giving a clean separation between the test suite and the System-under-test.

On the other hand, due to the tiger framework being open-source, we can now start to ship test suites to the vendors and enable them to test in-house. Before every vendor had to develop their own testsuite themselves, leading to a huge overhead. Now we can support them directly.

## Can this really work?

We had examples of sharing the test suite with external vendors even before finalizing the specification. We have introduced new test cases on the fly, releasing them to the vendor during development. This massively reduces the cost of bugs. In the past we had failed test runs causing a delay in projects for months or even years. With this approach we increase confidence in the products and tests while massively reducing overhead for the vendor.

## Tl;dr

The Zero-Line Testsuite is a great way to hit the ground running when writing a test suite. Testers can immediately start in their domain (test specification, test reporting) while developers can concentrate on the hard problems. Do not get bogged down by being strict about the zero in the name: Writing a quick Rest-Call should not be a dealbreaker neither for a developer nor a tester.

# About the author

Julian Peters is a programmer at heart, writing code for 20 years. He has joined gematik 9 years ago and held positions ranging from tech lead to software architect. He is the leader of the chapter on testing technologies.
