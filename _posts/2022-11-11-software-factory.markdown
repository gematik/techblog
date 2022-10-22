---
layout: post
title:  "Software Factory and Release Engineering"
date:   2022-11-11 09:00:00 +0200
author: Michele Adduci
categories: infrastructure
tags: Jenkins Git SDLC Kubernetes Docker IaC DevOps
---

Since May 2022 we've opened at gematik officially the _Software Factory_ to all the employees. It is a collection of services that help developers, test analysts and software architects to build, test and release their projects in a stable, trackable and consistent way. 

Stable, because the systems that compose the Software Development Life Cycle (SDLC) chain are running 100% on cloud, with little-to-zero downtime, in contrast to the previous on Premise architeture, and it's easily scalable and highly configurable, thanks to the infrastructure being written as code using Terraform and Helm.

Trackable, because all the code that lands on the main branch, through push or merge-request events (we use Git as CVS), is tagged with human-readable names. The built artifacts are then fingerprinted and associated to the particular tag and git commit, so eventual problems in a library or application can be easily tracked back to the original code that was used to build it.

Consistent, because we've developed our own internal set of functions to be used with our build server and writing also build pipelines themselves as a Blue Print for many projects. In this way, users of the build server don't need to spend a lot of time writing their own pipeline, but can use ready-to-go solutions that fit their projects. This accelerates the development process, shifting the focus on solving business problems, not infrastructure ones.

As the time writing, our Software Factory is using as major services:

* GitLab
* Jenkins and custom Build-Agents for the different projects
* SonaType Nexus, for hosting our build artifacts and Helm Charts
* SonarQube, enforcing Quality Gates on code
* OWASP Scan and Trivy, scanning applications and Docker Images for security issues
* Google Container Registry to store Docker Images built in the process
* Kubernetes and some on Premise Servers for hosting applications

# About the author
