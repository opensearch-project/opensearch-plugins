- [Auto Create Documentation Issues](#auto-create-documentation-issues)
  - [Integration](#integration)

# Auto Create Documentation Issues

When new features are introduced or changes are added that need to be documented, developers have to remember to create issues in the [`documentation-website`](https://github.com/opensearch-project/documentation-website) in order to document the changes. This is a manual process. To eliminate this manual process, there is a [create-documentation-issue](https://github.com/opensearch-project/OpenSearch/blob/main/.github/workflows/create-documentation-issue.yml) workflow which can be added to automatically create issues in the `documentation-website` repo by adding a label `needs-documentation` to the pull request.

## Integration

1. Integrate the `opensearch-trigger-bot` in your repository, if not done already. This requires adding some GitHub secrets. See the [backport documentation](./BACKPORT.md) for details.
2. Add an issue template `.ci/documentation/issue.md` ([example](https://github.com/opensearch-project/OpenSearch/blob/main/.ci/documentation/issue.md)) to your repository. This markdown file defines the template that will be used to create issues in the documentation-website repo.
3. Add a GitHub workflow `.github/workflows/create-documentation-issue.yml` ([example](https://github.com/opensearch-project/OpenSearch/blob/main/.github/workflows/create-documentation-issue.yml)) to your repository, this workflow gets triggered when a label `needs-documentation` is added to a pull request. It creates an issue with the title and issue template provided to the workflow.