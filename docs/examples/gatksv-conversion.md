# GATK-SV Conversion Example

This example demonstrates converting GATK-SV (Genome Analysis Toolkit for Structural Variants) workflows from WDL to CWL using AWLKit.

## Background

GATK-SV is a comprehensive pipeline for discovering structural variants in whole-genome sequencing data. It consists of multiple modules that work together:

- **Module00a**: Sample-level QC
- **Module00b**: Evidence collection 
- **Module00c**: Batch-level QC
- **Module01-06**: Various stages of SV calling and filtering

## Using sv-agent

The easiest way to convert GATK-SV workflows is using sv-agent:

```python
from sv_agent import SVAgent
from pathlib import Path

# Initialize agent
agent = SVAgent()

# Convert all GATK-SV workflows
results = agent.convert_gatksv_to_cwl(
    output_dir=Path("gatksv_cwl")
)

print(f"Converted {len(results['converted'])} workflows")
print(f"Failed: {len(results['failed'])}")

# Convert specific modules
results = agent.convert_gatksv_to_cwl(
    output_dir=Path("gatksv_cwl"),
    modules=["Module00a", "Module00b"]
)
```

## Direct AWLKit Usage

For more control, use AWLKit directly:

```python
from awlkit import WDLParser, CWLWriter
from pathlib import Path

# Paths
gatksv_dir = Path("gatk-sv/wdl")
output_dir = Path("cwl_output")
output_dir.mkdir(exist_ok=True)

# Parse Module00a
parser = WDLParser()
module00a = parser.parse_file(gatksv_dir / "Module00aSampleQC.wdl")

# GATK-SV uses many imports - handle them
for import_stmt in module00a.imports:
    # Resolve import path
    import_path = gatksv_dir / import_stmt
    
    # Parse imported file
    imported = parser.parse_file(import_path)
    
    # Merge tasks into main workflow
    module00a.tasks.update(imported.tasks)

# Write complete CWL
writer = CWLWriter()
writer.write_file(module00a, output_dir / "Module00aSampleQC.cwl")
```

## Handling GATK-SV Specifics

### 1. Complex Runtime Requirements

GATK-SV tasks often have specific runtime requirements:

```python
# Ensure all tasks have proper resource allocation
for task in workflow.tasks.values():
    if task.runtime:
        # GATK-SV uses preemptible instances
        if hasattr(task.runtime, 'preemptible'):
            # CWL doesn't directly support preemptible
            # Add as custom requirement or hint
            task.metadata['preemptible'] = task.runtime.preemptible
        
        # Ensure memory format is consistent
        if task.runtime.memory:
            # Convert "4 GB" to "4G" format
            task.runtime.memory = task.runtime.memory.replace(" ", "")
```

### 2. Docker Images

Update Docker images if needed:

```python
# Map GATK-SV Docker images to your registry
docker_mapping = {
    "us.gcr.io/broad-dsde-methods/": "myregistry.io/gatksv/",
    "gcr.io/broad-gcp-workflows/": "myregistry.io/broad/"
}

for task in workflow.tasks.values():
    if task.runtime and task.runtime.docker:
        for old_prefix, new_prefix in docker_mapping.items():
            if task.runtime.docker.startswith(old_prefix):
                task.runtime.docker = task.runtime.docker.replace(
                    old_prefix, new_prefix
                )
```

### 3. Scatter Operations

GATK-SV heavily uses scatter for parallel processing:

```python
# Example: Module00b scatters over samples
workflow = parser.parse_file("Module00b.wdl")

# Check scatter operations
for call in workflow.calls:
    if call.scatter:
        print(f"Call {call.call_id} scatters over {call.scatter}")
        # Ensure the scatter variable is properly defined
```

## Complete Example Script

Here's a complete script to convert GATK-SV Module00a:

```python
#!/usr/bin/env python3
"""Convert GATK-SV Module00a to CWL format."""

from pathlib import Path
from awlkit import WDLParser, CWLWriter
from awlkit.utils import WorkflowValidator
import logging

# Setup logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def convert_gatksv_module(module_name: str, gatksv_dir: Path, output_dir: Path):
    """Convert a GATK-SV module to CWL."""
    
    # Create parser and writer
    parser = WDLParser()
    writer = CWLWriter()
    validator = WorkflowValidator()
    
    # Parse main module
    wdl_file = gatksv_dir / "wdl" / f"{module_name}.wdl"
    logger.info(f"Parsing {wdl_file}")
    workflow = parser.parse_file(wdl_file)
    
    # Handle imports
    logger.info(f"Processing {len(workflow.imports)} imports")
    for import_path in workflow.imports:
        full_path = wdl_file.parent / import_path
        if full_path.exists():
            imported = parser.parse_file(full_path)
            workflow.tasks.update(imported.tasks)
        else:
            logger.warning(f"Import not found: {import_path}")
    
    # Validate
    is_valid, errors = validator.validate_workflow(workflow)
    if not is_valid:
        logger.warning(f"Validation issues found: {len(errors)}")
        for error in errors[:5]:  # Show first 5
            logger.warning(f"  {error}")
    
    # Write CWL
    output_file = output_dir / f"{module_name}.cwl"
    writer.write_file(workflow, output_file)
    logger.info(f"Written to {output_file}")
    
    return workflow

# Convert Module00a
if __name__ == "__main__":
    gatksv_dir = Path("gatk-sv")
    output_dir = Path("cwl_output")
    output_dir.mkdir(exist_ok=True)
    
    workflow = convert_gatksv_module("Module00aSampleQC", gatksv_dir, output_dir)
    
    print(f"Converted workflow: {workflow.name}")
    print(f"Tasks: {len(workflow.tasks)}")
    print(f"Inputs: {len(workflow.inputs)}")
    print(f"Outputs: {len(workflow.outputs)}")
```

## Validation

After conversion, validate the CWL files:

```bash
# Install cwltool
pip install cwltool

# Validate
cwltool --validate cwl_output/Module00aSampleQC.cwl
```

## Running Converted Workflows

To run the converted workflows:

1. **Prepare inputs JSON**:
```json
{
  "sample_id": "SAMPLE001",
  "bam_or_cram_file": {
    "class": "File",
    "path": "/data/sample001.bam"
  },
  "reference_fasta": {
    "class": "File",
    "path": "/data/reference.fa"
  }
}
```

2. **Run with cwltool**:
```bash
cwltool Module00aSampleQC.cwl inputs.json
```

## Troubleshooting GATK-SV Conversions

### Common Issues

1. **Missing Imports**: GATK-SV has many interdependent WDL files
   - Solution: Ensure all imports are resolved

2. **Complex Types**: Some GATK-SV types are complex
   - Solution: May need manual type mapping

3. **Custom Functions**: GATK-SV uses custom WDL functions
   - Solution: Implement equivalent CWL expressions

4. **Resource Requirements**: Specific to Google Cloud
   - Solution: Adapt for your compute environment

### Debug Tips

```python
# Enable debug logging
import logging
logging.basicConfig(level=logging.DEBUG)

# Parse with error recovery
try:
    workflow = parser.parse_file("problematic.wdl")
except Exception as e:
    logger.error(f"Parse failed: {e}")
    # Try partial parsing or manual fixes
```

## Next Steps

- Explore other [Examples](../examples/)
- Learn about [Extending AWLKit](../developer-guide/extending-awlkit.md)
- Read the [API Reference](../user-guide/api-reference.md)