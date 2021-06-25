<!-- TOC -->
- [Testing](#testing)
    - [Code Coverage Collecting](#code-coverage-collecting)
    - [Code Coverage Reporting](#code-coverage-reporting)
<!-- TOC -->

## Testing

### Code Coverage Collecting
Code coverage is a measurement of how many lines of code are executed under the test suites.



### Code Coverage Reporting
OpenSearch plugins are using [Codecov](https://about.codecov.io/) for code coverage reporting and analysis. The dashboard can be seen [here](https://app.codecov.io/gh/opensearch-project/).

All the OpenSearch plugins are required to have the following items:

#### Code coverage report upload through CI workflow
1. Generate code coverage report in `xml` format in a CI workflow of Github Actions
2. Upload the report to Codecov by adding a step of [codecov-action](https://github.com/codecov/codecov-action) in the workflow.
3. For private repositories, an upload token is needed, visit the repository page of Codecov to get your token: https://codecov.io/gh/opensearch-project/REPO-NAME-HERE. Please use Github Secrets to store the token and not expose to the public.
4. Set the workflow to be triggered by `push` and `pull request` on `main` and release branches.
5. To see your code coverage results from Codecov, visiting https://codecov.io/gh/opensearch-project/REPO-NAME-HERE after the CI workflow is triggered.

See [CI Workflows](STANDARDS.md#ci-workflows) for example.

#### A status badge in README file of the repository to show the code coverage.  ([example](https://github.com/opensearch-project/index-management#readme))
Add the follwing line at the header of the README markdown file:
```
[![codecov](https://codecov.io/gh/opensearch-project/REPO-NAME-HERE/branch/main/graph/badge.svg)](https://codecov.io/gh/opensearch-project/REPO-NAME-HERE)
```
Link of the badge can also be accessed from the repository's settings page of Codecov: 
https://app.codecov.io/gh/opensearch-project/REPO-NAME-HERE/settings/badge

#### Codecov integration in Pull Requests
1. Add a `codecov.yml` file into the repository ([example](https://github.com/opensearch-project/k-NN/commit/f7d1985230ce851cb97a7e41d8bce32127a4f33b))
2. Set the target code coverage properly in case it blocks your PR in Github checks.

See Codecov docs to learn more about [codecov yaml file](https://docs.codecov.com/docs/codecov-yaml) and [common configurations](https://docs.codecov.com/docs/common-recipe-list).