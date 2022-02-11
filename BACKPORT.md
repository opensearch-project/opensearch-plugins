- [Managing Backports in OpenSearch Plugins](#managing-backports-in-opensearch-plugins)
  - [Features](#features)
  - [Integration](#integration)
  - [Usage](#usage)

## Managing Backports in OpenSearch Plugins

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

1. Label the original PR according to the release-branch. For example, to backport to 1.x , please add a label `backport 1.x`. You can add multiple backport labels to different release branches. Labels can be added in any order: meaning before or after the PR is merged.
2. Once the label is added and the original PR is in a merged state, the auto backport workflow will create backport PR if there are no merge conflicts using `opensearch-trigger-bot`. If there are merge conflicts, it will comment on the original PR on the steps to take.
3. Once the backport PR is merged, the branch created for backport will be auto-deleted.