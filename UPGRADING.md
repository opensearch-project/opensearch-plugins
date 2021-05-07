- [Upgrading Plugins](#upgrading-plugins-to-opensearchopensearch-dashboards)
    - [Steps to Upgrade](#upgrading)
        - [OpenSearch Plugins](#opensearch-plugins)
            - [Naming Conventions](#naming-convention)
        - [OpenSearch Dashboard Plugins](#opensearch-plugins)
            - [Naming Conventions](#naming-convention)
    - [Backwards Compatibility](#backwards-compatibility-support)
        - [OpenSearch Plugins](#opensearch-plugins)
            - [Rest APIs](#rest-apis)

## Upgrading Plugins to OpenSearch/OpenSearch Dashboards

These are all the steps to upgrade plugins to work with OpenSearch and OpenSearch Dashboards, including building, and passing all tests.

### Upgrading
We will upgrade the plugins to use OpenSearch/Dashboards 1.0.0.

#### OpenSearch Plugins

1. Consume artifacts from OpenSearch by publishing them locally. Please see [Readme](./README.md#building-with-opensearch).
2. Change the namespaces from `org.elasticsearch` to `org.opensearch`. Use the naming convention below.
3. Consuming dependencies with OpenSearch/Dashboards 1.0.0. 
   * For dependencies `securemock`, `mocksocket` and `jna-build` please continue to use `org.elasticsearch` namespace.
4. Build and test your plugin.
5. Report all runtime failures of OpenSearch to [OpenSearch Issues](http://github.com/opensearch-project/opensearch/issues) and 
runtime failures of plugins in [OpenSearch plugin Issues](https://github.com/opensearch-project/opensearch-plugins/issues).

Here is an example for the changes which were done for OpenSearch plugin.
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

#### OpenSearch Dashboard Plugins

1. Change the namespaces from `Kibana` to `OpenSearch-Dashboards`. Use the naming convention below.
2. Download and install OpenSearch-Dashboards. Make sure the version specific in `package.json` matches the OpenSearch version.
3. Create plugins directory in the root of the project, if plugins directory doesn't exist already.
4. Run `yarn osd bootstrap` inside `opensearch-dashboards/plugins/opensearch-dashboards-plugin`.
5. Build and test your plugin.
6. Report all runtime failures of OpenSearch to [OpenSearch Dashboards Issues](http://github.com/opensearch-project/opensearch-dashboards/issues) and 
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

Here is an example for the changes which were done for OpenSearch Dashboard plugin.
https://github.com/opensearch-project/anomaly-detection-dashboards-plugin/pull/1

### Backwards Compatibility support

There are a bunch of items to think about and support backwards compatibility.

* Renaming Rest APIs
* Renaming Indexes
* Renaming Settings
* Renaming Namespaces (E.g. com.amazon.opendistroforelasticsearch to org.opensearch)
* Renaming all remaining identifiers (E.g Opendistro to OpenSearch).
* 100% Backward compatibility with ES v.?
* Drop in replacement for Opendistro plugins with OpenSearch plugins

#### OpenSearch Plugins

##### Rest APIs

All APIs which have to be migrated can follow this design.
From here on the doc will be focussed on an ODFE plugin as an example for the design.

* All plugins extend `BaseRestHandler` class to implement their own RestAPIs.
* All the existing APIs with `_opendistro` should be supported but for maintenance mode only.
* All new APIs will be implemented with `_opensearch` (TBD based on [Issue](https://github.com/opensearch-project/opensearch-plugins/issues/13))
* The method `routes()` is overriden and this should now have both `_opendistro` and `_opensearch` APIs.
* Add new tests to cover both old and new APIs.
* Add documentation for the new APIs.

_Example_: Adding an API in Anomaly Detection plugin.
```java
public static final String AD_BASE_URI = "/_opendistro/_anomaly_detection";
public static final String AD_BASE_OPENSEARCH_URI = "/_opensearch/_anomaly_detection";
public static final String AD_BASE_DETECTORS_URI = AD_BASE_URI + "/detectors";
public static final String AD_BASE_DETECTORS_OPENSEARCH_URI = AD_BASE_OPENSEARCH_URI + "/detectors";

@Override
public List<RestHandler.Route> routes() {
    return ImmutableList
        .of(
            // get the count of number of detectors
            new RestHandler.Route(
                RestRequest.Method.GET,
                String.format(Locale.ROOT, "%s/%s", AnomalyDetectorPlugin.AD_BASE_DETECTORS_URI, COUNT)
            ),
            // get if a detector name exists with name
            new RestHandler.Route(
                RestRequest.Method.GET,
                String.format(Locale.ROOT, "%s/%s", AnomalyDetectorPlugin.AD_BASE_DETECTORS_URI, MATCH)
            ),
            // get the count of number of detectors (opensearch API)
            new RestHandler.Route(
                    RestRequest.Method.GET,
                    String.format(Locale.ROOT, "%s/%s", AnomalyDetectorPlugin.AD_BASE_DETECTORS_OPENSEARCH_URI, COUNT)
            ),
            // get if a detector name exists with name (opensearch API)
            new RestHandler.Route(
                    RestRequest.Method.GET,
                    String.format(Locale.ROOT, "%s/%s", AnomalyDetectorPlugin.AD_BASE_DETECTORS_OPENSEARCH_URI, MATCH)
            )
        );
}
```

_Example_: Pull Request from Anomaly Detection https://github.com/opensearch-project/anomaly-detection/pull/35



