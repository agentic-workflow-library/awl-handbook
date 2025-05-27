# Domain-Specific Agents (DSAs)

## Overview

Domain-Specific Agents (DSAs) are intelligent agents that combine AWLKit's workflow manipulation capabilities with deep domain knowledge in specific scientific areas. Each DSA serves as an expert assistant that can:

- Answer questions about their domain
- Guide users through complex analyses
- Convert and optimize workflows
- Troubleshoot common issues
- Provide best practices

## Key Characteristics

### 1. **Domain Expertise**

Each DSA contains comprehensive knowledge about:
- Scientific concepts in their domain
- Specific tools and pipelines
- Best practices and guidelines
- Common problems and solutions
- Parameter recommendations

### 2. **Interactive Communication**

DSAs can be interacted with through:
- **Command Line Chat**: Interactive terminal sessions
- **Single Questions**: Quick command-line queries
- **Python API**: Programmatic access
- **Jupyter Notebooks**: Rich interactive environments

### 3. **Workflow Intelligence**

Beyond AWLKit's conversion capabilities, DSAs understand:
- What each workflow module does
- How parameters affect results
- Which modules to use for specific analyses
- How to optimize for different scenarios

## Example: SV-Agent

SV-Agent is a DSA for structural variant analysis using GATK-SV:

```python
from sv_agent import create_agent

agent = create_agent()

# Domain expertise
agent.ask("What coverage do I need for detecting small deletions?")
# → "For small deletions (50bp-1kb), you need at least 30x coverage..."

# Workflow guidance
agent.explain("Module00a")
# → Detailed explanation of sample QC module

# Best practices
agent.best_practices("cohort selection")
# → Recommendations for sample selection

# Troubleshooting
agent.troubleshoot("low variant calls")
# → Step-by-step debugging guide
```

## Architecture of a DSA

```
Domain-Specific Agent
├── Core Agent (agent.py)
│   ├── AWLKit integration
│   ├── Pipeline-specific methods
│   └── Configuration handling
├── Knowledge Base (knowledge.py)
│   ├── Domain concepts
│   ├── Tool descriptions
│   ├── FAQs
│   └── Best practices
├── Chat Interface (chat.py)
│   ├── Intent parsing
│   ├── Response generation
│   └── Context management
└── Notebook Interface (notebook.py)
    ├── Rich display
    ├── Interactive widgets
    └── Visualizations
```

## Creating Your Own DSA

To create a domain-specific agent:

1. **Define Your Domain**
   - What scientific area?
   - Which pipelines/tools?
   - Who are the users?

2. **Build Knowledge Base**
   ```python
   class MyKnowledgeBase:
       def __init__(self):
           self.concepts = {...}
           self.tools = {...}
           self.faq = {...}
           self.best_practices = {...}
   ```

3. **Implement Chat Logic**
   ```python
   class MyAgentChat:
       def chat(self, message: str) -> str:
           intent, params = self._parse_intent(message)
           return self._generate_response(intent, params)
   ```

4. **Create Interfaces**
   - CLI with subcommands
   - Python API
   - Notebook interface

## DSA Design Principles

### 1. **Be Helpful**
- Provide clear, actionable answers
- Suggest next steps
- Offer examples

### 2. **Be Domain-Aware**
- Use correct terminology
- Understand common workflows
- Know tool limitations

### 3. **Be Proactive**
- Anticipate user needs
- Suggest best practices
- Warn about common pitfalls

### 4. **Be Accessible**
- Support multiple interaction modes
- Provide different detail levels
- Include visual aids when helpful

## Current DSAs

### SV-Agent
- **Domain**: Structural variant analysis
- **Pipeline**: GATK-SV
- **Expertise**: SV types, detection methods, quality control
- **Repository**: [sv-agent](https://github.com/agentic-workflow-library/sv-agent)

### Future DSAs (Planned)
- **RNASeq-Agent**: RNA sequencing analysis
- **SingleCell-Agent**: Single-cell analysis
- **Proteomics-Agent**: Mass spectrometry workflows
- **Imaging-Agent**: Bioimage analysis pipelines

## Benefits of DSAs

1. **Lower Barrier to Entry**: Users don't need deep pipeline knowledge
2. **Faster Troubleshooting**: Immediate access to solutions
3. **Better Results**: Built-in best practices
4. **Workflow Portability**: Easy conversion between formats
5. **Interactive Learning**: Learn while doing

## Using DSAs Effectively

### For Users
- Start with general questions
- Be specific about your data/goals
- Follow recommended practices
- Use troubleshooting guides

### For Developers
- Contribute domain knowledge
- Report issues and edge cases
- Suggest new features
- Share successful analyses

## Integration with AWL Ecosystem

DSAs integrate seamlessly with:
- **AWLKit**: For workflow conversion
- **CWL/WDL**: For workflow execution
- **Cloud Platforms**: For scalable computing
- **Data Management**: For organized analysis

## Conclusion

Domain-Specific Agents represent the future of scientific computing assistance - combining the power of workflow automation with the intelligence of domain expertise. They make complex analyses accessible while maintaining scientific rigor.