# AWL Handbook

Welcome to the Agentic Workflow Library (AWL) Handbook. This repository contains comprehensive documentation for AWLKit and other AWL projects.

## What is AWLKit?

AWLKit is a universal framework for parsing, converting, and manipulating computational workflow definitions. It serves as the foundation for multiple agents in the AWL ecosystem, providing:

- **Workflow Language Conversion**: Convert between WDL, CWL, Nextflow, and other formats
- **Workflow Analysis**: Understand workflow structure, dependencies, and complexity
- **Extensible Architecture**: Easy to add support for new workflow languages
- **Agent Integration**: Shared framework for all AWL agents

## Quick Start

```bash
# Install AWLKit
pip install awlkit

# Convert a WDL workflow to CWL
awlkit convert workflow.wdl workflow.cwl

# Analyze workflow structure
awlkit analyze workflow.wdl
```

## Documentation Structure

- **[Concepts](docs/concepts/)** - Core concepts and philosophy
  - [Domain-Specific Agents](docs/concepts/domain-specific-agents.md)

- **[Getting Started](docs/getting-started/)** - Installation and basic usage
  - [Installation](docs/getting-started/installation.md)
  - [Quick Start Guide](docs/getting-started/quick-start.md)
  - [Basic Usage](docs/getting-started/basic-usage.md)

- **[User Guide](docs/user-guide/)** - Comprehensive usage documentation
  - [Converting Workflows](docs/user-guide/converting-workflows.md)
  - [CLI Reference](docs/user-guide/cli-usage.md)
  - [Python API](docs/user-guide/api-reference.md)

- **[Developer Guide](docs/developer-guide/)** - For contributors and extenders
  - [Architecture Overview](docs/developer-guide/architecture.md)
  - [Creating Agents](docs/developer-guide/creating-agents.md)
  - [Extending AWLKit](docs/developer-guide/extending-awlkit.md)
  - [Contributing Guidelines](docs/developer-guide/contributing.md)
  - [Testing](docs/developer-guide/testing.md)

- **[Examples](docs/examples/)** - Practical examples and tutorials
  - [Simple Conversion](docs/examples/simple-conversion.md)
  - [GATK-SV Conversion](docs/examples/gatksv-conversion.md)
  - [Custom Converters](docs/examples/custom-converters.md)

- **[Reference](docs/reference/)** - Technical reference
  - [AWLKit Summary](docs/reference/awlkit-summary.md)
  - [WDL Support](docs/reference/wdl-support.md)
  - [CWL Support](docs/reference/cwl-support.md)
  - [Troubleshooting](docs/reference/troubleshooting.md)

## AWL Ecosystem

AWLKit is part of the larger Agentic Workflow Library ecosystem:

- **[sv-agent](https://github.com/agentic-workflow-library/sv-agent)** - Structural variant analysis agent using GATK-SV
- **[awlkit](https://github.com/agentic-workflow-library/awlkit)** - Core workflow conversion framework
- More agents coming soon...

## Contributing

We welcome contributions! Please see our [Contributing Guidelines](docs/developer-guide/contributing.md) for details.

## License

This documentation is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.