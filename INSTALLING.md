## Installing Plugins

- [Installing Plugins](#installing-plugins)
  - [Installing OpenSearch Plugins](#installing-opensearch-plugins)
  - [Installing OpenSearch Dashboards Plugins](#installing-opensearch-dashboards-plugins)

### Installing OpenSearch Plugins

Assemble, extract and run OpenSearch `1.0.0-beta1` using [Building OpenSearch package](https://github.com/opensearch-project/OpenSearch/blob/main/TESTING.md#creating-packages).  

_Example_: For Linux platform.

```
~/ > git clone https://github.com/opensearch-project/OpenSearch.git
~/OpenSearch (main)> git checkout 1.0.0-beta1
~/OpenSearch (main)> ./gradlew :distribution:archives:linux-tar:assemble -Dbuild.version_qualifier=beta1 -Dbuild.snapshot=false
~/OpenSearch (main)> tar vfxz distribution/archives/linux-tar/build/distributions/opensearch-1.0.0-beta1-linux-x64.tar.gz
~/OpenSearch (main)> cd opensearch-1.0.0-beta1
~/OpenSearch/opensearch-1.0.0-beta1 (main)> ./bin/opensearch
```

Checkout, build and install the plugin.

_Example_: Install Anomaly Detection(Job Scheduler plugin which is a dependency).

To build the plugins with OpenSearch, plugins may require building all their dependencies and publishing them to Maven local along with OpenSearch using [Building Plugins with OpenSearch](https://github.com/opensearch-project/opensearch-plugins/blob/main/BUILDING.md).

```
~/OpenSearch (main)> git clone https://github.com/opensearch-project/job-scheduler.git
~/OpenSearch/job-scheduler (main)> ./gradlew assemble -Dopensearch.version=1.0.0-beta1 -Dbuild.snapshot=false
~/OpenSearch/job-scheduler (main)> cd build/distributions && cp opensearch-job-scheduler-1.0.0.0-beta1.zip ~/
~/OpenSearch/opensearch-1.0.0-beta1 (main)> ./bin/opensearch-plugin install file:///opensearch-job-scheduler-1.0.0.0-beta1.zip
~/OpenSearch (main)> git clone https://github.com/opensearch-project/anomaly-detection.git
~/OpenSearch/anomaly-detection (main)> ./gradlew assemble -Dopensearch.version=1.0.0-beta1 -Dbuild.snapshot=false
~/OpenSearch/anomaly-detection (main)> cd build/distributions && cp opensearch-anomaly-detection-1.0.0.0-beta1.zip ~/
~/OpenSearch/opensearch-1.0.0-beta1 (main)> ./bin/opensearch-plugin install file:///opensearch-anomaly-detection-1.0.0.0-beta1.zip
```

Install the plugins in order if they are dependent on other plugins.

### Installing OpenSearch Dashboards Plugins

Build and run OpenSearch Dashboards `1.0.0-beta1`. For setting up `yarn` and `nvm` follow instructions [Getting Started](https://github.com/opensearch-project/OpenSearch-Dashboards#getting-started).

_Example_: For Linux platform.

```
~/ > git clone https://github.com/opensearch-project/OpenSearch-Dashboards.git
~/OpenSearch-Dashboards (main)> git checkout 1.0.0-beta1
~/OpenSearch-Dashboards (main)> yarn build --release --version-qualifier beta1
~/OpenSearch-Dashboards (main)> cd build/opensearch-dashboards-1.0.0-beta1-linux-x64
~/OpenSearch-Dashboards/build/opensearch-dashboards-1.0.0-beta1-linux-x64 (main)> ./bin/opensearch-dashboards
```

Checkout, boostrap and run OpenSearch Dashboards with the plugin.

_Example_: Install Anomaly Detection Dashboards.

```
~/OpenSearch-Dashboards (main)> cd plugins
~/OpenSearch-Dashboards/plugins (main)> git clone https://github.com/opensearch-project/anomaly-detection-dashboards-plugin.git
~/OpenSearch-Dashboards/plugins (main)> cd ../ && yarn osd bootstrap
~/OpenSearch-Dashboards (main)> yarn start
```

