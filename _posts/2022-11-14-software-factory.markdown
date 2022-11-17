---
layout: post
title:  "Software Factory"
date:   2022-11-14 10:00:00 +0200
author: Michele Adduci, Christian Lange
categories: tech
tags: Jenkins Git SDLC Kubernetes Docker IaC DevOps
---

Since May 2022 and after a long preparation, we've officially opened at gematik the _Software Factory_ to all our colleagues. It is a collection of services that help developers, test analysts, and software architects to build, test, and release their projects in a stable, trackable, and consistent way.

**Stable** because the systems that compose the Software Development Life Cycle (SDLC) chain are running 100% on the cloud, with little-to-zero downtime. In stark contrast to the previous on-premise architecture, it's easily scalable and highly configurable, thanks to the infrastructure being written as code using Terraform and Helm.

**Trackable** because all the code that lands on the main branch through push or merge-request events (we use Git as CVS) is tagged with human-readable names. The built artifacts are then fingerprinted and associated with the particular tag and git commit, so eventual problems in a library or application can be easily tracked back to the original code used to build it.

**Consistent** because we've developed our own internal set of functions to be used with our build server and writing also build pipelines as a Blue Print for many projects. This way, users of the build server don't need to spend a lot of time writing their own pipeline but can use ready-to-go solutions that fit their projects. This accelerates the development process, shifting the focus to solving business problems, not infrastructure ones.

## Migration

We moved gematik onPrem Build- and Deployment System to a full cloud-based scalable Software Factory in several steps with minimal work and workflow disruption on product teams in the last years.

The first step was renewing our old windows based Jenkins System with static agents to an entirely new Linux-based Jenkins Master and declarative Pipeline definitions check in with the project source code. The active projects have a long time to migrate the old Jenkins-Jobs to the new declarative pipelines, and old projects were archived.

The number of builds increased the following month because we started with attentional Feature-Branch and Merge-Request builds. The virtual machine-based static Jenkins agents were insufficient, so we started with dynamic Docker-based agents. This new agent concept helps us to fulfill the requirement of reproducible builds on a defined environment without several builds on the same agent. Every build starts a new clean Agent.
The types of our Jenkins agents vary from Maven-Java to Angular agents and result in nine different agent types.

The number of teams and projects increased monthly, and the available hardware for the agents became increasingly scarce, leading to ever longer build times. The next step we took was integrating resources from the google cloud and moving Jenkins agents to a Kubernetes cluster. The number of parallel builds increased significantly, but network communication with the onPrem systems (Git, Nexus, Sonarqube, ...) increased significantly and slowed down the builds.

The last step we took was the migration of all build system components into a google Kubernetes cluster which we named "Software Factory."

## What is the 'Software Factory'?

At the time of writing, the Software Factory is a composition of the following tools:

* GitLab as central CVS server
* Jenkins and custom build agents for the different projects
* SonaType Nexus, for hosting our build artifacts and Helm charts
* SonarQube, enforcing quality gates on code
* Google Container Registry to store Docker images built during the process
* Different sets of scanners for analyzing code and build artifacts
* Kubernetes and some on-premise servers for hosting applications
* Monitoring Solutions

![Sketch of the matrix organization @gematik]({{site.baseurl}}/assets/img/20221114-swfactory/Software-Factory.webp)
*Logo of Software Factory @gematik (gematik GmbH)*

Most of the above systems run on Google Cloud, except for some Jenkins agents and custom monitoring solutions hosted on gematik private servers. Applications that require particular constraints and access to the _Telematik Infrastruktur_ (TI) are also hosted on private servers.

The whole environment is built using Infrastructure-as-Code with Terraform and Helm, following the industry's best practices and being supported by our technical partners.

Access to those systems is granted based on group policies and using a multi-factor authentication Single-Sign-On (SSO) experience for frictionless platform usage.

## Entering the gematik Software Development Life Cycle

The main goals of the Software Factory are:

* provide a seamless experience with the build tools
* require as little-as-possible steps to start using given build tools
* give feedback on what's going wrong during builds

All of the above goals can be achieved by defining build profiles, and flexible agents backed up with our own set of internal functions (called _Jenkins Shared Library_ or _JSL_). These functions encourage and, in particular projects, enforce to address critical issues discovered in our software supply chain.

The _JSL_ simplifies functionalities for building, testing, and deploying applications. The developers do not have to make any adjustments to their pipelines when the environment or default settings for functions change, as these can be made globally for everyone in the JSL. Furthermore, we have clustered the JSL functions according to functionalities such as Git, JIRA, Maven, Gradle, Nexus, Trivy, Docker, and others. For example, the Maven functions offer simple options for querying the version of the current Jira project, just with the Jira project ID, and setting it in the Maven project:

```groovy
@Library('gematik-jenkins-shared-library') _
def JIRA_PROJECT_ID = "..."
pipeline {
    ...
    stages {
        ...
        stage('set Version from Jira-Project') {
            steps {
                mavenSetVersionFromJiraProject(JIRA_PROJECT_ID)
            }
        }
    }
    ...
}
```

Another example is the vulnerability scan functionalities for Docker images. With one command, an HTML report is created for an image and saved as a result for the pipeline:

```groovy
@Library('gematik-jenkins-shared-library') _
def IMAGE_NAME= "MyImage"
def VERSION = "latest"

pipeline {
    ...
    stages {
        ...
        stage('Step') {
            steps {
                trivyVulnerabilitiesScanAllAsHtml(IMAGE_NAME, VERSION)
            }
        }
    }
    ...
}
```

Before hitting production, artifacts are properly scanned for security or misconfiguration issues and tested using [our own test-suites]({{site.baseurl}}/testing/2022/10/13/zeroline-test-suite). If problems are found, they are visible in the form of visual feedback from Jenkins and GitLab and chat notifications through Microsoft Teams.

Another example of frictionless usage of the Software Factory can be proven by the following scenario: imagine you want to publish an internally built Docker image to Docker Hub. The JSL contains not only individual functionalities but also complete pipelines. 
The following Jenkins pipeline uses the gematik Template pipeline _pipelineDockerHubPublish_ and needs only 2 parameters:

```groovy
@Library('gematik-jenkins-shared-library') _
pipelineDockerHubPublish {
    INTERNAL_IMAGE_NAME = "demis/hospital-location-service"
    EXTERNAL_IMAGE_NAME = "gematik1/demis-hospital-location-service"
}
```

which then translates to the following job:

![Sketch of the matrix organization @gematik]({{site.baseurl}}/assets/img/20221114-swfactory/pipeline.webp)
*Example of pipeline resulting from Groovy source code*

That's it! 

How we are constantly validating that the Software Factory was for us the right choice? Teams could increase their productivity in general, especially during the setup of new projects and their related CI/CD pipeline, dedicating their focus on solving business problems. The build times have incredibly been reduced, especially for jobs requiring more computing resources and parallel execution of processes, which all translates to faster iterations and reduced deployment time.

# About the author

Michele Adduci is a Software Engineer with 10+ years of experience in different topics such as Computer Vision, eID/eIDAS, and DevOps. At gematik since 2021, he focuses on Projects such as the [electronic Patient Record (ePA)](https://www.gematik.de/anwendungen/e-patientenakte) and [DEMIS](https://www.gematik.de/anwendungen/demis).

Christian Lange is a software architect with more than 15 years of expertise and leads the Chapter Cloud & Deployment Technologies at gematik. His focus includes software architectures, cloud technologies, DevOps, and tech leadership. Christian Lange works on different projects like [DEMIS](https://www.gematik.de/anwendungen/demis) and Software Factory.
