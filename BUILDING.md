- [Developing Plugins for OpenSearch](#developing-plugins-for-opensearch)
  - [`plugin-descriptor.properties`](#plugin-descriptorproperties)
- [Building within the OpenSearch Build System](#building-within-the-opensearch-build-system)
  - [Define a Version Based on the OpenSearch Dependency](#define-a-version-based-on-the-opensearch-dependency)
  - [Consume Dynamic Versions of OpenSearch Dependencies](#consume-dynamic-versions-of-opensearch-dependencies)
  - [Always Build SNAPSHOT Artifacts](#always-build-snapshot-artifacts)
  - [Consume Maven Artifacts in Order](#consume-maven-artifacts-in-order)
  - [Include Checksums in Maven Publications](#include-checksums-in-maven-publications)
  - [Custom Gradle Plugins](#custom-gradle-plugins)
    - [opensearch.pluginzip](#opensearchpluginzip)
        - [Plugin Design](#plugin-design)
        - [Plugin Usage](#plugin-usage)
- [Updating documentation](#updating-documentation)

## Developing Plugins for OpenSearch

The easiest way to get started authoring a new OpenSearch plugin is to follow instruction in [opensearch-project/opensearch-plugin-template-java](https://github.com/opensearch-project/opensearch-plugin-template-java). See also [My First Steps in OpenSearch Plugins](https://logz.io/blog/opensearch-plugins/).

### `plugin-descriptor.properties`

All OpenSearch plugins require a `plugin-descriptor.properties` file. The fields contained in the `plugin-descriptor.properties` file are documented [here](https://github.com/opensearch-project/OpenSearch/blob/main/buildSrc/src/main/resources/plugin-descriptor.properties). When using the gradle build system, these fields can be specified in the `build.gradle` file and the build system will generate the properties file.

Use `opensearchplugin` in `build.gradle` to generate a `plugin-descriptor.properties`. 

```gradle
opensearchplugin {
    name 'opensearch-plugin'
    description 'Example OpenSearch plugin'
    classname 'org.opensearch.example.Plugin'
}
```

## Building within the OpenSearch Build System

OpenSearch builds and releases the core engine at the same time as plugins as one monolithic distribution. This is documented in [the build workflow](https://github.com/opensearch-project/opensearch-build). To accommodate this system plugins are required to make build scripts configurable as follows.

1. Define the plugin version based on the version of OpenSearch.
2. Consume dynamic versions of OpenSearch dependencies with `-Dopensearch.version=`.
3. Support building `-SNAPSHOT` vs. release artifacts with `-Dbuild.snapshot=`, default to always building snapshot.
4. Prioritize Maven local for dependencies, including OpenSearch.
5. Publish Maven checksums.

### Define a Version Based on the OpenSearch Dependency

Do not include `version=` in `build.properties`. Instead, dynamically define it in `build.gradle`. Adjust the version when building a snapshot.

```gradle
buildscript {
    ext {
        opensearch_group = "org.opensearch"
        opensearch_version = System.getProperty("opensearch.version", "1.1.0-SNAPSHOT")
    }
}

allprojects {
    group = 'org.opensearch'

    version = opensearch_version - "-SNAPSHOT" + ".0"
}
```

### Consume Dynamic Versions of OpenSearch Dependencies

OpenSearch uses a 3-digit versioning scheme while plugins have a 4th free digit for patches. Patch the version number to consume dependencies such as common-utils or job-scheduler.

```gradle
buildscript {
    ext {
        // 1.1.0 -> 1.1.0.0, and 1.1.0-SNAPSHOT -> 1.1.0.0-SNAPSHOT
        opensearch_build = opensearch_version.replaceAll(/(\.\d)([^\d]*)$/, '$1.0$2')
        common_utils_version = System.getProperty("common_utils.version", opensearch_build)
        job_scheduler_version = System.getProperty("job_scheduler.version", opensearch_build)
    }
}
```

Use the above versions as follows.

```gradle
dependencies {
    compile "org.opensearch:opensearch:${opensearch_version}"
    compile "org.opensearch:common-utils:${common_utils_version}"
    compileOnly "org.opensearch:opensearch-job-scheduler-spi:${job_scheduler_version}"
}
```

### Always Build SNAPSHOT Artifacts

OpenSearch SNAPSHOT artifacts are [built continuously](https://github.com/opensearch-project/opensearch-build) and made available in [AWS OSS Sonatype Maven](https://aws.oss.sonatype.org/content/repositories/). 

Plugins that depend on OpenSearch artifacts should always consume `-SNAPSHOT` artifacts in all branches in their GHA workflows. While it's typical for Java projects to consume snapshot dependencies early and stabilize on released artifacts, OpenSearch builds and releases the core engine at the same time as plugins as one monolithic distribution, thus switching from `-SNAPSHOT` to released artifacts is not necessary. This substitution is performed by [the build workflow](https://github.com/opensearch-project/opensearch-build) by passing `-Dbuild.snapshot=false` into Gradle build.

```gradle
buildscript {
    ext {
        opensearch_group = "org.opensearch"
        // always default to -SNAPSHOT
        opensearch_version = System.getProperty("opensearch.version", "1.1.0-SNAPSHOT")
    }
}

ext {
    // support -Dbuild.snapshot=false, but default to true
    isSnapshot = "true" == System.getProperty("build.snapshot", "true")
}

allprojects {
    group = 'org.opensearch'

    version = opensearch_version - "-SNAPSHOT" + ".0"

    // append -SNAPSHOT when building snapshot, which is the default 
    if (isSnapshot) {
        version += "-SNAPSHOT"
    }
}
```

Note that signed OpenSearch release artifacts are also available in [Maven Central](https://repo1.maven.org/maven2/org/opensearch/) for components that don't release with the monolithic distribution.

### Consume Maven Artifacts in Order 

Plugins must declare the dependencies for the root projects and all sub-projects in the following order. The first two repositories `mavenLocal()` and `maven(aws.oss.sonatype.org)` are required, while the rest are optional as needed for external dependencies.

```gradle
repositories {
    mavenLocal()
    maven { url "https://aws.oss.sonatype.org/content/repositories/snapshots" }
    mavenCentral()
    maven { url "https://plugins.gradle.org/m2/" }
    ...
}
```

Same applies for dependencies for the build script itself, in-case the `buildscript { }` needs to consume the OpenSearch `build-tools` artifact.

```gradle
buildscript {
    repositories {
        mavenLocal()
        maven { url "https://aws.oss.sonatype.org/content/repositories/snapshots" }
        mavenCentral()
        maven { url "https://plugins.gradle.org/m2/" }
    }
    dependencies {
       classpath "org.opensearch.gradle:build-tools:${opensearch_version}"
       ...
    }
}
```

### Include Checksums in Maven Publications

If your component publishes maven artifacts, use [shadow](https://github.com/johnrengelman/shadow) and publish to a local repository at a non default location.
The maven publish plugin used by gradle will not include checksums when publishing to maven local `~/m2/repository`.

```gradle
publishing {
    repositories {
        maven {
            name = 'staging'
            url = "${rootProject.buildDir}/local-staging-repo"
        }
    }
    publications {
        shadow(MavenPublication) { publication ->
        ...
```
Use `./gradlew publishShadowPublicationToStagingRepository` to produce maven artifacts. When building as part of the monolithic distribution customize `scripts/build.sh` to collect maven artifacts. See [job-scheduler#71](https://github.com/opensearch-project/job-scheduler/pull/82/files) for an example.

## Custom Gradle Plugins

[OpenSearch](https://github.com/opensearch-project/OpenSearch/tree/main/buildSrc) provides a set of custom gradle plugins that avoid boilerplate code for building and testing plugins.

### opensearch.pluginzip

`opensearch.pluginzip` is designed to facilitate OpenSearch plugin distributions (ZIPs) to be available as Apache Maven artifacts which can then be fetched as dependency using Apache Maven dependency management. This plugin identifies the OpenSearch plugin ZIP file, the output of `bundlePlugin` task and publishes to local Apache Maven repository.
[Plugin Code](https://github.com/opensearch-project/OpenSearch/tree/main/buildSrc/src/main/java/org/opensearch/gradle/pluginzip), [Plugin META-INF](https://github.com/opensearch-project/OpenSearch/blob/main/buildSrc/src/main/resources/META-INF/gradle-plugins/opensearch.pluginzip.properties), [Plugin Tests](https://github.com/opensearch-project/OpenSearch/tree/main/buildSrc/src/test/java/org/opensearch/gradle/pluginzip), and [gradle build script examples](https://github.com/opensearch-project/OpenSearch/tree/main/buildSrc/src/test/resources/pluginzip) used by Plugin Tests. 


#### Plugin Design

* `opensearch.pluginzip` is java based code published into build-tools.
* The maven coordinates `groupID`, `version` and `artifcatID` will be inferred from gradle project properties. (Prior to OpenSearch 2.4 the `groupId` was fixed as `org.openserach.plugin` regardless of actual gradle project `group` value or pluginZip publication POM customizations.)
* User can also pass custom POM extensions, that will add desired properties to Apache Maven POM file generated during runtime.
* Once the plugin is added to the `build.gradle` as `apply plugin: 'opensearch.pluginzip'`, this will add a new custom publish task `publishPluginZipPublicationToZipStagingRepository`, the purpose of this task is to publish the ZIP distribution to the Apache Maven local repository (file system).
* Since OpenSearch 2.4 this plugin internally applies both the '[nebula.maven-base-publish](https://plugins.gradle.org/plugin/nebula.maven-base-publish)' and '[maven-publish](https://docs.gradle.org/current/userguide/publishing_maven.html)' plugins which means that it is not necessary to explicitly apply these in your project. 
* The Apache Maven local artifacts could then be published to Apache Maven central/nexus using [CI workflows](https://github.com/opensearch-project/opensearch-build/tree/main/jenkins).
* The plugin will not publish generated JARs (`sourcesJar`, `javadocJar`), this is done on purpose to separate `jars` and `zip` publications.

#### Plugin Usage

1. Ensure that the project uses `bundlePlugin` task that comes from existing [PublishPlugin](https://github.com/opensearch-project/OpenSearch/blob/main/buildSrc/src/main/java/org/opensearch/gradle/PublishPlugin.java). 
`./gradlew tasks --all` should list `bundlePlugin`.

2. Add build-tools dependency to the current gradle project. (If already exists, dont need to re-add again).
Example:
```gradle
buildscript {
    ext {
        opensearch_version = System.getProperty("opensearch.version", "2.0.0-rc1-SNAPSHOT")
    }
    repositories {
        mavenLocal()
    }
    dependencies {
        classpath "org.opensearch.gradle:build-tools:${opensearch_version}"
    }
}
```

3. Add `apply plugin: 'opensearch.pluginzip'` to the build.gradle file.
Once added, this should list the new task `publishPluginZipPublicationToZipStagingRepository`.

4. Run the task `publishPluginZipPublicationToZipStagingRepository` (add it to managed build script `build.sh`)  
```./gradlew publishPluginZipPublicationToZipStagingRepository -Dopensearch.version=$VERSION -Dbuild.snapshot=$SNAPSHOT -Dbuild.version_qualifier=$QUALIFIER```

    Note: The gradle task responsible to generate the distribution zip should be called first before executing `publishPluginZipPublicationToZipStagingRepository`, as the zip file needs to be generated before publishing.

5. To add custom POM extensions to zip publication: 
Example: 
```gradle
publishing {
    publications {
        pluginZip(MavenPublication) { publication ->
            pom {
              name = pluginName
              description = pluginDescription
              licenses {
                license {
                  name = "The Apache License, Version 2.0"
                  url = "http://www.apache.org/licenses/LICENSE-2.0.txt"
                }
              }
              developers {
                developer {
                  name = "OpenSearch"
                  url = "https://github.com/opensearch-project/opensearch-plugin-template-java"
                }
              }
            }
        }
    }
}
```

6. To run `build` task using `./gradlew build` for the entire project without having proper POM reference, add ```startParameter.excludedTaskNames=["validatePluginZipPom"]``` in `settings.gradle` file, this task `validatePluginZipPom` is not required if POM is formatted right with `name`, `description`, `licenses`, `developers` fields. If these fields not added and executed `./gradlew build` would throw an error as

```
name is missing in [/usr/share/opensearch/git/opensearch-plugin-template-java/build/distributions/rename-unspecified.pom]
description is missing in [/usr/share/opensearch/git/opensearch-plugin-template-java/build/distributions/rename-unspecified.pom]
licenses is empty in [/usr/share/opensearch/git/opensearch-plugin-template-java/build/distributions/rename-unspecified.pom]
developers is empty in [/usr/share/opensearch/git/opensearch-plugin-template-java/build/distributions/rename-unspecified.pom]
```

   * **Note:** If gradle project already has the setting as shown in step 5, then its not required to add `startParameter.excludedTaskNames=["validatePluginZipPom"]`. This setting is only required when ran `./gradlew build` without proper POM reference.

7. Exclude the following tasks in `settings.gradle` file, if the build script exists for the plugin and has the tasks `publishToMavenLocal` and `publishAllPublicationsToStagingRepository`, these will also include the tasks `publishPluginZipPublicationToMavenLocal` and `publishPluginZipPublicationToStagingRepository` which is not required to be called with this plugin.
To exclude add the following in `settings.gradle` file ```startParameter.excludedTaskNames=["publishPluginZipPublicationToMavenLocal", "publishPluginZipPublicationToStagingRepository"]```

    Note: If there is a managed `build.sh` file and do not have any publish tasks, then its not required to exclude these tasks, only required if it is calling publish tasks that targets `ALL` repos and `ALL` publications.

8. Quick Example:
    Job-scheduler:
    [build.gradle](https://github.com/opensearch-project/job-scheduler/blob/main/build.gradle)
    [settings.gradle](https://github.com/opensearch-project/job-scheduler/blob/main/settings.gradle)
    [build.sh](https://github.com/opensearch-project/job-scheduler/blob/main/scripts/build.sh)

## Updating Documentation

The available [OpenSearch plugins](https://opensearch.org/docs/latest/install-and-configure/install-opensearch/plugins/#available-plugins) and [OpenSearch Dashboards plugins](https://opensearch.org/docs/latest/install-and-configure/install-dashboards/plugins/#available-plugins) are listed on the [documentation website](https://opensearch.org/docs/latest/). 

If you create a new plugin or update the name of an existing one, make sure to update the list of plugins. You can either edit the [OpenSearch plugins doc page](https://github.com/opensearch-project/documentation-website/blob/main/_install-and-configure/install-opensearch/plugins.md) or [OpenSearch Dashboards plugins doc page](https://github.com/opensearch-project/documentation-website/blob/main/_install-and-configure/install-dashboards/plugins.md) in the documentation repo, or [open a documentation issue](https://github.com/opensearch-project/documentation-website/issues/new/choose).