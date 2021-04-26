## Upgrading Plugins to OpenSearch/OpenSearch Dashboards

For the plugins to work with OpenSearch here are all the steps you can take and get them building, passing all the tests.

### Upgrading
We will upgrade the plugins to use OpenSearch/Dashboards 1.0.0.

#### OpenSearch Plugins

1. Consume artifacts from OpenSearch by publishing them locally. Please see [Readme](./README.md#building-with-opensearch)
2. Change the namespaces from `org.elasticsearch` to `org.opensearch`. Use the naming convention below.
3. Consuming dependencies with OpenSearch/Dashboards 1.0.0. 
   * For dependencies `securemock`, `mocksocket` and `jna-build` please continue to use `org.elasticsearch` namespace.
4. Build and test your plugin.
5. Please report all run time failures of OpenSearch to [OpenSearch Issues](http://github.com/opensearch-project/opensearch/issues) and 
runtime failures of plugins in [OpenSearch plugin Issues](https://github.com/opensearch-project/opensearch-plugins/issues).

Here is an example for the changes which were done for OpenSearch plugin:
https://github.com/opensearch-project/anomaly-detection/pull/1

##### Naming Convention

Naming convention followed:

* `Elasticsearch` -> `OpenSearch`
* `elasticsearch` -> `opensearch`
* `es` -> `opensearch`
* `Es` -> `OpenSearch`
* `ES` -> `OpenSearch`
* `ELASTICSEARCH` -> `OPENSEARCH`
* Anything with Kibana follow the naming convention below.

#### OpenSearch Dashboard plugins

1. Change the namespaces from `Kibana` to `OpenSearch-Dashboards`. Use the naming convention below.
2. Download and install OpenSearch-Dashboards. Make sure the version specific in `package.json` matches the OpenSearch version.
3. Create plugins directory in the root of the project, if plugins directory doesn't exist already.
4. Run `yarn osd bootstrap` inside `opensearch-dashboards/plugins/opensearch-dashboards-plugin`.
5. Build and test your plugin.
6. Please report all run time failures of OpenSearch to [OpenSearch Dashboards Issues](http://github.com/opensearch-project/opensearch-dashboards/issues) and 
runtime failures of plugins in [OpenSearch plugin Issues](https://github.com/opensearch-project/opensearch-plugins/issues).


##### Naming Convention

* `kibana` -> `opensearchDashboards`
* `Kibana` -> `OpenSearchDashboards`
* `KIBANA` -> `OPENSEARCH_DASHBOARDS`
* `kbn` -> `osd`
* `Kbn` -> `Osd`
* `KBN` -> `OSD`
* `kib` -> `osd`
* `Kib` -> `Osd`
* `KIB` -> `OSD`

Here is an example for the changes which were done for OpenSearch Dashboard plugin:
https://github.com/opensearch-project/anomaly-detection-dashboards-plugin/pull/1