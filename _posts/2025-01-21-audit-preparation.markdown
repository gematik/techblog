---
layout: post
title:  "Audit Preparation in E-Rezept App"
date:   2025-01-21 10:10:10 +0200
author: Dinesh Gangatharan
categories: tech
tags: Audits
---

As an open-source and publicly funded application, the E-Rezept app must consistently pass rigorous audits. These audits are critical, and while auditors have access to the entire codebase, it’s our responsibility to make their task easier by providing well-documented and structured audit artifacts.

To achieve this, we’ve developed a process that leverages Kotlin’s annotation and HTML libraries. Let’s explore how we make our app audit-ready.


# Preparing for Audits in the E-Rezept App

---

## Chapter 1: Making the Code Audit-Ready

### Introducing the `@Requirement` Annotation

The cornerstone of our process is the `@Requirement` annotation. This custom annotation is used throughout the app to embed detailed information about requirements directly into the codebase:

```kotlin
@Repeatable
annotation class Requirement(
    vararg val requirements: String,
    val sourceSpecification: String,
    val rationale: String,
    val codeLines: Int = 15
)
```

#### Example Usage

Here’s an example of how we use the annotation:

```kotlin
@Requirement(
    "O.Auth_8#4",
    sourceSpecification = "BSI-eRp-ePA",
    rationale = "The timer is set to 60 seconds.",
    codeLines = 2
)
```

#### Annotation Breakdown

- **`requirements`**: A comma-separated list of requirement identifiers. Each identifier corresponds to a specific regulation or standard.
- **`sourceSpecification`**: Identifies the group or source of the requirement (e.g., BSI or Gematik specifications).
- **`rationale`**: Provides a descriptive explanation of how the implementation satisfies the requirement. This can range from a brief sentence to a detailed paragraph.
- **`codeLines`**: Specifies how many lines of code below the annotation are relevant for documentation.

---

### Centralized Documentation with `requirements.properties`

To maintain a high-level overview, we use a `requirements.properties` file. This file captures a comprehensive description of each requirement, allowing us to link multiple code blocks to a single requirement. This structure ensures that auditors can easily correlate code implementations with overarching requirements.

---

## Chapter 2: The Plugin That Brings It All Together

To streamline the process, we created a Gradle plugin called `technical-requirements`. This plugin automates the generation of audit artifacts in three steps: setup, execution, and presentation.

[![Audit Preparation Workflow](/assets/img/20250121-audit-preparation/process-uml.svg)](/assets/img/20250121-audit-preparation/process-uml.svg)

*Fig.1: Workflow*

### 1. Setup: Preparing the Specifications

The first task, `downloadBsiSpecs`, downloads and parses requirements from BSI and Gematik specification sources. Using an `enum` class, we map specifications to their sources:

```kotlin
enum class SpecificationSource(
    val spec: String,
    val url: String,
    val isFromGemSpec: Boolean = true
)
```

This mapping ensures that specifications are consistently parsed, regardless of their source. Any changes to specifications are detected during this setup phase.

---

### 2. Execution: Extracting and Matching Requirements

The second task, `generateTechnicalRequirements`, ties the annotations in the codebase to the specifications. Here’s how it works:

#### Obtaining the Source

- The specifications are downloaded as HTML files.
- These files are parsed to extract relevant requirement details, which are then matched with the `@Requirement` annotations in the codebase.

#### Extracting Annotation Data

We analyze the repository to collect information from the `@Requirement` annotations, including:

- **Rationale**: The purpose of the implementation.
- **File Name and Line Number**: The exact location of the code.
- **Code Block**: The relevant lines of code.

This data is then matched with the parsed requirements from the HTML source.

---

### 3. Presentation: Generating the Final Document

The final task, `extractRequirements`, generates an HTML document using `kotlinx.html`. Here’s what the document includes:

#### Structured View

- **Grouped by Specification**: Requirements are displayed in groups based on their source.
- **Grid View**: Each requirement is presented in a grid format for easy navigation.

#### Requirement Details

Clicking on a requirement reveals:

- **Specification Details**: Displayed as a table, similar to the original specification page.
- **Notes Section**: Information from the `requirements.properties` file.
- **Code Block**: The annotated code, formatted and styled using `prism.js` for readability.

---

### Packaging for Auditors

The final step involves packaging the HTML document with the necessary CSS, JavaScript, and supporting files into an audit folder. This folder is included in the repository, ensuring the audit artifacts are always available.

<table style="border-collapse: collapse; border: none; width: 100%; text-align: center;">
  <tr>
    <td style="border: none;">
      <a href="{{ site.baseurl }}/assets/img/20250121-audit-preparation/audit-report-screenshot-1.png">
        <img src="{{ site.baseurl }}/assets/img/20250121-audit-preparation/audit-report-screenshot-1.png" alt="Screenshot 1" width="350" />
      </a>
    </td>
    <td style="border: none;">
      <a href="{{ site.baseurl }}/assets/img/20250121-audit-preparation/audit-report-screenshot-2.png">
        <img src="{{ site.baseurl }}/assets/img/20250121-audit-preparation/audit-report-screenshot-2.png" alt="Screenshot 2" width="350"/>
      </a>
    </td>
  </tr>
</table>

*Fig.2: Final Results*

---

## Conclusion

By embedding requirements directly into the codebase and automating the generation of structured documentation, we’ve significantly streamlined the audit process. This approach not only saves time but also ensures transparency and compliance with standards.

If you’re working on a similar project, consider adopting annotations and plugins to make your code audit-ready. It’s a worthwhile investment that pays off during every audit.

The source code for the above process can be found [here](https://github.com/gematik/E-Rezept-App-Android/tree/master/plugins/technical-requirements-plugin).


# About the author
Dinesh Gangatharan has worked in mobile technologies since 2016, specializing in Android, KMP, Flutter, and iOS. He has contributed to health tech projects like ePA and E-Rezept, as well as large e-commerce platforms and eID systems.

