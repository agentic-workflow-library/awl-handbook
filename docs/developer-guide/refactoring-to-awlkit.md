# Refactoring Domain Agents to Use AWLKit

This guide explains how to refactor domain-specific agents to leverage AWLKit's generic functionality while maintaining domain expertise.

## Overview

AWLKit should provide the generic infrastructure for building domain agents:
- Base agent classes with common patterns
- LLM integration framework
- Chat and notebook interfaces
- Workflow analysis tools
- Batch processing utilities

Domain agents (like sv-agent) should focus solely on:
- Domain-specific knowledge
- Custom business logic
- Specialized handlers
- Domain-specific configurations

## Example: Refactoring sv-agent

### Current Structure

sv-agent currently contains both generic and domain-specific code:

```
sv-agent/
├── src/sv_agent/
│   ├── agent.py         # Mix of generic + SV-specific
│   ├── chat.py          # Mix of generic chat + SV handlers
│   ├── knowledge.py     # Pure SV domain knowledge ✓
│   ├── llm/            # Generic LLM framework
│   └── notebook.py      # Mix of generic + SV-specific
```

### Target Structure

After refactoring:

```
sv-agent/
├── src/sv_agent/
│   ├── agent.py         # Only SV-specific overrides
│   ├── chat.py          # Only SV chat handlers
│   ├── knowledge.py     # Pure SV domain knowledge ✓
│   └── notebook.py      # Only SV notebook customizations

awlkit/
├── src/awlkit/
│   ├── agents/          # Generic agent framework
│   ├── llm/            # LLM integration
│   └── utils/          # Workflow analysis, batch processing
```

## Step-by-Step Refactoring

### 1. Move LLM Framework to AWLKit

The entire LLM integration is generic and should move to AWLKit:

```python
# Move from sv_agent/llm/ to awlkit/llm/
awlkit/
├── llm/
│   ├── __init__.py
│   ├── providers.py     # All LLM provider classes
│   ├── memory.py        # ConversationMemory
│   └── utils.py         # Detection, formatting utilities
```

### 2. Create AWLKit Base Classes

```python
# awlkit/agents/base.py
from abc import ABC, abstractmethod

class Agent(ABC):
    """Base class for all AWL agents."""
    
    def __init__(self, config=None):
        self.config = config or {}
    
    def process_batch(self, batch_config):
        """Generic batch processing."""
        self._validate_batch_config(batch_config)
        return self._execute_batch(batch_config)
    
    @abstractmethod
    def _execute_batch(self, batch_config):
        """Domain-specific batch execution."""
        pass
    
    def analyze_workflow(self, workflow_path):
        """Generic workflow analysis."""
        from awlkit.utils import WorkflowAnalyzer
        analyzer = WorkflowAnalyzer()
        return analyzer.analyze(workflow_path)
```

```python
# awlkit/agents/chat.py
class ChatInterface:
    """Generic chat interface for agents."""
    
    def __init__(self, agent, llm_provider=None):
        self.agent = agent
        self.llm = llm_provider or self._detect_provider()
        self.handlers = {}
        self.memory = ConversationMemory()
    
    def register_handler(self, intent, handler):
        """Register domain-specific handlers."""
        self.handlers[intent] = handler
    
    def chat_loop(self):
        """Generic chat loop."""
        # Implementation of generic chat flow
```

### 3. Update Domain Agent

```python
# sv_agent/agent.py
from awlkit.agents import Agent

class SVAgent(Agent):
    """SV-specific agent implementation."""
    
    def __init__(self, config=None):
        super().__init__(config)
        self.knowledge = SVKnowledgeBase()
        self.gatksv_path = self._find_gatksv_path()
    
    def _execute_batch(self, batch_config):
        """SV-specific batch processing."""
        # Domain-specific implementation
```

```python
# sv_agent/chat.py
from awlkit.agents import ChatInterface

class SVAgentChat(ChatInterface):
    """SV-specific chat interface."""
    
    def __init__(self, agent):
        super().__init__(agent)
        self._register_sv_handlers()
    
    def _register_sv_handlers(self):
        """Register SV-specific handlers."""
        self.register_handler('explain', self._handle_sv_explain)
        self.register_handler('recommend', self._handle_sv_recommend)
        self.register_handler('troubleshoot', self._handle_sv_troubleshoot)
```

## Benefits of This Approach

### For AWLKit
- Becomes a true framework for building domain agents
- Reusable components across all domain agents
- Consistent patterns and interfaces
- Easier to maintain and test generic functionality

### For Domain Agents
- Focused solely on domain expertise
- Smaller, cleaner codebases
- Inherit improvements from AWLKit automatically
- Easier to understand and maintain

### For Users
- Consistent interfaces across all agents
- Better documentation and examples
- More reliable and tested components
- Easier to create new domain agents

## Migration Checklist

When refactoring a domain agent:

- [ ] Identify generic vs domain-specific code
- [ ] Move generic utilities to awlkit/utils/
- [ ] Move LLM integration to awlkit/llm/
- [ ] Create base classes in awlkit/agents/
- [ ] Update domain agent to inherit from bases
- [ ] Move only domain handlers and knowledge
- [ ] Update imports and dependencies
- [ ] Test thoroughly with existing tests
- [ ] Update documentation

## Creating New Domain Agents

With AWLKit providing the infrastructure:

```python
# my_agent/agent.py
from awlkit.agents import Agent

class MyDomainAgent(Agent):
    def __init__(self):
        super().__init__()
        self.knowledge = MyDomainKnowledge()
    
    def _execute_batch(self, batch_config):
        # Domain-specific logic only
```

This makes creating new domain agents much simpler and more focused on the actual domain expertise rather than infrastructure.