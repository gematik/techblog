---
layout: post
title:  "Maintaining a Legacy System"
date:   2024-11-21 08:00:00 +0200
author: Zhenwu Duan
categories: tech
tags: Legacy System Modernization Security Strategy Measures Experience
excerpt: "<br/>This article helps understanding Legacy Systems. It shows that maintaining legacy systems can be challenging due to outdated technology, lack of documentation. It supplies experience from practice the strategies and measures through gradual modernization, regularly plan for gradual updates and replacement of the system to modern technologies.<br/><br/>"
---

## What is a Legacy System?

A legacy system, or legacy software, refers to an older system that has evolved over time and is still in use today. In many companies, IT systems and mission-critical applications have been in production for a long time and have become outdated. These legacy applications are often based on outdated technologies, a historically developed codebase, or monolithic architectures. The problem is that they no longer meet the modern demands of agile IT infrastructures and increasingly pose a security risk.

## How Does a Legacy System Occur?

Let's assume there is a product called "Prod.". The development phase is exciting, and then the "maintenance" phase begins. After extensive bug fixing and some optimizations, the software is accepted by the customer. Occasional bug fixes are still possible. Then comes a long quiet phase. The customers use the software, and the developers focus on other projects.

This quiet phase can last for several years. During this time, new techniques in the programming language have been established, and the gap between the version of “Prod” and the current state of the technology grows larger. The old code is no longer elegant and often feels outdated. The company's operating systems have likely been updated to the latest versions. Tests might have been moved to new computers, but the operating system on the new machine is no longer Windows, it changes to Linux. Half of the tests run no longer. No one has the time to address these issues. Gradually, the legacy system begins to deteriorate.

## What Problems Can a Legacy System Cause?

We have several legacy systems in practice, which have been modernized step by step. The problems in these legacy systems vary from each other. They can be summarized as follows:
1.	No CI/CD in some projects, missing pipelines.
2.	Source code for some projects is not committed to GitLab. Third-party source code is not in a version control system.
3.	Some software no longer works after the introduction of a new operating system because one core library doesn't support the new OS.
4.	Clean Code principles are not followed.
5.	Redundancies are produced due to the involvement of many different developers over a long period.
6.	Projects haven't been released for a long time. The software's state is unclear. No clear development process to be recognized.
7.	Tests run no longer. They are not adapted to the changed environment.
8.	Use of outdated libraries leads to security vulnerabilities. Numerous weaknesses have been discovered through inspection tools.
9.	With the introduction of modern inspection tools for source codes, more and more violations of the clean-code criteria were discovered.

## How Do You Deal with Problematic Legacy Systems?

There are many different reasons why a legacy system must be modernized. One example is the technologies used to develop the software. Technologies, whether open-source or commercial, have their own life cycles.

Another reason may relate to the internal quality of legacy systems. Software is subject to a continuous erosion process, where the internal quality can gradually deteriorate. A degenerating system structure, the accumulation of technical debt, increasing code duplication, the loss of knowledge, or poor documentation can all be important factors. Typical effects include drastically reduced maintainability, decreasing stability, and longer release cycles.

Additionally, security flaws may make improvements to the legacy software necessary. IT security plays a critical role in modern IT projects.

Discarding the problematic legacy system and developing a new one is an option, in this case you have to consider many factors such as time, money and personnel. A far more practical and cost-effective option is to modernize the existing project.

## Modernization Strategies

From our experience, reactivating the tests has the top priority. The first thing to do is to get the tests running again. These tests will then serve as a guarantee for the following refactoring measures.  
Automated tests provide a safety net for developers by confirming that changes don’t inadvertently cause regressions. In a legacy system, tests often become outdated or neglected. Bringing them back to life ensures the integrity of the system, especially when refactoring or adding new features.

The modernization can be done through two strategies.
* The first one is that the developing team focuses fully on modernization in a period, the team does nothing but modernize the legacy system only. This strategy requires a long-time gap till to several weeks or more.

* The second one prefers a step-by-step approach with partial refactoring, aiming for continuous improvement over a longer period, carried out in parallel with other work. The source code will be improved whenever possible.

Since we cannot afford such a long break, we proceed with the second strategy.

In our experience, a gradual, step-by-step approach is much more effective. This method is often referred to as incremental refactoring, and it allows the team to introduce changes without overwhelming the system and the team self.
A full system overhaul can introduce significant risk, especially when working with complex, monolithic legacy systems. By refactoring small sections over time, you reduce the risk of major issues, while also enabling continuous improvements.

## Modernization Measures

**Focus on one module or feature at a time**: For example, if a particular module in the legacy system is responsible for card processing, modernize that module first before moving to others.

**Gradually replace outdated components**: As you refactor the code, replace old libraries with modern alternatives.

**Refactor with an eye on maintainability**: When making changes, follow modern coding standards (e.g., Clean Code principles), ensuring the refactored code is modular, readable, and easier to maintain in the future. e.g., Replace Getters and setters by using Lombok, which can save large amount of line in source code.

The result is quite evident. The source code becomes gradually more readable, because it becomes shorter and more logical. Redundancies are reduced. Tests are restored and new unit tests are added where they are missing. Missing Jenkins pipelines are set up. All source codes are committed to GitLab. Outdated libraries are replaced with modern alternatives. Redundant classes are removed through structural optimization. Releases are made at regular intervals.

Security is one of the most important considerations when maintaining legacy systems, as older software may be vulnerable to a range of security issues.  
Legacy systems often use outdated libraries or technologies that have known security flaws. These can be exploited if not addressed. Regular security audits and timely updates to libraries and frameworks are essential for maintaining a secure environment.

**Conduct regular vulnerability assessments**:   Our tools like OWASP Dependency-Check or SonarQube identify vulnerabilities in libraries and dependencies.

Replace outdated libraries with a newer, more secure alternative.

**Update security patches regularly**: Even if the system’s core functionality does not require major changes, ensure that security patches are applied consistently to reduce the risk of attacks.

Our approach in practice looks as below:

The "critical" vulnerabilities in the used libraries should be replaced first with secure alternatives. Then, the "high" rated vulnerabilities should be addressed. The project is gradually transformed into secure software. Sonar findings are addressed, the test coverage is increased. OWASP findings are processed, and libraries are updated to enhance the software’s security.

## How to Prevent Problems with Legacy Systems

Based on the above analysis, it’s clear that the deficiencies arise after the intensive “maintenance” phase. After this phase, the software is almost abandoned by the development team. The team only responds to serious bugs in the software but no longer actively maintains it. As the external world is constantly changing, the software remains constant internally, but its functionality does not remain stable forever.

Systems left without regular attention will inevitably become harder to maintain and more prone to failure

A legacy software system is born this way. But what can be done to address this?

It’s not enough to only patch the software in isolated spots after a long period of time. Eventually, the software will become unmaintainable. In the worst case, a legacy system may even reach the point of being abandoned due to fundamental flaws.

A vehicle has to go through the TÜV (vehicle inspection) every two years—this is a good lesson. Why not perform a similar check on legacy systems? It makes sense to regularly check whether all tests are still running, whether Clean Code measures are needed, or if there are any security vulnerabilities in the libraries.

By setting up a routine schedule, the team ensures that problems are identified and resolved before they become major issues.

Implementing therefore a “Software TÜV” (software inspection) is worthwhile to ensure that a legacy system functions reliably and is future- oriented. The inspection should be formally scheduled, and necessary tickets should be created for it. The results of the inspection and any actions taken should be well documented.

## Conclusion

Maintaining and modernizing legacy systems is a balancing act that requires careful planning, ongoing attention, and a structured approach.

**Establish regular maintenance intervals**: No matter how often, establish a routine for checking the system’s health, applying patches, updating dependencies, and improving the codebase.

**Plan for future-oriented modernization**: Regularly review the technologies being used and assess whether they are still suitable for long-term use. Replace outdated technologies before they become a liability.

By following a step-by-step modernization strategy, reactivating tests, addressing security issues, and documenting every change, teams like Smartcard can ensure their legacy systems continue to function reliably while preparing them for future needs. Regular maintenance, proactive security measures, and continuous improvements help prevent legacy systems from becoming liabilities, ensuring they remain valuable assets over time.

---

## About the author

Zhenwu Duan is responsible in the Smartcard team for the development and maintenance of test tools. His interests include various programming languages such as ​​Java, Go, C#, Typescript, as well as system testing and PKI technology.

---
