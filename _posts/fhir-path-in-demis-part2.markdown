## Executive Summary

FHIRPath is a compact, declarative language to navigate and filter FHIR resources optimized for hierarchical data
structures used in the public health sector. In DEMIS it makes complex rules transparent, testable, and reusable across
services. Instead of writing long complex code platform-independent path expressions can be used to read out content
and for decision-making or analysis. This post explains the why and how, with practical examples you can adapt.

> Note: This is Part II of a three-part series. The first part dives into the fundamentals of FHIRPath, and Part III into routing.

---

## Part 2 – Scenario-driven validation of FHIR notifications

### Motivation and Value
FHIR notifications in DEMIS have a defined lifecycle depending on the type of notification.
The lifecycle management (LCM) defines various valid scenarios for notifications. Each scenario has rules based on the combination of values of specific elements and of a notification.
Part of the validation of incoming FHIR notifications in DEMIS is to check if it is valid regarding the lifecycle.
Due to the complexity of the rules, it is hard to implement them in a maintainable way using traditional programming languages.
Using FHIRPath expressions to define the rules has several advantages: 
- **Compactness**: FHIRPath expressions are concise and can express complex logic in a few lines.
- **Readability**: The declarative nature of FHIRPath makes it easier to understand the rules at a glance.
- **Maintainability**: Changes to the rules can be made by updating the FHIRPath expressions without modifying the underlying codebase.

### Scenarios in LCM

A Scenario should be understood as a specific state or condition that a FHIR notification can be in during its lifecycle.
A notification contains information about the detection of a pathogen or the suspicion of a disease in a patient.
A notification can be initial or follow-up. A follow-up notification amends or corrects information contained in a previously sent notification.
A scenario represents a real, valid state of a notification.