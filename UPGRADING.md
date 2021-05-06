## Upgrading Plugins to OpenSearch and OpenSearch Dashboards

These are all the steps to upgrade plugins to work with OpenSearch and OpenSearch Dashboards 1.0.0, including building, and passing all tests.

- [OpenSearch Plugins](#opensearch-plugins)
   - [Building](#building)
   - [Naming Conventions](#naming-conventions)
   - [Settings Backwards Compatibility](#settings-backwards-compatibility)
- [OpenSearch Dashboard Plugins](#opensearch-dashboard-plugins)
   - [Building](#building)
   - [Naming Conventions](#naming-conventions)

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

#### Settings Backwards Compatibility

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
            "opensearch.jobscheduler.request_timeout",
            LegacyOpenDistroJobSchedulerSettings.REQUEST_TIMEOUT,
            Setting.Property.NodeScope, Setting.Property.Dynamic);

   }
   ```

   Note that this setting does not have a default value, but defaults to falling back to `LegacyOpenDistroJobSchedulerSettings.REQUEST_TIMEOUT`.

5. Make sure you have implemented a listener to update the setting value. If a setting is set in `opensearch.yml`, you're all set. But if the setting is set via API, the node will start, settings will be read, defaults applied (fallback value will be used), and only afterwards fetched from the cluster and set on your node. Thus your setting will end up with the default value of the fallback.

   To fix this, add a listener. For example:

   ```java
   clusterService.getClusterSettings().addSettingsUpdateConsumer(JobSchedulerSettings.REQUEST_TIMEOUT,
      value -> {
         this.requestTimeout = value;
         log.debug("Setting request timeout: " + this.requestTimeout.getMinutes());
      });
   ```

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

7. Document your newly renamed settings.

   TODO

See [job-scheduler#20](https://github.com/opensearch-project/job-scheduler/pull/20) for an example.

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