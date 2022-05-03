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

## Developing Plugins for OpenSearch

The easiest way to get started authoring a new OpenSearch plugin is to follow instruction in [opensearch-project/opensearch-plugin-template-java](https://github.com/opensearch-project/opensearch-plugin-template-java). See also [My First Steps in OpenSearch Plugins](https://logz.io/blog/opensearch-plugins/).

### `plugin-descriptor.properties`

All OpenSearch plugins require a `plugin-descriptor.properties` file. The fields contained in the `plugin-descriptor.properties` file are documented [here](https://github.com/opensearch-project/OpenSearch/blob/main/buildSrc/src/main/resources/plugin-descriptor.properties). When using the gradle build system, these fields can be specified in the `build.gradle` file and the build system will generate the properties file.

Use `opensearchplugin` in `build.gradle` to generate a `plugin-descriptor.properties`. 

```
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

```
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

This plugin identifies the generated zip file distribution which is the ouput of `bundlePlugin` task and publishes to local maven repo with standardized maven coordinates.
[Plugin Code](https://github.com/opensearch-project/OpenSearch/tree/main/buildSrc/src/main/java/org/opensearch/gradle/pluginzip), [Plugin Tests](https://github.com/opensearch-project/OpenSearch/tree/main/buildSrc/src/test/java/org/opensearch/gradle/pluginzip), [Plugin META-INF](https://github.com/opensearch-project/OpenSearch/blob/main/buildSrc/src/main/resources/META-INF/gradle-plugins/opensearch.pluginzip.properties)


#### Plugin Design

* `opensearch.pluginzip` is java based code published into build-tools.
* The maven coordinates groupID is fixed as `org.openserach.plugin`, `version` and `artifcatID` will be inferred from gradle project properties.
* User can also pass custom POM extensions, that will add desired xml to the zip maven POM file geneaterd during runtime.
* Once the plugin is added to the `build.gradle` as `apply plugin: 'opensearch.pluginzip'`, this will add a new custom publish task `publishPluginZipPublicationToZipStagingRepository`, this task will publish the zip distribution to the maven local (file system).
* The maven local artifacts can then be published to maven central/nexus.
* The plugin will not add jars generated by tasks `sourcesJar` and `javadocJar`, this is done in purpose to exclude `jars` for `zip` publications.

#### Plugin Usage

1. The Key requirement for this plugin to work is that the zip should be generated by `bundlePlugin` task that comes from existing [PublishPlugin](https://github.com/opensearch-project/OpenSearch/blob/main/buildSrc/src/main/java/org/opensearch/gradle/PublishPlugin.java). 
`./gradlew tasks ` should list `bundlePlugin`.

2. Add build-tools dependency to the current gradle project. (If already exists, dont need to re-add again)
Example:
```
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

4. Run the task `publishPluginZipPublicationToZipStagingRepository` (add it to managed build script build.sh)  
```./gradlew publishPluginZipPublicationToZipStagingRepository -Dopensearch.version=$VERSION -Dbuild.snapshot=$SNAPSHOT -Dbuild.version_qualifier=$QUALIFIER```

    Note: The gradle assemble task `./gradlew assemble` should be called first before calling `publishPluginZipPublicationToZipStagingRepository`, as zip file need to be generated first before publishing.

5. To add custom POM extensions: 
Example: 
```
allprojects {
    project.ext.licenseName = 'The Apache Software License, Version 2.0'
    project.ext.licenseUrl = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
    publishing {
            repositories {
                maven {
                    name = 'staging'
                    url = "${rootProject.buildDir}/local-staging-repo"
                }
            }
            publications {
                all {
                    pom.withXml { XmlProvider xml ->
                        Node node = xml.asNode()
                        node.appendNode('inceptionYear', '2021')

                        Node license = node.appendNode('licenses').appendNode('license')
                        license.appendNode('name', project.licenseName)
                        license.appendNode('url', project.licenseUrl)

                        Node developer = node.appendNode('developers').appendNode('developer')
                        developer.appendNode('name', 'OpenSearch')
                        developer.appendNode('url', 'https://github.com/opensearch-project/security')
                    }
                }
            }
        }
}
```
6. To run `build` task using `./gradlew build` for the entire project instead of running specific tasks, add the line ```startParameter.excludedTaskNames=["validatePluginZipPom"]``` in `settings.gradle` file, this task `validatePluginZipPom` is not required for this plugin.

    Note: If gradle project is executed by specific task, then its not required to add ```startParameter.excludedTaskNames=["validatePluginZipPom"]```, only required when ran `./gradlew build`. 

7. Exclude the following tasks in `settings.gradle` file, if the build script exists for the plugin and has the tasks `publishToMavenLocal` and `publishAllPublicationsToStagingRepository`, these will also include the tasks `publishPluginZipPublicationToMavenLocal` and `publishPluginZipPublicationToStagingRepository` which is not required to be called with this plugin.
To exclude add the following in `settings.gradle` file ```startParameter.excludedTaskNames=["publishPluginZipPublicationToMavenLocal", "publishPluginZipPublicationToStagingRepository"]```

    Note: If there is a managed `build.sh` file and do not have any publish tasks, then its not required to exlcude these tasks, only required if it is calling publish tasks that targets ALL repos and ALL publications.

8. Quick Example:
    Job-scheduler:
    [build.gradle](https://github.com/prudhvigodithi/job-scheduler/blob/gradleplugin/build.gradle#L33)
    [settings.gradle](https://github.com/prudhvigodithi/job-scheduler/blob/gradleplugin/settings.gradle#L13)
    [build.sh](https://github.com/prudhvigodithi/job-scheduler/blob/gradleplugin/scripts/build.sh#L80)

