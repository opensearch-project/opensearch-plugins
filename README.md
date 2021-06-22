- [OpenSearch Plugins](#opensearch-plugins)
  - [Naming Conventions](#naming-conventions)
  - [Managing OpenSearch Plugins](#managing-opensearch-plugins)
  - [Building Plugins with OpenSearch](#building-plugins-with-opensearch)
  - [Upgrading Plugins to work with OpenSearch](#upgrading-plugins-to-work-with-opensearch)
  - [Installing Plugins](#installing-plugins)
  - [Plugin Release Notes](#plugin-release-notes)
  - [Headers](#headers)
  - [Standards](#standards)
- [Contributing](#contributing)
- [License](#license)

## OpenSearch Plugins

This repository contains all the things you ever wanted to know about OpenSearch plugins.

### Naming Conventions

See [CONVENTIONS](CONVENTIONS.md).
### Managing OpenSearch Plugins

We use [meta](https://github.com/mateodelnorte/meta) to manage OpenSearch plugins as a set. See [META](META.md) for more information.

### Building Plugins with OpenSearch

Until OpenSearch and other artifacts are published to Maven Central [OpenSearch#581](https://github.com/opensearch-project/OpenSearch/issues/581), plugins may require building all their dependencies and publishing them to Maven local.

See [BUILDING](BUILDING.md) for details.

### Upgrading Plugins to work with OpenSearch

To upgrade your existing plugins to work with OpenSearch, see [UPGRADING](./UPGRADING.md).

### Installing Plugins

See [INSTALLING](INSTALLING.md) for details.

### Plugin Release Notes

Plugins generally use a standard format for release notes, see [RELEASE_NOTES](./RELEASE_NOTES.md).

### Headers

See [HEADERS](HEADERS.md) for details of headers including licenses.

### Standards

See [STANDARDS](STANDARDS.md) for information regarding plugin standards.

## Contributing

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This project is licensed under the Apache-2.0 License.
