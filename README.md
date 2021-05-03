- [OpenSearch Plugins](#opensearch-plugins)
    - [Building with OpenSearch](#building-with-opensearch)
        - [Publish OpenSearch to Maven Local](#publish-opensearch-to-maven-local)
        - [Use OpenSearch from Maven Local in Plugins](#use-opensearch-from-maven-local-in-plugins)
    - [Upgrading Plugins to work with OpenSearch](#upgrading-plugins-to-work-with-opensearch)
    - [Installing Plugins in OpenSearch](#installing-plugins-in-opensearch)
    - [Plugin Release Notes](#plugin-release-notes)
- [Contributing](#contributing)
- [License](#license)

## OpenSearch Plugins

This repository contains all the things you ever wanted to know about OpenSearch plugins.

### Building with OpenSearch

Until OpenSearch and other artifacts are published to Maven Central [OpenSearch#581](https://github.com/opensearch-project/OpenSearch/issues/581), plugins may require building all their dependencies and publishing them to Maven local.

#### Publish OpenSearch to Maven Local
Use the `1.0.0-beta1` tag to have a stable version [1.0.0-beta1 Tag](https://github.com/opensearch-project/OpenSearch/releases/tag/1.0.0-beta1) 

This will publish artifacts which are part of release version `1.0.0-beta1`.
This will support running integration tests. Please note that the limitation for integration tests is that it is only supported for builds on linux platform.

```
~/OpenSearch (main)> git checkout 1.0.0-beta1 -b beta1-release
~/OpenSearch (main)> ./gradlew publishToMavenLocal -Dbuild.version_qualifier=beta1 -Dbuild.snapshot=false
```

On Unix, the local Maven repository is the `~/.m2` folder. For example, the above command produces `~/.m2/repository/org/opensearch/opensearch/1.0.0-beta1/opensearch-1.0.0-beta1.jar`.

#### Use OpenSearch from Maven Local in Plugins

Your plugin may have more dependencies than just OpenSearch. For example, [anomaly-detection](https://github.com/opensearch-project/anomaly-detection) requires [OpenSearch](https://github.com/opensearch-project/OpenSearch), [common-utils](https://github.com/opensearch-project/common-utils) and [job-scheduler](https://github.com/opensearch-project/job-scheduler) to be successfully published to Maven local, manually. 

The following trivial build script changes were made to successfully run `./gradlew publishToMavenLocal` in each of these.

* [common-utils#9](https://github.com/opensearch-project/common-utils/pull/9)
* [job-scheduler#11](https://github.com/opensearch-project/job-scheduler/pull/11)
* [anomaly-detection#18](https://github.com/opensearch-project/anomaly-detection/pull/18)

We plan to remove the need for this work-around via [OpenSearch#581](https://github.com/opensearch-project/OpenSearch/issues/581).

### Upgrading Plugins to work with OpenSearch

To upgrade your existing plugins to work with OpenSearch, see [UPGRADING](./UPGRADING.md).

### Installing Plugins

#### OpenSearch Plugins

Assemble, extract and run OpenSearch `1.0.0-beta1` using [Building OpenSearch package](https://github.com/opensearch-project/OpenSearch/blob/main/TESTING.asciidoc#creating-packages).  

_Example_: For Linux platform

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

#### OpenSearch Dashboards Plugins
Build and run OpenSearch Dashboards `1.0.0-beta1`. For setting up `yarn` and `nvm` follow instructions [Getting Started](https://github.com/opensearch-project/OpenSearch-Dashboards#getting-started).

_Example_: For Linux platform

```
~/ > git clone https://github.com/opensearch-project/OpenSearch-Dashboards.git
~/OpenSearch-Dashboards (main)> git checkout 1.0.0-beta1
~/OpenSearch-Dashboards (main)> yarn build --release --version-qualifier beta1
~/OpenSearch-Dashboards (main)> cd build/opensearch-dashboards-1.0.0-beta1-linux-x64
~/OpenSearch-Dashboards (main)> ./bin/opensearch-dashboards
```

Checkout, boostrap and run OpenSearch Dashboards with the plugin

_Example_: Install Anomaly Detection Dashboards.

```
~/OpenSearch-Dashboards (main)> cd plugins
~/OpenSearch-Dashboards/plugins (main)> git clone https://github.com/opensearch-project/anomaly-detection-dashboards-plugin.git
~/OpenSearch-Dashboards/plugins (main)> cd ../ && yarn osd bootstrap
~/OpenSearch-Dashboards (main)> yarn start
```


### Plugin Release Notes

Plugins generally use a standard format for release notes, see [RELEASE_NOTES](./RELEASE_NOTES.md).

## Contributing

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This project is licensed under the Apache-2.0 License.
