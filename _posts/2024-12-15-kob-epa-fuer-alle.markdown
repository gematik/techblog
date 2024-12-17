---
layout: post
title:  "How the KOB Test Suite Improves Development and Compliance Processes"
date:   2024-12-16 16:51:16 +0200
author: Julian Peters
categories: testing
tags: testing test-suite tiger kob kig epa
---

Gematik’s open-source KOB Test Suite, powered by Tiger, transforms healthcare digitization by enabling local testing, CI integration, and streamlined development. Discover how it fosters trust and efficiency in ePA rollout.

## Tiger Sets New Standards in Conformity Assessment

Digitization in healthcare is inherently complex, particularly when it comes to creating open systems that enable seamless collaboration among diverse stakeholders. This challenge is particularly evident in the rollout of the new electronic patient record (ePA für alle).

Millions of patients must be able to access and manage data across approximately 100,000 medical practices.  
This requires systems developed by around 200 manufacturers to not only securely process data and establish connections but also ensure smooth interoperability.

To guarantee this level of collaboration, the German legislature tasked **gematik** with implementing a conformity assessment (KOB). Historically, this process involved submitting software systems to gematik for review—an effort-intensive approach for both parties. Tests were time-consuming, requiring the entire process to be repeated even for minor errors, accompanied by detailed reports at the manufacturer’s expense. Alternatively, tests were conducted on central servers, which were equally cumbersome and challenging to integrate into the manufacturers’ development workflows.

With the rollout of the 'ePA für alle', a new approach has been introduced: the **KOB Test Suite**, based on Tiger. Tiger, developed by gematik, is a test framework designed for rapid, transparent, and efficient test suites. The KOB Test Suite is publicly available on GitHub, free of charge, allowing manufacturers to download and execute it locally early in their development process. This supports integration into existing CI pipelines and use on individual developer machines. Test results can be uploaded to the confirmation portal, where the largely automated review process certifies conformity in a short time.

## A Paradigm Shift in Testing

This new process offers significant advantages for all parties involved: faster public availability of test suites, early support during development, precise and transparent test results, and flexible integration into internal workflows. This iterative approach fosters better collaboration between gematik and manufacturers, enabling more efficient development.

By entrusting testing to manufacturers while maintaining complete transparency regarding test content, gematik builds trust. The open-source nature of the test suite ensures all code can be reviewed and understood. Early testing (a "shift-left" approach) further supports manufacturers by improving time-to-market outcomes.

## The Tiger Framework

Tiger is a comprehensive test solution designed to simplify the creation of various test types, including end-to-end, acceptance-, integration-, and interface tests. Ideal for resource-constrained teams and agile processes, it is open-source [], Java-based, and built on Cucumber, offering flexibility and customizability for automated testing.

Tiger’s core feature is a proxy that intercepts and analyzes network traffic between components. This traffic is converted into a protocol-agnostic hierarchical structure (a tree), searchable with **RbelPath**, a query language inspired by XPath and JsonPath. This eliminates the need for advanced programming skills, making the framework accessible even for non-technical users.

To enhance collaboration, Tiger provides the **WorkflowUI**, an HTML-based visualization of captured traffic, simplifying documentation and communication around test scenarios. The framework's plugin-based architecture supports extensions tailored to specific needs, such as Docker container provisioning or FHIR resource validation. Additionally, **Zion** enables the rapid deployment of mock services with simple logic. Future blog posts will explore these technical aspects in greater detail, diving further into the Tiger framework and its capabilities.

## KOB Tests with Tiger

The KOB test suite is open source and available for download [on GitHub](https://github.com/gematik/kob-Testsuite/)[^1]. Tests are executed directly in the manufacturers' environments, enabling the use of custom setups, execution on developer machines, or on centralized build servers. This allows tests to adapt to different workflows while providing rapid and clear results.

The test suite can be integrated into CI pipelines and supports direct development against the final test suite. This approach helps streamline processes and align manufacturers’ development with testing requirements. gematik can also reduce its workload by delegating parts of the test setup to manufacturers.

Optional test cases included in the suite allow for technical clarification of specific issues, particularly for non-functional aspects such as endpoint usage patterns. These cases complement gematik’s existing documentation and support services, aiming to capture potential challenges in an automated manner.

## Implementation Challenges and Long-Term Benefits

The implementation effort for manufacturers is significant and should not be underestimated. A key challenge lies in integrating the test driver interface, which must be tailored to align with the user journeys being tested. This adaptation requires additional development work and can become complex depending on the manufacturer’s existing infrastructure and technical constraints. Moreover, building the expertise to effectively use the test framework demands time and resources, particularly for smaller manufacturers with limited development capacity.

Despite these initial difficulties, the effort pays off in the long term. Once implemented, the KOB test suite enables continuous, automated testing through integration with CI processes. This improves error detection, enhances software quality, and supports faster market readiness for products that meet compliance requirements. Over time, maintenance costs can be reduced due to improved test coverage. Additionally, the test suite increasingly standardizes testing across different products, simplifying workflows for manufacturers. Adopting the KOB test suite can contribute to more efficient development processes and closer collaboration between manufacturers and gematik.

## Conclusion

The introduction of the KOB Test Suite marks a pivotal advancement in conformity assessment. While the initial implementation, particularly of the test driver interface, presents challenges, the suite offers significant long-term benefits: Reduced maintenance costs, automated testing, and faster time-to-market for new products are achievable goals. This new transparency from gematik fosters greater trust among manufacturers and patients. By investing in this approach, both manufacturers and gematik establish a solid foundation for efficient and successful collaboration.

# About the author

Julian Peters is a programmer at heart, writing code for 20 years. He joined gematik 11 years ago and held positions ranging from tech lead to software architect. He is the tech-lead of the Tiger-Team developing the Tiger Test framework.

[^1]: <https://github.com/gematik/kob-Testsuite/>
[^2]: <https://github.com/gematik/app-Tiger>
