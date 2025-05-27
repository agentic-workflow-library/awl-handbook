# AWLKit Reference Summary

Quick reference for AWLKit components and APIs.

## Core Classes

### Workflow IR

```python
from awlkit import Workflow, Task, Input, Output, Runtime

# Create a workflow
workflow = Workflow(name="my_workflow")

# Create a task
task = Task(
    name="process_data",
    command="python process.py ${input_file}",
    inputs=[Input(name="input_file", type_spec=TypeSpec(DataType.FILE))],
    outputs=[Output(name="result", type_spec=TypeSpec(DataType.FILE))],
    runtime=Runtime(docker="python:3.9", memory="4G", cpu=2)
)

# Add task to workflow
workflow.add_task(task)
```

### Type System

```python
from awlkit.ir.types import TypeSpec, DataType

# Basic types
string_type = TypeSpec(base_type=DataType.STRING)
file_type = TypeSpec(base_type=DataType.FILE)
int_type = TypeSpec(base_type=DataType.INT)

# Optional types
optional_string = TypeSpec(base_type=DataType.STRING, optional=True)

# Array types
file_array = TypeSpec(
    base_type=DataType.ARRAY,
    item_type=TypeSpec(base_type=DataType.FILE)
)

# Map types
string_map = TypeSpec(
    base_type=DataType.MAP,
    key_type=TypeSpec(base_type=DataType.STRING),
    value_type=TypeSpec(base_type=DataType.STRING)
)
```

## Parsers

### WDL Parser

```python
from awlkit import WDLParser

parser = WDLParser()

# Parse from file
workflow = parser.parse_file("workflow.wdl")

# Parse from string
wdl_content = """
version 1.0
task hello {
    input { String name }
    command <<< echo "Hello, ~{name}!" >>>
    output { File greeting = stdout() }
}
"""
task = parser.parse_string(wdl_content)
```

### CWL Parser

```python
from awlkit import CWLParser

parser = CWLParser()

# Parse CWL file
tool = parser.parse_file("tool.cwl")

# Parse from dict
cwl_dict = {
    "class": "CommandLineTool",
    "inputs": {"input": "File"},
    "outputs": {"output": "File"}
}
tool = parser.parse_dict(cwl_dict)
```

## Writers

### CWL Writer

```python
from awlkit import CWLWriter

writer = CWLWriter()

# Write to string
cwl_content = writer.write(workflow)

# Write to file
writer.write_file(workflow, "output.cwl")
```

### WDL Writer

```python
from awlkit import WDLWriter

writer = WDLWriter(version="1.0")

# Write to string
wdl_content = writer.write(task)

# Write to file
writer.write_file(workflow, "output.wdl")
```

## Converters

### WDL to CWL

```python
from awlkit import WDLToCWLConverter

converter = WDLToCWLConverter()

# Simple conversion
converter.convert_file("input.wdl", "output.cwl")

# Convert string
cwl_content = converter.convert_string(wdl_content)

# Batch conversion
converter.convert_directory(
    wdl_dir=Path("wdl_files/"),
    cwl_dir=Path("cwl_output/"),
    recursive=True
)
```

## Utilities

### Workflow Analysis

```python
from awlkit.utils import WorkflowGraphAnalyzer

analyzer = WorkflowGraphAnalyzer(workflow)

# Get execution order
order = analyzer.get_execution_order()

# Find dependencies
deps = analyzer.get_dependencies("task_name")

# Get statistics
stats = analyzer.get_statistics()
print(f"Max parallelism: {stats['max_parallelism']}")
print(f"Has cycles: {stats['has_cycles']}")
```

### Validation

```python
from awlkit.utils import WorkflowValidator

validator = WorkflowValidator()

# Validate workflow
is_valid, errors = validator.validate_workflow(workflow)

if not is_valid:
    for error in errors:
        print(f"{error.level}: {error.message}")

# Validate task
is_valid, errors = validator.validate_task(task)
```

## CLI Commands

```bash
# Convert file
awlkit convert input.wdl output.cwl

# Convert with validation
awlkit convert input.wdl output.cwl --validate

# Convert directory
awlkit convert-dir wdl_dir/ cwl_dir/

# Parse and display
awlkit parse workflow.wdl

# Show version
awlkit --version

# Verbose mode
awlkit -v convert input.wdl output.cwl
```

## Common Patterns

### Custom Converter

```python
from awlkit.converters import WDLToCWLConverter

class CustomConverter(WDLToCWLConverter):
    def convert_file(self, wdl_path, cwl_path):
        # Parse
        workflow = self.parser.parse_file(wdl_path)
        
        # Custom modifications
        for task in workflow.tasks.values():
            # Update all docker images
            if task.runtime and task.runtime.docker:
                task.runtime.docker = self.update_docker(task.runtime.docker)
        
        # Write
        self.writer.write_file(workflow, cwl_path)
```

### Error Handling

```python
from awlkit import WDLParser
import logging

# Setup logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

parser = WDLParser()

try:
    workflow = parser.parse_file("workflow.wdl")
except ParseError as e:
    logger.error(f"Parse error: {e}")
    # Handle error
except Exception as e:
    logger.error(f"Unexpected error: {e}")
    raise
```

### Workflow Modification

```python
# Parse workflow
workflow = parser.parse_file("original.wdl")

# Modify inputs
workflow.inputs.append(
    Input(
        name="new_param",
        type_spec=TypeSpec(base_type=DataType.STRING),
        default="default_value"
    )
)

# Update task commands
for task in workflow.tasks.values():
    task.command = task.command.replace("old_path", "new_path")

# Write modified workflow
writer.write_file(workflow, "modified.wdl")
```

## Type Conversions

| WDL Type | CWL Type | AWLKit IR |
|----------|----------|-----------|
| String | string | DataType.STRING |
| Int | int | DataType.INT |
| Float | float | DataType.FLOAT |
| Boolean | boolean | DataType.BOOLEAN |
| File | File | DataType.FILE |
| Array[T] | array of T | DataType.ARRAY |
| Map[K,V] | record | DataType.MAP |
| T? | ["null", T] | TypeSpec(optional=True) |

## Environment Variables

```bash
# Set log level
export AWLKIT_LOG_LEVEL=DEBUG

# Set default output format
export AWLKIT_OUTPUT_FORMAT=cwl

# Cache directory for parsed files
export AWLKIT_CACHE_DIR=/tmp/awlkit_cache
```

## Quick Tips

1. **Always validate** converted workflows with external tools
2. **Use logging** to debug parsing issues
3. **Handle imports** explicitly for complex workflows
4. **Test incrementally** with simple workflows first
5. **Check compatibility** between WDL and CWL features