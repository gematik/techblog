---
layout: post
title: "FHIRPath In Production: Why It Matters In DEMIS"
date: 2025-12-22 14:00:00 +0200
author: Daniel Reckel, Verena Sadlack
category: tech
tags: FHIR FHIRPath DEMIS Validation Routing HAPI-FHIR
excerpt: <br />How DEMIS uses FHIRPath to make validation and routing transparent, testable, and efficient.<br /><br />
---

## Executive Summary

> Note: This is Part II of a three-part series. Part I gives a overview of the use of FHIRPath in DEMIS and Part III
> dives III into routing.

---

## Part II –

In Part II, we’ll dive into scenario‑driven validation: robust rule design, edge handling, testing, and performance
patterns.

### DEMIS scenarios

Was ist das, was macht das, wer macht das?

The functional aspects of the DEMIS interface are handled by the project partner RKI. RKI consolidates the domain
expertise on legal and substantive questions, while the Gematik team is responsible for the technical implementation and
operations.

In addition to FHIR profiling for generating the data model, RKI also conducted a comprehensive content review of the
notifications. These notifications should never be seen in isolation, as several notifications can relate to a single
infection. Laboratory notifications confirm the infection and include information about sample testing, such as sample
type, specific test used, or detected antibiotic resistance. Disease notifications, meanwhile, contain details about
the patient’s condition and contextual background, for example the location of infection.

The relationship between notifications is crucial and can be established through IDs linking them together. However,
these links only make sense if the corresponding values are populated within each notification. Quality management of
the notifications is therefore handled through lifecycle management. Encoding these rules directly into the profiled
resources would make them more rigid and harder to maintain, so it was decided to communicate them via the additional
guidance in the Implementation Guide.

### Robust Rule Design

Wie werden die Regeln gebaut? von Szenario-Tabellen zu Szenario-Baum zur Umsetzung. Zusammenarbeit mit dem RKI

### Wie werden die Regeln hinterlegt und wie werden sie ausgeführt.

### Performance

Einfache Mittel zur Performance Steigerung

- mehrere "Paths"
- externe Dienste erst zum Schluss abfragen
- "cleveres" Regeldesign

## What’s Next

In Part III, we’ll cover smart routing and orchestration with practical integration notes.

---

## About The Authors

Daniel is a Senior Software Developer and works on the DEMIS backend since 2021 with great expertise in Java and
FHIRPath.

Tabea is a Junior Software Developer and has only recently joined the team. Using FHIRPath was one of her first tasks,
and she has been an enthusiastic user ever since.
