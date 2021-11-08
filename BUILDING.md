- [Developing Plugins for OpenSearch](#developing-plugins-for-opensearch)
  - [`plugin-descriptor.properties`](#plugin-descriptorproperties)
- [Building within the OpenSearch Build System](#building-within-the-opensearch-build-system)
  - [Define a Version Based on the OpenSearch Dependency](#define-a-version-based-on-the-opensearch-dependency)
  - [Consume Dynamic Versions of OpenSearch Dependencies](#consume-dynamic-versions-of-opensearch-dependencies)
  - [Always Build SNAPSHOT Artifacts](#always-build-snapshot-artifacts)
  - [Consume Maven Artifacts in Order](#consume-maven-artifacts-in-order)
  - [Include Checksums in Maven Publications](#include-checksums-in-maven-publications)

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
    jcenter()
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
        jcenter()
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