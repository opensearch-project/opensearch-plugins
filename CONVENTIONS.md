- [Plugin Naming Conventions](#plugin-naming-conventions)
  - [GitHub](#github)
    - [OpenSearch Plugins](#opensearch-plugins)
    - [OpenSearch Dashboard Plugins](#opensearch-dashboard-plugins)
  - [Metadata](#metadata)
    - [OpenSearch Plugins](#opensearch-plugins-1)
    - [OpenSearch Dashboard Plugins](#opensearch-dashboard-plugins-1)
  - [Artifacts](#artifacts)
    - [OpenSearch Plugins](#opensearch-plugins-2)
    - [OpenSearch Dashboards Plugins](#opensearch-dashboards-plugins)
  - [Folders](#folders)
    - [OpenSearch Plugins](#opensearch-plugins-3)
    - [OpenSearch Dashboard Plugins](#opensearch-dashboard-plugins-2)
  - [Classes](#classes)
  - [Settings](#settings)
  - [APIs](#apis)
  - [Indices](#indices)
  - [Identifiers](#identifiers)
  - [Variables](#variables)

## Plugin Naming Conventions

### GitHub

- Within the [opensearch-project org](https://github.com/opensearch-project), do not include the word `OpenSearch` or `OpenSearch Dashboards` in the repo name.
- Inherit templates, CoC, etc., from [opensearch-project/.github](https://github.com/opensearch-project/.github).
- Use lowercase repo names.
- Provide a meaningful description, e.g. `An OpenSearch Dashboards plugin to perform real-time and historical anomaly detection on OpenSearch data.`

#### OpenSearch Plugins

- Do not include the word `plugin` in the repo name, e.g. [job-scheduler](https://github.com/opensearch-project/job-scheduler).

#### OpenSearch Dashboard Plugins

- Prefix OpenSearch Dashboards plugins with `dashboards-`, e.g. [dashboards-reports](https://github.com/opensearch-project/dashboards-reports).

### Metadata

#### OpenSearch Plugins

- Plugin name starts with `opensearch-`.
- Include `opensearch-` in the plugin name.
- Description does not end with a period.
- Include `OpenSearch` and `plugin` in the descripion. 

```groovy
opensearchplugin {
    name 'opensearch-job-scheduler'
    description 'OpenSearch Job Scheduler plugin'
    classname 'org.opensearch.jobscheduler.JobSchedulerPlugin'
}
```

#### OpenSearch Dashboard Plugins

- Plugin ID is lowercase camelcase, e.g. `notebooksDashboards`.
- Version follows [semver](https://semver.org/).

For example, [dashboards-notebooks/opensearch_dashboards.json](https://github.com/opensearch-project/dashboards-notebooks/blob/main/dashboards-notebooks/opensearch_dashboards.json).

```json
{
  "id": "notebooksDashboards",
  "version": "1.0.0.0-beta1",
  "opensearchDashboardsVersion": "1.0.0-beta1",
  "server": true,
  "ui": true,
  "requiredPlugins": ["navigation", "embeddable", "dashboard"],
  "optionalPlugins": []
}
```

### Artifacts

#### OpenSearch Plugins

- Artifacts are of the `<package name>-<version>` format, e.g. `opensearch-job-scheduler-1.0.0.0-beta1.zip`.

#### OpenSearch Dashboards Plugins

- Artifacts are of the `<package name>-<version>` format, e.g. `notebooks-dashboards-1.0.0.0-beta1.zip`.

### Folders

#### OpenSearch Plugins

- Always use kebab-case.
- Include full plugin name, e.g. `plugins/opensearch-knn`.

#### OpenSearch Dashboard Plugins

- Always use kebab-case.
- Include full plugin name, e.g. `plugins/notebooks-dashboards`. The change to automatically convert folders to kebab-case is still pending, see [OpenSearch-Dashboards#322](https://github.com/opensearch-project/OpenSearch-Dashboards/issues/322).

### Classes

- Lowercase namespaces, e.g. `org.opensearch.jobscheduler`.
- CamelCase classes, e.g. `org.opensearch.jobscheduler.JobSchedulerPlugin`.

### Settings

- Settings do not have a prefix.
- Use `plugins.` for plugin settings, e.g. `plugins.jobscheduler.request_timeout`.
- Do not use `opensearch` in the setting name, e.g. `plugins.jobscheduler.request_timeout` and not `opensearch.jobscheduler.request_timeout`.

### APIs

- Use `_` as prefix for APIs, e.g. `_plugins/_anomaly_detection/*`.

### Indices

- Use `.` as prefix for indices.

### Identifiers

- TODO

### Variables

- Use common sense for your plugin.
