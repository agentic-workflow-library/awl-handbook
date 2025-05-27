# Installation

This guide covers installing AWLKit and its dependencies.

## Requirements

- Python 3.8 or higher
- pip (Python package installer)
- Git (for development installation)

## Install from PyPI (Coming Soon)

```bash
pip install awlkit
```

## Install from Source

### For Users

```bash
# Clone the repository
git clone https://github.com/agentic-workflow-library/awlkit.git
cd awlkit

# Install in user mode
pip install .
```

### For Developers

```bash
# Clone with development branch
git clone https://github.com/agentic-workflow-library/awlkit.git
cd awlkit

# Install in development mode with extras
pip install -e ".[dev]"
```

## Install as Part of an Agent

When using AWLKit as part of an AWL agent (like sv-agent):

```bash
# Clone agent with submodules
git clone --recursive https://github.com/agentic-workflow-library/sv-agent.git
cd sv-agent

# Install AWLKit submodule
pip install -e awlkit/

# Install the agent
pip install -e .
```

## Verify Installation

```bash
# Check AWLKit is installed
python -c "import awlkit; print(awlkit.__version__)"

# Check CLI is available
awlkit --version

# Run a simple conversion test
echo 'version 1.0
task hello {
  command { echo "Hello, World!" }
  output { String message = read_string(stdout()) }
}' > test.wdl

awlkit convert test.wdl test.cwl
```

## Dependencies

AWLKit automatically installs these dependencies:

- **lark** (>=1.1.0) - Parsing framework
- **pydantic** (>=2.0) - Data validation
- **networkx** (>=3.0) - Graph analysis
- **ruamel.yaml** (>=0.17) - YAML handling
- **click** (>=8.0) - CLI framework

## Optional Dependencies

### For Development

```bash
pip install -e ".[dev]"
```

Includes:
- pytest - Testing framework
- black - Code formatter
- flake8 - Linting
- mypy - Type checking

### For Documentation

```bash
pip install -e ".[docs]"
```

Includes:
- sphinx - Documentation generator
- sphinx-rtd-theme - ReadTheDocs theme

## Troubleshooting

### Import Errors

If you get import errors after installation:

```bash
# Ensure you're in the right environment
which python
which pip

# Reinstall
pip uninstall awlkit
pip install -e /path/to/awlkit
```

### Permission Errors

If you get permission errors during installation:

```bash
# Install in user space
pip install --user awlkit

# Or use a virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install awlkit
```

## Next Steps

- Read the [Quick Start Guide](quick-start.md)
- Learn [Basic Usage](basic-usage.md)
- Explore [Examples](../examples/)