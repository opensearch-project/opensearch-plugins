- [OpenSearch Plugins](#opensearch-plugins)
  - [Building with OpenSearch](#building-with-opensearch)
    - [Publish OpenSearch to Maven Local](#publish-opensearch-to-maven-local)
    - [Use OpenSearch from Maven Local in Plugins](#use-opensearch-from-maven-local-in-plugins)
- [Contrbuting](#contrbuting)
- [License](#license)

## OpenSearch Plugins

This repository contains all the things you ever wanted to know about OpenSearch plugins.

### Building with OpenSearch

Until OpenSearch and other artifacts are published to Maven Central [OpenSearch#581](https://github.com/opensearch-project/OpenSearch/issues/581), plugins may require building all their dependencies and publishing them to Maven local.

#### Publish OpenSearch to Maven Local
Use the `1.0.0-alpha2` tag to have a stable version [1.0.0-alpha2 Tag](https://github.com/opensearch-project/OpenSearch/releases/tag/1.0.0-alpha2) 

This will publish artifacts which are part of release version `1.0.0-alpha2`.
This will support running integration tests. Please note that the limitation for integration tests is that it is only supported for builds on linux platform.

```
~/OpenSearch (main)> git checkout 1.0.0-alpha2 -b alpha2-release
~/OpenSearch (main)> ./gradlew publishToMavenLocal -Dbuild.version_qualifier=alpha2 -Dbuild.snapshot=false
```

On Unix, the local Maven repository is the `~/.m2` folder. For example, the above command produces `~/.m2/repository/org/opensearch/opensearch/1.0.0-alpha2/opensearch-1.0.0-alpha2.jar`.

#### Use OpenSearch from Maven Local in Plugins

Your plugin may have more dependencies than just OpenSearch. For example, [anomaly-detection](https://github.com/opensearch-project/anomaly-detection) requires [OpenSearch](https://github.com/opensearch-project/OpenSearch), [common-utils](https://github.com/opensearch-project/common-utils) and [job-scheduler](https://github.com/opensearch-project/job-scheduler) to be successfully published to Maven local, manually. 

The following trivial build script changes were made to successfully run `./gradlew publishToMavenLocal` in each of these.

* [common-utils#9](https://github.com/opensearch-project/common-utils/pull/9)
* [job-scheduler#11](https://github.com/opensearch-project/job-scheduler/pull/11)
* [anomaly-detection#18](https://github.com/opensearch-project/anomaly-detection/pull/18)

We plan to remove the need for this work-around via [OpenSearch#581](https://github.com/opensearch-project/OpenSearch/issues/581).

## Contrbuting

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This project is licensed under the Apache-2.0 License.
