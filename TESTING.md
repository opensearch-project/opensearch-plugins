<!-- TOC -->
- [Testing](#testing)
    - [Code Coverage Report](#code-coverage-report)
<!-- TOC -->

## Testing

### Code Coverage Report
Code coverage is a measurement of how many lines of code are executed under the test suites.
OpenSearch plugins are using [Codecov](https://about.codecov.io/) for code coverage reporting and analysis. The dashboard can be seen [here](https://app.codecov.io/gh/opensearch-project/).

All the OpenSearch plugins are required to have the following items:

#### Code coverage report upload through CI workflow
1. Generate code coverage report in `xml` format in a CI workflow of Github Actions
2. Upload the report to Codecov by adding a step of [codecov-action](https://github.com/codecov/codecov-action) in the workflow.
3. Set the workflow to be triggered by `push` and `pull request` on `main` and release branches.

See [CI Workflows](STANDARDS.md#ci-workflows) for example.

#### A status badge in README file of the repository to show the code coverage.  ([example](https://github.com/opensearch-project/index-management#readme))
Add the follwing line in the markdown file.
```
[![codecov](https://codecov.io/gh/opensearch-project/REPO-NAME-HERE/branch/main/graph/badge.svg)](https://codecov.io/gh/opensearch-project/REPO-NAME-HERE)
```
Link of the badge can also be accessed from the Repository's Settings page: 
https://app.codecov.io/<code_provider>/<org_name>/<repo_name>/settings/badge