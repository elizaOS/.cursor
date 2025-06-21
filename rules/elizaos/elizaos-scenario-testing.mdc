---
description: Comprehensive guide for creating, running, and debugging scenario tests in ElizaOS
globs: **/*
alwaysApply: true
---
# ElizaOS Scenario Testing

This guide explains how to create, run, and debug scenario tests for ElizaOS agents, plugins, and complex workflows using the integrated scenario testing framework.

## Overview

ElizaOS scenario tests are **real-world agent simulations** that:

- Create multi-agent conversations with defined roles and scripts
- Test complex workflows and plugin integrations
- Verify agent behaviors in realistic scenarios
- Use actual AI models, databases, and external services
- Provide LLM-based verification of outcomes
- Support concurrent execution and performance benchmarking

**A scenario should be created to demonstrate every single feature running in a production-like environment.**

## Core Architecture

### Scenario Structure

```typescript
interface Scenario {
  id: UUID;                    // Unique identifier
  name: string;               // Human-readable name
  description: string;        // Purpose and context
  category: string;           // Classification (integration, unit, stress)
  tags: string[];             // Searchable tags
  actors: ScenarioActor[];    // Participating agents
  setup?: ScenarioSetup;      // Environment configuration
  execution?: ScenarioExecution; // Runtime parameters
  verification: ScenarioVerification; // Success criteria
}
```

### Actor Types

```typescript
interface ScenarioActor {
  id: UUID;
  name: string;
  role: 'subject' | 'observer' | 'assistant' | 'adversary';
  bio?: string;               // Agent personality
  system?: string;            // System prompt
  plugins?: string[];         // Required plugins
  script?: ScenarioScript;    // Scripted behaviors
}

// Role Definitions:
// - subject: Primary agent being tested
// - observer: Passive monitoring agent
// - assistant: Helpful secondary agent
// - adversary: Creates challenging scenarios
```

## Creating Scenario Tests

### 1. Basic Scenario Template

```typescript
import type { Scenario } from '../../src/scenario-runner/types.js';

export const myScenario: Scenario = {
  id: 'uuid-here',
  name: 'My Test Scenario',
  description: 'Tests specific agent functionality',
  category: 'integration',
  tags: ['plugin-name', 'feature-area'],

  actors: [
    {
      id: 'subject-uuid',
      name: 'Test Agent',
      role: 'subject',
      bio: 'Agent description',
      system: 'System prompt for the agent',
      plugins: ['@elizaos/plugin-required'],
      script: {
        steps: [
          {
            type: 'message',
            content: 'Hello, I want to test something specific.',
            description: 'Initial test message'
          }
        ]
      }
    }
  ],

  verification: {
    rules: [
      {
        id: 'response-check',
        type: 'llm',
        description: 'Verify agent responds appropriately',
        config: {
          successCriteria: 'Agent should acknowledge the request and take appropriate action',
          priority: 'high'
        }
      }
    ]
  }
};
```

### 2. Multi-Agent Scenario

```typescript
export const multiAgentScenario: Scenario = {
  id: 'multi-agent-uuid',
  name: 'Multi-Agent Collaboration',
  description: 'Tests agents working together',
  category: 'integration',
  tags: ['collaboration', 'multi-agent'],

  actors: [
    {
      id: 'agent1-uuid',
      name: 'Primary Agent',
      role: 'subject',
      plugins: ['@elizaos/plugin-rolodex'],
      script: { steps: [] }
    },
    {
      id: 'agent2-uuid', 
      name: 'Helper Agent',
      role: 'assistant',
      plugins: ['@elizaos/plugin-research'],
      script: {
        steps: [
          {
            type: 'message',
            content: 'I can help with research tasks.',
            description: 'Offer assistance'
          },
          {
            type: 'wait',
            waitTime: 2000,
            description: 'Wait for response'
          },
          {
            type: 'message', 
            content: 'What would you like me to look up?',
            description: 'Follow-up query'
          }
        ]
      }
    }
  ],

  execution: {
    maxDuration: 30000,
    maxSteps: 20
  },

  verification: {
    rules: [
      {
        id: 'collaboration-check',
        type: 'llm',
        description: 'Verify agents collaborate effectively',
        config: {
          successCriteria: 'Agents should work together, share information, and complete the task',
          category: 'collaboration'
        }
      }
    ]
  }
};
```

### 3. Plugin Testing Scenario

```typescript
export const pluginTestScenario: Scenario = {
  id: 'plugin-test-uuid',
  name: 'Plugin Functionality Test',
  description: 'Tests specific plugin actions and behaviors',
  category: 'integration',
  tags: ['plugin-test', 'actions'],

  actors: [
    {
      id: 'plugin-agent-uuid',
      name: 'Plugin Test Agent',
      role: 'subject',
      bio: 'Agent specialized in testing plugin functionality',
      system: 'You are testing a specific plugin. Execute the requested actions and report results.',
      plugins: ['@elizaos/plugin-rolodex'],
      script: {
        steps: [
          {
            type: 'message',
            content: 'I need to create an entity for Alice Chen, CTO at TechCorp.',
            description: 'Test entity creation'
          },
          {
            type: 'action',
            actionName: 'CREATE_ENTITY',
            actionParams: {
              name: 'Alice Chen',
              role: 'CTO',
              organization: 'TechCorp'
            },
            description: 'Execute entity creation action'
          },
          {
            type: 'assert',
            assertion: {
              type: 'contains',
              value: 'entity created',
              description: 'Verify entity was created'
            }
          }
        ]
      }
    }
  ],

  verification: {
    rules: [
      {
        id: 'action-execution',
        type: 'llm', 
        description: 'Verify plugin actions execute correctly',
        config: {
          successCriteria: 'Plugin actions should execute successfully and produce expected results',
          priority: 'critical'
        }
      },
      {
        id: 'data-persistence',
        type: 'llm',
        description: 'Verify data is stored correctly',
        config: {
          successCriteria: 'Created entities should be stored and retrievable',
          category: 'persistence'
        }
      }
    ]
  }
};
```

## Script Step Types

### Message Steps
```typescript
{
  type: 'message',
  content: 'Text to send',
  description: 'What this step does'
}
```

### Wait Steps  
```typescript
{
  type: 'wait',
  waitTime: 2000, // milliseconds
  description: 'Pause for processing'
}
```

### Action Steps
```typescript
{
  type: 'action',
  actionName: 'CREATE_ENTITY',
  actionParams: { name: 'John', role: 'Developer' },
  description: 'Execute specific action'
}
```

### Assertion Steps
```typescript
{
  type: 'assert',
  assertion: {
    type: 'contains' | 'regex' | 'count' | 'timing' | 'custom',
    value: 'expected value',
    description: 'What to verify'
  },
  critical: true // Fail scenario if assertion fails
}
```

### Condition Steps
```typescript
{
  type: 'condition',
  condition: 'message.includes("success")',
  description: 'Conditional logic'
}
```

## Verification Rules

### LLM-Based Verification
All verification uses LLM analysis for intelligent evaluation:

```typescript
{
  id: 'unique-rule-id',
  type: 'llm',
  description: 'Human-readable description',
  config: {
    successCriteria: 'Detailed criteria for success',
    priority: 'high' | 'medium' | 'low',
    category: 'functionality' | 'performance' | 'integration',
    context: { additionalInfo: 'value' }
  },
  weight: 1.0 // Impact on overall score
}
```

### Common Verification Patterns

```typescript
// Response Quality
{
  id: 'response-quality',
  type: 'llm',
  description: 'Verify response quality and relevance',
  config: {
    successCriteria: 'Response should be relevant, helpful, and accurate',
    priority: 'high'
  }
}

// Action Execution
{
  id: 'action-success',
  type: 'llm', 
  description: 'Verify actions execute successfully',
  config: {
    successCriteria: 'All requested actions should complete without errors',
    category: 'functionality'
  }
}

// Data Consistency
{
  id: 'data-consistency',
  type: 'llm',
  description: 'Verify data remains consistent',
  config: {
    successCriteria: 'Data should be stored correctly and remain consistent across operations',
    category: 'persistence'
  }
}
```

## Running Scenario Tests

### Command Line Interface

```bash
# Run all scenarios
elizaos scenario run

# Run specific scenario file
elizaos scenario run path/to/scenario.ts

# Run scenarios in directory
elizaos scenario run -d path/to/scenarios/

# Filter by name or tag
elizaos scenario run -f "Entity Management"
elizaos scenario run -f "plugin-rolodex"

# Verbose output for debugging
elizaos scenario run --verbose

# Benchmark mode
elizaos scenario run --benchmark

# Parallel execution
elizaos scenario run --parallel --max-concurrency 3

# Output to file
elizaos scenario run -o results.json --format json
```

### Environment Setup

Scenarios automatically use:
- **Database**: In-memory PGLite for isolation
- **Environment**: Loads from `.env` file
- **Plugins**: Dynamically loaded as specified
- **Models**: Uses configured AI models

### Output Formats

```bash
# Text format (default)
elizaos scenario run --format text

# JSON format for CI/CD
elizaos scenario run --format json

# HTML report
elizaos scenario run --format html
```

## File Organization

### Project Structure
```
my-project/
├── scenarios/
│   ├── plugin-tests/
│   │   ├── 01-basic-functionality.ts
│   │   ├── 02-integration-test.ts
│   │   └── index.ts
│   ├── stress-tests/
│   │   └── high-load.ts
│   └── integration/
│       └── multi-agent.ts
├── src/
└── package.json
```

### Scenario Export Pattern
```typescript
// scenarios/plugin-tests/my-test.ts
import type { Scenario } from '../../src/scenario-runner/types.js';

export const myTestScenario: Scenario = {
  // scenario definition
};

// scenarios/plugin-tests/index.ts  
export { myTestScenario } from './my-test';
export { otherScenario } from './other-test';
```

## Debugging and Troubleshooting

### Verbose Mode
```bash
elizaos scenario run --verbose
```
Shows:
- Agent initialization details
- Message exchanges
- Action executions
- Verification results
- Error stack traces

### Common Issues

#### Database Migration Errors
```bash
# Clear test database
rm -rf .scenario-test-db
elizaos scenario run
```

#### Plugin Loading Failures
- Verify plugin names in scenario actors
- Check plugin dependencies are installed
- Ensure plugins export correctly

#### Verification Failures
- Check success criteria are realistic
- Review agent responses for edge cases
- Adjust verification rules as needed

### Manual Testing
For debugging specific scenarios:

```typescript
// Create simple test character
{
  "name": "Test Agent",
  "bio": ["I am a test agent"],
  "system": "You are helpful test agent", 
  "plugins": ["@elizaos/plugin-rolodex"]
}
```

## Performance and Benchmarking

### Benchmark Mode
```bash
elizaos scenario run --benchmark
```

Measures:
- Response latency (min, max, avg, p95)
- Token usage (input, output, total)
- Memory operations count
- Action execution counts
- Overall duration

### Parallel Execution
```bash
elizaos scenario run --parallel --max-concurrency 3
```

Benefits:
- Faster test execution
- Load testing capabilities
- Resource utilization insights

## Best Practices

### Scenario Design
1. **Single Responsibility**: Each scenario tests one specific feature
2. **Realistic Data**: Use actual names, companies, scenarios
3. **Clear Verification**: Define specific, measurable success criteria
4. **Error Handling**: Include both positive and negative test cases

### Script Steps
1. **Descriptive Names**: Clear description for each step
2. **Appropriate Timing**: Add waits for async operations
3. **Critical Assertions**: Mark important assertions as critical
4. **Incremental Testing**: Build complexity gradually

### Verification Rules
1. **Specific Criteria**: Avoid vague success criteria
2. **Weighted Importance**: Use weights for rule importance
3. **Category Organization**: Group related verification rules
4. **Context Inclusion**: Provide context for complex rules

### Maintenance
1. **Regular Updates**: Keep scenarios current with features
2. **Dependency Management**: Update plugin references
3. **Performance Monitoring**: Track scenario execution times
4. **Result Analysis**: Review failures and adjust criteria

## Integration with CI/CD

### GitHub Actions Example
```yaml
name: Scenario Tests
on: [push, pull_request]

jobs:
  scenario-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v1
      - run: bun install
      - run: bun run build
      - run: elizaos scenario run --format json --output results.json
      - uses: actions/upload-artifact@v4
        with:
          name: scenario-results
          path: results.json
```

### Continuous Testing
- Run scenarios on every commit
- Compare performance over time  
- Alert on regression failures
- Generate trending reports

## Advanced Features

### Dynamic Scenarios
Generate scenarios programmatically:

```typescript
function createPluginTestScenario(pluginName: string): Scenario {
  return {
    id: `plugin-${pluginName}-test`,
    name: `${pluginName} Plugin Test`,
    // ... rest of scenario
  };
}
```

### Environment-Specific Testing
```typescript
setup: {
  environment: {
    plugins: process.env.NODE_ENV === 'production' 
      ? ['@elizaos/plugin-prod'] 
      : ['@elizaos/plugin-dev']
  }
}
```

### Stress Testing
```typescript
execution: {
  maxDuration: 300000, // 5 minutes
  maxSteps: 1000,      // High interaction count
  stopConditions: [
    {
      type: 'custom',
      value: 'memory_threshold_exceeded',
      description: 'Stop if memory usage too high'
    }
  ]
}
```

Remember: Scenario tests are **real agent simulations** - they execute actual code, use real models, and create real data. Use them to verify your agents work correctly in realistic conditions.
