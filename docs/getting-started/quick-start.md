# Quick Start Guide

Get started with AWLKit in 5 minutes!

## Basic Workflow Conversion

### 1. Convert a Simple WDL to CWL

Create a simple WDL file:

```wdl
version 1.0

task count_lines {
    input {
        File input_file
    }
    
    command <<<
        wc -l < ~{input_file}
    >>>
    
    output {
        Int line_count = read_int(stdout())
    }
    
    runtime {
        docker: "ubuntu:latest"
    }
}
```

Convert it to CWL:

```bash
awlkit convert count_lines.wdl count_lines.cwl
```

### 2. Using Python API

```python
from awlkit import WDLToCWLConverter

# Create converter
converter = WDLToCWLConverter()

# Convert file
converter.convert_file("workflow.wdl", "workflow.cwl")

# Or convert from string
wdl_content = """
version 1.0
task hello {
    input { String name }
    command <<< echo "Hello, ~{name}!" >>>
    output { File greeting = stdout() }
}
"""

cwl_content = converter.convert_string(wdl_content)
print(cwl_content)
```

### 3. Analyze Workflow Structure

```python
from awlkit import WDLParser
from awlkit.utils import WorkflowGraphAnalyzer

# Parse workflow
parser = WDLParser()
workflow = parser.parse_file("pipeline.wdl")

# Analyze
analyzer = WorkflowGraphAnalyzer(workflow)
stats = analyzer.get_statistics()

print(f"Total tasks: {stats['total_calls']}")
print(f"Max parallelism: {stats['max_parallelism']}")
print(f"Has cycles: {stats['has_cycles']}")
```

## Common Use Cases

### Convert Multiple Files

```bash
# Convert all WDL files in a directory
awlkit convert-dir wdl_files/ cwl_output/

# Convert with validation
awlkit convert workflow.wdl --validate
```

### Working with GATK-SV

```python
from sv_agent import SVAgent

agent = SVAgent()

# Convert GATK-SV modules
results = agent.convert_gatksv_to_cwl(
    output_dir="cwl_output",
    modules=["Module00a", "Module00b"]
)

print(f"Converted {len(results['converted'])} files")
```

### Custom Conversion

```python
from awlkit import WDLParser, CWLWriter

# Parse WDL
parser = WDLParser()
workflow = parser.parse_file("workflow.wdl")

# Modify workflow (e.g., update docker images)
for task in workflow.tasks.values():
    if task.runtime and task.runtime.docker:
        task.runtime.docker = "myregistry/" + task.runtime.docker

# Write modified workflow
writer = CWLWriter()
writer.write_file(workflow, "modified.cwl")
```

## Command Line Interface

### Basic Commands

```bash
# Show help
awlkit --help

# Convert file
awlkit convert input.wdl output.cwl

# Parse and display structure
awlkit parse workflow.wdl

# Convert directory
awlkit convert-dir src/ dest/
```

### Advanced Options

```bash
# Convert with verbose output
awlkit -v convert workflow.wdl

# Validate conversion
awlkit convert workflow.wdl --validate

# Convert only specific files
awlkit convert-dir src/ dest/ --pattern "Module*.wdl"
```

## What's Next?

- Learn more about [Converting Workflows](../user-guide/converting-workflows.md)
- Explore the [Python API](../user-guide/api-reference.md)
- See [Examples](../examples/) for real-world usage
- Read about [Architecture](../developer-guide/architecture.md) to understand how AWLKit works