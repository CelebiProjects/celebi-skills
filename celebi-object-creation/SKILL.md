---
name: celebi-object-creation
description: Creation and management of Celebi objects including tasks, algorithms, and directories. Use when working with Celebi analysis projects to create new objects with celebi-cli create-task and celebi-cli create-algorithm commands, link algorithms to tasks with celebi-cli add-algorithm, or manage existing Celebi object configurations and relationships.
---

# Celebi Object Creation

## Overview

This skill provides comprehensive guidance for creating and managing Celebi objects including tasks, algorithms, and directories. Celebi follows a strict Object-Oriented File System model where each object type has specific requirements and constraints.

## Quick Start

### Basic Object Creation Commands

1. **Create a new Task**:
   ```bash
   celebi-cli create-task [task-name]
   ```
   Creates a Task object directory with `.celebi/config.json` and `README.md`.

2. **Create a new Algorithm**:
   ```bash
   celebi-cli create-algorithm [algorithm-name]
   ```
   Creates an Algorithm object directory with `.celebi/config.json` and `README.md`.

3. **Link Algorithm to Task**:
   ```bash
   celebi-cli add-algorithm [path/to/algorithm]
   ```
   Links an existing Algorithm to the current Task directory.

4. **Create other object types**:
   ```bash
   celebi-cli create-data [name]      # Create a data object
   celebi-cli mkdir [name]            # Create a directory object
   ```
   Data objects represent external datasets; directories organize tasks/algorithms.

## Understanding Celebi Object Types

### Task Objects ("The What")
- **Purpose**: Executable instances of algorithms
- **Contains**: Parameters, inputs, dependencies, execution context
- **Rules**: Must map to exactly one Algorithm via `code` alias
- **Creation**: `celebi-cli create-task [name]`

### Algorithm Objects ("The How")
- **Purpose**: Stateless, reusable templates for computation
- **Contains**: Definitions, environments, commands, logic
- **Rules**: Stateless, don't "know" about workflow
- **Creation**: `celebi-cli create-algorithm [name]`

### Directory Objects
- **Purpose**: Organizational units to group related objects
- **Constraint**: Cannot contain nested Directory objects
- **Use**: Group related Tasks or Algorithms

### Data Objects ("The Source")
- **Purpose**: Represent external datasets or static data
- **Contains**: Metadata about data sources, file references
- **Rules**: Can be referenced as inputs by tasks
- **Creation**: `celebi-cli create-data [name]`

## Object Creation Workflows

### Workflow 1: Creating a Complete Analysis Pipeline
1. **Create Algorithm** (the computation logic):
   ```bash
   celebi-cli create-algorithm data-processor
   ```
2. **Create Task** (the execution instance):
   ```bash
   celebi-cli create-task process-experiment-data
   ```
3. **Link Algorithm to Task**:
   ```bash
   cd process-experiment-data
   celebi-cli add-algorithm ../data-processor
   ```
4. **Configure Task** (using appropriate `celebi-cli *` commands):
   - Set parameters
   - Define inputs/outputs
   - Declare dependencies

### Workflow 2: Creating Dependent Tasks
1. **Create first Task**:
   ```bash
   celebi-cli create-task data-preprocessing
   ```
2. **Create dependent Task**:
   ```bash
   celebi-cli create-task analysis
   ```
3. **Set up dependencies** (using appropriate `celebi-cli *` commands):
   ```json
   "dependencies": ["data-preprocessing"]
   ```

### Workflow 3: Creating Algorithm Library
1. **Create multiple Algorithms**:
   ```bash
   celebi-cli create-algorithm statistical-tests
   celebi-cli create-algorithm visualization
   celebi-cli create-algorithm data-validation
   ```
2. **Organize in Directory**:
   ```bash
   mkdir algorithms-library
   # Move algorithms into directory
   ```
3. **Create Tasks that use different Algorithms**

## Command Reference

### `celebi-cli create-task [name]`
- **Purpose**: Initializes a new Task object directory
- **Creates**: Directory with `.celebi/config.json` and `README.md`
- **Location**: Run in parent directory where Task should be created
- **Next steps**: Configure parameters, link to Algorithm, set dependencies

### `celebi-cli create-algorithm [name]`
- **Purpose**: Initializes a new Algorithm object directory
- **Creates**: Directory with `.celebi/config.json` and `README.md`
- **Location**: Run in parent directory where Algorithm should be created
- **Next steps**: Define computation logic, environments, commands

### `celebi-cli add-algorithm [path]`
- **Purpose**: Links an existing Algorithm to the current Task
- **Usage**: Run inside Task directory, provide path to Algorithm
- **Effect**: Updates Task's `.celebi/config.json` with `code` alias
- **Requirement**: Algorithm must already exist

### `celebi-cli create-data [name]`
- **Purpose**: Initializes a new Data object directory
- **Creates**: Directory with `.celebi/config.json` and `README.md`
- **Location**: Run in parent directory where Data should be created
- **Next steps**: Configure data source and metadata

### `celebi-cli mkdir [name]`
- **Purpose**: Creates a Directory object for organizing tasks/algorithms
- **Creates**: Directory with `.celebi/config.json` and `README.md`
- **Location**: Run in parent directory where Directory should be created
- **Constraint**: Cannot contain nested Directory objects

## Configuration Guidelines

**Important**: Use `celebi-cli *` commands to modify configuration instead of editing `.celebi/config.json` files directly. The commands ensure proper syntax, validate relationships, and maintain Celebi system integrity.

### Task Configuration (.celebi/config.json)
```json
{
  "type": "task",
  "code": "path/to/algorithm",  // Linked Algorithm
  "parameters": {},             // Execution parameters
  "inputs": [],                 // Required input files
  "outputs": [],                // Expected output files
  "dependencies": []            // Other Tasks this depends on
}
```

### Algorithm Configuration (.celebi/config.json)
```json
{
  "type": "algorithm",
  "environment": {},           // Execution environment
  "commands": [],              // Computation commands
  "definitions": {}            // Logic definitions
}
```

## Best Practices

1. **Follow the Algorithm–Task Contract**: Keep computation logic in Algorithms, execution context in Tasks
2. **Explicit Declaration**: Tasks must explicitly declare all inputs and expected outputs
3. **Stateless Algorithms**: Algorithms should not "know" about workflow or other Tasks
4. **Single Reference**: Each Task maps to exactly one Algorithm
5. **Task-to-Task Dependencies**: Dependencies are between Tasks, not files
6. **Document README.md**: Every object needs human-readable documentation
7. **Use `celebi-cli *` commands for configuration**: Avoid manual editing of `.celebi/config.json` files; use appropriate commands to ensure system integrity

## Common User Requests

This skill should be used when users ask:
- "How do I create a new Celebi task?"
- "How do I create a Celebi algorithm?"
- "How do I link an algorithm to a task?"
- "How do I set up task dependencies?"
- "What's the difference between a task and an algorithm?"
- "How do I configure Celebi objects?"

## Error Handling

### Common Issues
1. **Missing `.celebi/config.json`**: Object not properly initialized
2. **Invalid `code` alias**: Algorithm path doesn't exist or isn't accessible
3. **Circular dependencies**: Tasks depending on each other creating cycles
4. **Missing dependencies**: Referenced Tasks don't exist

### Troubleshooting
1. Use `celebi-cli ls` to verify object types
2. Check `.celebi/config.json` syntax and paths (read-only; use `celebi-cli *` commands for modifications)
3. Verify Algorithm exists before linking to Task
4. Ensure dependency Tasks exist before referencing them

## References

For detailed information about Celebi architecture and the Algorithm–Task contract, see the Celebi documentation in the project's `.claude/CLAUDE.md` file or consult the specialized Celebi expert skills:

- **celebi-algorithm-expert** – Creating and modifying algorithms
- **celebi-task-expert** – Orchestrating and configuring tasks
- **celebi-amend-expert** – Diagnosing and debugging existing components

For other Celebi operations, consider these complementary skills:
- **celebi-navigation**: Explore project structure with `celebi-cli tree` and `celebi-cli ls`
- **celebi-workflow-execution**: Execute workflows with `celebi-cli submit` and manage dependencies
- **celebi-output-checker**: View task outputs, logs, and download result files after execution

## Skill Invocation Instructions

When working as `celebi-object-creation`, follow these instructions for invoking other Celebi skills:

1. **Configuring algorithms**: After creating an algorithm, if the user needs to implement its logic, **invoke the `/celebi-algorithm-expert` skill**.
2. **Configuring tasks**: After creating a task, if the user needs to configure parameters and dependencies, **invoke the `/celebi-task-expert` skill**.
3. **Debugging objects**: If created objects have configuration errors or issues, **invoke the `/celebi-amend-expert` skill** for diagnosis and repair.
4. **Exploring project structure**: If you need to understand where to place new objects or navigate the hierarchy, **invoke the `/celebi-navigation` skill**.
5. **Executing workflows**: Once objects are created and configured, **invoke the `/celebi-workflow-execution` skill** for submission and monitoring.
6. **Maintaining patience during execution**: If tasks are running and need monitoring without premature debugging, **invoke the `/celebi-workflow-patience` skill**.
7. **Viewing outputs and results**: After tasks complete, if the user needs to view outputs, logs, or download results, **invoke the `/celebi-output-checker` skill**.

**Important**: Always use the Skill tool to invoke these skills. Do not proceed with manual fixes when specialized skills exist.
