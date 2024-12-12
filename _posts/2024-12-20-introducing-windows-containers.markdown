---
layout: post
title:  "Introducing Windows Containers"
date:   2024-12-20 08:00:00 +0100
author: Michele Adduci
categories: tech
tags: Jenkins Docker Legacy Modernization
excerpt: "<br/>How Windows Containers help us keeping legacy projects working and at the same time supporting new requirements.<br/><br/>"
---

## About our Software Landscape

Every Product Team and Department at gematik maintains or develops a set of projects, based on a particular technological stack, depending on the goal that has to be achieved. We develop a variety of different solutions, including backend services, desktop applications, mobile applications, libraries, testing frameworks and test suites. The majority of these projects (~90%) is using a JVM-based language, most of the time this language is Java and can be built and run in a Linux or Windows environment.

Some particular projects, such as the gematik Authenticator, the ePA Library for Primary Systems (ePA PrimärSysteme) and Konnektor Tests with TTCN-3 require a Windows build.

Until the beginning of 2024, all these projects, built through our Build System, called [Software Factory](https://code.gematik.de/tech/2022/11/14/software-factory.html), were using the back-then existing Build Agents, which were based on many Windows Virtual Machines and Linux-based Docker Containers, all with their dedicated software packages.

## Sustainability and Maintainability

Since our Software Factory is running mostly on Google Cloud since 2022, having Virtual Machines running and being underutilized most of the time represents an opportunity for optimising our monthly recurring costs.

Generally speaking, keeping Virtual Machines regularly updated, in terms of operating system patches and software updates, poses a challenge in terms of sustainability and maintainability. Performing slow update cycles, executing custom operations in the virtual machine or replacing the whole VM with a newer one by big updates are all operations that cost time and requires a big effort, but has also some risks: what if there is a malicious access in the VM or some software packages don't work anymore? 

We have faced, and addressed, similar challenges before, moving away from Linux Virtual Machines and adopting Linux Containers, so the question was: why not give Windows Containers a try?

## Approaching Windows Containers

Citing the Microsoft Learn [article about Windows Containers](https://learn.microsoft.com/en-us/virtualization/windowscontainers/about/), Container Technology has been for years a disrupting one, but today represents a "must", or, using the [Thoughtworks TechRadar](https://www.thoughtworks.com/radar) terminology, Containers belong now in the "Adopt" quadrant.

Microsoft offers several base images, that can be used as a starting point:

* Windows - contains the full set of Windows APIs and system services (minus server roles).
* Windows Server - contains the full set of Windows APIs and system services.
* Windows Server Core - a smaller image that contains a subset of the Windows Server APIs–namely the full .NET framework. It also includes most but not all server roles (for example Fax Server is not included).
* Nano Server - the smallest Windows Server image and includes support for the .NET Core APIs and some server roles.

<p align="center">
<a href="#img1">
<img src="{{ site.baseurl }}/assets/img/20241220-win-containers/images.webp" alt="Types of Windows Base Images"/>
</a>
</p>

As stated in the [article on the Container Images](https://learn.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/container-base-images), there's an important distinction between `Server Core` and `Nano Server`:

> The key difference between these images is that Nanoserver has a significantly smaller API surface. PowerShell, WMI, and the Windows servicing stack are absent from the Nanoserver image.

## Requirements for Windows Containers

Windows Containers were initially kicked-off at the time of Windows Server 2016 and Windows 10 Professional/Enterprise (version 1607). Their capabilities were also very limited, compared to what they can do today.

At the time of writing, Windows Containers are officially supported up to Windows Server 2022, although initial Images of Windows Server 2025 are already available. Also Windows 11 Pro and Enterprise can run Windows Containers. 

Independently from what kind of Windows setup you are using, the basic requirements are the same: 

* Hyper-V must be installed and activated
* the Hyper-V role must be installed before running the Hyper-V isolation
* A container runtime must be installed (Docker for Windows 10/11, one between containerd, Mirantis or Docker/Moby for Windows Server)
* Enough RAM (at least 8 GB)
* Enough Storage Capacity (at least 50 GB)

There is also an important requirement: you still need a build host, in form of physical or virtual machine, since a Windows Docker Image cannot be built or run inside another container: the "docker-in-docker" mechanism like in Linux is not possible. 

## Assessing the (many) limitations

Microsoft itself offers a documentation page containing a list of existing limitations of their containers and what is possible, today, use with them, instead of relying on virtual machines, called ["Lift and shift to Windows containers"](https://learn.microsoft.com/en-us/virtualization/windowscontainers/quick-start/lift-shift-to-containers).

Since at gematik we looked for replacing our virtual machines with containers to perform headless build tasks and not run any interactive or long-running application, we have accepted the limitations and digged further into using Windows Containers.

Running Windows Containers requires a well-defined orchestration of setups and configurations: you have a given version of Windows as your host, then depending on it, you can only use a compatible version of one of the four available Base Images. As an example: you can't build or have Containers based on Windows Server 2022 but your host ist a Windows 10 machine. 

In our case, we had to make sure that in Google Cloud, not only we still needed a Virtual Machine for building the Docker Images, but also our Kubernetes Cluster had to be extended, to have Windows Worker Nodes, with the same compatible version of Windows. Since in Google Cloud the latest available version was Server Core 2022, we opted for it, after failing tests with the Server Core 2019.

## Shifting Build Agents to Immutable Infrastructure

In our task to move Build Agents to Containers, we have adopted the approach "cattle, not pets" also with the host Virtual Machine, responsible for building the Windows Docker Image: the VM is defined by a template in a Terraform project, so we can regularly update the VM itself by simply destroying and recreating it in a simple step.

The Windows Docker Images themselves, once built, are also immutable and easly updatable, by simply upgrading the tag or sha256 digest of the base image.

For our use cases, we havea at the moment 3 different Windows Images, in a hierarchy:

* a base builder, with common tools pre-installed
* a general builder for backend services and test suites, with many JDKs, Browsers, Visual Studio Build Tools, NodeJS
* a builder for Konnektor Tests with TTCN-3 with JDK 11 and Maven

For the base builder, we've decided to use `chocolatey` for installing all the needed packages with a given version and define PowerShell as default interpreter for the commands, instead of the default CMD:

```dockerfile
# Configure PowerShell as default execution shell
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'Continue'; $verbosePreference='Continue';"]

# Activate TLSv1.2 and install Chocolatey with basic packages
RUN [Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12; `
    $null=New-Item -Force -Path \"C:/\" -Name \"ProgramData\" -ItemType \"directory\"; `
    iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1')); `
    Write-Verbose \"### Installing basic packages\" -Verbose; `
    choco install git --version 2.47.0.20241025 -y --no-progress; `
    choco install openssl --version 3.4.0 -y --no-progress; `
    choco install jq --version 1.7.1 -y --no-progress; `
    choco install yq --version 4.40.2 -y --no-progress; `
    choco install curl --version 8.11.0 -y --no-progress; `
    choco install unzip --version 6.0 -y --no-progress; `
    choco install 7zip --version 24.8.0 -y --no-progress; `
    Write-Verbose \"### Installing Snyk\" -Verbose; `
    Write-Verbose \"### Cleanup TEMP folder\" -Verbose; `
    Remove-Item -Path $env:TEMP\* -Recurse -Force -ErrorAction SilentlyContinue;    
```

In addition to these tools, we've also integrated some static analisys tools for monitoring and improving our Software Quality Gates, such as the OWASP Dependency Scanner, Sonar Scanner CLI and Snyk, whose releases are available through GitHub.

We had also to configure some custom symbolic links, to make sure that Jenkins could work with these agents, instead of being blocked, looking for the `nohup` operation:

```dockerfile
RUN Write-Verbose \"### Configuring nohup for Jenkins\" -Verbose; `
    $null=New-Item -Force -Path \"C:/Program Files/Git/bin/nohup.exe\" -ItemType SymbolicLink -Value \"C:/Program Files/git/usr/bin/nohup.exe\"; `
    $null=New-Item -Force -Path \"C:/Program Files/Git/bin/msys-2.0.dll\" -ItemType SymbolicLink -Value \"C:/Program Files/git/usr/bin/msys-2.0.dll\"; `
    $null=New-Item -Force -Path \"C:/Program Files/Git/bin/msys-iconv-2.dll\" -ItemType SymbolicLink -Value \"C:/Program Files/git/usr/bin/msys-iconv-2.dll\"; `
    $null=New-Item -Force -Path \"C:/Program Files/Git/bin/msys-intl-8.dll\" -ItemType SymbolicLink -Value \"C:/Program Files/git/usr/bin/msys-intl-8.dll\";
```

Other software packages that we have installed in the derived agents are: Visual Studio Build Tools, different JDKs, Maven, Gradle, NodeJS, Flutter, Google Chrome and Microsoft Edge (for browser testing). Installing browsers required the installation of some fonts, which are not shipped withing the Server Core base images. We couldn't add Firefox, since it requires other system APIs, which are not available.

## How far we got?

The Windows Docker Images are quite big. The Server Core Base Image occupies already ~4.5 GBytes. Installing VS Build Tools and other packages bumps the size to 13 GBytes. It is still much less than a Virtual Machine and the launch of the container is considerably _faster_. The build time varies from agent to agent, since some packages (such as Visual Studio Build Tools) require a lot of network traffic, but approximately the build time is around 30 Minutes, including pull and push of images.

We have successfully transitioned many Jenkins Jobs to the newer Container-based agents, supporting in a shorter period of time many newer use cases that previously were not existing (e.g. monitoring of Quality Gates, Signing Applications and Packages, building C# applications). At the same time, we could isolate legacy projects that rely on older versions of Java and that require very specific packages to work with: once these projects will be phased out, we can simply remove the build agent.

Running Container Agents has been for us cheaper, not only in terms of costs, having workloads running only when it is really required, by also in terms of maintenance: in few steps, new software packages can be added or the existing ones upgraded or removed, generally saving hours of time.

---

## About the author

Michele Adduci is a Software Architect with a proven track record of helping organisations implementing modern solutions, focusing a focus on maintainability, reliability and efficiency. At gematik since 2021, he works on [DEMIS](https://www.gematik.de/anwendungen/demis) and is part of the Chapter "Cloud and Build Technologies".

---
