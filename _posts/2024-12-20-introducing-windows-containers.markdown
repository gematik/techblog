---
layout: post
title:  "Introducing Windows Containers"
date:   2024-12-17 08:00:00 +0100
author: Michele Adduci
categories: tech
tags: Jenkins Docker Legacy Modernization
excerpt: "<br/>How Windows containers are helping us to keep legacy projects working and to support new requirements at the same time.<br/><br/>"
---

## About Our Software Landscape

Every product team and department at gematik maintains or develops a set of projects based on a particular technological stack, depending on the goals that need to be achieved. 
We develop a variety of solutions, including:

* backend services,
* desktop applications,
* mobile applications,
* libraries,
* testing frameworks,
* test suites.

The majority of these projects (~90%) are using a JVM-based language, most of the time Java, and can be built and run on both Linux and Windows environments.

Some particular projects, such as the gematik Authenticator, the ePA Library for Primary Systems (ePA Primärsysteme), and connector tests with TTCN-3, require a Windows build.

Until the beginning of 2024, all these projects, built through our build system called [Software Factory](https://code.gematik.de/tech/2022/11/14/software-factory.html), were using the existing build agents at the time. These agents were based on numerous Windows virtual machines and Linux-based Docker containers, each with their dedicated software packages.

## Sustainability and Maintainability

Since 2022, our Software Factory has been running mostly on Google Cloud. Running underutilized virtual machines presents an opportunity to optimize our monthly recurring costs.

Generally speaking, keeping virtual machines regularly updated in terms of operating system patches and software updates poses a challenge for sustainability and maintainability. Performing slow update cycles costs time and requires significant effort. Executing custom operations in the virtual machine or replacing the entire VM during major updates also adds to the workload. Additionally, these actions carry risks: what if there is malicious access to the VM, or some software packages no longer work?

We have faced and addressed similar challenges before by moving away from Linux virtual machines and adopting Linux containers. This led us to the question: why not give Windows containers a try?

## Approaching Windows Containers

Citing the Microsoft Learn [article about Windows containers](https://learn.microsoft.com/en-us/virtualization/windowscontainers/about/), Container Technology has been for years a disrupting one, but today represents a "must", or, using the [Thoughtworks TechRadar](https://www.thoughtworks.com/radar) terminology, containers belong now in the "Adopt" quadrant.

Microsoft offers several base images, that can be used as a starting point:

* Windows - contains the full set of Windows APIs and system services (minus server roles).
* Windows Server - contains the full set of Windows APIs and system services.
* Windows Server Core - a smaller image that contains a subset of the Windows Server APIs-namely the full .NET framework. It also includes most but not all server roles (for example Fax Server is not included).
* Nano Server - the smallest Windows Server image and includes support for the .NET Core APIs and some server roles.

<p align="center">
<a href="#img1">
<img src="{{ site.baseurl }}/assets/img/20241220-win-containers/images.webp" alt="Types of Windows base images"/>
</a>
</p>

As stated in the [article on the Container Images](https://learn.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/container-base-images), there's an important distinction between `Server Core` and `Nano Server`:

> The key difference between these images is that Nanoserver has a significantly smaller API surface. PowerShell, WMI, and the Windows servicing stack are absent from the Nanoserver image.

## Requirements for Windows Containers

Windows containers were initially kicked-off at the time of Windows Server 2016 and Windows 10 Professional/Enterprise (version 1607). Their capabilities were also very limited, compared to what they can do today.

At the time of writing, Windows containers are officially supported up to Windows Server 2022, although initial Images of Windows Server 2025 are already available. Also Windows 11 Pro and Enterprise can run Windows containers. 

Independently from what kind of Windows setup you are using, the basic requirements are the same: 

* Hyper-V must be installed and activated
* the Hyper-V role must be installed before running the Hyper-V isolation
* A container runtime must be installed (Docker for Windows 10/11, one between containerd, Mirantis or Docker/Moby for Windows Server)
* Enough RAM (at least 8 GB)
* Enough Storage Capacity (at least 50 GB)

Another important requirement is the necessity of a build host, either a physical or virtual machine, because Windows Docker images cannot be built or run within another container. Unlike on Linux, the _docker-in-docker_ mechanism is not supported for Windows, due to Hyper-V limitations.

## Assessing the (Many) Limitations

Microsoft offers a [documentation page](https://learn.microsoft.com/en-us/virtualization/windowscontainers/quick-start/lift-shift-to-containers), titled **"Lift and Shift to Windows Containers"**, which outlines the existing limitations of their containers and the current capabilities available for use without relying on virtual machines.

Since our goal was to replace virtual machines with containers for performing headless build tasks, rather than running interactive or long-running applications, we accepted these limitations and invested more time learning about Windows containers.

Running Windows containers necessitates a well-defined orchestration of setups and configurations:

* **Host Operating System**: You must have a specific version of Windows as your host.
* **base images Compatibility**: Depending on the host OS version, only compatible versions of the four available base images can be used.

**Example**: You cannot build or run containers based on Windows Server 2022 if your host is a Windows 10 machine.

In our scenario, we needed to ensure that within Google Cloud:

1. **Virtual Machine Requirement**: A virtual machine was still necessary for building Docker images
2. **Kubernetes Cluster Extension**: Our Kubernetes cluster had to be extended to include a Windows Worker Node Pool with the same compatible Windows version, that can launch a compatible Kubernetes node, running the containers

Since the latest available version in Google Cloud was Server Core 2022, we opted for it after our tests with Server Core 2019 failed.

## Shifting Build Agents to Immutable Infrastructure

In our effort to migrate Build Agents to containers, we have adopted the "cattle, not pets" approach for the host virtual machines responsible for building Windows Docker images. The VM is defined by a template in a Terraform project, allowing us to regularly update the VM itself by simply destroying and recreating it in a straightforward step.

The Windows Docker images themselves are immutable and easily updatable. Updates can be performed by simply upgrading the tag or the SHA256 digest of the base image.

For our use cases, we currently maintain three different Windows images organized in a hierarchy:

* Base Builder: Includes common tools pre-installed.
* General Builder: Designed for backend services and test suites, equipped with multiple JDKs, browsers, Visual Studio Build Tools, and Node.js.
* Konnektor Test Builder: Tailored for connector tests with TTCN-3, featuring JDK 11 and Maven.

For the base builder, we've decided to use `chocolatey` for installing all the needed packages with a given version and define PowerShell as default interpreter for the commands, instead of the default CMD:

```dockerfile
# Configure PowerShell as default execution shell
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'Continue'; $verbosePreference='Continue';"]

# Activate TLSv1.2 and install Chocolatey with basic packages
RUN [Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12; `
    $null=New-Item -Force -Path \"C:/\" -Name \"ProgramData\" -ItemType \"directory\"; `
    iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1')); `
    Write-Verbose \"### Installing basic packages\" -Verbose; `
    choco install git --version 2.47.1 -y --no-progress; `
    choco install openssl --version 3.4.0 -y --no-progress; `
    choco install jq --version 1.7.1 -y --no-progress; `
    choco install yq --version 4.40.2 -y --no-progress; `
    choco install curl --version 8.11.1 -y --no-progress; `
    choco install unzip --version 6.0 -y --no-progress; `
    choco install 7zip --version 24.9.0 -y --no-progress; `
    Write-Verbose \"### Installing Snyk\" -Verbose; `
    Write-Verbose \"### Cleanup TEMP folder\" -Verbose; `
    Remove-Item -Path $env:TEMP\* -Recurse -Force -ErrorAction SilentlyContinue;    
```

In addition to these tools, we've also integrated some static analysis tools for monitoring and improving our Software Quality Gates, such as the OWASP Dependency Scanner, Sonar Scanner CLI and Snyk, whose releases are available through GitHub.

We had also to configure some custom symbolic links, to make sure that Jenkins could work with these agents, instead of being blocked, looking for the `nohup` operation:

```dockerfile
RUN Write-Verbose \"### Configuring nohup for Jenkins\" -Verbose; `
    $null=New-Item -Force -Path \"C:/Program Files/Git/bin/nohup.exe\" -ItemType SymbolicLink -Value \"C:/Program Files/git/usr/bin/nohup.exe\"; `
    $null=New-Item -Force -Path \"C:/Program Files/Git/bin/msys-2.0.dll\" -ItemType SymbolicLink -Value \"C:/Program Files/git/usr/bin/msys-2.0.dll\"; `
    $null=New-Item -Force -Path \"C:/Program Files/Git/bin/msys-iconv-2.dll\" -ItemType SymbolicLink -Value \"C:/Program Files/git/usr/bin/msys-iconv-2.dll\"; `
    $null=New-Item -Force -Path \"C:/Program Files/Git/bin/msys-intl-8.dll\" -ItemType SymbolicLink -Value \"C:/Program Files/git/usr/bin/msys-intl-8.dll\";
```

Other software packages that we have installed in the derived agents are: Visual Studio Build Tools, different JDKs, Maven, Gradle, NodeJS, Flutter, Google Chrome and Microsoft Edge (for browser testing). Installing browsers required the installation of some fonts, which are not shipped withing the Server Core base images. We couldn't add Firefox, since it requires other system APIs, which are not available.

## How Far Have We Come?

The Windows Docker images are quite large. The Server Core base image already occupies approximately 4.5 GB. Installing Visual Studio (VS) Build Tools and other packages increases the size to 13 GB. Despite their size, Docker containers remain significantly smaller than virtual machines, and container launch times are considerably faster. Build times vary between agents, as some packages (such as Visual Studio Build Tools) require substantial network traffic. However, the overall build time is approximately 30 minutes, including the pull and push of images.

We have successfully transitioned many Jenkins jobs to the newer container-based agents, enabling support for various new use cases that previously did not exist, such as monitoring Quality Gates, signing applications and packages, and building C# applications. At the same time, we have been able to isolate legacy projects that rely on older versions of Java and require very specific packages. Once these legacy projects are phased out, we can simply remove the corresponding build agents.

Running container agents has been more cost-effective for us, not only in terms of reduced expenses due to workloads running only when required, but also in terms of maintenance. In just a few steps, new software packages can be added, or existing ones upgraded or removed, typically saving hours of time.

---

## About The Author

Michele Adduci is a Software Architect with a proven track record of assisting organizations in implementing modern solutions, with a strong focus on maintainability, reliability, and efficiency. Since joining gematik in 2021, he has been working on [DEMIS](https://www.gematik.de/anwendungen/demis) and is an integral member of the "Cloud and Deployment Technologies" chapter.

---
