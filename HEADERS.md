<!-- TOC -->
- [Headers](#headers)
  - [License Headers](#license-headers)
  - [Automated License Headers' Checks](#automated-license-headers-checks)
<!-- TOC -->
## Headers

### License Headers
OpenSearch-project uses Apache License v2 for all the projects.  
Link [Apache License v2](LICENSE)

All files in opensearch-project should use OpenSearch SPDX license.  
```
/*
 * Copyright OpenSearch Contributors
 * SPDX-License-Identifier: Apache-2.0
 *
 * The OpenSearch Contributors require contributions made to
 * this file be licensed under the Apache-2.0 license or a
 * compatible open source license.
 *
*/
```

If files have any existing headers keep them and add OpenSearch SPDX license header on top.  
_Example_: [Version.java](https://github.com/opensearch-project/OpenSearch/blob/main/server/src/main/java/org/opensearch/Version.java) file has license header from Elasticsearch.
 The existing license is not removed and OpenSearch SPDX license header is added on top of it.   

### Automated License Headers' Checks

OpenSearch plugins have an inbuilt `:licenseHeaders` Gradle task to perform automated license headers' checks. All plugins should keep this check enabled in their `build.gradle` file.

Plugins can also use [IntelliJ's Copyright](https://www.jetbrains.com/help/idea/copyright.html) feature to generate the license headers. The copyright profile can be shared in the plugin's repository so that license headers are consistently generated for all contributors.

Example: [Enable license headers check + Add IntelliJ copyright profile](https://github.com/opensearch-project/k-NN/pull/41)

For complex multi-module projects and libraries, the `:licenseHeaders` check may not be available outside the plugin module. Such projects should consider using plugins like [Spotless](https://github.com/diffplug/spotless) to validate and/or generate the license headers.

Example: [Spotless configuration to generate the license headers](https://github.com/opensearch-project/performance-analyzer/blob/1357d86e3936c1de8de951e85f2db792e49d8a02/build.gradle#L89-L104)
