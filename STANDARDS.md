# Plugin Standards

This document lists down the standards for OpenSearch plugins which can be used by plugin authors.

- [Plugin Standards](#plugin-standards)
  - [Broken Links Checker](#broken-links-checker)
  - [CI Workflows](#ci-workflows)
  - [License Headers](#license-headers)
  - [Public Documentation](#public-documentation)
  - [Release Notes](#release-notes)  
  - [Code Coverage Report](#code-coverage-report)

## Broken Links Checker

Create a Github Actions workflow to check for broken links in text files (.md, .html, .txt, .json) on pull requests and push. See [.github/workflows/links.yml](.github/workflows/links.yml) for an example.

See [lycheeverse/lychee-action](https://github.com/lycheeverse/lychee-action) for more information.

## CI Workflows

CI Workflows should be setup to run and verify plugin unit and integration tests.  
Workflows should run on main and release branches including pull requests merging into them.

_Example_: See CI Workflow in [anomaly-detection](https://github.com/opensearch-project/anomaly-detection/blob/main/.github/workflows/CI.yml). 

## License Headers

License Header details can be found at [HEADERS.md](HEADERS.md).

## Public Documentation

For all new features and functionality changes for every release should update the documentation via [documentation-website](https://github.com/opensearch-project/documentation-website).  
Public documentation can be found at [docs-beta](https://docs-beta.opensearch.org/).

## Release Notes

Release notes for plugins can be found at [RELEASE_NOTES.md](RELEASE_NOTES.md).

## Code Coverage

Code coverage reporting can be found at [TESTING.md](TESTING.md#code-coverage-reporting).
