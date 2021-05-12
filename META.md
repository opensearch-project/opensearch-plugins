## Managing OpenSearch Plugins

We use [meta](https://github.com/mateodelnorte/meta) to manage OpenSearch plugins as a set.

- [Install GH](#install-gh)
- [Install Meta](#install-meta)
- [Check Out All Plugins](#check-out-all-plugins)
- [Get Repo Info](#get-repo-info)
- [Add a New Plugin](#add-a-new-plugin)
- [Create an Issue in All Plugins](#create-an-issue-in-all-plugins)

### Install GH

Install and configure GitHub CLI from [cli.github.com/manual/installation](https://cli.github.com/manual/installation). Authenticate with `gh auth login` and ensure that it works, e.g. `gh issue list`.

### Install Meta

```sh
npm install -g meta
```

### Check Out All Plugins

OpenSearch plugins:

```sh
cd plugins
meta git update
```

OpenSearch Dashboards plugins:

```sh
cd dashboards-plugins
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

### Create an Issue in All Plugins

Create a file for the issue body, e.g. `backwards-compat/issue.md`.

```
meta exec "gh issue create --label backwards-compatibility --title 'Ensure backwards compatibility with ODFE' --body-file ../backwards-compat/issue.md"
```