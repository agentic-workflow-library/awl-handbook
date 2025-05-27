# AWLKit Architecture

This document describes the architecture and design principles of AWLKit.

## Overview

AWLKit follows a modular architecture designed for extensibility and maintainability. The core principle is to use an Intermediate Representation (IR) that serves as a bridge between different workflow languages.

```
WDL → Parser → IR → Writer → CWL
                ↓
             Analysis
             Validation
             Transformation
```

## Core Components

### 1. Intermediate Representation (IR)

The IR is the heart of AWLKit, providing a language-agnostic representation of workflows.

```
awlkit/ir/
├── __init__.py
├── types.py       # Type system (String, Int, File, Array, etc.)
├── io.py          # Input/Output definitions
├── runtime.py     # Runtime requirements (docker, memory, cpu)
├── task.py        # Task definitions
└── workflow.py    # Workflow and call definitions
```

**Key Classes:**

- `Workflow`: Top-level workflow container
- `Task`: Individual computational task
- `Input/Output`: Parameter definitions with types
- `Runtime`: Resource and environment requirements
- `WorkflowCall`: Task invocation within a workflow

### 2. Parsers

Parsers convert workflow languages into IR.

```
awlkit/parsers/
├── __init__.py
├── wdl_parser.py  # WDL → IR
└── cwl_parser.py  # CWL → IR
```

**Design Principles:**

- **Incremental Parsing**: Support partial parsing for large workflows
- **Error Recovery**: Continue parsing despite errors
- **Import Resolution**: Handle external dependencies
- **Type Inference**: Deduce types when not explicit

### 3. Writers

Writers generate workflow definitions from IR.

```
awlkit/writers/
├── __init__.py
├── cwl_writer.py  # IR → CWL
└── wdl_writer.py  # IR → WDL
```

**Features:**

- **Version Support**: Generate compatible output versions
- **Formatting**: Readable, well-formatted output
- **Validation**: Ensure generated output is valid
- **Optimization**: Simplify redundant constructs

### 4. Converters

High-level conversion interfaces that combine parsers and writers.

```
awlkit/converters/
├── __init__.py
└── wdl_to_cwl.py  # Complete WDL → CWL pipeline
```

### 5. Utilities

Helper functions for analysis and validation.

```
awlkit/utils/
├── __init__.py
├── graph.py       # Dependency analysis
└── validator.py   # Workflow validation
```

## Data Flow

### Conversion Pipeline

1. **Input**: WDL file or string
2. **Lexing/Parsing**: Convert to parse tree
3. **AST Construction**: Build abstract syntax tree
4. **IR Generation**: Create intermediate representation
5. **Transformation**: Apply any modifications
6. **Output Generation**: Write target format
7. **Validation**: Verify output correctness

### Example Flow

```python
# 1. Parse WDL
wdl_content = "task hello { ... }"
parser = WDLParser()
ir = parser.parse_string(wdl_content)

# 2. Transform (optional)
for task in ir.tasks.values():
    task.runtime.docker = "updated:latest"

# 3. Generate CWL
writer = CWLWriter()
cwl_content = writer.write(ir)
```

## Extension Points

### Adding a New Parser

1. Create `new_lang_parser.py` in `parsers/`
2. Implement `parse_file()` and `parse_string()` methods
3. Map language constructs to IR objects
4. Register in `parsers/__init__.py`

### Adding a New Writer

1. Create `new_lang_writer.py` in `writers/`
2. Implement `write()` and `write_file()` methods
3. Map IR objects to language constructs
4. Register in `writers/__init__.py`

### Custom Transformations

```python
class WorkflowOptimizer:
    def optimize(self, workflow: Workflow) -> Workflow:
        # Remove unused inputs
        # Merge compatible tasks
        # Optimize resource allocation
        return optimized_workflow
```

## Design Patterns

### 1. Visitor Pattern

Used for traversing and transforming IR:

```python
class IRVisitor:
    def visit_workflow(self, workflow: Workflow): ...
    def visit_task(self, task: Task): ...
    def visit_input(self, input: Input): ...
```

### 2. Builder Pattern

Used for constructing complex IR objects:

```python
workflow = WorkflowBuilder() \
    .name("my_workflow") \
    .add_input("file", TypeSpec(DataType.FILE)) \
    .add_task(task) \
    .build()
```

### 3. Strategy Pattern

Used for different parsing/writing strategies:

```python
class ParsingStrategy:
    def parse(self, content: str) -> IR: ...

class StrictStrategy(ParsingStrategy): ...
class LenientStrategy(ParsingStrategy): ...
```

## Performance Considerations

### Memory Efficiency

- **Lazy Loading**: Parse imports only when needed
- **Streaming**: Support large workflow files
- **Caching**: Cache parsed imports and type information

### Parsing Performance

- **Regex Caching**: Compile and cache regular expressions
- **Incremental Parsing**: Parse only changed sections
- **Parallel Processing**: Parse independent modules concurrently

## Error Handling

### Error Types

1. **Parse Errors**: Syntax errors in input
2. **Type Errors**: Type mismatches
3. **Validation Errors**: Semantic errors
4. **Conversion Errors**: Unsupported features

### Error Recovery

```python
try:
    workflow = parser.parse_file(path)
except ParseError as e:
    # Collect errors but continue
    errors.append(e)
    workflow = parser.parse_with_recovery(path)
```

## Testing Strategy

### Unit Tests

- Test each component in isolation
- Mock dependencies
- Test edge cases and error conditions

### Integration Tests

- Test complete conversion pipelines
- Use real-world workflows
- Verify output with external validators

### Performance Tests

- Benchmark parsing/writing speed
- Memory usage profiling
- Scalability with large workflows

## Future Enhancements

1. **Additional Languages**: Nextflow, Snakemake support
2. **Workflow Optimization**: Automatic performance improvements
3. **Interactive Mode**: REPL for workflow development
4. **Cloud Integration**: Direct execution on cloud platforms
5. **Visual Tools**: Workflow visualization and debugging