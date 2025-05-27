# Converting Workflows

This guide covers converting workflows between different languages using AWLKit.

## Basic Conversion

### WDL to CWL

The most common conversion is from WDL (Workflow Description Language) to CWL (Common Workflow Language).

```python
from awlkit import WDLToCWLConverter

converter = WDLToCWLConverter()
converter.convert_file("input.wdl", "output.cwl")
```

### Command Line

```bash
# Simple conversion
awlkit convert workflow.wdl workflow.cwl

# With validation
awlkit convert workflow.wdl workflow.cwl --validate

# Verbose output
awlkit -v convert workflow.wdl workflow.cwl
```

## Handling Complex Workflows

### Workflows with Imports

Many WDL workflows import other WDL files. AWLKit handles these automatically:

```python
from awlkit import WDLToCWLConverter
from pathlib import Path

converter = WDLToCWLConverter()

# AWLKit will resolve imports relative to the WDL file location
converter.convert_file(
    Path("workflows/main.wdl"),
    Path("output/main.cwl")
)
```

### Manual Import Resolution

For more control over import resolution:

```python
from awlkit import WDLParser, CWLWriter

parser = WDLParser()

# Parse main workflow
main_wf = parser.parse_file("main.wdl")

# Manually parse and merge imports
for import_path in main_wf.imports:
    imported = parser.parse_file(import_path)
    main_wf.tasks.update(imported.tasks)

# Convert to CWL
writer = CWLWriter()
writer.write_file(main_wf, "complete.cwl")
```

### Scatter Operations

AWLKit converts WDL scatter operations to CWL scatter:

**WDL:**
```wdl
scatter (sample in samples) {
    call process_sample { input: sample = sample }
}
```

**Generated CWL:**
```yaml
steps:
  process_sample:
    run: '#process_sample'
    scatter: sample
    in:
      sample: samples
```

## Customizing Conversion

### Modifying During Conversion

```python
from awlkit import WDLParser, CWLWriter

# Parse WDL
parser = WDLParser()
workflow = parser.parse_file("workflow.wdl")

# Modify before conversion
for task in workflow.tasks.values():
    # Update Docker images
    if task.runtime and task.runtime.docker:
        task.runtime.docker = task.runtime.docker.replace(
            "old-registry", "new-registry"
        )
    
    # Add missing memory requirements
    if task.runtime and not task.runtime.memory:
        task.runtime.memory = "4G"

# Write CWL
writer = CWLWriter()
writer.write_file(workflow, "modified.cwl")
```

### Custom Converters

Create specialized converters for your needs:

```python
from awlkit.converters import WDLToCWLConverter

class MyCustomConverter(WDLToCWLConverter):
    def convert_file(self, wdl_path, cwl_path):
        # Parse
        workflow = self.parser.parse_file(wdl_path)
        
        # Apply custom transformations
        self.add_metadata(workflow)
        self.optimize_resources(workflow)
        
        # Write
        self.writer.write_file(workflow, cwl_path)
    
    def add_metadata(self, workflow):
        """Add custom metadata to workflow."""
        workflow.metadata = {
            "author": "My Team",
            "version": "1.0"
        }
    
    def optimize_resources(self, workflow):
        """Optimize resource allocations."""
        for task in workflow.tasks.values():
            if task.runtime and task.runtime.cpu:
                # Ensure even CPU counts
                task.runtime.cpu = (task.runtime.cpu + 1) // 2 * 2
```

## Batch Conversion

### Convert Directory

```bash
# Convert all WDL files in a directory
awlkit convert-dir wdl_workflows/ cwl_output/

# With pattern matching
awlkit convert-dir wdl_workflows/ cwl_output/ --pattern "Module*.wdl"

# Non-recursive
awlkit convert-dir wdl_workflows/ cwl_output/ --no-recursive
```

### Python API for Batch

```python
from awlkit import WDLToCWLConverter
from pathlib import Path

converter = WDLToCWLConverter()

# Convert entire directory
converter.convert_directory(
    wdl_dir=Path("workflows/"),
    cwl_dir=Path("cwl_output/"),
    recursive=True
)

# With error handling
wdl_files = Path("workflows/").glob("**/*.wdl")
results = {"success": [], "failed": []}

for wdl_file in wdl_files:
    try:
        cwl_file = Path("cwl_output") / wdl_file.with_suffix(".cwl").name
        converter.convert_file(wdl_file, cwl_file)
        results["success"].append(str(wdl_file))
    except Exception as e:
        results["failed"].append({
            "file": str(wdl_file),
            "error": str(e)
        })

print(f"Converted: {len(results['success'])}")
print(f"Failed: {len(results['failed'])}")
```

## Validation

### Post-Conversion Validation

```python
from awlkit import WDLToCWLConverter
import subprocess

converter = WDLToCWLConverter()

# Convert
workflow = converter.convert_file("input.wdl", "output.cwl")

# Validate with cwltool
result = subprocess.run(
    ["cwltool", "--validate", "output.cwl"],
    capture_output=True,
    text=True
)

if result.returncode == 0:
    print("✓ Valid CWL generated")
else:
    print(f"✗ Validation failed: {result.stderr}")
```

### Built-in Validation

```python
from awlkit.utils import WorkflowValidator

validator = WorkflowValidator()
is_valid, errors = validator.validate_workflow(workflow)

if not is_valid:
    for error in errors:
        print(f"{error.level}: {error.message}")
```

## Handling Unsupported Features

Some WDL features may not have direct CWL equivalents:

```python
from awlkit import WDLParser
import logging

# Enable warnings for unsupported features
logging.basicConfig(level=logging.WARNING)

parser = WDLParser()
try:
    workflow = parser.parse_file("complex.wdl")
except NotImplementedError as e:
    print(f"Unsupported feature: {e}")
    # Handle gracefully or use alternative approach
```

## Best Practices

1. **Always Validate**: Use external validators to ensure correctness
2. **Test Incrementally**: Convert and test small workflows first
3. **Preserve Semantics**: Ensure the converted workflow behaves identically
4. **Document Changes**: Note any manual modifications needed
5. **Version Control**: Track both source and converted workflows

## Troubleshooting

### Common Issues

1. **Import Errors**: Ensure all imported files are accessible
2. **Type Mismatches**: Some type conversions may need manual adjustment
3. **Runtime Differences**: Docker images and resource requirements may need updates
4. **Feature Gaps**: Some advanced features may require workarounds

### Debug Mode

```python
import logging

# Enable debug logging
logging.basicConfig(level=logging.DEBUG)

converter = WDLToCWLConverter()
converter.convert_file("problematic.wdl", "output.cwl")
```

## Next Steps

- Learn about the [Python API](api-reference.md)
- See [Examples](../examples/) for real-world conversions
- Read about [Extending AWLKit](../developer-guide/extending-awlkit.md)