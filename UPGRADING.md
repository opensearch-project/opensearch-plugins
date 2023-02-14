<!-- TOC -->

- [Upgrading Plugins to OpenSearch and OpenSearch Dashboards](#upgrading-plugins-to-opensearch-and-opensearch-dashboards)
  - [Upgrading to OpenSearch](#upgrading-to-opensearch)
  - [OpenSearch Plugins](#opensearch-plugins)
    - [Building](#building)
    - [Naming Conventions](#naming-conventions)
    - [Backwards Compatibility](#backwards-compatibility)
      - [Settings](#settings)
      - [Rest APIs on 9200 port](#rest-apis-on-9200-port)
      - [Rest APIs on 9600 or other ports](#rest-apis-on-9600-or-other-ports)
      - [Indices](#indices)
  - [OpenSearch Dashboards Plugins](#opensearch-dashboards-plugins)
    - [Building](#building-1)
    - [Naming Conventions](#naming-conventions-1)
    - [REST API Compatibility with OpenSearch Plugins](#rest-api-compatibility-with-opensearch-plugins)

<!-- /TOC -->

## Upgrading Plugins to OpenSearch and OpenSearch Dashboards

In [introducing OpenSearch](https://aws.amazon.com/blogs/opensource/introducing-opensearch/) we promised that _"The Amazon OpenSearch Service APIs will be backward compatible with the existing service APIs to eliminate any need for customers to update their current client code or applications. Additionally, just as we did for previous versions of Elasticsearch, we will provide a seamless upgrade path from existing Elasticsearch 6.x and 7.x managed clusters to OpenSearch."_ We also [asked customers](https://discuss.opendistrocommunity.dev/t/upgrade-path-to-opensearch/5788/10) about what that would look like, and scoped what _upgrading_ and _backwards compatibility_ mean. 

### Upgrading to OpenSearch

OpenSearch and OpenSearch Dashboards 1.0 North Star is the following upgrade path.

> Upgrading from Elasticsearch OSS and Kibana OSS or Open Distro for Elasticsearch (ODFE) to OpenSearch and OpenSearch Dashboards is just like upgrading between versions of Elasticsearch OSS and Kibana OSS. Specifically, OpenSearch supports rolling upgrades and restart upgrades from Elasticsearch OSS 6.8.0 through Elasticsearch OSS 7.10.2 to OpenSearch 1.0. OpenSearch Dashboards supports restart upgrades from Kibana OSS 6.8.0 through Kibana OSS 7.10.2 to OpenSearch Dashboards 1.0. All 1.x versions of ODFE similarly support upgrades to OpenSearch and OpenSearch Dashboards 1.0.

When making changes, ensure that you are not breaking anything that prevents rolling upgrades to OpenSearch or restart upgrades to OpenSearch Dashboards. Do not do any refactoring or renaming that would risk breaking everything.

You must make branding changes (e.g. text labels, icons).

You do not need to rename any "magic words" (e.g. variable names, index names, parameters) that are required by the system to function in its ALv2 form.

### OpenSearch Plugins

The following steps are generally known as required to upgrade plugins to work with OpenSearch and OpenSearch Dashboards 1.0.0, with backwards compatibility, including building, and passing all tests.

#### Building

1. Consume artifacts from your dependencies, including OpenSearch, see [BUILDING](BUILDING.md).
2. Change the namespaces from `org.elasticsearch` to `org.opensearch`. Use the naming convention below.
3. Consume dependencies with OpenSearch/Dashboards 1.0.0-beta1. Continue using `org.elasticsearch` dependencies for `securemock`, `mocksocket` and `jna-build`.
4. Ensure CI works end-to-end, build and test your plugin.
5. Report all runtime failures of OpenSearch to [OpenSearch Issues](http://github.com/opensearch-project/opensearch/issues) and runtime failures of plugins in plugin repositories.

See [anomaly-detection#1](https://github.com/opensearch-project/anomaly-detection/pull/1) for an example.

#### Naming Conventions

The following naming convention has been adopted for renaming from Elasticsearch.

| Before          | After        |
|-----------------|--------------|
| `Elasticsearch` | `OpenSearch` |
| `elasticsearch` | `opensearch` |
| `es`            | `opensearch` |
| `Es`            | `OpenSearch` |
| `ES`            | `OpenSearch` |
| `ELASTICSEARCH` | `OPENSEARCH` |

You will need to rename namespaces, classes, methods, variables and identifiers.

See below for Kibana-related naming conventions.

#### Backwards Compatibility

##### Settings

1. Assuming your settings live in a single class, make a copy of that class and prepend `LegacyOpenDistro` to the copy. Do not rename the settings here in the legacy class. For example:

   ```java
   package com.amazon.opendistroforelasticsearch.jobscheduler;

   import org.opensearch.common.settings.Setting;
   import org.opensearch.common.unit.TimeValue;

   public class LegacyOpenDistroJobSchedulerSettings {

   }
   ```

2. Add `Setting.Property.Deprecated` to each existing setting to mark them deprecated. For example:

   ```java
   public class LegacyOpenDistroJobSchedulerSettings {

      public static final Setting<TimeValue> REQUEST_TIMEOUT = Setting.positiveTimeSetting(
               "opendistro.jobscheduler.request_timeout",
               TimeValue.timeValueSeconds(10),
               Setting.Property.NodeScope, Setting.Property.Dynamic, Setting.Property.Deprecated);
   }
   ```

3. Include legacy settings in `Plugin#getSettings`. Example:

   ```java
   public class JobSchedulerPlugin extends Plugin implements ExtensiblePlugin {

      @Override
      public List<Setting<?>> getSettings() {
         List<Setting<?>> settingList = new ArrayList<>();
         settingList.add(LegacyOpenDistroJobSchedulerSettings.REQUEST_TIMEOUT);
         settingList.add(JobSchedulerSettings.REQUEST_TIMEOUT);
         return settingList;
      }
   }
   ```

   If you do not add your legacy settings here they will be archived upon the node joining the OpenSearch cluster.
   
4. In the non-legacy class, rename settings and re-declare them to fallback on the legacy settings.

   ```java
   public class JobSchedulerSettings {

       public static final Setting<TimeValue> REQUEST_TIMEOUT = Setting.positiveTimeSetting(
            "plugins.jobscheduler.request_timeout",
            LegacyOpenDistroJobSchedulerSettings.REQUEST_TIMEOUT,
            Setting.Property.NodeScope, Setting.Property.Dynamic);
   }
   ```

   Note that this setting does not have a default value, but defaults to falling back to `LegacyOpenDistroJobSchedulerSettings.REQUEST_TIMEOUT`.

   Note that the naming convention for plugin settings has changed from `opendistro.` to `plugins.` If you have renamed any settings to `opensearch.`, please change them again to `plugins.`.

5. Make sure you have implemented a listener to update the setting value. If a setting is set in `opensearch.yml`, you're all set. But if the setting is set via API, the node will start, settings will be read, defaults applied (fallback value will be used), and only afterwards fetched from the cluster and set on your node. Thus your setting will end up with the default value of the fallback.

   To fix this, add a listener. For example:

   ```java
   clusterService.getClusterSettings().addSettingsUpdateConsumer(JobSchedulerSettings.REQUEST_TIMEOUT,
      value -> {
         this.requestTimeout = value;
         log.debug("Setting request timeout: " + this.requestTimeout.getMinutes());
      });
   ```

   Note that only listener on new settings are required. Updates on an old setting will be synced to the corresponding new setting automatically until the new setting is changed directly.

6. Write tests.

   ```java
    public void testSettingsGetValueWithLegacyFallback() {
        Settings settings = Settings.builder()
            .put("opendistro.jobscheduler.retry_count", 3)
        .build();
        
        assertEquals(JobSchedulerSettings.RETRY_COUNT.get(settings), Integer.valueOf(3)); 

        assertSettingDeprecationsAndWarnings(new Setting[]{
            LegacyOpenDistroJobSchedulerSettings.RETRY_COUNT
        });
    }   
   ```

7. Document your newly renamed settings. For example, [documentation-website/.../alerting/settings.md](https://github.com/opensearch-project/documentation-website/blob/1.0/_monitoring-plugins/alerting/settings.md). Many settings and names were bulk-changed in the docs, so this is a good time to ensure that the documentation is accurate.

See [job-scheduler#20](https://github.com/opensearch-project/job-scheduler/pull/20) for an example.

##### Rest APIs on 9200 port

All APIs on 9200 which have to be migrated can follow this design.
From here on the doc will be focussed on an ODFE plugin as an example.

1. All plugins extend `BaseRestHandler` class to implement their own RestAPIs.
2. All the existing APIs with `_opendistro` should be supported but for maintenance mode only.
3. All new APIs will be implemented with `_plugins` as defined in the [Naming Conventions](./CONVENTIONS.md).
4. The method `replacedRoutes()` is overriden and this should now have both `_opendistro` and `_plugins` APIs.
    ```java
    public static final String LEGACY_OPENDISTRO_AD_BASE = "/_opendistro/_anomaly_detection";
    public static final String LEGACY_OPENDISTRO_AD_BASE_URI = LEGACY_OPENDISTRO_AD_BASE + "/detectors";
    public static final String AD_BASE_URI = "/_plugins/_anomaly_detection";
    public static final String AD_BASE_DETECTORS_URI = AD_BASE_URI + "/detectors";

    @Override
    public List<ReplacedRoute> replacedRoutes() {
        return ImmutableList
            .of(
                // delete anomaly detector document
                new ReplacedRoute(
                    RestRequest.Method.DELETE,
                    String.format(Locale.ROOT, "%s/{%s}", AnomalyDetectorPlugin.AD_BASE_DETECTORS_URI, DETECTOR_ID),
                    RestRequest.Method.DELETE,
                    String.format(Locale.ROOT, "%s/{%s}", AnomalyDetectorPlugin.LEGACY_OPENDISTRO_AD_BASE_URI, DETECTOR_ID)
                )
            );
    }
   ```
5. Add new tests to cover both old and new APIs.
    ```java
    public void testBackwardCompatibilityWithOpenDistro() throws IOException {
        // Create a detector
        AnomalyDetector detector = TestHelpers.randomAnomalyDetector(TestHelpers.randomUiMetadata(), null);
        String indexName = detector.getIndices().get(0);
        TestHelpers.createIndex(client(), indexName, toHttpEntity("{\"name\": \"test\"}"));

        // Verify the detector is created using legacy _opendistro API
        Response response = TestHelpers
            .makeRequest(
                client(),
                "POST",
                TestHelpers.LEGACY_OPENDISTRO_AD_BASE_DETECTORS_URI,
                ImmutableMap.of(),
                toHttpEntity(detector),
                null
            );
        assertEquals("Create anomaly detector failed", RestStatus.CREATED, restStatus(response));
        Map<String, Object> responseMap = entityAsMap(response);
        String id = (String) responseMap.get("_id");
        int version = (int) responseMap.get("_version");
        assertNotEquals("response is missing Id", AnomalyDetector.NO_ID, id);
        assertTrue("incorrect version", version > 0);

        // Get the detector using new _plugins API
        AnomalyDetector createdDetector = getAnomalyDetector(id, client());
        assertEquals("Get anomaly detector failed", createdDetector.getDetectorId(), id);

        // Delete the detector using legacy _opendistro API
        response = TestHelpers
            .makeRequest(
                client(),
                "DELETE",
                TestHelpers.LEGACY_OPENDISTRO_AD_BASE_DETECTORS_URI + "/" + createdDetector.getDetectorId(),
                ImmutableMap.of(),
                "",
                null
            );
        assertEquals("Delete anomaly detector failed", RestStatus.OK, restStatus(response));
   }
   ```
6. Add documentation for the new APIs. For example, [documentation-website/.../ad/api.md](https://github.com/opensearch-project/documentation-website/blob/1.0/_monitoring-plugins/ad/api.md)

See [anomaly-detection#35](https://github.com/opensearch-project/anomaly-detection/pull/35) for an example.

##### Rest APIs on 9600 or other ports

All APIs on 9600 or other ports which have to be migrated can follow this design.
From here on the doc will be focussed on an ODFE plugin as an example.

1. All plugins uses `HttpServer` to create a server which handles HTTPS requests and is bound to an IP address and port number.
2. All the existing APIs with `_opendistro` should be supported but for maintenance mode only.
3. All new APIs will be implemented with `_plugins` as defined in the [Naming Conventions](./CONVENTIONS.md).
4. The method `createContext` is used to create HttpContext's for both `_opendistro` and `_plugins` APIs.
   ```java
   public static final String QUERY_URL = "/_plugins/_performanceanalyzer/metrics";
   public static final String LEGACY_OPENDISTRO_QUERY_URL =
               "/_opendistro/_performanceanalyzer/metrics";
   public static final String BATCH_METRICS_URL = "/_plugins/_performanceanalyzer/batch";
   public static final String LEGACY_OPENDISTRO_BATCH_METRICS_URL =
               "/_opendistro/_performanceanalyzer/batch";
   QueryMetricsRequestHandler queryMetricsRequestHandler =
                       new QueryMetricsRequestHandler(netClient, metricsRestUtil, appContext);
   httpServer.createContext(QUERY_URL, queryMetricsRequestHandler);
   httpServer.createContext(LEGACY_OPENDISTRO_QUERY_URL, queryMetricsRequestHandler);
   
   QueryBatchRequestHandler queryBatchRequestHandler =
                       new QueryBatchRequestHandler(netClient, metricsRestUtil);
   httpServer.createContext(BATCH_METRICS_URL, queryBatchRequestHandler);
   httpServer.createContext(LEGACY_OPENDISTRO_BATCH_METRICS_URL, queryBatchRequestHandler);
   ```    
5. Add new tests to cover both old and new APIs.
6. Add documentation for the new APIs. For example, [documentation-website/.../ad/api.md](https://github.com/opensearch-project/documentation-website/blob/1.0/_monitoring-plugins/ad/api.md)

See [performance-analyzer#18](https://github.com/opensearch-project/performance-analyzer-rca/pull/18) for an example.

##### Indices

Do not change index names at this time to preserve backwards compatibility.

### OpenSearch Dashboards Plugins

#### Building

1. Change the namespaces from `Kibana` to `OpenSearch-Dashboards`. Use the naming convention below.
2. Download and install OpenSearch-Dashboards. Make sure the version specific in `package.json` matches the OpenSearch version.
3. Create a `plugins` directory in the root of the project, if it doesn't exist.
4. Run `yarn osd bootstrap` inside `opensearch-dashboards/plugins/opensearch-dashboards-plugin`.
5. Build and test your plugin.
6. Report all runtime failures of OpenSearch Dashboards to [OpenSearch Dashboards Issues](https://github.com/opensearch-project/opensearch-dashboards/issues) and runtime failures of plugins in plugin repositories.

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

#### REST API Compatibility with OpenSearch Plugins

If your OpenSearch Dashboards plugin is communicating with a corresponding OpenSearch plugin, you will want to update any migrated API calls (see API changes [here](#backwards-compatibility)) to be compatible with the new endpoints.

For example, the [Anomaly Detection OpenSearch Plugin](https://github.com/opensearch-project/anomaly-detection) migrated all of its APIs to support the `_plugins/*` prefix (PR [here](https://github.com/opensearch-project/anomaly-detection/pull/35)).

The corresponding change for the [Anomaly Detection Dashboards Plugin](https://github.com/opensearch-project/anomaly-detection-dashboards-plugin) is captured in the PR [here](https://github.com/opensearch-project/anomaly-detection-dashboards-plugin/pull/25).