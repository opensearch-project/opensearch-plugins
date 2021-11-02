# Plugin Standards

This document lists down the standards for OpenSearch plugins which can be used by plugin authors.

- [Plugin Standards](#plugin-standards)
  - [Development Best Practices](#development-best-practices)
    - [Separation of Concerns](#separation-of-concerns)
    - [Use Namespacing](#use-namespacing)
    - [Code Styling](#code-styling)
    - [Use Asynchronous Method Invocation](#use-asynchronous-method-invocation)
    - [Validate User Input](#validate-user-input)
    - [API Versioning](#api-versioning)
    - [Accurate Logging](#accurate-logging)
  - [Broken Links Checker](#broken-links-checker)
  - [CI Workflows](#ci-workflows)
  - [Code Coverage Report](#code-coverage-report)
  - [License Headers](#license-headers)
  - [Public Documentation](#public-documentation)
  - [Release Notes](#release-notes)

## Development Best Practices

Follow these best practices while developing any plugin.

### Separation of Concerns

Make sure your plugin follows the SoC design principle and has different functionality organized as separate entities. Each component in the plugin should have a singularity of purpose which would then make the over-all plugin easier to maintain. If each component focusses on a single purpose it is easier to reuse that in other plugins with different contexts. Also ensure the component follows the best practise of encapsulation by having well-defined interfaces.

### Use Namespacing

Make effective use of namespacing since it will allow co-existence of ambiguous names / classes with the same name and help avoid collisions.

### Code Styling

There are two formatting plugins used currently:
1. Spotless - Provides code formatting and style checking
2. CheckStyle - Provides style checking
The advantage of using Spotless over CheckStyle is in addition to check the formatting of the code it also has apply goal that fixes all the style and formatting.

Format Java files with the Eclipse JDT (Java Development Tools) formatter, using the [Spotless Gradle](https://github.com/diffplug/spotless/tree/master/plugin-gradle) plugin. Sample formatting rules can be found [here](https://github.com/opensearch-project/OpenSearch/blob/main/buildSrc/formatterConfig.xml). 

Please follow these formatting guidelines:

* Java indent is 4 spaces
* Line width is 140 characters
* Lines of code surrounded by `// tag::NAME` and `// end::NAME` comments are included in the documentation and should only be 76 characters wide not counting leading indentation. Such regions of code are not formatted automatically as it is not possible to change the line length rule of the formatter for part of a file. Please format such sections sympathetically with the rest of the code, while keeping lines to maximum length of 76 characters.
* Wildcard imports (`import foo.bar.baz.*`) are forbidden and will cause the build to fail.
* If *absolutely* necessary, you can disable formatting for regions of code with the `// tag::NAME` and `// end::NAME` directives, but note that these are intended for use in documentation, so please make it clear what you have done, and only do this where the benefit clearly outweighs the decrease in consistency.
* Note that JavaDoc and block comments i.e. `/* ... */` are not formatted, but line comments i.e `// ...` are.
* There is an implicit rule that negative boolean expressions should use the form `foo == false` instead of `!foo` for better readability of the code. While this isn't strictly enforced, if might get called out in PR reviews as something to change.

### Use Asynchronous Method Invocation

Make sure to use the asynchronous method invocation which avoids the calling thread to not block while waiting for a response. You can make effective use of callbacks for retrieving the results and parallelly process multiple independent tasks.

### Validate User Input

Make sure to validate all the user input before it is processed by the backend. For example, APIs that contain data in its body should have the data validated so that no malicious or invalid data is sent along with the API.

### API Versioning

API Versioning provides the support for the producers to incorporate latest changes in a new version of the API while also making sure that the consumers still have access to the older version of the API.  

### Accurate Logging

Log different events at accurate severity levels and thereby preventing excessive logging. Use a good logging framework which provides the support for multiple severity levels, setting up different appenders which provide support for lossy and log summarizing appenders.

## Broken Links Checker

Create a Github Actions workflow to check for broken links in text files (.md, .html, .txt, .json) on pull requests and push. See [.github/workflows/links.yml](.github/workflows/links.yml) for an example.

See [lycheeverse/lychee-action](https://github.com/lycheeverse/lychee-action) for more information.

## CI Workflows

CI Workflows should be setup to run and verify plugin unit, integration and backwards compatibility tests.
Workflows should run on main and release branches including pull requests merging into them.

_Example_: See CI Workflow in [anomaly-detection](https://github.com/opensearch-project/anomaly-detection/blob/main/.github/workflows/CI.yml). 

## Code Coverage Report

Code coverage reporting can be found at [TESTING.md](TESTING.md#code-coverage-reporting).

## License Headers

License Header details can be found at [HEADERS.md](HEADERS.md).

## Public Documentation

For all new features and functionality, changes for every release should update the documentation via [documentation-website](https://github.com/opensearch-project/documentation-website).  
Public documentation can be found on [OpenSearch website](https://opensearch.org/docs).

## Release Notes

Release notes for plugins can be found at [RELEASE_NOTES.md](RELEASE_NOTES.md).
