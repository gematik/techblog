---
layout: post
title: "How we established a Secure Software Development Lifecycle"
date:   2022-12-09 12:00:00 +0200
author: Dr. Alexey Tschudnowsky
categories: tech
tags: SDL SDLC SSDLC security e-prescription
excerpt: "<br/>Building secure software is hard. In this post, we share our experience with establishing a Secure Software Development Lifecycle during the development of the E-Prescription app.<br/><br/>"
---

Building secure software is hard. We learned it once again as we developed the [E-Prescription app](https://www.das-e-rezept-fuer-deutschland.de/app). We faced a lot of security-related design requirements, the need to establish a secure development lifecycle, and, finally, had to prepare for external assessments. Especially the secure lifecycle was a new topic for the team, and we doubted if it could be established without giving up the agile way of software development. 

## What is the Secure Software Development Lifecycle?

The Secure Software Development Lifecycle (SSDLC) is a method to improve software quality by integrating security-focused activities in all phases of software development. Different models for SSDLC emerged. Some examples are [Microsoft SDL](https://www.microsoft.com/en-us/securityengineering/sdl), [OWASP SAMM](https://owaspsamm.org/) or [BSI Guidelines for Secure Web Application Development](https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Publikationen/Studien/Webanwendungen/Webanw_Auftragnehmer.pdf?__blob=publicationFile&v=1). Some of them are generic and can be applied to any software, the other ones have a particular focus on, e.g., Web applications.

In the context of the E-Prescription app, we went for Microsoft SDL as one of the "established" and "proven" approaches. It prescribes several activities before a project starts and during each subsequent phase. Although it follows the spirit of a waterfall process model, we managed to integrate its activities into iterations of Scrum, which we were using to develop increments of the app. The following picture shows the resulting model:

<p align="center">
<img src="{{ site.baseurl }}/assets/img/221209-gematik-sdl/sdl.png" alt="gematik SDL"/>
</p>
 
One should note that before the app development even started, our colleagues from Systems Engineering and Security departments made many considerations regarding the E-Prescription application's overall design and security concept. From the architectural point of view, the E-Prescription app is only one front end in the whole system, with E-Prescription-Server and various healthcare administrative systems being the others. The architecture, threat models, and security requirements for the E-Prescription have already been set, i.e., we had a solid basis to build upon. 

## Integrating Security Activities into Scrum

So first, we started with building a team, setting up the process, and preparing the toolchain. We set up roles and responsibilities in the team, defined quality gates, identified relevant app-specific security requirements, and established a training plan for developers. We knew things might change with time, so especially relevant security requirements and training needs emerged during the process. During the training, which unfortunately took place a bit late, we focused on mobile-specific topics such as [OWASP Top 10 Mobile](https://owasp.org/www-project-mobile-top-10/) and [OWASP MASVS](https://github.com/OWASP/owasp-masvs). Especially the latter was extremely useful, and many team members wished we had conducted the training at the very beginning of the project.

During the subsequent iterations, threat models and risk assessments were continuously refined and extended. A significant impact had a security expert, who was integrated into the team and took part in all agile ceremonies, especially backlog refinement. This way, we could identify new security requirements, topics for deeper analysis, and the need for security-oriented code reviews. During implementation, we made use of Static Application Security Testing (SAST) tools such as [Micro Focus Fortify](https://www.microfocus.com/en-us/cyberres/application-security/static-code-analyzer) and [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/). The reports were analyzed both by developers and security experts. While the SAST tool helped us to identify some code issues at the beginning, there were little to no relevant findings in the later stage of the development.  

Before the first release, external experts extensively assessed and tested the app and the development process. We had to prove and explain that each (security) requirement on the app imposed by the E-Prescription applications as a whole is fulfilled. After the assessment, we noticed how important the traceability of requirements is and how challenging it can be for crosscutting concerns. We made dedicated annotations in code to reference security requirements, enabling us to monitor and prove their fulfillment much easier. Finally, the app has been penetration tested and approved for release by responsible roles. The app's first release went live on the 1st of June 2021. 

Independently of the agile iterations, we established an incident response plan, kept our toolset and pipelines up-to-date, and looked for ways to optimize the process. For example, code reviews were pretty inefficient at the beginning, as we tried to review the complete OWASP MASVS checklist for each and every feature. Later, we only used the checklist on milestones and focused the reviews on specific security requirements defined in the user story. 

More information on the Secure Software Development Lifecycle can be found in our [presentation on Youtube](https://www.youtube.com/watch?v=Ydo3kjnSZ0o&t=1s).

## What's Next

Today, we have a deeper understanding of the SDLC and perform its activities much more efficiently. We can say that they contributed to the app's quality and security. Still, one of the most valuable aspects of an established SDLC was the more profound knowledge of security topics and the increased awareness of security risks for the whole team. We continuously review the process and notice things that can go better. For example, we repeat the evaluation of SAST tools to meet our (increased) demands. Furthermore, we plan to optimize our JIRA workflows to better support us in the above activities. Finally, we learn other process models to apply them in further contexts besides mobile development. 

# About The Author

Dr. Alexey Tschudnowsky is a software architect and works on the [gematik Reference Validator](https://github.com/gematik/app-referencevalidator) project. He is an expert in software architectures, agile development processes, distributed systems, and Web technologies. He helped to establish the Secure Software Development Lifecycle at gematik and continuously consults teams regarding execution. 
