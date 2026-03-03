---
name: celebi-navigation
description: Comprehensive navigation and exploration of Celebi project structures. Use when working with Celebi analysis projects to visualize project hierarchy with celebi-cli tree, list and identify object types with celebi-cli ls, understand Celebi object relationships and dependencies, navigate through Project/Directory/Algorithm/Task objects, or explore Celebi project structure before making modifications.
---

# Celebi Project Navigation

## Overview

This skill provides comprehensive guidance for navigating and exploring Celebi analysis projects. Celebi is a workflow-driven analysis management framework with a strict Object-Oriented File System model. Use this skill to understand project structure, identify object types, and navigate through the Celebi hierarchy.

## Quick Start

### Basic Navigation Commands

1. **Visualize project hierarchy**:
   ```bash
   celebi-cli tree
   ```
   Shows the entire project structure with object types and relationships.

2. **List directory contents**:
   ```bash
   celebi-cli ls
   ```
   Identifies object types in the current directory and shows Celebi-specific information.

3. **Check current directory type**:
   ```bash
   celebi-cli ls .
   ```
   Identifies what type of Celebi object the current directory represents.

4. **Navigate within project**:
   ```bash
   cd <path>          # Change directory within Celebi project
   celebi-cli short-ls           # Compact listing of objects
   ```

## Understanding Celebi Object Hierarchy

Celebi projects follow a strict object hierarchy:

1. **Project**: Root container and top-level namespace
2. **Directory**: Organizational units (cannot contain nested Directory objects)
3. **Algorithm**: Stateless, reusable template for computation ("The How")
4. **Task**: Executable instance of an algorithm ("The What")

### Key Characteristics

- Every Celebi object is a directory containing at minimum:
  - `.celebi/config.json`: Metadata, type definitions, and configuration
    - **Important**: Use `celebi-cli *` commands to modify configuration instead of editing this file directly
  - `README.md`: Human-readable documentation
- Objects follow the Algorithm–Task Contract: strict separation of concerns
- Workflows are Task-centric and data-agnostic (DAG logic)

## Common Navigation Scenarios

### Scenario 1: Exploring a New Project
When first encountering a Celebi project:
1. Run `celebi-cli tree` from the project root to see the full hierarchy
2. Use `celebi-cli ls` in any directory to identify object types
3. Look for `.celebi/config.json` files to understand object configurations (read-only; use `celebi-cli *` commands for modifications)

### Scenario 2: Finding Specific Object Types
To locate algorithms or tasks:
1. Navigate to likely directories (e.g., `algorithms/`, `tasks/`)
2. Use `celebi-cli ls` to confirm object types
3. Check `README.md` files for documentation

### Scenario 3: Understanding Dependencies
To understand task dependencies:
1. Use `celebi-cli tree` to visualize relationships
2. Examine `.celebi/config.json` in task directories for dependency declarations (read-only; use `celebi-cli *` commands for modifications)
3. Follow the DAG logic: Task-to-Task dependencies only

## Command Reference

### `celebi-cli tree`
- **Purpose**: Visualizes the entire project hierarchy
- **Usage**: `celebi-cli tree`
- **Output**: Colored tree showing object types and relationships
- **Best for**: Understanding overall project structure and object relationships

### `celebi-cli ls`
- **Purpose**: Lists contents and identifies object types
- **Usage**: `celebi-cli ls [path]`
- **Output**: Identifies Celebi object types with additional metadata
- **Best for**: Checking current directory type or exploring specific paths

### `celebi-cli cd`
- **Purpose**: Change directory within Celebi project
- **Usage**: `celebi-cli cd <path>`
- **Output**: Changes current working directory (similar to `cd`)
- **Best for**: Navigating to specific objects

### `celebi-cli cdproject`
- **Purpose**: Switch to another Celebi project
- **Usage**: `celebi-cli cdproject <project_name>`
- **Output**: Changes to project root directory
- **Best for**: Working across multiple projects

### `celebi-cli short-ls`
- **Purpose**: Compact listing of objects
- **Usage**: `celebi-cli short-ls`
- **Output**: Simplified list without detailed metadata
- **Best for**: Quick overview of directory contents

## Best Practices

1. **Always start with `celebi-cli tree`** when exploring a new Celebi project
2. **Use `celebi-cli ls` instead of regular `ls`** to get Celebi-specific information
3. **Check object types** before making modifications
4. **Follow the object hierarchy** - understand what type of object you're working with
5. **Respect the Algorithm–Task Contract** - don't mix concerns between object types
6. **Use `celebi-cli *` commands for configuration** - modify `.celebi/config.json` through appropriate commands instead of manual editing

## Common User Requests

This skill should be used when users ask:
- "Show me the structure of this Celebi project"
- "What type of Celebi object is this directory?"
- "How do I navigate this Celebi project?"
- "Help me understand the Celebi hierarchy"
- "What objects are in this Celebi directory?"

## References

For detailed information about Celebi architecture and operational rules, see the Celebi documentation in the project's `.claude/CLAUDE.md` file or consult the specialized Celebi expert skills:
- `celebi-algorithm-expert` – Creating and modifying algorithms
- `celebi-task-expert` – Orchestrating and configuring tasks
- `celebi-amend-expert` – Diagnosing and debugging existing components

For other Celebi operations, consider these complementary skills:
- **celebi-object-creation**: Create tasks, algorithms, and directories using `celebi-cli create-*` commands
- **celebi-workflow-execution**: Execute workflows with `celebi-cli submit` and manage dependencies
- **celebi-output-checker**: View task outputs, logs, and download result files after execution

## Skill Invocation Instructions

When working as `celebi-navigation`, follow these instructions for invoking other Celebi skills:

1. **Creating algorithms**: If exploration reveals a need to create or modify an algorithm, **invoke the `/celebi-algorithm-expert` skill**.
2. **Creating tasks**: If exploration reveals a need to create or modify a task, **invoke the `/celebi-task-expert` skill**.
3. **Debugging components**: If exploration reveals issues with existing components, **invoke the `/celebi-amend-expert` skill** for diagnosis and repair.
4. **Creating objects**: If exploration reveals a need to create new objects (tasks, algorithms, directories), **invoke the `/celebi-object-creation` skill**.
5. **Executing workflows**: Once the project structure is understood and tasks are ready, **invoke the `/celebi-workflow-execution` skill** for submission and monitoring.
6. **Maintaining patience during execution**: If tasks are running and need monitoring without premature debugging, **invoke the `/celebi-workflow-patience` skill**.
7. **Viewing outputs and results**: After tasks complete, if the user needs to view outputs, logs, or download results, **invoke the `/celebi-output-checker` skill**.

**Important**: Always use the Skill tool to invoke these skills. Do not proceed with manual fixes when specialized skills exist.
