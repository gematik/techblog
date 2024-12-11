---
layout: post
title:  "How the KOB Test Suite Improves Development and Compliance Processes"
date:   2024-12-15 06:51:16 +0200
author: Julian Peters
categories: testing
tags: testing test-suite tiger kob kig epa
---

## Tiger Sets New Standards in Conformity Assessment

Digitization in healthcare is inherently complex, particularly when it comes to creating open systems that enable seamless collaboration among diverse stakeholders. This challenge is particularly evident in the rollout of the new electronic patient record (ePA für alle).

Millions of patients must be able to access and manage data across approximately 100,000 medical practices.  
This requires systems developed by around 200 manufacturers to not only securely process data and establish connections but also ensure smooth interoperability.

To guarantee this level of collaboration, the German legislature tasked **gematik** with implementing a conformity assessment (KOB). Historically, this process involved submitting software systems to gematik for review—an effort-intensive approach for both parties. Tests were time-consuming, requiring the entire process to be repeated even for minor errors, accompanied by detailed reports at the manufacturer’s expense. Alternatively, tests were conducted on central servers, which were equally cumbersome and challenging to integrate into the manufacturers’ development workflows.

With the rollout of the 'ePA für alle', a new approach has been introduced: the **KOB Test Suite**, based on Tiger. Tiger, developed by gematik, is a test framework designed for rapid, transparent, and efficient test suites. The KOB Test Suite is publicly available on GitHub, free of charge, allowing manufacturers to download and execute it locally early in their development process. This supports integration into existing CI pipelines and use on individual developer machines. Test results can be uploaded to the confirmation portal, where the largely automated review process certifies conformity in a short time.

## A Paradigm Shift in Testing

This new process offers significant advantages for all parties involved: faster public availability of test suites, early support during development, precise and transparent test results, and flexible integration into internal workflows. The iterative approach fosters better collaboration between gematik and manufacturers, enabling more efficient development.

By entrusting testing to manufacturers while maintaining complete transparency regarding test content, gematik builds trust. The open-source nature of the test suite ensures all code can be reviewed and understood. Early testing (a "shift-left" approach) further supports manufacturers by improving time-to-market outcomes.

## The Tiger Framework

Tiger is a comprehensive test solution designed to simplify the creation of various test types, including end-to-end, acceptance-, integration-, and interface tests. Ideal for resource-constrained teams and agile processes, it is open-source, Java-based, and built on Cucumber, offering flexibility and customizability for automated testing.

Tiger’s core feature is a proxy that intercepts and analyzes network traffic between components. This traffic is converted into a protocol-agnostic hierarchical structure (a tree), searchable with **RbelPath**, a query language inspired by XPath and JsonPath. This eliminates the need for advanced programming skills, making the framework accessible even for non-technical users.

To enhance collaboration, Tiger provides the **WorkflowUI**, an HTML-based visualization of captured traffic, simplifying documentation and communication around test scenarios. The framework's plugin-based architecture supports extensions tailored to specific needs, such as Docker container provisioning or FHIR resource validation. Additionally, **Zion** enables the rapid deployment of mock services with simple logic.

## KOB Tests with Tiger

The KOB Test Suite, available on GitHub, allows manufacturers to run tests in their environments, providing significant flexibility. Internal setups can be used, and tests can be executed on individual developer machines or central build servers. Test results are immediate and actionable.

Key benefits include integration into CI pipelines to prevent regression and the ability to develop directly against the final test suite, potentially saving significant time and costs. For gematik, delegating parts of the test setup to manufacturers reduces its operational burden.

Optional test cases included with the KOB Test Suite allow for enhanced technical communication. These tests address common challenges, such as non-functional requirements, by offering automated solutions to typical pitfalls. Manufacturers also benefit from guidance through implementation manuals and consultations with gematik experts.

## Implementation Challenges and Long-Term Benefits

The initial adaptation of the KOB Test Suite does pose challenges, particularly in integrating the test driver interface. Manufacturers must adapt to this interface to align with the tested user journeys, requiring additional development effort. Smaller manufacturers with limited resources may find this especially demanding. Building the knowledge base needed to work effectively with the new framework also consumes resources initially.

However, the long-term benefits outweigh these hurdles. Once integrated, the KOB Test Suite enables continuous automated testing within CI pipelines, accelerating bug detection and improving software quality. This leads to faster time-to-market for compliant products and reduced maintenance costs through enhanced test coverage. Moreover, as more products are to be integrated into the KOB Test Suite, manufacturers benefit from streamlined processes and standardized procedures.

## Conclusion

The introduction of the KOB Test Suite marks a pivotal advancement in conformity assessment. While the initial implementation, particularly of the test driver interface, presents challenges, the suite offers significant long-term benefits: reduced maintenance costs, automated testing, and accelerated product launches. This new level of transparency fosters trust among manufacturers and patients alike. By embracing this innovative approach, manufacturers and gematik lay the foundation for efficient and successful collaboration.

# About the author

Julian Peters is a programmer at heart, writing code for 20 years. He joined gematik 11 years ago and held positions ranging from tech lead to software architect. He is the leader of the chapter on testing technologies.
