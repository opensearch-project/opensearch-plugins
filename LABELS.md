- [Labeling PRs](#labeling-prs)
  - [Integration](#integration)

# Labeling PRs

Many of the automated workflows to generate [release notes](./RELEASE_NOTES.md) or [backport PRs](./BACKPORT.md) require labels on the PRs to correctly categorize them and perform the right actions. To eliminate having to do this manually, there is an [auto-labeling GitHub action](https://github.com/actions/labeler), which allows for automatically labeling opened PRs based on the files that the PR changes.

## Integration

1. Integrate the `github-trigger-bot` in your repository, if not done already. This requires adding some GitHub secrets. See the [backport documentation](./BACKPORT.md) for details.
2. Add a labeling config `.github/labeler.yml` ([example](https://github.com/actions/labeler#common-examples)) to your repository. This is where labels can be defined, along with their associated glob patterns, such that if any matching files are changed in a PR, that label will be applied.
3. Add a GitHub workflow `.github/workflows/labeler.yml` ([example](https://github.com/opensearch-project/anomaly-detection-dashboards-plugin/blob/main/.github/workflows/labeler.yml)) to your repository, to utilize the configuration. Note this uses permissions provided from the `github-trigger-bot` instead of the default `github-actions`, due to security limitations regarding forked repository pull requests (details on the limitations [here](https://github.com/actions/first-interaction/issues/10)).