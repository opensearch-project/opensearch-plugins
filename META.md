- [Managing OpenSearch Plugins](#managing-opensearch-plugins)
  - [Install GH](#install-gh)
  - [Install Meta](#install-meta)
  - [Check Out All Plugins](#check-out-all-plugins)
  - [Get Repo Info](#get-repo-info)
  - [Add a New Plugin](#add-a-new-plugin)
  - [Create or Update Labels in All Plugin Repos](#create-or-update-labels-in-all-plugin-repos)
  - [Create an Issue in All Plugin Repos](#create-an-issue-in-all-plugin-repos)
  - [Open a Pull Request in Each Repo](#open-a-pull-request-in-each-repo)
  - [Increment a Version in Every Plugin](#increment-a-version-in-every-plugin)
    - [Increment Version in OpenSearch](#increment-version-in-opensearch)
    - [Create a 1.2.3 Manifest](#create-a-123-manifest)
    - [Prepare for Increments](#prepare-for-increments)
    - [Increment Version in Plugins](#increment-version-in-plugins)
    - [Commit and Push Changes](#commit-and-push-changes)
    - [Create Pull Requests](#create-pull-requests)
      - [common-utils](#common-utils)
      - [job-scheduler](#job-scheduler)
      - [min-SNAPSHOT](#min-snapshot)
      - [alerting](#alerting)
      - [Remaining Plugins](#remaining-plugins)
      - [Update job-scheduler Snapshots](#update-job-scheduler-snapshots)
    - [Update the Manifest](#update-the-manifest)
  - [Backporting in OpenSearch Plugins](#backporting-in-opensearch-plugins)

## Managing OpenSearch Plugins

We use [meta](https://github.com/mateodelnorte/meta) to manage OpenSearch and OpenSearch Dashoards plugins as a set. There are three sets: [all plugins](.meta), [OpenSearch Plugins](plugins/.meta) and [OpenSearch Dashboards Plugins](dashboards-plugins/.meta). If you need a meta project for all components included in OpenSearch, see [project-meta](https://github.com/opensearch-project/project-meta).

### Install GH

Install and configure GitHub CLI from [cli.github.com/manual/installation](https://cli.github.com/manual/installation). Authenticate with `gh auth login` and ensure that it works, e.g. `gh issue list`.

### Install Meta

```sh
npm install
```

See [package.json](package.json) for all dependencies being installed.

### Check Out All Plugins

```sh
meta git update
```

Use `meta git pull` to subsequently pull the latest revisions.

### Get Repo Info

```sh
meta gh issue list
```

### Add a New Plugin

```sh
meta project import plugin git@github.com:opensearch-project/plugin.git
```

### Create or Update Labels in All Plugin Repos

Install [ghi](https://github.com/stephencelis/ghi), e.g. `brew install ghi`.

```
meta exec "ghi label 'backwards-compatibility' -c '#773AA8'
```

This makes it easy to create version labels.

```
meta exec "ghi label 'untriaged' -c '#fbca04'"
meta exec "ghi label 'v1.0.0' -c '#d4c5f9'"
meta exec "ghi label 'v1.1.0' -c '#c5def5'"
meta exec "ghi label 'v1.2.0' -c '#bfdadc'"
meta exec "ghi label 'v2.0.0' -c '#b94c47'"
```

### Create an Issue in All Plugin Repos

Create a file for the issue body, e.g. `issue.md`.

```
meta exec "gh issue create --label backwards-compatibility --title 'Ensure backwards compatibility with ODFE' --body-file ../issue.md"
```

### Open a Pull Request in Each Repo

In [opensearch-build#497](https://github.com/opensearch-project/opensearch-build/issues/497) we needed to remove `integtest.sh` from each repo.

We'll be pushing to forks. Make sure to replace `username` with your GitHub username below and to fork all the repos first.

```
meta exec "gh repo fork"
```

Ensure that a remote is setup for each plugin pointing to our forks.

```
meta exec "git remote get-url origin | sed s/opensearch-project/username/g | xargs git remote add username"
```

Remove a file, e.g. `integtest.sh`, commit and push.

```
meta exec "git rm integtest.sh"
meta exec "git add --all"
meta exec "git checkout -b remove-integtest-sh"
meta exec "git commit -s -m 'Removed integtest.sh.'"
meta exec "git push username remove-integtest-sh"
meta exec "gh pr create --title 'Removing default integtest.sh.' --body='Coming from https://github.com/opensearch-project/opensearch-build/issues/497, removing default integtest.sh.'"
```

### Increment a Version in Every Plugin

Because one cannot install an older version of a plugin on a newer version of OpenSearch (see [OpenSearch#1707](https://github.com/opensearch-project/OpenSearch/issues/1707)), it's common to have to increment versions in all plugins, without making other changes. This was the case in [the 1.2.3 release](https://github.com/opensearch-project/opensearch-build/issues/1365).

See also [opensearch-build#1375](https://github.com/opensearch-project/opensearch-build/issues/1375) which aims to supersede this labor-intensive process with plugins auto-incrementing versions for the next development iteration.

#### Increment Version in OpenSearch

The auto-increment workflow in [OpenSearch#1816](https://github.com/opensearch-project/OpenSearch/pull/1816) will increment the version in OpenSearch patch branch, e.g. 1.2, [OpenSearch#1758](https://github.com/opensearch-project/OpenSearch/pull/1758), and backport the version increment change in OpenSearch to `1.x` ([OpenSearch#1759](https://github.com/opensearch-project/OpenSearch/pull/1759)) and `main` ([OpenSearch#1760](https://github.com/opensearch-project/OpenSearch/pull/1760)).

#### Create a 1.2.3 Manifest

A workflow will create a new manifest that only contains OpenSearch, e.g. [opensearch-build#1369](https://github.com/opensearch-project/opensearch-build/pull/1369). After this manifest is merged, wait for a successful SNAPSHOT build.

#### Prepare for Increments

In the examples below we are incrementing 1.2.2 to 1.2.3.

```
export INCREMENT_BASE=1.2
export INCREMENT_FROM=1.2.2
export INCREMENT_TO=1.2.3
```

#### Increment Version in Plugins

Check out and update the 1.2 branch.

```
meta git update
meta git pull origin
meta git checkout $INCREMENT_BASE
meta git pull origin $INCREMENT_BASE
```

Replace known versions.

```
find . -name build.gradle -print0 | xargs -0 sed -i "s/$INCREMENT_FROM-SNAPSHOT/$INCREMENT_TO-SNAPSHOT/g"
find . -wholename "*/.github/workflows/*.yml" -print0 | xargs -0 sed -i "s/$INCREMENT_FROM-SNAPSHOT/$INCREMENT_TO-SNAPSHOT/g"
find . -wholename "*/.github/workflows/*.yml" -print0 | xargs -0 sed -i "s/$INCREMENT_FROM.0-SNAPSHOT/$INCREMENT_TO.0-SNAPSHOT/g"
```

Plugins such as k-nn and security need some exact version replacements.

```
find . -name build.gradle -print0 | xargs -0 sed -i "s/$INCREMENT_FROM/$INCREMENT_TO/g"
find . -name CMakeLists.txt -print0 | xargs -0 sed -i "s/$INCREMENT_FROM.0/$INCREMENT_TO.0/g"
find . -name pom.xml -print0 | xargs -0 sed -i "s/$INCREMENT_FROM/$INCREMENT_TO/g"
find . -name plugin-descriptor.properties -print0 | xargs -0 sed -i "s/$INCREMENT_FROM/$INCREMENT_TO/g"
```

The cross-cluster-replication plugin needs an update in `SecurityAdminWrapper.sh`.

```
find . -name SecurityAdminWrapper.sh -print0 | xargs -0 sed -i "s/$INCREMENT_FROM/$INCREMENT_TO/g"
```

#### Commit and Push Changes

```
meta git checkout -b increment-to-$INCREMENT_TO
meta git add .
meta git commit -s -m "Incremented version to $INCREMENT_TO."
```

Create a remote for your own forks of the plugins. Replace `<your-github-username>`.

```
meta exec "gh repo fork --remote --remote-name <your-github-username>"
```

Ensure that a remote exists for your fork for every repo.

```
meta exec "git remote get-url origin | sed s/opensearch-project/<your-github-username>/g | xargs git remote add <your-github-username>"
```

Push the changes to your fork.

```
meta exec "git push <your-github-username> increment-to-$INCREMENT_TO"
```

#### Create Pull Requests

##### common-utils

An auto-increment workflow (see [common-utils#106](https://github.com/opensearch-project/common-utils/pull/106)) will auto-increment the version, e.g. [common-utils#105](https://github.com/opensearch-project/common-utils/pull/105).

##### job-scheduler

Make a pull request incrementing the version into `job-scheduler` that depends on `OpenSearch`, e.g. [job-scheduler#110](https://github.com/opensearch-project/job-scheduler/pull/110).

```
cd job-scheduler
gh pr create --title "Incremented version to $INCREMENT_TO." --body "Coming from https://github.com/opensearch-project/opensearch-build/issues/1365." --base $INCREMENT_BASE --label v$INCREMENT_TO'
```

Add `common-utils` and `job-scheduler` to the 1.2.3 manifest, e.g. [opensearch-build#1374](https://github.com/opensearch-project/opensearch-build/pull/1374) and [opensearch-build#1376](https://github.com/opensearch-project/opensearch-build/pull/1376) (you can combine both as `job-scheduler` doesn't depend on `common-utils`), and wait for a successful SNAPSHOT build.

##### min-SNAPSHOT

Check that a snapshot build has been published, e.g. [opensearch-min-1.2.3-SNAPSHOT-linux-x64-latest.tar.gz](https://artifacts.opensearch.org/snapshots/core/opensearch/1.2.3/opensearch-min-1.2.3-linux-x64-latest.tar.gz), which is required by most plugins' backwards compatibility tests. See [opensearch-build#1261](https://github.com/opensearch-project/opensearch-build/issues/1261) for automating this.

##### alerting

Make a pull request incrementing the version into `alerting`, which contains `notifications` that `index-management` depends on, e.g. [alerting#261](https://github.com/opensearch-project/alerting/pull/261). Add that plugin to the manifest and wait for a successful SNAPSHOT build, e.g. [opensearch-build#1379](https://github.com/opensearch-project/opensearch-build/pull/1379).

##### Remaining Plugins

Create pull requests referencing the [parent release issue in opensearch-build](https://github.com/opensearch-project/opensearch-build/issues/1365). You will be prompted for where to push code, choose your fork.

```
meta exec 'gh pr create --title "Incremented version to $INCREMENT_TO." --body "Coming from https://github.com/opensearch-project/opensearch-build/issues/1365." --base $INCREMENT_BASE --label v$INCREMENT_TO'
```

##### Update job-scheduler Snapshots

In addition to the above changes, build and replace the job-scheduler SNAPSHOT jar in `anomaly-detection`, `dashboards-reports`, and `index-management`.

```
cd job-scheduler
git checkout $INCREMENT_BASE
git pull
./gradlew assemble

rm ../anomaly-detection/src/test/resources/job-scheduler/*
cp ./build/distributions/opensearch-job-scheduler-$INCREMENT_TO.0-SNAPSHOT.zip ../anomaly-detection/src/test/resources/job-scheduler/
rm -rf ../anomaly-detection/src/test/resources/org/opensearch/ad/bwc/job-scheduler/$INCREMENT_FROM.0-SNAPSHOT
mkdir -p ../anomaly-detection/src/test/resources/org/opensearch/ad/bwc/job-scheduler/$INCREMENT_TO.0-SNAPSHOT
cp ./build/distributions/opensearch-job-scheduler-$INCREMENT_TO.0-SNAPSHOT.zip ../anomaly-detection/src/test/resources/org/opensearch/ad/bwc/job-scheduler/$INCREMENT_TO.0-SNAPSHOT

rm ../dashboards-reports/reports-scheduler/src/test/resources/job-scheduler/*
cp ./build/distributions/opensearch-job-scheduler-$INCREMENT_TO.0-SNAPSHOT.zip ../dashboards-reports/reports-scheduler/src/test/resources/job-scheduler/

rm ../index-management/src/test/resources/job-scheduler/*
cp ./build/distributions/opensearch-job-scheduler-$INCREMENT_TO.0-SNAPSHOT.zip ../index-management/src/test/resources/job-scheduler/
```

For each of `anomaly-detection`, `dashboards-reports`, and `index-management`, use `git add .` to add the above updated-files, commit, and push an update, e.g. `git push <your-github-username> increment-to-$INCREMENT_TO`.

#### Update the Manifest

Ensure all plugins pass CI and the version increments have been merged. Add the remaining components to the manifest, e.g. [opensearch-build#1380](https://github.com/opensearch-project/opensearch-build/pull/1380).

### Backporting in OpenSearch Plugins

See [BACKPORT](BACKPORT.md).