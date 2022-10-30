---
layout: post
title:  "Speeding up the release cycle"
author: Michele Adduci, Christian Lange
categories: devops
tags: Jenkins Git SDLC Kubernetes Docker IaC DevOps
---

Since May 2022 and after a long preparation, we've opened officially at gematik the _Software Factory_ to all our colleagues. It is a collection of services that help developers, test analysts and software architects to build, test and release their projects in a stable, trackable and consistent way.

**Stable**, because the systems that compose the Software Development Life Cycle (SDLC) chain are running 100% on cloud, with little-to-zero downtime, in contrast to the previous on Premise architeture, and it's easily scalable and highly configurable, thanks to the infrastructure being written as code using Terraform and Helm.

**Trackable**, because all the code that lands on the main branch, through push or merge-request events (we use Git as CVS), is tagged with human-readable names. The built artifacts are then fingerprinted and associated to the particular tag and git commit, so eventual problems in a library or application can be easily tracked back to the original code that was used to build it.

**Consistent**, because we've developed our own internal set of functions to be used with our build server and writing also build pipelines themselves as a Blue Print for many projects. In this way, users of the build server don't need to spend a lot of time writing their own pipeline, but can use ready-to-go solutions that fit their projects. This accelerates the development process, shifting the focus on solving business problems, not infrastructure ones.

## Migration

We moved gematik onPrem Build- and Deploymentsystem to fully cloud based scalable Software Factory in several steps with minimal work and workflow disruption on product teams at the last years.

First step was the renew of our old windows based Jenkins System with static Agents, to a completely new Linux based Jenkins Master and declarative Pipeline definitions checkin with the project sourcecode. The active Projects has a long time to migrate the old Jenkins-Jobs to the new declarative Pipelines and old Projects was archived.

The number of builds increase in the following month because we start with attentional Feature-Branch and Merge-Request Builds. The virtual Machine based static Jenkins Agents was not enough and we start with dynamic Docker based Agents. This new Agent concept help us to fulfill the requirement of reproducible builds on defined environment without several builds on the same Agent, every build starts a new clean Agent.
The types of our Jenkins Agents various from Maven-Java to Angular Agents and result in nine Agent-types.

The number of teams and projects increased from month to month and the available hardware for the agents became increasingly scarce, which led to ever longer build times. The next step we took was integrating resources from the google cloud and moving Jenkins agents to a Kubernetes cluster. The number of parallel builds increased significantly, but network communication with the onPrem systems (Git, Nexus, Sonarqube, ...) increased significantly and slowed down the builds.

The last step we took is the migration of all build system components into a google Kubernetes cluster and name it 'Software Factory'.

## What is 'Software Factory'?

At the time of writing, the Software Factory is a composition of following tools:

* GitLab as central CVS server
* Jenkins and custom Build-Agents for the different projects
* SonaType Nexus, for hosting our build artifacts and Helm Charts
* SonarQube, enforcing Quality Gates on code
* Google Container Registry to store Docker Images built in the process
* Different sets of scanners for analyzing code and built artifacts
* Kubernetes and some on Premise Servers for hosting applications
* Monitoring Solutions

![Sketch of the matrix organization @gematik]({{site.baseurl}}/assets/img/221111-swfactory/Software-Factory.webp)
*Logo of Software Factory @gematik (gematik GmbH)*

Most of the above systems are running on Google Cloud, with the exception of some Jenkins Agents and some custom Monitoring solutions, hosted on gematik private servers. Applications that require particular constraints and access to the _Telematik Infrastruktur_ (TI) are hosted on private servers as well.

The whole environment is built using Infrastructure-as-Code with Terraform and Helm, following the industry best practices and being supported by our technical partners.

The access to those systems is granted based on group policies and using a multi-factor authentication Single-Sign-On (SSO) experience for a friction-less usage of the platform.

## Entering the gematik Software Development Life Cycle

The main goals of the Software Factory are:

* provide a seamless experience with the build tools
* require as little-as-possible steps to start using given build tools
* give feedbacks on what's going wrong during builds

All of the above goals can be achieved defining build profiles and flexible agents, backed up with our own set of internal functions (called _Jenkins Shared Library_ or _JSL_), encouraging and, in particular projects, enforcing to address critical issues discovered in our software supply chain.

<!--more about JSL functions and clustering-->

Before hitting the production, artifacts are properly scanned for security or misconfiguration issues and tested using [our own test-suites]({{site.baseurl}}/testing/2022/10/13/zeroline-test-suite). If problems are found, they are visible in form of visual feedback from Jenkins and GitLab but also in form of chat notifications through Microsoft Teams.

### A small example

An example of friction-less usage of the Software Factory can be proven by the following Scenario: imagine you want to publish an internally built Docker Image to Docker Hub.  This translates in the following Jenkins Pipeline file:

```groovy
@Library('gematik-jenkins-shared-library') _
pipelineDockerHubPublish {
    INTERNAL_IMAGE_NAME = "demis/hospital-location-service"
    EXTERNAL_IMAGE_NAME = "gematik1/demis-hospital-location-service"
}
```

which then translates to following Job:

![Sketch of the matrix organization @gematik]({{site.baseurl}}/assets/img/221111-swfactory/pipeline.webp)
*Example of pipeline resulting from Groovy source code*

That's it!

# About the author

Michele Adduci is a Software Engineer with 10+ years experience in different topics such as Computer Vision, eID/eIDAS and DevOps. At gematik since 2021, he focuses on Projects such as the [electronic Patient Record (ePA)](https://www.gematik.de/anwendungen/e-patientenakte) and [DEMIS](https://www.gematik.de/anwendungen/demis).