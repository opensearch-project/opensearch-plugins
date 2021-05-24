<!-- TOC -->

- [Managing OpenSearch Plugins](#managing-opensearch-plugins)
    - [Install GH](#install-gh)
    - [Install Meta](#install-meta)
    - [Check Out Plugins](#check-out-plugins)
    - [Get Repo Info](#get-repo-info)
    - [Add a New Plugin](#add-a-new-plugin)
    - [Create or Update Labels in All Plugin Repos](#create-or-update-labels-in-all-plugin-repos)
    - [Create an Issue in All Plugin Repos](#create-an-issue-in-all-plugin-repos)
    - [Build Multiple Plugin Artifacts](#build-multiple-plugin-artifacts)

<!-- /TOC -->

## Managing OpenSearch Plugins

We use [meta](https://github.com/mateodelnorte/meta) to manage OpenSearch plugins as a set.

### Install GH

Install and configure GitHub CLI from [cli.github.com/manual/installation](https://cli.github.com/manual/installation). Authenticate with `gh auth login` and ensure that it works, e.g. `gh issue list`.

### Install Meta

```sh
npm install -g meta
```

### Check Out Plugins

```sh
cd plugins
meta git update
```

Use `meta git pull` to subsequently pull the latest revisions.

### Get Repo Info

```sh
plugins> meta gh issue list
```

### Add a New Plugin

```sh
cd plugins
meta project import new-plugin git@github.com:opensearch-project/new-plugin.git
```

### Create or Update Labels in All Plugin Repos

Install [ghi](https://github.com/stephencelis/ghi), e.g. `brew install ghi`.

```
meta exec "ghi label 'backwards-compatibility' -c '#773AA8'"
```

### Create an Issue in All Plugin Repos

Create a file for the issue body, e.g. `issue.md`.

```
meta exec "gh issue create --label backwards-compatibility --title 'Ensure backwards compatibility with ODFE' --body-file ../issue.md"
```

### Build Multiple Plugin Artifacts

You might want to do some bookkeeping, first.

```
rm -rf ~/.m2
```

All the work will be done inside `plugins`, which includes the [meta](plugins/.meta) project that lists all the plugins.

```
> cd plugins
plugins>
```

First, build OpenSearch, e.g. 1.x.

```
plugins> git clone git@github.com:opensearch-project/OpenSearch.git
plugins> cd OpenSearch
plugins/OpenSearch (main)> git checkout 1.x
plugins/OpenSearch (1.x)> ./gradlew publishToMavenLocal -Dbuild.version_qualifier=rc1 -Dbuild.snapshot=false
plugins/OpenSearch (1.x)> cd ..
```

Update code in all plugins.

```
meta git update
meta git pull
```

Build all Gradle plugins into maven local (to make dependencies available), then assemble artifacts and copy them into one place.

```
meta exec "test -e ./gradlew && ./gradlew publishToMavenLocal -Dopensearch.version=1.0.0-rc1 -Dbuild.snapshot=false || echo Skipping ..."

meta exec "test -e ./gradlew && ./gradlew assemble -Dopensearch.version=1.0.0-rc1 -Dbuild.snapshot=false || echo Skipping ..."

mkdir dist
meta exec "test -e ./gradlew && cp build/distributions/*.zip ../dist || echo Skipping ..."
```