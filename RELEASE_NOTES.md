# Plugin Release Notes

Release notes help OpenSearch users learn about new features, and make educated decisions with regards to upgrading.

- [Release Notes Folder](#release-notes-folder)
- [Naming Convention](#naming-convention)
- [Categories](#categories)
    - [Breaking Changes](#breaking-changes)
    - [Features](#features)
    - [Enhancements](#enhancements)
    - [Bug Fixes](#bug-fixes)
    - [Infrastructure](#infrastructure)
    - [Documentation](#documentation)
    - [Maintenance](#maintenance)
    - [Refactoring](#refactoring)
- [Change Lines](#change-lines)
- [Automation](#automation)

## Release Notes Folder

Release notes are located in the **release-notes** folder of the plugin repo.

_Example:_ [anomaly-detection/tree/main/release-notes](https://github.com/opensearch-project/anomaly-detection/tree/main/release-notes)

## Naming Convention

OpenSearch plugins use the following release notes naming convention: `*opensearch-*<plugin repo name>*.release-notes-*<4 digit plugin version number>*.md`.

_Example:_ [opensearch-**anomaly-detection**.release-notes-**1.0.0.0-beta1**.md](https://github.com/opensearch-project/anomaly-detection/blob/main/release-notes/opensearch-anomaly-detection.release-notes-1.0.0.0-beta1.md)

## Categories

Release notes have the following categories, highlighted by a **Header 3** (`###`).

_Example:_ `### Bug Fixes`

### Breaking Changes

1. A change in one part of a software system that causes other parts to fail.
2. A change that will require users to make a corresponding change.

_Example:_ `* Invalidate HTTP GET method ([#414](https://github.com/opensearch-project/sql/pull/414))`

### Features

A change that introduce a *net new* unit of functionality of a software system that satisfies a requirement, represents a design decision, and provides a potential configuration option. As for improvement on existing features, use the *Enhancement* category. As for fixes on existing features, use the *Bug Fixes* category.

_Example:_ `* Add start/stop batch actions on detector list page ([#195](https://github.com/opensearch-project/anomaly-detection/pull/195))`

### Enhancements

A product change or upgrade that increases software capabilities beyond original client specifications, or an improvement on the existing feature’s functionality.

_Example:_ `* Improve batch action modal loading state ([#216](https://github.com/opensearch-project/anomaly-detection/pull/216))`

### Bug Fixes

A change to a system or product designed to handle a programming bug/glitch.

_Example:_ `* Fix Jacoco coverage issue introduced in odfe 1.8.0 ([#134](https://github.com/opensearch-project/k-NN/pull/134))`

### Infrastructure

A change to infrastructure, testing, CI/CD, pipelines, etc.

_Example:_ `* Add CI for e2e ([#208](https://github.com/opensearch-project/anomaly-detection/pull/208))`

### Documentation

A change to update existing documentations such as README, docs, etc.

_Example:_  `* Update build instructions ([#93](https://github.com/opensearch-project/sql-odbc/pull/93))`

### Maintenance

A change to add support for new versions of OpenSearch or OpenSearch Dashboards from upstream.

_Example:_ `* Add support for OpenSearch 1.0 ([#219](https://github.com/opensearch-project/alerting/pull/219))`

### Refactoring

A change intended to improve the design, structure, and/or implementation of the software, while preserving its functionality.

_Example:_ `* Make ClusterDetailsEventProcessor and all its access methods non-static ([#283](https://github.com/opensearch-project/performance-analyzer-rca/pull/283))`

## Change Lines

Under each category, each line represents a change with its corresponding pull-request number formatted as a hyperlink. 

**Do not add additional Prefix before the change description or PR nsumber.**

_Incorrect:_ `* BUGFIX: (Issue [#123](http:/#123)): Fix the out of memory issue (PR [#456](http:/#456))`

_Correct:_ `* Fix the out of memory issue ([#456](https:/#456))`

**Always use * (asterisk) for the bulleted list, do not use alternatives such as - (hyphen/dash).**

_Incorrect:_ `- Fix the out of memory issue ([#456](http:/#456))`

_Correct:_ `* Fix the out of memory issue ([#456](http:/#456))`

**Always surround the PR number/hyperlink with a pair of parentheses to highlight it.**

_Incorrect:_ `* Fix the out of memory issue [#456](http:/#456)`

_Correct:_ `* Fix the out of memory issue ([#456](http:/#456))`       

## Automation

Create a GitHub Actions workflow to help draft release notes. See [.github/workflows/draft-release-notes.yml](.github/workflows/draft-release-notes.yml) for an example. This workflow is configured using [.github/draft-release-notes-config.yml](.github/draft-release-notes-config.yml).

See [release-drafter](https://github.com/release-drafter/release-drafter) for more information.