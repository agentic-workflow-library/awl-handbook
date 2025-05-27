# Creating AWL Agents

This guide explains how to create new agents using AWLKit as the foundation.

## What is an AWL Agent?

An AWL agent is a specialized tool that uses AWLKit to work with specific scientific workflows or pipelines. Examples include:

- **sv-agent**: Works with GATK-SV for structural variant analysis
- **rnaseq-agent**: Could work with RNA-seq pipelines
- **proteomics-agent**: Could handle mass spectrometry workflows

## Agent Architecture

```
your-agent/
├── awlkit/                    # Submodule: shared framework
├── target-pipeline/           # Submodule: the pipeline you're wrapping
├── src/
│   └── your_agent/
│       ├── __init__.py
│       ├── agent.py          # Main agent class
│       ├── main.py           # CLI entry point
│       └── utils/            # Agent-specific utilities
├── tests/
├── examples/
├── pyproject.toml
└── README.md
```

## Step-by-Step Guide

### 1. Set Up Repository

```bash
# Create new repository
mkdir my-agent
cd my-agent
git init

# Add AWLKit as submodule
git submodule add https://github.com/agentic-workflow-library/awlkit.git

# Add your target pipeline (if applicable)
git submodule add https://github.com/example/my-pipeline.git

# Create directory structure
mkdir -p src/my_agent tests examples docs
```

### 2. Create Agent Class

```python
# src/my_agent/agent.py
from pathlib import Path
from typing import Dict, Any, List, Optional

from awlkit import WDLToCWLConverter, WDLParser
from awlkit.utils import WorkflowGraphAnalyzer


class MyAgent:
    """Agent for working with my-pipeline workflows."""
    
    def __init__(self, config: Optional[Dict[str, Any]] = None):
        """Initialize agent with optional configuration."""
        self.name = "MyAgent"
        self.config = config or {}
        self.pipeline_path = Path(__file__).parent.parent.parent / "my-pipeline"
        self.converter = WDLToCWLConverter()
        self.parser = WDLParser()
    
    def convert_pipeline_to_cwl(self, 
                               output_dir: Path,
                               modules: Optional[List[str]] = None) -> Dict[str, Any]:
        """Convert pipeline WDL workflows to CWL format."""
        output_dir.mkdir(parents=True, exist_ok=True)
        results = {"converted": [], "failed": []}
        
        # Get WDL files
        wdl_dir = self.pipeline_path / "workflows"
        wdl_files = self._get_workflow_files(wdl_dir, modules)
        
        # Convert each file
        for wdl_file in wdl_files:
            try:
                cwl_file = output_dir / wdl_file.with_suffix('.cwl').name
                self.converter.convert_file(wdl_file, cwl_file)
                results["converted"].append(str(wdl_file.name))
            except Exception as e:
                results["failed"].append({
                    "file": str(wdl_file.name),
                    "error": str(e)
                })
        
        return results
    
    def analyze_workflow(self, workflow_name: str) -> Dict[str, Any]:
        """Analyze a workflow structure."""
        wdl_file = self.pipeline_path / "workflows" / f"{workflow_name}.wdl"
        
        if not wdl_file.exists():
            raise FileNotFoundError(f"Workflow not found: {wdl_file}")
        
        workflow = self.parser.parse_file(wdl_file)
        analyzer = WorkflowGraphAnalyzer(workflow)
        
        return {
            "name": workflow.name,
            "inputs": len(workflow.inputs),
            "outputs": len(workflow.outputs),
            "tasks": len(workflow.tasks),
            "statistics": analyzer.get_statistics()
        }
    
    def _get_workflow_files(self, wdl_dir: Path, 
                           modules: Optional[List[str]] = None) -> List[Path]:
        """Get list of workflow files to process."""
        if modules:
            files = []
            for module in modules:
                files.extend(wdl_dir.glob(f"*{module}*.wdl"))
            return files
        return list(wdl_dir.glob("*.wdl"))
```

### 3. Create CLI Interface

```python
# src/my_agent/main.py
import argparse
import json
import sys
from pathlib import Path

from my_agent import MyAgent


def main():
    """Main CLI entry point."""
    parser = argparse.ArgumentParser(
        description="MyAgent - AWLKit-based agent for my-pipeline"
    )
    
    subparsers = parser.add_subparsers(dest="command", help="Commands")
    
    # Convert command
    convert_parser = subparsers.add_parser("convert", help="Convert workflows")
    convert_parser.add_argument("--output", type=Path, required=True)
    convert_parser.add_argument("--modules", nargs="+", help="Specific modules")
    
    # Analyze command
    analyze_parser = subparsers.add_parser("analyze", help="Analyze workflow")
    analyze_parser.add_argument("workflow", help="Workflow name")
    
    args = parser.parse_args()
    
    if not args.command:
        parser.print_help()
        sys.exit(1)
    
    agent = MyAgent()
    
    try:
        if args.command == "convert":
            results = agent.convert_pipeline_to_cwl(
                output_dir=args.output,
                modules=args.modules
            )
            print(json.dumps(results, indent=2))
        
        elif args.command == "analyze":
            analysis = agent.analyze_workflow(args.workflow)
            print(json.dumps(analysis, indent=2))
    
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)


if __name__ == "__main__":
    main()
```

### 4. Create Package Configuration

```toml
# pyproject.toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "my-agent"
version = "0.1.0"
description = "AWLKit-based agent for my-pipeline"
readme = "README.md"
requires-python = ">=3.8"
dependencies = [
    "awlkit>=0.1.0",
    "click>=8.0",
    "pyyaml>=6.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "black>=23.0",
    "flake8>=6.0",
]

[project.scripts]
my-agent = "my_agent.main:main"
```

### 5. Add Tests

```python
# tests/test_agent.py
import pytest
from pathlib import Path
from my_agent import MyAgent


class TestMyAgent:
    def test_agent_initialization(self):
        agent = MyAgent()
        assert agent.name == "MyAgent"
        assert agent.pipeline_path.exists()
    
    def test_workflow_analysis(self):
        agent = MyAgent()
        # Mock or use test workflow
        with pytest.raises(FileNotFoundError):
            agent.analyze_workflow("nonexistent")
```

## Best Practices

### 1. Use AWLKit Effectively

- **Don't Duplicate**: Use AWLKit's parsers, writers, and converters
- **Extend When Needed**: Subclass AWLKit components for custom behavior
- **Contribute Back**: If you add general features, contribute to AWLKit

### 2. Agent Design

- **Single Responsibility**: Each agent should focus on one pipeline/domain
- **Configuration**: Support flexible configuration
- **Error Handling**: Provide clear error messages
- **Documentation**: Document pipeline-specific quirks

### 3. Integration

```python
# Allow agent to be used as a library
class MyAgent:
    def process_batch(self, config: Dict[str, Any]) -> Dict[str, Any]:
        """High-level processing method."""
        # Convert workflows if needed
        # Execute pipeline
        # Return results
        pass
    
    def as_cwl_tool(self) -> Dict[str, Any]:
        """Export agent as a CWL tool."""
        return {
            "class": "CommandLineTool",
            "baseCommand": ["my-agent", "process"],
            # ... tool definition
        }
```

## Example: Creating an RNA-seq Agent

```python
# src/rnaseq_agent/agent.py
from awlkit import WDLToCWLConverter
from pathlib import Path


class RNASeqAgent:
    """Agent for RNA-seq analysis pipelines."""
    
    def __init__(self):
        self.name = "RNASeqAgent"
        self.converter = WDLToCWLConverter()
        self.supported_pipelines = {
            "star": "STAR aligner based pipeline",
            "hisat2": "HISAT2 aligner based pipeline",
            "salmon": "Salmon quasi-mapping pipeline"
        }
    
    def prepare_analysis(self, 
                        samples: List[Dict[str, Any]],
                        pipeline: str = "star") -> Dict[str, Any]:
        """Prepare RNA-seq analysis configuration."""
        if pipeline not in self.supported_pipelines:
            raise ValueError(f"Unsupported pipeline: {pipeline}")
        
        # Generate configuration
        config = {
            "pipeline": pipeline,
            "samples": samples,
            "outputs": self._get_expected_outputs(pipeline, samples)
        }
        
        return config
    
    def convert_pipeline(self, pipeline: str, output_dir: Path):
        """Convert RNA-seq pipeline to CWL."""
        # Implementation specific to RNA-seq workflows
        pass
```

## Publishing Your Agent

1. **Documentation**: Create comprehensive docs in awl-handbook
2. **Examples**: Provide clear examples
3. **CI/CD**: Set up automated testing
4. **Registry**: Register with AWL organization
5. **Community**: Announce in AWL community channels

## Next Steps

- Review [sv-agent](https://github.com/agentic-workflow-library/sv-agent) as an example
- Read [AWLKit Architecture](architecture.md)
- Join the AWL community for support