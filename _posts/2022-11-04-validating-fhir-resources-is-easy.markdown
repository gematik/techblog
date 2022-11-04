---
layout: post
title:  "Validating FHIR resources is easy, right? Right??"
date:   2022-11-04 11:51:12 +0200
author: Dr. Alexey Tschudnowsky
categories: tech
tags: HL7 FHIR Java validation validator interoperability Simplifier
excerpt: "<br/>It's pretty easy to build a reference validator for a given FHIR specification. And we'll show you how to do it.<br/><br/>"
---
Distributed applications require commonly agreed data structures, formats, and protocols to be interoperable. With the help of so-called validation tools, developers can ensure that their systems respect this common agreement. At runtime, validators can filter out invalid traffic before further processing. The ability to validate a piece of data against its schema is crucial and leads to predictable communication and stable systems.

Many tools exist in the FHIR ecosystem to enable the validation of FHIR resource instances against the underlying specification. Let's take a look at some of them.

## How to validate a FHIR resource

To validate a FHIR resource, you can take, e.g., the [HL7 FHIR Validator](https://confluence.hl7.org/display/FHIR/Using+the+FHIR+Validator), point it to the implementation guide with structure definitions, and pass the resource to validate:

```txt
java -jar validator_cli.jar c:\temp\patient.xml -version 3.0 -ig hl7.fhir.us.core#1.0.1
```

Another option is to use the [Simplifier validation service](https://simplifier.net/validate) (free registration required):

<p align="center">
<img src="{{ site.baseurl }}/assets/img/221104-validating-fhir/simplifier-1.png" alt="Simplifier validation service"/>
</p>

The output contains found issues, categorized into errors, warnings, and notices:

<p align="center">
<img src="{{ site.baseurl }}/assets/img/221104-validating-fhir/simplifier-2.png" alt="Output of Simplifier validation service"/>
</p>

Pretty simple, isn't it?

Not really. Can the identified warnings and notices be really ignored? Where do they come from?

The answer is: FHIR validators have plenty of configuration options. They may or may not validate terminologies and check or ignore externally referenced resources. Projects like [Hammer](https://github.com/health-validator/Hammer) try even to combine outputs of several different tools to return more detailed validation results:

<p align="center">
<img src="{{ site.baseurl }}/assets/img/221104-validating-fhir/hammer.png" alt="Hammer validation tool"/>
</p>

So, what tool to use? Which configuration?

For new FHIR specifications, it's better to answer these questions as early as possible and to define a so-called reference validator - an authoritative tool and configuration to check the conformance of resources to this specification. Having such a tool from the early stages helps users of your specification (i.e., mostly developers) to avoid wrong assumptions, harden their systems, discover misunderstandings and resolve interoperability problems and integration issues. 

Surprisingly, it's pretty easy to build a reference validator for a given FHIR specification. And we'll show how. 

## Creating a reference validator

The toolkit we will use to build a reference validator is called [HAPI FHIR](https://github.com/hapifhir/hapi-fhir), an open-source Java API for HL7 FHIR Clients and Servers. The following examples stem from the E-Prescription validation module of the [gematik Reference Validator](https://github.com/gematik/app-referencevalidator) project.

We first need to download all the packages required to validate a particular profile. The packages can be retrieved, e.g., from [Simplifier](https://simplifier.net/packages/) or from [HL7 FHIR Registry](https://registry.fhir.org/). If a profile references external Code Systems and/or Value Sets, they should be downloaded and bundled as FHIR packages.

For example, to validate the E-Prescription profile `https://fhir.kbv.de/StructureDefinition/KBV_PR_ERP_Bundle` (Version 1.1.0), we need the following FHIR packages:

- kbv.ita.erp-1.1.0.tgz
- kbv.ita.for-1.1.0.tgz (dependency)
- kbv.basis-1.3.0.tgz (dependency)
- de.basisprofil.r4-1.3.2.tgz (dependency)
- gematik.kbv.sfhir.cs.vs-1.0.0.tgz, which is a manually crafted FHIR package containing external code systems and value sets referenced in the profile.

It is a good idea to put the information about which profiles the validator supports and the required packages into a separate configuration file, which can be extended as the validator supports more and more profiles. The packages should be distributed together with the validator to make it offline-capable. 

The next thing we need is to configure the validation algorithm. In the above E-Prescription validation module, the following configuration is used:

- Resources are considered to be invalid whenever they use profiles that are not defined in the above packages.
- Resources are considered to be invalid whenever they use extensions that are not defined in the above packages.
- Some code systems, like `http://fhir.de/CodeSystem/ifa/pzn`, are not validated due to the absence of freely available value sets.

Finally, we might want to post-process outputs of the HAPI FHIR validation library to adjust the severity level of some outputs. Some warnings and information messages can be critical in the context of a particular application. In the case of the E-Prescription validation module, we suppress specific errors for the `DAV-PR-ERP-AbgabedatenBundle|1.0.3` profile due to inconsistent constraints in the FHIR profile. 

The validation flow looks as follows (see [GenericValidator.java at GitHub](https://github.com/gematik/app-referencevalidator/blob/ab3cb6b35a1f76276fb8038e5eea439024c414a4/commons/src/main/java/de/gematik/refv/commons/validation/GenericValidator.java)) :

```java
public ValidationResult validate(
@NonNull
 String resourceBody,
@NonNull
 ValidationModuleConfiguration configuration) throws IllegalArgumentException {

Profile profileInResource = getProfileInResource(resourceBody);
var packageDefinition = getPackageDefinitionForProfile(profileInResource, configuration);
FhirValidator fhirValidator = getOrCreateCachedFhirValidatorFor(profileInResource, configuration, packageDefinition);

var intermediateResult = fhirValidator.validateWithResult(resourceBody);

logger.debug("Pre-Transformation ValidationResult: Valid: {}, Messages: {}", intermediateResult.isSuccessful(), intermediateResult.getMessages());

var filteredMessages = severityLevelTransformator.applyTransformations(intermediateResult.getMessages(), packageDefinition.getValidationMessageTransformations());
var result = new ValidationResult(intermediateResult, filteredMessages);

logger.debug("Final ValidationResult: {}", result);

return result;
}
```


The following code snippet demonstrates the initialization of the HAPI FHIR library (see [GenericValidatorFactory.java at GitHub](https://github.com/gematik/app-referencevalidator/blob/ab3cb6b35a1f76276fb8038e5eea439024c414a4/commons/src/main/java/de/gematik/refv/commons/validation/GenericValidatorFactory.java)):

```java
public FhirValidator createInstance(
        FhirContext ctx,
        Collection<String> packageFilenames,
        Collection<String> codeSystemsToIgnore
        )  {
        var npmPackageSupport = new NpmPackageLoader().loadPackagesAndCreatePrePopulatedValidationSupport(ctx, packageFilenames);

        IValidationSupport validationSupport = ctx.getValidationSupport();

        var validationSupportChain = new ValidationSupportChain(
                npmPackageSupport,
                validationSupport,
                new FixedSnapshotGeneratingValidationSupport(ctx)
                ,new IgnoreMissingValueSetValidationSupport(ctx, codeSystemsToIgnore)
        );

        FhirInstanceValidator hapiValidatorModule = new FhirInstanceValidator(
                validationSupportChain);
        hapiValidatorModule.setErrorForUnknownProfiles(true);
        hapiValidatorModule.setNoExtensibleWarnings(true);
        hapiValidatorModule.setAnyExtensionsAllowed(false);
        FhirValidator fhirValidator = ctx.newValidator();
        fhirValidator.registerValidatorModule(hapiValidatorModule);
        return fhirValidator;
    }
```

The complete source code can be found at [https://github.com/gematik/app-referencevalidator](https://github.com/gematik/app-referencevalidator)

It makes sense to distribute the validator as a Maven dependency and a stand-alone application with a command line interface to support different use cases. This way, the validator can be used locally, in a continuous integration pipeline, or even integrated into other (Java-based) applications.

## Are we done?

Not quite. It's important to extensively test the validator and ensure it makes its judgment correctly. For this purpose, one can create a rich test set containing both valid and invalid resource examples and discuss them with domain experts. The more the validator is used, the more instances will be found, which might require adjustments in the validation algorithm or configuration. Finally, it is crucial to continuously update the validator with new versions of profiles, external value sets, and packages, i.e., users can rely on the tool while adjusting their software to new versions of the specification.  

# About the author

Dr. Alexey Tschudnowsky is a software architect and works on the [gematik Reference Validator](https://github.com/gematik/app-referencevalidator) project. He is an expert in software architectures, agile development processes, distributed systems, and Web technologies.