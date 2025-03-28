---
layout: post
title:  "Audit Preparation in E-Rezept App (iOS)"
date:   2025-05-06 14:00:00 +0200
author: Martin Fiebig
categories: tech
tags: Audits iOS
excerpt: <br />Our technical approach for preparing for audits explained.<br /><br />
---

<div class="note">
<p>
  This is the continuation of a previous article <a href="/tech/2025/01/21/audit-preparation">Audit Preparation in E-Rezept App (Android)</a>. If you are interested in how we prepare audits for our Android E-Rezept App, feel free to read that article first.
</p>
</div>


# Preparing for Audits in the E-Rezept App iOS

The iOS Application is subject to audits much like the Android application. Our application is mainly written using Swift. At the point of writing this article, Swift does not support annotations as they can be used with Gradle plugins, so there is no formal way of annotating code as we would need.

In this article we will present a distinct approach to annotate the code differently from Android and walk through the steps of combining requirements with code to form an output document to help with auditing the code.

---

## Requirements

In order to deliver a secure product we have to implement various formal requirements. These requirements are published regularly for all gematik products. You can find them on our website *gemSpecPages* ([https://gemspec.gematik.de](https://gemspec.gematik.de)). An example (and our *main*) specification for the E-Rezept App is called `gemSpec_eRp_FdV`. You can find the latest version of the actual specification under [https://gemspec.gematik.de/docs/gemSpec/gemSpec_eRp_FdV/latest/](https://gemspec.gematik.de/docs/gemSpec/gemSpec_eRp_FdV/latest/).

While preparing the document for our auditors, we will use gemspec pages as a source for the specification text, as well as another specification that is provided by the BSI. The BSI specification is published as a PDF document, to use it within the scripts, we transformed parts of that PDF into an HTML document with the same structure as gemspecPages.

## Annotations

Auditors expect an explanation of how and where a specific requirement is implemented. For that purpose, we use special annotations within our code for each requirement as well as a text document for more general comments that do not fit to any parts of the code. The special annotations are simple comments within the code, that flag a line or a region of code to be important for a specific requirement.

The format of the annotation looks like this:

```
 [REQ:<SPEC>:<AFO>#<ORDER>|<LOC_TO_INCLUDE>] <NOTE> 
```

Annotations are composed with these parts:

  - `[REQ:`: marks the start of a requirement annotation.
  - `<SPEC>`: Technical name of the specification (document name), that originates the requirement.
  - `:<AFO>`:  Static `:` followed by the identifier of the implemented requirement.
  - `#<ORDER>` (optional): Static `#` followed by a number. All references of a single requirement are ordered by this number in the document. This allows a deterministic order of all code parts and makes the explanation easier to understand.
  - `|<LOC_TO_INCLUDE>` (optional): The number of lines that the annotation is meant for. Defaults to 1.
  - `] `: Marks the end of the technical part of the annotation.
  - `<NOTE>` (optional): Textual explanation of the implementation or why this part is important for the requirement.

### Example

Looking at [DefaultIDPSession.swift#L196](https://github.com/gematik/E-Rezept-App-iOS/blob/master/Sources/IDP/DefaultIDPSession.swift#L196) we can see the annotation:

```
[REQ:gemSpec_IDP_Frontend:A_21323,A_21324#1|6] Crypto box contains `Token-Key`
```

containing the following information:

  - `gemSpec_IDP_Frontend`: The specification document where the requirements can be found
  - `A_21323,A_21324#1`: The part of the code spans two requirements, one being `A_21323` the other being `A_21324`. The annotation is ranked #1 for the latter.
  - `|6`: The annotation is meant for the following 6 lines of code.

## Document generation

The actual generation of the document is written in Ruby, called by the `list_requirements` lane of our [Fastfile](https://github.com/gematik/E-Rezept-App-iOS/blob/master/fastlane/Fastfile). We extracted the actual code into a fastlane plugin to clean up the fastfile. You can find the plugin on Github [gematik/Fastlane-Plugin-CI-CD](https://github.com/gematik/Fastlane-Plugin-CI-CD). The action that generates the audit document is named `audit_generator`.

### Processing Annotations

Let's have a look into that plugin, to see how this works [audit_generator.rb#L15](https://github.com/gematik/Fastlane-Plugin-CI-CD/blob/main/lib/fastlane/plugin/ci_cd/actions/audit_generator.rb#L15):

```ruby
def self.run(params)
  UI.message("The audit_generator plugin is working!")

  # Load data and initialize processors
  data_loader = Fastlane::Helper::AuditDataLoader.new(params[:audit_afos_json])
  file_processor = Fastlane::Helper::FileProcessor.new
  spec_fetcher = Fastlane::Helper::SpecFetcher.new(data_loader.audit_afos_json)

  # Process requirement notes if provided
  if params[:requirement_notes_glob]
    UI.message("Processing requirement notes from: #{params[:requirement_notes_glob]}")
    Dir.glob(params[:requirement_notes_glob]).each do |file|
      file_processor.process_requirement_notes(file)
    end
  end

  # Process code files
  source_files = []
  params[:source_file_globs].each do |glob|
    UI.message("Adding source files from glob: #{glob}")
    source_files.concat(Dir.glob(glob))
  end

  source_files.each do |file|
    file_processor.process_code_file(file)
  end

  # Get specs and prepare data for templates
  original_texts = spec_fetcher.fetch_specs
  audit_data = prepare_audit_data(data_loader.audit_afos, file_processor.specs, original_texts)

  # Generate output files
  generate_output_files(params, audit_data)
end
```

After some basic initialization the process consists of 4 steps:

  1. Parsing the requirement notes document (if present) containing general information on some of the implemented requirements
  2. Scanning all given source files for requirement annotations
  3. Processing
      1. Download the original specifications
      2. Processing all found annotation information, grouping by spec, sorting
  4. Rendering the information into all given templates

#### Parsing requirement notes & scanning all given source files

The actual parsing of the requirement notes document as well as the given source files is similar. We identify blocks that contain annotations. While in the requirement notes each block will consist of one line in the document, the source files are scanned for comment sections. Subsequent commented lines are treated as one block.

Each block is then parsed using a regular expression:

```ruby
block.each do |line|
  # Format: [REQ:<SPEC>:<AFOS>:<SUBAFOS?>] <DESC>
  if (matches = line.match(/\[REQ:(?<SPEC>[^:]*):(?<AFOS>[^:^|\s]*)(?:(?::)(?<SUBAFOS>[^:^|\s]*))?(?:(?:\|)(?<NUMBEROFLINES>[^:\s]*))?\](?<DESC>.*)/))
    register(matches, file, number, trimmed_code, source, trimming)
  end
  number += 1
end
```

The `register(...)` method then gathers all the information in a structured way.

#### Processing

The processing starts by downloading all original requirements from the specifications listed in the `audit_afos.json` file. This allows us to include the official requirement text alongside our implementation details.

##### Original Requirement Texts

The `audit_afos.json` file serves as a central registry of all requirements (AFOs) that will be checked by the auditors, as well as a list of specifications that are used. Here's an example of what this file might look like:

```json
{
  "audit_afos": [
    "A_20742", "A_20743", "A_20744", "A_21323", "A_21324"
  ],
  "specs": {
    "gemSpec_eRp_FdV": "https://gemspec.gematik.de/docs/gemSpec/gemSpec_eRp_FdV/latest/",
    "gemSpec_IDP_Frontend": "https://gemspec.gematik.de/docs/gemSpec/gemSpec_IDP_Frontend/latest/",
    "BSI_eRp": "file:///path/to/local/bsi_requirements.html"
  }
}
```

For each specification, we download and parse the HTML content using the `Nokogiri` gem. Since each requirement within the HTML document is identified by a unique ID, we can extract the original requirement text and include it in our audit document:

```ruby
def fetch_requirement_text(spec_doc, afo)
  # Find the element with the AFO ID
  element = spec_doc.css("##{afo}")
  return "Original text not found" if element.empty?
  
  # Extract the requirement text
  element.inner_html
end
```

For BSI requirements, which are typically provided as PDF documents, we convert the relevant sections to HTML with the same structure as gemspecPages to maintain consistency in our processing pipeline.

##### Filtering and Organization

Before generating the final output, we apply several filters and organizational steps to ensure the audit document is comprehensive and easy to navigate:

  1. **Completeness Check:** We identify any requirements from the `audit_afos.json` file that don't have corresponding implementations, marking them as "missing" in the output.
  2. **Grouping by Specification:** All implementations are grouped by their source specification, making it easier for auditors to verify compliance with specific documents.
  3. **Sorting:** Within each specification, implementations are sorted by AFO ID and then by the optional order parameter, ensuring a logical and consistent presentation.
  4. **Deduplication:** If the same code section is referenced multiple times for the same requirement, we consolidate these references to avoid redundancy.

#### Output Generation

The final step is generating the output files based on the processed data and the provided ERB templates:

```ruby
def generate_output_files(params, audit_data)
  FileUtils.mkdir_p(params[:output_directory])
  
  params[:erb_templates].each do |template_path|
    template_name = File.basename(template_path, '.erb')
    output_path = File.join(params[:output_directory], template_name)
    
    UI.message("Generating output file: #{output_path}")
    
    # Load and render the ERB template
    template = ERB.new(File.read(template_path))
    result = template.result_with_hash(
      specs: audit_data[:specs],
      missing_afos: audit_data[:missing_afos],
      original_texts: audit_data[:original_texts]
    )
    
    # Write the output file
    File.write(output_path, result)
  end
end
```

By using ERB templates we support multiple output formats. This flexibility allows us to generate:

  1. **Markdown** documents for easy reading and version control
  2. **HTML** reports with interactive features for easier navigation
  3. **PDF** documents (via HTML or Markdown + `pandoc` CLI tool) for formal submission to auditors

### Conclusion

By using this systematic approach to documenting our implementation of security requirements, we've significantly streamlined the audit preparation process for the iOS E-Rezept App. The combination of code annotations, automated document generation, and integration with our CI/CD pipeline ensures that:

1. Developers clearly document how they've implemented each requirement
2. Auditors can easily trace requirements to their implementations
3. Missing implementations are quickly identified and addressed
4. The documentation stays synchronized with the codebase

This approach has not only made the audit process more efficient but has also improved the overall quality of our codebase by encouraging developers to think explicitly about security requirements as they write code.

The fastlane plugin we've developed for this purpose is open-source and available on GitHub, so other teams facing similar audit requirements can benefit from our approach.

---

# About the author

Martin Fiebig has worked as a software engineer for 21 years, 13 years on iOS and Apple platforms. His focus topics include large scale mobile applications and developer experience. Martin leads the iOS team at gematik since 2020 and the Mobile Chapter since 2023.