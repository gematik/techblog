---
layout: post
title: "How Setting Everything as Code Accelerated the DEMIS Development"
date: 2025-12-05 16:00:00 +0200
author: Michele Adduci
categories: tech
tags: demis,terraform,kubernetes,helm,docker,security
excerpt: <br />We show how Shifting-Left-and-Down contributed to the moderization of the DEMIS Platform, reducing the time-to-production and improving the overall quality of the product.<br /><br />
---


In the last decade, the world of IT has adopted Infrastructure as Code (IaC) more as a philosophy or style of work, instead of a technical achievement.
The introduction of Terraform, developed by HashiCorp by the vision of their former CEO, Mitchell Hashimoto, has shaken and shaped how organizations set up their infrastructure and deliver their products. The recent fork of Terraform, OpenTofu, has also proven that the industry does care about this particular philosophy, since big companies are now funding the new project and speeding up with the development of important new features.

## Why Does This Matters and Why Terraform?

The key feature of Terraform is its extensibility: its Plug‘n‘Play design let users download and use custom providers to define the desired resources. You can create a whole Kubernetes Cluster, configure Identities in Keycloak, manage Groups on Microsoft EntraID and much more, with just few lines of code. But it’s not done yet. 

The great thing about Terraform/OpenTofu in general is that a whole ecosystem has grown around it: linters, security scanning tools, dashboards, collaborative platforms have been developed and extended the developing IaC with the concepts of Auditing, Security, Policies as Code.

## Supporting the Monitoring of COVID Outbreak in Germany

At gematik, in particular for the [DEMIS Project](https://www.gematik.de/anwendungen/demis), we have also adopted this philosophy. DEMIS has been developed last decade outside the gematik and by an impulse from the [Robert-Koch-Institute](https://www.rki.de/DE/Themen/Infektionskrankheiten/Meldewesen/DEMIS/demis-node.html) (RKI) as a concept for monitoring the diffusion of major diseases. With the COVID-19 outbreak, gematik got the important task of „resuming“ the development of the DEMIS Project and make it work to track the status of infections during the pandemic times. Due the initial time pressure, we have just made the existing setup work again and adapt it to make it stable enough for ingesting daily requests in the hundreds of thousands COVID-19 reports during peak pandemic times, with the challenge of keeping a lot of data encrypted at rest for days and keep the platform operative as much as possible.

Once a stable operational status has been established, the project has entered a new phase of development of new features, mostly under suggestion of the RKI, and a modernization of the architecture of DEMIS itself. The services were back then all running as Java Servlets on different Tomcat Webservers, each of them running in a dedicated Virtual Machine, on a private datacenter, operated by our partners Release and Deployment cycles has been fairly long, usually with a cadence of 3 weeks, combined with the length of our Scrum sprints. A deployment was performed completely manually on the production infrastructure by external providers, since we hadn‘t the permission to do it ourselves.

## Modern Problems Require Modern Solutions

Adopting a shift in our mindset and organization‘s structure helped us bring development to a new level. The DEMIS group gradually embraced the Scaled Agile Framework, gotten its own Agile Release Train and involving directly the RKI for the planning and strategic decisions around the platform. The group itself has been splitted in three Stream Aligned Teams, working on parallel topics, while keeping the system working.  

On the technical side, we have containerized our applications while developing new features, so we could have a local environment for testing, but also spin up a test environment where changes could be deployed, using a CI/CD pipeline. This shortened the feedback loop and improved the development experience consistently. The real game changer here has been the adoption of Feature Flags, which strategically enabled/disabled some behavioral changes in the (legacy) applications, that were needed when running as Docker containers, rather than as on-Premises ones.

Next step was the adoption of Kubernetes, moving away from Virtual Machines and Tomcat and embracing Docker images as way to deliver software and establish significant changes in the configuration of the applications: properties and credentials have been fully externalized (see [here](https://docs.spring.io/spring-boot/reference/features/external-config.html) for more information about it) and injected, without the need to recompile the applications themselves. In this shift, also the way of updating the applications has changed, leveraging the strategies and possibilities offered by Kubernetes for updating or replacing existing components, with or without downtime. We committed ourselves to embrace the Blue/Green Deployment strategy, so we could perform some tests live, before activating a newer version and sunset a previous one. 

For the applications, we’ve opted for Helm Charts, which are prepared and versioned right after the building of the associated Docker image: in our case, the Helm Chart of an application is stored right in the same repository were the source code is stored.

The migration route started as soon as possible as a local project, since the target environment(s) where DEMIS was supposed to run wasn‘t available yet, due to regulatory requirements. The whole process of setting up a Kubernetes Cluster started with the adoption of [KIND](https://kind.sigs.k8s.io/) (Kubernetes In Docker), which supports a declarative configuration for its setup. For the Blue/Green Deployment strategy and for its additional security features, we‘ve adopted Istio as Service Mesh, enabling mutual-TLS between the Workloads, authorization and authentication policies and using its native observability features to track down the usage of our applications. We‘ve blended KIND with Istio, Prometheus, Jaeger and Kiali into a OpenTofu project, which configured a local environment in a simple way. From there, using the Helm provider, we could also deploy the DEMIS applications in a matter of minutes.

The major architectural feature of this setup is the tailored modularization of the services running into the Kubernetes Cluster: the main IaC code has been written in a way that is the same for all the different stages (local, development, quality-assurance, preview, reference, production) and zones (two at the time of writing) being used. The per-stage and per-zone configurations and credentials are provided separately, with the help of external repositories that act as an extra overlay, instrumenting the main IaC code. Here takes place the definitions of e.g. Feature Flags, Workload resources and replicas, version of applications being deployed, definition of Istio Policies. Credentials come from a Secrets Engine and these are injected in the system as Kubernetes Secrets. The main goal of this approach has always been the reduction of differences across the external repositories (typically known as „snowflake effect“) and embrace GitOps in a way that we can always track and audit changes in the system. 

## Benefits from Everything as Code

Embracing Terraform (and later on, OpenTofu) for the local development, but also for the continuous deployment of changes, backed by different levels of tests (integration, browser tests, End-to-End tests, load tests), run at different phases, has fundamentally improved the quality of the applications, but also offered a platform where developers have acquired a better confidence in introducing changes or new features. We now perform multiple deployments to the production stage per week, compared to one every 3 weeks and the mean time for deployment is around 10 minutes, including the execution of tests.

By using IaC code, but also Helm Charts and Dockerfiles for the applications, additional definitions of policies for Istio and Kyverno, we can continuously audit the code, scan it for security issues or scale the platform to enable new features. 
Introducing the right abstractions and clear interfaces has reduced the cognitive load on all the DEMIS Team members: for all the backend applications a golden template has been developed, on the base of SpringBoot, including preconfigured features such as metrics, observability, logging, but also a Dockerfile and a pre-defined Helm Chart, so the developers can actually focus on the business logic from the beginning. Thanks to our [Software Factory](https://code.gematik.de/tech/2022/11/14/software-factory.html), we could design Pipeline Templates, that require only few input parameters for each project, to perform the classical steps such as build, test, scan for issues and quality gates, push to a registry, making the integration of new code in the platform simple.

The project, including all the source code of the single applications, but also the OpenTofu „engine“ responsible for the deployment of the platform, is available on GitHub, at [this link](https://github.com/gematik?q=demis-)


---

## About the author

Michele Adduci is a Software Architect with 15 years of experience and with a strong focus on maintainability, reliability, and efficiency. Since joining gematik in 2021, he has been working on different projects, such as [electronic Patient Record (ePA)](https://www.gematik.de/anwendungen/epa-fuer-alle), [DEMIS](https://www.gematik.de/anwendungen/demis) and the [National FHIR Terminology Server](https://terminologien.bfarm.de/) and is an integral member of the "Cloud and Deployment Technologies" chapter.
