---
layout: post
title: "FHIRPath In Production: Why It Matters In DEMIS"
date: 2026-01-30 14:00:00 +0200
author: Tabea Harper, Daniel Reckel
category: tech
tags: FHIR FHIRPath DEMIS Validation HAPI-FHIR
excerpt: <br />How DEMIS uses FHIRPath to make validation and routing transparent, testable, and efficient.<br /><br />
---

## Executive Summary

DEMIS uses FHIRPath to implement scenario-driven validation of FHIR notifications. Close collaboration with project
partner RKI and the clever integration of FHIRPath expressions into a Java-based microservice has resulted in a flexibly
expandable, easily maintainable construct that enables rapid technical validation.

> Note: This is Part II of a three-part series. Part I gives an overview of the use of FHIRPath in DEMIS and Part III
> dives into routing.

---

## Part II – Lifecycle Management As Scenario-Driven Validation of FHIR Notifications

### Motivation And Value

FHIR notifications in DEMIS have a defined lifecycle depending on the type of notification. A notification contains
information about the detection of a pathogen or the suspicion of a disease in a patient.
The lifecycle management (LCM) defines various valid scenarios for notifications. Each scenario has rules based on the
combination of values of specific elements of a notification.
Part of the validation of incoming FHIR notifications in DEMIS is to check if it is valid regarding the lifecycle.
Due to the complexity of the rules, it is hard to implement them in a maintainable way using traditional programming
languages.
Using FHIRPath expressions to define the rules has several advantages:

- **Compactness**: FHIRPath expressions are concise and can express complex logic in a few lines.
- **Readability**: The declarative nature of FHIRPath makes it easier to understand the rules at a glance.
- **Maintainability**: Changes to the rules can be made by updating the FHIRPath expressions without modifying the
  underlying codebase.

### DEMIS Lifecycle Management And Scenarios

The functional aspects of the DEMIS interface are handled by our project partner Robert Koch-Institut (RKI). The RKI
consolidates the domain
expertise on legal and substantive questions, while the gematik-team is responsible for the technical implementation and
operations.

In addition to FHIR profiling for creating the data model and the api, the RKI-team also conducted a comprehensive
content review of the
notifications. These notifications should never be seen in isolation, as several notifications can relate to a single
infection. Laboratory notifications confirm the infection and include information about sample testing, such as sample
type, specific test used, or detected antibiotic resistance. Meanwhile, disease notifications contain details about
the patient’s condition and contextual background, for example the location of infection depending on the type of
infectious disease.

The relationship between notifications is crucial and can be established through IDs linking them together. Each initial
notification should have a unique ID, to not link it to another notification of a different disease category or patient
by mistake. However,
these links only make sense if the corresponding values are populated accordingly within each notification. Quality
management of
the notifications is therefore handled through the LCM. Encoding these rules directly into the profiled
resources would make them more rigid and harder to maintain, so it was decided to communicate them via the additional
guidance in the Implementation Guides
for [laboratory notifications](https://simplifier.net/guide/rki.demis.laboratory?version=current)
and [disease notifications](https://simplifier.net/guide/rki.demis.disease?version=current).

The scenarios therefore revolve around initial notifications, follow-up notifications, supplementary notifications, and
correction notifications. While for example initial notifications must not contain an ID that already belongs to
existing notifications, correction, supplementary, or follow-up notifications can only be processed if a corresponding
initial notification has already been received.

While an external reader may not see any differences between follow-up, supplementary, and correction notifications,
these messages do differ either in their technical structure or in the values they carry. Follow-up notifications, for
instance, use a relatesTo relationship via the Composition resource and do not require a named individual. They are
typically sent by secondary laboratories that do not have access to personal data of patients. A supplementary
notification, on the
other hand, is sent by the primary laboratory to contribute additional information. This happens, for example, for a
long-running test or a
test confirmation. This overlaps with a correction notification, which revises the overall result of a previous
analysis, although in theory such a correction could also be issued as a follow-up notification. Drawing clear
boundaries is therefore difficult, a challenge already hinted at in the RKI’s initial scenario table.

### Robust Rule Design

The RKI defines functional, real world scenarios based on their and german health office experiences. These scenarios
have to be translated into unique technical scenarios with unambiguous rules.
Therefore, the RKI provided a scenario table through their Implementation Guides that lists all relevant scenarios and
the allowed values of elements.
Some of the defined functional scenarios could be removed due to technical overlap in field values, as the remaining
differences are purely functional and irrelevant for the lifecycle validation.
To improve traceability, both project partner together created a scenario decision tree that breaks down the scenarios
into smaller decision nodes, making it easier to understand the LCM.
From a domain driven perspective this approach results in common language between the technical and functional teams in
form of the decision tree.

<figure>
  <img src="{{ site.baseurl }}/assets/img/20260130-fhirpath/LCM-Disease_IM_EM.jpg" alt="DEMIS high-level flow with FHIRPath evaluation points"/>
  <figcaption>Decision Tree for Disease Notifications</figcaption>
</figure>

### Rule Implementation and Integration

We implemented the actual validation using HAPI FHIR. The framework provides an API for executing FHIRPath expressions
but requires an extension of the resolve() method, which must be implemented by the user 
(see [part one of this series](https://code.gematik.de/tech/2025/12/22/fhir-path-in-demis-part1.html)).

#### Static Validation via FHIRPath Expressions

For each scenario in the rule tree, we generate matching FHIRPath expressions and apply them to the message under
review. Because the rule tree—unlike the RKI table—does not allow duplicate scenarios, we reduce the number of checks to
the minimum necessary. This static validation can be made without any other context. It just validates the state of
fields in the current notification. If all static checks are valid, the external checks will be performed.

#### Dynamic Validation via External Checks

External Checks prove for initial notifications, that no other notification with the same ID exists.
Furthermore, the LCM describes scenarios for supplementary and follow-up notifications that require the reference to an
initial notification with the same ID and disease category.

These external checks can't be made static due to the limited scope to the state of the current notification. They require
the context of all previous notifications in the lifecycle via additional validation logic in Java. The notification or
relatesTo ID can be extracted from the current notification via FHIRPath in the external check and is used to make a
dynamic request to a service, that serves
the notification category of a previous notification, with the same Notification ID. For supplementary or follow-up
notifications the received disease category will be compared with the one extracted by FHIRPath from the current
notification.

If the external check for the scenario is valid as well, the notification is valid in its lifecycle.

### Performance

There are several straightforward measures to improve performance:

1. Instead of relying on a single, lengthy FHIRPath expression, we break the requirements into smaller chunks. The
   advantage is that we can terminate a validation much earlier; in the long run, we could also consider evaluating them
   in parallel.
2. We only query external services—for example, to validate IDs from previous notifications—after all other checks have
   passed. This reduces the number of calls within the microservice cluster.
3. The rule design avoids duplicating rules wherever possible.
4. We view rapid maintenance and updates as a performance gain, so we work closely with our project partner on a shared
   foundation: the decision tree.

In Part III we will discuss our routing implementation which is closer to a decision tree. We will examine which option
is
more practical and enables better cooperation with our project partner. The development process is not yet complete at
this point and will certainly be part of a later blog entry.

### DEMIS Example

The following example shows the validation configuration for the scenario of an initial preliminary disease
notification, where the disease is not confirmed
yet, but suspected. This is described by the purple path in the decision tree above.
The expression "Patient.profile(byName)" mirrors the first node in the decision tree and checks whether the patient
resource in the bundle has the profile "NotifiedPerson" for identified patients. Each other FHIRPath expression should
be also displayed over a node in the tree.
The external check for this scenario is of type NOTIFICATION_ID_NOT_EXISTING, meaning it is an initial notification
scenario.

```json
{
  "name": "S_IM_V",
  "fhirPathExpression": [
    {
      "fhirPath": "Patient.profile(byName)"
    },
    {
      "fhirPath": "Composition.status(preliminary)"
    },
    {
      "fhirPath": "Condition.clinicalStatus(active)"
    },
    {
      "fhirPath": "Condition.verificationStatus(unconfirmed)"
    }
  ],
  "externalChecks": [
    {
      "type": "NOTIFICATION_ID_NOT_EXISTING",
      "inputs": {
        "id": "NotificationId"
      }
    }
  ]
}
```

The next example shows the configuration for a supplementary notification, were the suspected disease from the initial
notification before is confirmed. This is described by the orange path in the decision tree above.
As already mentioned, supplementary notifications must reference an existing initial notification. Therefore, an
external check of type CATEGORY_MAPPING is
performed to verify that the referencing notification ID exists in the system and is linked to a notification with the
same disease category.

```json
{
  "name": "S_FM_V2EoT",
  "fhirPathExpression": [
    {
      "fhirPath": "Patient.profile(byName)"
    },
    {
      "fhirPath": "Composition.status(final)"
    },
    {
      "fhirPath": "Condition.clinicalStatus(active)"
    },
    {
      "fhirPath": "Condition.verificationStatus(confirmed)"
    }
  ],
  "externalChecks": [
    {
      "type": "CATEGORY_MAPPING",
      "inputs": {
        "id": "NotificationId",
        "hasToExist": true,
        "notificationCategory": "NotificationDiseaseCategory"
      }
    }
  ]
}
```

For better readability and reuse, we have replaced the FHIRPath expressions with short codes.
In a second file, we have documented the concrete FHIRPath expressions that these codes resolve to.

```json
{
  "Patient.profile(byName)": "Bundle.entry.resource.where($this is Patient).meta.where(profile = 'https://demis.rki.de/fhir/StructureDefinition/NotifiedPerson').exists()",
  "Composition.status(preliminary)": "Bundle.entry.resource.where($this is Composition).where(status = 'preliminary').exists()",
  "Composition.status(final)": "Bundle.entry.resource.where($this is Composition).where(status = 'final').exists()",
  "Condition.clinicalStatus(active)": "Bundle.entry.resource.where($this is Condition).clinicalStatus.coding.where(code = 'active').exists() or (Bundle.meta.profile = 'https://demis.rki.de/fhir/StructureDefinition/NotificationBundleDisease' and Bundle.entry.resource.where($this is Condition).clinicalStatus.empty())",
  "Condition.verificationStatus(unconfirmed)": "Bundle.entry.resource.where($this is Condition).verificationStatus.coding.where(code = 'unconfirmed').exists()",
  "Condition.verificationStatus(confirmed)": "Bundle.entry.resource.where($this is Condition).verificationStatus.coding.where(code = 'confirmed').exists()",
  "NotificationId": "Bundle.entry.resource.where($this is Composition).identifier.where(system='https://demis.rki.de/fhir/NamingSystem/NotificationId').value",
  "NotificationDiseaseCategory": "Bundle.entry.resource.where($this is Condition).code.coding.where(system='https://demis.rki.de/fhir/CodeSystem/notificationDiseaseCategory').code"
}
```

## What’s Next

In Part III, we’ll cover smart routing and orchestration with practical integration notes.

---

## About The Authors

Tabea is a Junior Software Developer and has only recently joined the team. Using FHIRPath was one of her first tasks,
and she has been an enthusiastic user ever since.

Daniel is a Senior Software Developer and works on the DEMIS backend since 2021 with great expertise in Java and
FHIRPath.

Tabea and Daniel worked together to revise the implementation of lifecycle management in DEMIS. Their goal was to create
an easy-to-understand and clear configuration that could also be understood and maintained by non-developers from
project partner RKI. The previous version, which was based purely on Java code, had proven itself, but had become very
communication-intensive to maintain. 
