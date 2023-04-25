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
  - [Publishing Java Plugins Snapshots](#publishing-java-plugins-snapshots)
    - [Configuration](#configuration)
    - [Testing](#testing)

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

## Publishing Java Plugins Snapshots

OpenSearch project uses GitHub Actions to publish snapshots to [Maven](https://aws.oss.sonatype.org/content/repositories/snapshots/). 

### Configuration
In order to onboard, follow the below steps:
1. Determine what artifacts you publish to maven. Most plugins publish only zips. [Example](https://aws.oss.sonatype.org/content/repositories/snapshots/org/opensearch/plugin/opensearch-security/3.0.0.0-SNAPSHOT/).
2. Add the below configuration under `publishing` section of build.gradle:
```
    repositories {
        maven {
            name = "Snapshots"
            url = "https://aws.oss.sonatype.org/content/repositories/snapshots"
            credentials {
                username "$System.env.SONATYPE_USERNAME"
                password "$System.env.SONATYPE_PASSWORD"
            }
        }
    }
```
3. Determine what gradle task will publish the content to snapshots repo. Example: ./gradlew publishPluginZipPublicationToSnapshotsRepository.
4. Create a pull request to set up a GitHub Action workflow that will fetch the credentials and publish the snapshots to https://aws.oss.sonatype.org/ See [sample workflow](https://github.com/opensearch-project/security/blob/main/.github/workflows/maven-publish.yml).
5. Add @opensearch-project/engineering-effectiveness as a reviewer, who will also take care of creating the IAM role that is required to fetch the credentials and add it to the GitHub Secrets.

### Testing 
In order to test the maven snapshot publication, update the Snapshots target repository URL from maven to a local file system path in your build.gradle file.

```
  maven {
      name = "Snapshots" //  optional target repository name
      url = "snapshots"
  }
  
```
Run the command `./gradlew <taskName>` which will publish the files to the local file system under the 'snapshots' folder, then scan with `tree` or your favor directory scanner to see the content along with sha and md5 files.

Example:
```
tree snapshots
snapshots
└── org
    └── opensearch
        └── plugin
            └── opensearch-security
                ├── 2.7.0.0-SNAPSHOT
                │   ├── maven-metadata.xml
                │   ├── maven-metadata.xml.md5
                │   ├── maven-metadata.xml.sha1
                │   ├── maven-metadata.xml.sha256
                │   ├── maven-metadata.xml.sha512
                │   ├── opensearch-security-2.7.0.0-20230424.235359-1.pom
                │   ├── opensearch-security-2.7.0.0-20230424.235359-1.pom.md5
                │   ├── opensearch-security-2.7.0.0-20230424.235359-1.pom.sha1
                │   ├── opensearch-security-2.7.0.0-20230424.235359-1.pom.sha256
                │   ├── opensearch-security-2.7.0.0-20230424.235359-1.pom.sha512
                │   ├── opensearch-security-2.7.0.0-20230424.235359-1.zip
                │   ├── opensearch-security-2.7.0.0-20230424.235359-1.zip.md5
                │   ├── opensearch-security-2.7.0.0-20230424.235359-1.zip.sha1
                │   ├── opensearch-security-2.7.0.0-20230424.235359-1.zip.sha256
                │   └── opensearch-security-2.7.0.0-20230424.235359-1.zip.sha512
                ├── maven-metadata.xml
                ├── maven-metadata.xml.md5
                ├── maven-metadata.xml.sha1
                ├── maven-metadata.xml.sha256
                └── maven-metadata.xml.sha512
```