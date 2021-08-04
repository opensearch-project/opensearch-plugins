# Building and Developing Plugins with OpenSearch

## Building Plugins with OpenSearch

Until OpenSearch and other artifacts are published to Maven Central [OpenSearch#581](https://github.com/opensearch-project/OpenSearch/issues/581), plugins may require building all their dependencies and publishing them to Maven local.

- [Building and Developing Plugins with OpenSearch](#building-and-developing-plugins-with-opensearch)
  - [Building Plugins with OpenSearch](#building-plugins-with-opensearch)
    - [Publish OpenSearch to Maven Local](#publish-opensearch-to-maven-local)
    - [Use OpenSearch from Maven Local in Plugins](#use-opensearch-from-maven-local-in-plugins)
  - [Developing new Plugins for OpenSearch](#developing-new-plugins-for-opensearch)
    - [`plugin-descriptor.properties`](#plugin-descriptorproperties)
    - [`build.gradle`](#buildgradle)
    - [Elements for Plugins](#elements-for-plugins)
      - [Mandatory elements](#mandatory-elements)
      - [Optional elements](#optional-elements)
    - [Notes](#notes)

### Publish OpenSearch to Maven Local

Use the `1.0.0-beta1` tag to have a stable version [1.0.0-beta1 Tag](https://github.com/opensearch-project/OpenSearch/releases/tag/1.0.0-beta1).

This will publish artifacts which are part of release version `1.0.0-beta1`.
This will support running integration tests. Please note that the limitation for integration tests is that it is only supported for builds on linux platform.
In order to run the integration tests on non-linux platforms, temporarily change the `testClusters.integTest.testDistribution` param in the `build.gradle` file of the plugin from `"ARCHIVE"` to `"INTEG_TEST"` ([see related issue](https://github.com/opensearch-project/opensearch-plugins/issues/40)).

```
~/OpenSearch (main)> git checkout 1.0.0-beta1 -b beta1-release
~/OpenSearch (main)> ./gradlew publishToMavenLocal -Dbuild.version_qualifier=beta1 -Dbuild.snapshot=false
```

On Unix, the local Maven repository is the `~/.m2` folder. For example, the above command produces `~/.m2/repository/org/opensearch/opensearch/1.0.0-beta1/opensearch-1.0.0-beta1.jar`.

### Use OpenSearch from Maven Local in Plugins

Your plugin may have more dependencies than just OpenSearch. For example, [anomaly-detection](https://github.com/opensearch-project/anomaly-detection) requires [OpenSearch](https://github.com/opensearch-project/OpenSearch), [common-utils](https://github.com/opensearch-project/common-utils) and [job-scheduler](https://github.com/opensearch-project/job-scheduler) to be successfully published to Maven local, manually. 

The following trivial build script changes were made to successfully run `./gradlew publishToMavenLocal` in each of these.

* [common-utils#9](https://github.com/opensearch-project/common-utils/pull/9)
* [job-scheduler#11](https://github.com/opensearch-project/job-scheduler/pull/11)
* [anomaly-detection#18](https://github.com/opensearch-project/anomaly-detection/pull/18)

We plan to remove the need for this work-around via [OpenSearch#581](https://github.com/opensearch-project/OpenSearch/issues/581).

## Developing new Plugins for OpenSearch

All OpenSearch plugins must contain the `plugin-descriptor.properties` file. The fields contained in the `plugin-descriptor.properties` file are mentioned [here](https://github.com/opensearch-project/OpenSearch/blob/main/buildSrc/src/main/resources/plugin-descriptor.properties). If using the gradle build system, these fields can be specified in the `build.gradle` file and the build system will create the properties file for us.

### `plugin-descriptor.properties`

The properties in the `plugin-descriptor.properties` file are specified as below:

```
### mandatory elements for all plugins:
#
# 'description': simple summary of the plugin
description=${description}
#
# 'version': plugin's version
version=${version}
#
# 'name': the plugin name
name=${name}
#
# 'classname': the name of the class to load, fully-qualified.
classname=${classname}
#
# 'java.version': version of java the code is built against
# use the system property java.specification.version
# version string must be a sequence of nonnegative decimal integers
# separated by "."'s and may have leading zeros
java.version=${javaVersion}
#
# 'opensearch.version': version of opensearch compiled against
opensearch.version=${opensearchVersion}
### optional elements for plugins:
#
# 'custom.foldername': the custom name of the folder in which the plugin is installed.
custom.foldername=${customFolderName}
#
#  'extended.plugins': other plugins this plugin extends through SPI
extended.plugins=${extendedPlugins}
#
# 'has.native.controller': whether or not the plugin has a native controller
has.native.controller=${hasNativeController}
```

### `build.gradle`

If adding the properties in `build.gradle` file, the gradle plugin will build the `plugin-descriptor.properties` file. Below is an example for the `opensearchplugin` in the `build.gradle`:

```
opensearchplugin {
    name 'opensearch-plugin'
    description 'Example OpenSearch plugin'
    classname 'org.opensearch.example.Plugin'
}
```

### Elements for Plugins

#### Mandatory elements

| Element | Description |
| --- | --- |
| description | simple summary of the plugin |
| version | version of the plugin |
| name | name of the plugin. See [CONVENTIONS](./CONVENTIONS.md#plugin-naming-conventions) for more information. |
| classname | name of the class to load, fully-qualified |
| java.version | version of java the code is built against. use the system property java.specification.version. version string must be a sequence of nonnegative decimal integers separated by "."'s and may have leading zeros |
| opensearch.version | version of OpenSearch the plugin is compiled against |

#### Optional elements

| Element | Description |
| --- | --- |
| custom.foldername | custom folder name for the plugin. This property is available for OpenSearch versions after 1.0.0. If this property is specified, the plugin gets installed in a folder specified in this property. See [opensearch#848](https://github.com/opensearch-project/OpenSearch/pull/848) for more information. |
| extended.plugins | list of plugins this plugin extends |
| has.native.controller | whether or not the plugin has a native controller |

### Notes

Refer to the blog post: [My First Steps in OpenSearch Plugins](https://logz.io/blog/opensearch-plugins/) for steps on creating an OpenSearch Plugin.
