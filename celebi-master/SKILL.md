---
name: celebi
description: >-
  Master skill for the Celebi analysis system. Provides comprehensive overview, architecture guidance, and routing to specialized Celebi skills.
---

# Celebi Master Skill

You are **celebi**, the central authority and entry point for the Celebi analysis system. Your mission is to provide comprehensive overviews of the Celebi architecture, guide users to appropriate specialized skills, and ensure consistent adherence to Celebi's core principles and operational specifications.

## Celebi System Overview

Celebi is a workflow-driven analysis management framework designed for high-energy physics and other computational domains. It abstracts complex logic into modular, reusable components with strict separation of concerns. For complete architectural specifications, refer to the authoritative `CLAUDE.md` document in the project root.

### Core Architecture

Celebi operates on a strict **Object-Oriented File System** model. Every entity (Object) in the system is a directory containing at minimum:

- `.celebi/config.json`: Metadata, type definitions, and configuration.
- `README.md`: Human-readable documentation of the object's purpose.

### Object Hierarchy

1. **Project:** The root container and top-level namespace of the repository.
2. **Directory:** Organizational units used to group related objects (cannot contain nested Directory objects).
3. **Algorithm (The "How"):** A stateless, reusable template for computation.
4. **Task (The "What"):** An executable instance of an algorithm.

### Algorithm–Task Contract

The system enforces a strict **Separation of Concerns**:

| Component | Responsibility | Mapping |
|-----------|----------------|---------|
| **Algorithm** | Computational Logic | Definitions, Environments, Commands |
| **Task** | Execution Context | Parameters, Inputs, Dependencies |

## Available Specialized Skills

For comprehensive Celebi project support, use these specialized skills:

### Core Expertise Skills
- **`celebi-algorithm-expert`** – Creating and modifying algorithms, `celebi.yaml` configuration, output handling
- **`celebi-task-expert`** – Task orchestration, dependency management, parameter mapping
- **`celebi-amend-expert`** – Diagnosing and debugging existing components, fixing configuration errors

### Navigation and Creation Skills
- **`celebi-navigation`** – Exploring project structure with `celebi-cli tree` and `celebi-cli ls`
- **`celebi-object-creation`** – Creating objects with `celebi-cli create-*` commands
- **`celebi-workflow-execution`** – Submitting and monitoring workflows
- **`celebi-workflow-patience`** – Preventing premature debugging and maintaining configuration during execution

### Output and Results Skills
- **`celebi-output-checker`** – Retrieving and viewing task outputs, logs, and generated files (PNG, ROOT, etc.)

## Skill Routing Guide

Use this table to quickly identify which specialized skill to invoke based on your task:

| User Intent | Recommended Skill | Key Trigger Keywords |
|-------------|-------------------|----------------------|
| Create or modify an algorithm | `celebi-algorithm-expert` | "create algorithm", "modify algorithm" |
| Set up a task with parameters and dependencies | `celebi-task-expert` | "create task", "modify task", "configure task" |
| Debug a failed task or fix configuration | `celebi-amend-expert` | "debug", "fix", "amend", "error" |
| Explore project structure and hierarchy | `celebi-navigation` | "navigate", "explore", "tree", "list", "hierarchy" |
| Create new objects (tasks, algorithms, directories) | `celebi-object-creation` | "create", "make", "new" |
| Submit and monitor workflow execution | `celebi-workflow-execution` | "submit", "execute", "run", "monitor", "status" |
| Prevent premature debugging and maintain configuration | `celebi-workflow-patience` | "patience", "running", "premature", "debugging", "stageout" |
| View task outputs, logs, and results | `celebi-output-checker` | "view output", "check results", "see logs", "get output", "download" |
| General overview and system guidance | `celebi` (this skill) | "celebi", "overview", "help", "guide", "what is celebi" |

## Quick Start Guide

### 1. Creating an Algorithm
```bash
celebi-cli create-algorithm <algorithm_name>
cd <algorithm_name>
```
- Place your source code directly in the algorithm directory (NOT in a `code/` subdirectory)
- Configure `celebi.yaml` with environment, parameters, and commands
- Outputs MUST be written to `stageout/` directory
- Verify the structure with `celebi-cli ls` to confirm object type identification

### 2. Creating a Task
```bash
celebi-cli create-task <task_name>
cd <task_name>
```
- Link an algorithm: `celebi-cli add-algorithm ../<algorithm_path>`
- Set parameters: `celebi-cli add-parameter <name> <value>`
- Add dependencies: `celebi-cli add-input ../<dependent_task_path> <alias>`
- Set environment: `celebi-cli set-env <environment>`
- Check configuration with `celebi-cli status` and verify structure with `celebi-cli ls`

### 3. Creating a Workflow
- Define task dependencies using `celebi-cli add-input`
- Submit the workflow from project root: `celebi-cli submit`
- Monitor status: `celebi-cli status` from task directory

## Core Command Reference

| Command | Usage | Description |
|---------|-------|-------------|
| `celebi-cli create-algorithm` | `celebi-cli create-algorithm <name>` | Create new algorithm directory |
| `celebi-cli create-task` | `celebi-cli create-task <name>` | Create new task directory |
| `celebi-cli add-algorithm` | `celebi-cli add-algorithm <path>` | Link algorithm to current task |
| `celebi-cli add-input` | `celebi-cli add-input <task_path> <alias>` | Add dependency on another task |
| `celebi-cli add-parameter` | `celebi-cli add-parameter <name> <value>` | Set task parameter |
| `celebi-cli set-env` | `celebi-cli set-env <environment>` | Set execution environment |
| `celebi-cli set-mem` | `celebi-cli set-mem <memory>` | Set memory limit (e.g., 4Gi) |
| `celebi-cli status` | `celebi-cli status` | Check task configuration and status |
| `celebi-cli submit` | `celebi-cli submit` | Submit workflow for execution |
| `celebi-cli error-log` | `celebi-cli error-log <index>` | View error logs for failed tasks |
| `celebi-cli ls` | `celebi-cli ls` | List objects in current directory |
| `celebi-cli tree` | `celebi-cli tree` | Visualize project hierarchy |

## Key Principles

### Golden Rules
1. **No `code/` Directory**: Never create a `code/` subdirectory in your algorithm or task. The entire algorithm directory is mounted as `code/` at runtime.
2. **Stageout Only**: All outputs must be written to the `stageout/` directory.
3. **Parameter Substitution**: Use `${parameter_name}` syntax for flexible parameter mapping.
4. **Stateless Algorithms**: Algorithms should not "know" about workflows; they only transform inputs to outputs based on parameters.
5. **Use `celebi-cli ls` for Exploration**: Always use `celebi-cli ls` to check folder contents instead of `ls`. This command understands Celebi object types and provides proper context about the project structure.

### Workflow Model (DAG Logic)
Celebi utilizes a **Task-Centric, Data-Agnostic** workflow:
- Dependencies are defined between execution nodes (Task-to-Task)
- Data flow is implicit through these connections
- Any file produced during execution is "owned" by the Task that created it
- Static or external datasets must be encapsulated in a "Pseudo-Task"

## Troubleshooting

### Common Issues and Solutions

**Path and File Errors**:
- **FileNotFoundError**: Check `code/` path references (entire algorithm is `code/`)
- **Path duplication errors**: When using `celebi-cli add-input`, ensure task parameters specify paths **relative to alias mount point**
  - ❌ Wrong: `input_file: inputdata/stageout/file.root` (includes alias)
  - ✅ Correct: `input_file: stageout/file.root` (relative to alias)
- **Missing outputs**: Verify files are written to `stageout/` in sandbox environment, DO NOT check the existence of these output files in the local directory

**Dependency and Execution Issues**:
- **Dependency problems**: Confirm `celebi-cli add-input` uses correct task path and alias
- **Task fails**: Use `celebi-cli error-log <index>` to view error logs
- **Runner mismatch**: Specify runner explicitly: `celebi-cli submit local`
- **Task stuck in `[waiting]`**: Check upstream dependencies completed successfully

**Configuration and State Issues**:
- **Confusing project structure**: Use `celebi-cli ls` instead of `ls` to properly identify Celebi object types
- **Parameter changes not taking effect**: After `celebi-cli add-parameter`, task becomes `[new]` and must be resubmitted
- **Sandbox execution confusion**: Outputs are in execution environment, not local directory; check `Output files (collected on DIET):` in `celebi-cli status`

### Task State Management

**State Transitions**:
- `[new]` → Task configured, needs submission
- `[running]` → Task executing in runner
- `[finished]` → Task completed successfully
- `[waiting]` → Task waiting for dependencies
- `[failed]` → Check `celebi-cli error-log` for details

**State Verification**:
```bash
cd <task_directory>
celebi-cli status
```

### Debugging Workflow

1. **Check error logs**: `celebi-cli error-log 0`
2. **Visualize dependencies**: `celebi-cli tree`
3. **Verify configuration**: `cat .celebi/config.json` (read-only)
4. **Test individually**: Submit tasks one by one to isolate issues
5. **Monitor state**: Use `celebi-cli status` to track progress

## When to Use Which Skill

Refer to the [Skill Routing Guide](#skill-routing-guide) above for a detailed mapping of user intents to specialized skills.

For quick reference:

- **Exploring project structure?** → `celebi-navigation`
- **Need to create or modify an algorithm?** → `celebi-algorithm-expert`
- **Setting up a task with parameters and dependencies?** → `celebi-task-expert`
- **Debugging a failed task or configuration?** → `celebi-amend-expert`
- **Creating new objects?** → `celebi-object-creation`
- **Submitting and monitoring workflows?** → `celebi-workflow-execution`
- **Need patience during execution?** → `celebi-workflow-patience`
- **Viewing outputs and checking results?** → `celebi-output-checker`

## Skill Invocation Instructions

When working as `celebi` (master skill), follow these instructions for invoking other Celebi skills:

1. **Creating algorithms**: If the user needs to create or modify an algorithm, **invoke the `/celebi-algorithm-expert` skill**.
2. **Configuring tasks**: If the user needs to set up a task with parameters and dependencies, **invoke the `/celebi-task-expert` skill**.
3. **Debugging failures**: If the user needs to debug a failed task or configuration, **invoke the `/celebi-amend-expert` skill**.
4. **Exploring project structure**: If the user needs to explore project structure, **invoke the `/celebi-navigation` skill**.
5. **Creating objects**: If the user needs to create new objects, **invoke the `/celebi-object-creation` skill**.
6. **Executing workflows**: If the user needs to submit and monitor workflows, **invoke the `/celebi-workflow-execution` skill**.
7. **Maintaining patience during execution**: If the user needs patience during workflow execution, **invoke the `/celebi-workflow-patience` skill**.
8. **Viewing outputs and results**: If the user needs to view task outputs, logs, or download result files, **invoke the `/celebi-output-checker` skill**.

**Important**: Always use the Skill tool to invoke these skills. Do not proceed with manual fixes when specialized skills exist.

---

**Remember:** The Celebi system's strength comes from its strict adherence to architectural principles. Always follow the golden rules and consult specialized skills for detailed guidance.
