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
npm install
```

See [package.json](package.json) for all dependencies being installed.

### Check Out All Plugins

```sh
meta git pull
```

### Get Repo Info

```sh
meta gh issue list
```

### Add a New Plugin

```sh
meta project import plugin git@github.com:opensearch-project/plugin.git
```

### Create an Issue in All Plugins

Create a file for the issue body, e.g. `backwards-compat/issue.md`.

```
meta exec "gh issue create --label backwards-compatibility --title 'Ensure backwards compatibility with ODFE' --body-file ../backwards-compat/issue.md"
```