## Upgrading Plugins to OpenSearch and OpenSearch Dashboards

These are all the steps to upgrade plugins to work with OpenSearch and OpenSearch Dashboards 1.0.0, including building, and passing all tests.

### OpenSearch Plugins

#### Building

1. Consume artifacts from your dependencies, including OpenSearch, see [BUILDING](BUILDING.md).
2. Change the namespaces from `org.elasticsearch` to `org.opensearch`. Use the naming convention below.
3. Consume dependencies with OpenSearch/Dashboards 1.0.0. Continue using `org.elasticsearch` dependencies for `securemock`, `mocksocket` and `jna-build`.
4. Ensure CI works end-to-end, build and test your plugin.
5. Report all runtime failures of OpenSearch to [OpenSearch Issues](http://github.com/opensearch-project/opensearch/issues) and runtime failures of plugins in plugin repositories.

See [anomaly-detection#1](https://github.com/opensearch-project/anomaly-detection/pull/1) for an example.

#### Naming Conventions

The following naming convention has been adopted for Elasticsearch.

| Before          | After        |
|-----------------|--------------|
| `Elasticsearch` | `OpenSearch` |
| `elasticsearch` | `opensearch` |
| `es`            | `opensearch` |
| `Es`            | `OpenSearch` |
| `ES`            | `OpenSearch` |
| `ELASTICSEARCH` | `OPENSEARCH` |

See below for Kibana-related naming conventions.

### OpenSearch Dashboard Plugins

#### Building

1. Change the namespaces from `Kibana` to `OpenSearch-Dashboards`. Use the naming convention below.
2. Download and install OpenSearch-Dashboards. Make sure the version specific in `package.json` matches the OpenSearch version.
3. Create a `plugins` directory in the root of the project, if it doesn't exist.
4. Run `yarn osd bootstrap` inside `opensearch-dashboards/plugins/opensearch-dashboards-plugin`.
5. Build and test your plugin.
6. Report all runtime failures of OpenSearch Dashboards to [OpenSearch Dashboards Issues](http://github.com/opensearch-project/opensearch-dashboards/issues) and runtime failures of plugins in plugin repositories.

#### Naming Conventions

| Before   | After                   |
|----------|-------------------------|
| `kibana` | `opensearchDashboards`  |
| `Kibana` | `OpenSearchDashboards`  |
| `KIBANA` | `OPENSEARCH_DASHBOARDS` |
| `kbn`    | `osd`                   |
| `Kbn`    | `Osd`                   |
| `KBN`    | `OSD`                   |
| `kib`    | `osd`                   |
| `Kib`    | `Osd`                   |
| `KIB`    | `OSD`                   |

See [anomaly-detection-dashboards-plugin#1](https://github.com/opensearch-project/anomaly-detection-dashboards-plugin/pull/1) for an example.