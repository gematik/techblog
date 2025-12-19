---
layout: post
title: "FHIRPath in production: Why it matters in DEMIS"
date: 2025-11-28 14:00:00 +0200
author: Verena Sadlack, Daniel Reckel
category: tech
tags: FHIR FHIRPath DEMIS Validation Routing HAPI-FHIR
excerpt: "<br />How DEMIS uses FHIRPath to make validation and routing transparent, testable, and efficient.<br />"
---

## Executive summary

FHIRPath is a compact, declarative language to navigate and filter FHIR resources optimized for hierarchical data
structures used in the public health sector. In DEMIS it makes complex rules transparent, testable, and reusable across
services. Instead of writing long complex code platform-independent path, expressions can be used to read out content
and
for decision-making or analysis. This post explains the why and how, with practical examples you can adapt.

> Note: This is Part I of a three-part series. Part II dives into scenario-based validation, and Part III into routing.

---

## Part I – Why FHIRPath matters in DEMIS: Fundamentals and quick wins

### Motivation and value

Health data from various sources must be processed reliably, transparently, and flexibly in the "Deutsches
Elektronisches
Melde- und Informationssystem für den Infektionsschutz" project which translates roughly to German Electronic Reporting
and Information System for Infection Protection or short DEMIS. Requirements for traceability, auditability,
and rapid adaptability are high, especially when reporting paths or validation rules change. This is where FHIRPath
comes
in: FHIRPath allows complex checks and filtering to be expressed directly on FHIR resources, without burying logic deep
in the code. This makes collaboration between developers, domain experts, and testers easier and enables quick wins when
implementing new requirements.

### What is FHIRPath?

FHIRPath is a declarative expression language designed specifically for navigating, filtering, transforming and
evaluating HL7 FHIR data models. It works similarly to XPath for XML, but is tailored to FHIR's structure which is
optimized for the public health sector. With FHIRPath, you can target elements, values, and structures within a FHIR
resource, filter and check them compactly and understandably. HAPI FHIR, a Java based framework, uses FHIRPath
expression when showing positions of warnings, errors, and information in its validation outputs.

**Example 1:**

```
Patient.name.where(use = 'official').exists()
```

This expression checks whether a patient has an official name entry using navigation and filtering.

**Example 2:**

```
Patient.name.where(use = 'official')
```

This expression returns the official name entry or null, when none is part of the resource.

---

## FHIRPath in a nutshell

FHIRPath is an expression language tailored for FHIR. You use it to navigate resource trees, filter collections, and
compute booleans for decisions.

### Core operators in the DEMIS context

In DEMIS, the following FHIRPath operators are especially relevant:

- **Navigation:** Access nested elements (`resource.element.subElement`)
- **Filtering:** Select by condition (`where(predicate)`)
- **Existence check:** Is an element present? (`exists()`)
- **Counting:** How many elements are present? (`count()`)
- **Type check:** Is a value of the desired type? (`is(Type)`, `as(Type)`)
- **Reference resolution:** Find linked resources (`resolve()`)
- **String and set operations:** Contains, starts with, is in (`contains`, `startsWith`, `in`, `union`)

With these operators, the most important validation and routing rules in DEMIS can be mapped directly and transparently.

### DEMIS notification example

As DEMIS notification bundles can be quite large, we do not want to go beyond the scope of this blog. However, you can
find an  [example](/assets/examples/2025-11-28-fhir-path-in-demis-CVDP-example.json) of a laboratory report on a
SARS-CoV-2 pathogen here.
These reports consist of a composition, a patient, up to two practitioners, multiple organizations, pathogen detection,
multiple observations and at least one specimen.

### Navigation and targeted selection

These are some examples of FHIRPath expressions we use to navigate notifications or to find and return data.
You can look up the expressions in the [example file](/assets/examples/2025-11-28-fhir-path-in-demis-CVDP-example.json)
mentioned earlier.

```
// Find LOINC-coded Observation for a specific test
Bundle.entry.resource
  .where($this is Observation)
  .where(code.coding.where(system = 'http://loinc.org' and code = '94309-2').exists())
```

Used on the example file this expression would return 'true'.

```
// At least one phone number present
Bundle.entry.resource.where($this is Patient).telecom.where(system = 'phone').count() > 0
```

Used on the example file this expression would return 'true'.

```
// Bundle adheres to a specific profile
Bundle.meta.profile.contains('https://demis.rki.de/fhir/StructureDefinition/NotificationBundleLaboratory')
```

Used on the example file this expression would return 'true'.

```
// Code contained in a controlled set
Bundle.entry.resource.where($this is DiagnosticReport).code.coding.code in ('cvpd'|'mytp'|'hivp'|'invp')
```

Used on the example file this expression would return 'true'.

We typically strive to work with unambigous FHIR PATH expressions that result in a true or false. These are sufficent
for our purposes of validation or traversing a binary search tree. Of course, it is possible to create significantly
more complex evaluations on FHIR resources.
---

## Context — DEMIS and the need for clear rules

Public health reporting requires timely, reliable data flows from multiple senders (labs, hospitals, physicians) to
various recipients (local health authorities, Robert Koch Institute (RKI), armed forces, and others). Each notification
travels as a FHIR Bundle that must be validated, routed, and sometimes transformed before delivery. In this environment:

- Rules must be explicit and auditable to all stakeholders as well as programmers, testers, and architects.
- Implementations should be resilient to heterogeneous payloads and clear to a heterogeneous group of colleagues.
- Changes should be localized and testable without touching every service.
- Since FHIR forms the basis or data model for the messages, it is the common language of all parties involved, with
  FHIRPath forming only one branch of this common language.

FHIRPath helps by expressing rules directly against resource structures, providing a shared, declarative “rule language”
that engineers, analysts, and QA can review together.
---

## How DEMIS applies FHIRPath (preview of Parts II and III)

Right now we have implemented two core services that use FHIRPath to process notifications: Lifecycle-Validation-Service
and
Notification-Routing-Service. They are part of a larger validation and routing step, which is firstly executed when a
notification arrives in the system. While the schema is validated with the help of a HAPI-FHIR-validator based
Validation-Service (which, funnily enough, also uses FHIRPath) the Lifecycle-Validation-Service checks the broader
context of the report and verifies whether the values set correspond to specific points in the life cycle of a disease
or infection. We also check whether follow-up notifications were actually preceded by primary notifications. The
Notification-Routing-Service uses FHIRPath to determine which technical type a notification has and which paragraph of
the Infektionsschutzgesetz (IfSG), the German Infection Protection Act, is addressed by the notification.

<img src="{{ site.baseurl }}/assets/img/20251128-fhir-path-in-demis/01_demis_flow_fhirpath.png" alt="DEMIS high-level flow with FHIRPath evaluation points"/>

### Lifecycle Validation Service (LVS)

Combines multiple conditions to accept or reject a Bundle. Example pattern:

```
Bundle.entry.resource.where($this is Composition).where(status = 'final').exists()
and Bundle.entry.resource.where($this is Condition).clinicalStatus.coding.where(code = 'active').exists()
and Bundle.entry.resource.where($this is Condition).verificationStatus.coding.where(code = 'confirmed').exists()
```

This check makes sure that only notifications are accepted that have a final status and clinicalStatus active and the
verificationStatus is confirmed.

### Notification Routing Service (NRS)

Decides recipients and downstream steps using FHIRPath filters.

```
// Profile-based filter
Bundle.meta.profile.contains('https://demis.rki.de/fhir/StructureDefinition/NotificationBundleLaboratory')

// Disease-based routing on resolved entries
Bundle.entry.resource.where($this is Composition)
  .section.entry.reference.resolve().code.coding.where(code in {'cvdd','mytp','mytd','mybd'}).exists()
```

---

## Edge cases to plan for (production)

Designing resilient FHIRPath rules means being explicit about ambiguous cases. Below are the key edge domains and how we
address them in DEMIS.

### 1. Arrays and multiplicity

FHIR elements that can repeat (e.g. `Patient.name`, `Observation.component`, `Condition.evidence`) often tempt
developers to treat the first element as “the one”. That breaks as soon as additional entries appear.

Best practice:

```
// Prefer filtering + existence instead of positional access
Patient.name.where(use = 'official').exists()
```

Anti‑pattern:

```
// Fragile: assumes official name is first
Patient.name[0].use = 'official'
```

If you really need a single value, reduce explicitly:

```
Patient.name.where(use = 'official').given.first()
```

### 2. Polymorphic elements

FHIR uses choice types (e.g. `value[x]`, `onset[x]`, `effective[x]`). You must check the actual type before casting.

Safe pattern:

```
Observation.value is Quantity and
Observation.value.as(Quantity).unit = 'mg'
```

Combined predicate pattern:

```
Encounter.period.start.is(DateTime) and Encounter.period.end.is(DateTime)
  and Encounter.period.start.as(DateTime) < Encounter.period.end.as(DateTime)
```

Anti‑pattern:

```
// Unsafe: assumes DateTime, breaks if Period used
Encounter.period.start < today()
```

### 3. Empty vs. null semantics

FHIRPath treats absent elements as empty collections, not null references. Use `empty()` or `exists()` appropriately.

```
// Check that there is no official name
Patient.name.where(use = 'official').empty()

// Ensure at least one phone exists
Patient.telecom.where(system = 'phone').exists()
```

Avoid comparing to literal null; it will not behave as in Java or SQL.

### 4. Reference resolution boundaries

`resolve()` is powerful but can be costly and unsafe if used unguarded. Always:

- Check the reference collection exists.
- Resolve once, reuse the result via `let` bindings.
- Keep filtering local before resolving.

Safe multi‑step pattern:

```
let sectionRefs := Composition.section.entry.reference
sectionRefs.exists() and
let targets := sectionRefs.resolve()
  targets.where($this is Condition).code.coding.where(code = 'cvdd').exists()
```

Anti‑pattern:

```
// Repeated unnecessary resolve calls
Composition.section.entry.reference.resolve().code.coding.where(code = 'cvdd').exists()
Composition.section.entry.reference.resolve().code.coding.where(code = 'mytp').exists()
```

### 5. Profile evolution and drift

Rules tied to profile URLs (`Bundle.meta.profile.contains('...')`) must track version changes. Mitigations:

- Central registry of active profile URLs.
- Automated test that fails when a deprecated profile remains in rules.
- Prefer constants injected at runtime over hard‑coding inside expressions.

Example with injected constant:

```
Bundle.meta.profile.contains(%DEMIS_NOTIFICATION_PROFILE)
```

`%DEMIS_NOTIFICATION_PROFILE` is supplied by the evaluation engine.

### 6. Performance under load

Large Bundles (hundreds of entries) amplify inefficiencies.

- Filter early by resource type: `Bundle.entry.resource.where($this is Observation)`.
- Collapse predicates: combine related conditions inside one `where()`.
- Avoid wide `resolve()` across all entries when you only need a subset.

Optimized pattern:

```
// Narrow to Composition sections first
let comps := Bundle.entry.resource.where($this is Composition)
comps.section.entry.reference.exists() and
let refs := comps.section.entry.reference
let condTargets := refs.resolve().where($this is Condition)
condTargets.code.coding.where(code in {'cvdd','mytp'}).exists()
```

### 7. Ambiguous coding systems

Multiple codings may appear for the same concept (LOINC + local). Always restrict by system.

```
Observation.code.coding.where(system = 'http://loinc.org' and code = '94500-6').exists()
```

Anti‑pattern:

```
Observation.code.coding.where(code = '94500-6').exists() // may match unintended system
```

### 8. Temporal comparisons

FHIR uses `date`, `dateTime`, `instant`. Ensure consistent granularity.

```
Encounter.period.start.is(DateTime) and
Encounter.period.start.as(DateTime) >= today() - 30 days
```

Use `is()` + `as()` to avoid implicit truncation.

### 9. Boolean logic clarity

Complex chains can hide intent. Prefer intermediate `let` bindings.

```
let activeCond := Bundle.entry.resource.where($this is Condition)
  .where(clinicalStatus.coding.where(code = 'active').exists())
activeCond.exists() and activeCond.count() <= 3
```

### 10. Defensive resolve (concise)

```
Composition.section.entry.reference.exists() and
Composition.section.entry.reference.resolve().exists()
```

If external resolution (database/API) is introduced, wrap calls and cache.

---

## Performance considerations

Efficient expressions keep the pipeline fast:

- Filter early with the most selective predicates.
- Avoid unnecessary resolve(); short-circuit when prerequisites are missing.

```
// Early, selective filtering example
Bundle.entry.resource.where(
$this is Condition and verificationStatus.coding.where(code = 'confirmed').exists()
)
```

---

## Implementation note: HAPI FHIR and controlled resolve()

In DEMIS, resolve() was deliberately limited to references within the current bundle. References were to be resolved
deterministically, performantly, and fail-fast for unresolved references. However, requirements have expanded, so that
we now need to resolve IDs beyond the boundaries of a bundle with resolve.

```java
// Conceptual integration sketch with HAPI FHIR
FhirContext ctx = FhirContext.forR4();
FhirPathEngine engine = new FhirPathEngine(new HapiWorkerContext(ctx, ctx.getValidationSupport()));
engine.

setHostServices(new DemisEvaluationContext(bundle)); // custom resolve within bundle

boolean isFinal = engine.evaluateToBoolean(bundle,
        "Bundle.entry.resource.where($this is Composition).where(status = 'final').exists()");
```

---

## Sequence: bundle-only resolve()

Here, we will first present our original implementation for the resolve method and then discuss the use of an external
database in the Part II.

```java
public class CustomEvaluationContext implements IFhirPathEvaluationContext {

    private final Bundle bundle;

    public CustomEvaluationContext(Bundle bundle) {
        this.bundle = bundle;
    }

    @Override
    public IBase resolveReference(@Nonnull IIdType theReference, @Nullable IBase theContext) {
        String referenceValue = theReference.getValue(); // Full reference value
        String referenceIdPart = theReference.getIdPart(); // Just the ID part (e.g., "123456")

        for (Bundle.BundleEntryComponent entry : bundle.getEntry()) {
            // Check if the fullUrl matches
            if (referenceValue.equals(entry.getFullUrl())) {
                return entry.getResource();
            }

            // Check if the resource type and ID match
            if (entry.getResource().getIdElement().getIdPart().equals(referenceIdPart)
                    && entry
                    .getResource()
                    .getResourceType()
                    .toString()
                    .equals(theReference.getResourceType())) {
                return entry.getResource();
            }
        }

        throw new UnsupportedOperationException(
                "Reference resolution not supported for: " + referenceValue);
    }
}
```

---

## What’s next

In Part II, we’ll dive into scenario‑driven validation: robust rule design, edge handling, testing, and performance
patterns. In Part III, we’ll cover smart routing and orchestration with practical integration notes.

---

## About the authors

Verena and Daniel are software developers in the DEMIS NAVY team at gematik, working
on [DEMIS - Deutsches Elektronisches Melde- und Informationssystem für den Infektionsschutz](https://www.rki.de/DE/Themen/Infektionskrankheiten/Meldewesen/DEMIS/demis-node.html).
One aspect they focus on are reliable data pipelines and developer-friendly rule tooling.
