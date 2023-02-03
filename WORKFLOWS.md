- [Workflows](#workflows)
  - [Managing Backports](#managing-backports)
    - [Features](#features)
    - [Integration](#integration)
    - [Usage](#usage)
  - [Labeling PRs](#labeling-prs)
    - [Integration](#integration-1)
  - [Auto Create Documentation Issues](#auto-create-documentation-issues)
    - [Integration](#integration-2)
  - [Auto Github Releases](#auto-github-releases)
    - [Integration](#integration-3)

# Workflows

This document lists down useful workflows that can be added to repositories to alleviate manual processes.

## Managing Backports

Backporting a PR/commit to a release branch is a manual process and can lead to missed backports before a release. The auto backport Github Action allows to create automatic backport PRs just by labelling them.

### Features

When the auto backport workflow is integrated with a repo, the following features are available:

1. Allows auto backports on PRs that are merged and labelled (can be in any order). To backport a PR to 1.x, please add a label called `backport 1.x` to the PR and the backport workflow will do the rest.
2. The backport workflow is now created by a Github App called `opensearch-trigger-bot` instead of `github-actions` which allows CI to run on such PRs.
3. The backport branches are named in the form `backport/backport-<original PR number>-to-<base>`. These branches will be cleaned up by an auto delete workflow once the backport PR is merged.

### Integration

Integrating the backport workflow is easy and can be done with the following steps:

1. Add the [backport](https://github.com/opensearch-project/OpenSearch/blob/main/.github/workflows/backport.yml) workflow in your repo.
2. Add secrets in your repo for the Github App ID and Private key. This is used to generate a token for the Github App `opensearch-trigger-bot`. Using this token helps generate the automatic backport PR from the app token so that all the CI triggers work on the PR. ([OpenSearch#2071](https://github.com/opensearch-project/OpenSearch/pull/2071))
3. Add [auto delete](https://github.com/opensearch-project/OpenSearch/blob/main/.github/workflows/delete_backport_branch.yml) workflow for deleting the backport branches once the backport PRs are merged. This enables the backport workflow to clean up after itself.
4. Backport this auto delete workflow to release branches so the backport branches created against the release branches can be auto deleted.
5. Add appropriate branch permissions for backport related branches in your repo.
6. Add related labels for release branches. For auto-backports, the labels should be of the form `backport <release-branch-name>`. (`backport 1.x`, `backport 1.2` etc.)

### Usage

To use the auto backport workflow:

1. Label the original PR according to the release-branch. For example, to backport to 1.x, please add a label `backport 1.x`. You can add multiple backport labels to different release branches. Labels can be added in any order: meaning before or after the PR is merged. Note that there is an [auto-labeling](#labeling-prs) action that can be configured to help automate this process.
2. Once the label is added and the original PR is in a merged state, the auto backport workflow will create backport PR if there are no merge conflicts using `opensearch-trigger-bot`. If there are merge conflicts, it will comment on the original PR on the steps to take.
3. Once the backport PR is merged, the branch created for backport will be auto-deleted.

An example of auto backport:
- Original PR: https://github.com/opensearch-project/OpenSearch/pull/2094 with label `backport 1.x`.
- Backport PR: https://github.com/opensearch-project/OpenSearch/pull/2106 with merged backport branch auto deleted.


## Labeling PRs

Many of the automated workflows to generate [release notes](./RELEASE_NOTES.md) or [backport PRs](#managing-backports) require labels on the PRs to correctly categorize them and perform the right actions. To eliminate having to do this manually, there is an [auto-labeling GitHub action](https://github.com/actions/labeler), which allows for automatically labeling opened PRs based on the files that the PR changes.

### Integration

1. Integrate the `opensearch-trigger-bot` in your repository, if not done already. This requires adding some GitHub secrets. See the [backport documentation](#managing-backports) for details.
2. Add a labeling config `.github/labeler.yml` ([example](https://github.com/actions/labeler#common-examples)) to your repository. This is where labels can be defined, along with their associated glob patterns, such that if any matching files are changed in a PR, that label will be applied.
3. Add a GitHub workflow `.github/workflows/labeler.yml` ([example](https://github.com/opensearch-project/anomaly-detection-dashboards-plugin/blob/main/.github/workflows/labeler.yml)) to your repository, to utilize the configuration. Note this uses permissions provided from the `opensearch-trigger-bot` instead of the default `github-actions`, due to security limitations regarding forked repository pull requests (details on the limitations [here](https://github.com/actions/first-interaction/issues/10)).


## Auto Create Documentation Issues

When new features are introduced or changes are added that need to be documented, developers have to remember to create issues in the [`documentation-website`](https://github.com/opensearch-project/documentation-website) in order to document the changes. This is a manual process. To eliminate this manual process, there is a [create-documentation-issue](https://github.com/opensearch-project/OpenSearch/blob/main/.github/workflows/create-documentation-issue.yml) workflow which can be added to automatically create issues in the `documentation-website` repo by adding a label `needs-documentation` to the pull request.

### Integration

1. Integrate the `opensearch-trigger-bot` in your repository, if not done already. This requires adding some GitHub secrets. See the [backport documentation](#managing-backports) for details.
2. Add an issue template `.ci/documentation/issue.md` ([example](https://github.com/opensearch-project/OpenSearch/blob/main/.ci/documentation/issue.md)) to your repository. This markdown file defines the template that will be used to create issues in the documentation-website repo.
3. Add a GitHub workflow `.github/workflows/create-documentation-issue.yml` ([example](https://github.com/opensearch-project/OpenSearch/blob/main/.github/workflows/create-documentation-issue.yml)) to your repository, this workflow gets triggered when a label `needs-documentation` is added to a pull request. It creates an issue with the title and issue template provided to the workflow.


## Auto Github Releases

OpenSearch project releases add a [github tag](https://docs.github.com/en/rest/git/tags) for each repo. This workflow enables automatically publishing a new [github release](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases) based on the tag.

### Integration

1. Setup release notes directory with the repo, see OpenSearch [example](https://github.com/opensearch-project/OpenSearch/tree/main/release-notes). This workflow is dependent on predictable release notes based on a release tag.
2. Setup the workflow in your repo similar to [auto-release.yml](https://github.com/opensearch-project/OpenSearch/blob/main/.github/workflows/auto-release.yml) in OpenSearch.
3. Backport this workflow to all release branches i.e 2.x, 1.x etc.