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
Use the `1.0.0-alpha1` tag to have a stable version [1.0.0-alpha1 Tag](https://github.com/opensearch-project/OpenSearch/releases/tag/1.0.0-alpha1) 

This will publish artifacts which are part of release version `1.0.0-alpha1`.
```
~/OpenSearch (main)> git checkout 1.0.0-alpha1 -b alpha1-release
~/OpenSearch (main)> ./gradlew publishToMavenLocal -Dbuild.version_qualifier=alpha1 -Dbuild.snapshot=false
```

On Unix, the local Maven repository is the `~/.m2` folder. For example, the above command produces `~/.m2/repository/org/opensearch/opensearch/1.0.0-alpha1/opensearch-1.0.0-alpha1.jar`.

#### Use OpenSearch from Maven Local in Plugins

Your plugin may have more dependencies than just OpenSearch. For example, [anomaly-detection](https://github.com/opensearch-project/anomaly-detection) requires [OpenSearch](https://github.com/opensearch-project/OpenSearch), [common-utils](https://github.com/opensearch-project/common-utils) and [job-scheduler](https://github.com/opensearch-project/job-scheduler) to be successfully published to Maven local, manually. 

The following trivial build script changes were made to successfully run `./gradlew publishToMavenLocal` in each of these.

* [common-utils#8](https://github.com/opensearch-project/common-utils/pull/8)
* [job-scheduler#8](https://github.com/opensearch-project/job-scheduler/pull/8)
* [anomaly-detection#13](https://github.com/opensearch-project/anomaly-detection/pull/13)

We plan to remove the need for this work-around via [OpenSearch#581](https://github.com/opensearch-project/OpenSearch/issues/581).

## Contrbuting

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This project is licensed under the Apache-2.0 License.
