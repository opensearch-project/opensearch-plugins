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

- Prefix OpenSearch Dashboards plugins repo name with `dashboards-`, e.g. [dashboards-observability](https://github.com/opensearch-project/dashboards-observability).

### Metadata

#### OpenSearch Plugins

- Plugin name starts with `opensearch-`.
- Include `opensearch-` in the plugin name.
- Description does not end with a period.
- Include `OpenSearch` and `plugin` in the description.

```groovy
opensearchplugin {
    name 'opensearch-job-scheduler'
    description 'OpenSearch Job Scheduler plugin'
    classname 'org.opensearch.jobscheduler.JobSchedulerPlugin'
}
```

#### OpenSearch Dashboard Plugins

- Plugin ID is lowercase camelCase, e.g. `observabilityDashboards`.
- Package `name` in `package.json` is kebab-case, e.g. [observability-dashboards](https://github.com/opensearch-project/dashboards-observability/blob/2.5/package.json)
- Version follows [semver](https://semver.org/).

For example, [dashboards-observability/opensearch_dashboards.json](https://github.com/opensearch-project/dashboards-observability/blob/2.5/opensearch_dashboards.json).

```json
{
  "id": "observabilityDashboards",
  "version": "2.5.0.0",
  "opensearchDashboardsVersion": "2.5.0",
  "server": true,
  "ui": true,
  "requiredPlugins": []
}
```

### Artifacts

#### OpenSearch Plugins

- Artifacts are of the `<package name>-<version>` format, e.g. `opensearch-job-scheduler-1.0.0.0.zip`.

#### OpenSearch Dashboards Plugins

- Artifacts are of the `<package name>-<version>` format, e.g. `notebooks-dashboards-1.0.0.0.zip`.

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
