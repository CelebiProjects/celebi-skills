---
name: celebi-workflow-execution
description: Execution and management of Celebi workflows including task submission, dependency resolution, and workflow monitoring. Use when working with Celebi analysis projects to submit tasks with celebi-cli submit, manage workflow dependencies, monitor execution status, handle task outputs, or troubleshoot workflow execution issues.
---

# Celebi Workflow Execution

## Overview

This skill provides comprehensive guidance for executing and managing Celebi workflows. Celebi utilizes a Task-Centric, Data-Agnostic workflow model based on Directed Acyclic Graph (DAG) logic. Use this skill to submit tasks, manage dependencies, monitor execution, and handle workflow outputs.

## Key Workflow Commands

### Submission & Execution
- `celebi-cli submit` – Submit current task for execution
- `celebi-cli submit <path>` – Submit specific task
- `celebi-cli status` – Check execution status of tasks

### Dependency Management
- `celebi-cli tree` – Visualize task dependencies and project hierarchy
- `celebi-cli ls` – List dependencies and related tasks

### Monitoring & Debugging
- `celebi-cli error-log <index>` – View error logs for failed tasks
- `celebi-cli trace <task>` – Show execution logs and history
- `celebi-cli history` – Display command history

## Quick Start

### Basic Workflow Execution

1. **Submit a Task for execution**:
   ```bash
   celebi-cli submit
   ```
   Submits the current Task directory for execution.

2. **Submit specific Task**:
   ```bash
   celebi-cli submit [path/to/task]
   ```
   Submits the specified Task for execution.

3. **Check execution status**:
   ```bash
   # Check for output files in Task directory
   ls [task-directory]/
   ```
   Look for output files specified in the Task's configuration.

## Runner Selection

**Choosing the Right Runner**:
- Default runner: `local` for local execution
- Specify runner explicitly: `celebi-cli submit <runner>`
- Common pattern: `celebi-cli submit local`

**Runner Mismatch Errors**:
If you encounter "Runner mismatch" errors:
1. Check task configuration: `cat .celebi/config.json` for `"default_runner"`
2. Ensure submission command matches configured runner
3. Best practice: Always specify runner explicitly

**Runner Configuration**:
- Local execution: `celebi-cli submit local`
- Cluster execution: `celebi-cli submit cluster-01` (if configured)
- Verify runner availability before submission

## Understanding Celebi Workflow Model

### Task-Centric, Data-Agnostic DAG
- **Rule 1**: Dependencies are between Tasks, not files
- **Rule 2**: Any file produced is "owned" by the Task that created it
- **Rule 3**: External data must be wrapped in "Pseudo-Tasks"

### Workflow Characteristics
- **Directed Acyclic Graph (DAG)**: Tasks execute in dependency order
- **Implicit Data Flow**: Data flows through Task dependencies
- **Task Ownership**: Output files belong to producing Tasks
- **Explicit Dependencies**: Must be declared in Task configuration

## Workflow Execution Scenarios

### Scenario 1: Executing a Single Task
1. **Navigate to Task directory**:
   ```bash
   cd [task-directory]
   ```
2. **Verify Task configuration**:
   ```bash
   cat .celebi/config.json
   ```
   Check `code` (linked Algorithm), `inputs`, `outputs`, `dependencies`.
3. **Submit for execution**:
   ```bash
   celebi-cli submit
   ```
4. **Monitor execution**:
   Check for output files and execution logs.

### Scenario 2: Executing Dependent Tasks
1. **Check dependency graph**:
   Use `celebi-cli tree` to visualize dependencies.
2. **Execute in correct order**:
   Celebi automatically resolves dependencies.
3. **Submit root Task**:
   ```bash
   celebi-cli submit [final-task]
   ```
   Dependent Tasks will execute first.

### Scenario 3: Handling External Data
1. **Create Pseudo-Task** for external data:
   ```bash
   celebi-cli create-task external-data-wrapper
   ```
2. **Configure as data source**:
   Set up Task to "produce" the external data files.
3. **Make dependent Tasks reference it**:
   Add `external-data-wrapper` to dependencies.

## Command Reference

### `celebi-cli submit`
- **Purpose**: Submits a Task for execution
- **Usage**: `celebi-cli submit [path]` or `celebi-cli submit` (in Task directory)
- **Behavior**: Resolves dependencies, executes in DAG order
- **Output**: Creates files in Task directory as specified in configuration

## Workflow Management

### Dependency Resolution
Celebi automatically resolves Task dependencies:
1. **Parses dependency declarations** from `.celebi/config.json`
2. **Builds execution DAG** based on Task relationships
3. **Executes in topological order** (dependencies before dependents)
4. **Handles parallel execution** where possible

### Execution Monitoring
Monitor workflow execution by:
1. **Checking for output files** in Task directories
2. **Looking for execution logs** (location depends on Celebi setup)
3. **Verifying dependency satisfaction** (all inputs available)
4. **Checking for error indicators** (failed execution markers)

### Output Handling
Task outputs are "owned" by the producing Task:
1. **Output files** are created in the Task directory
2. **File ownership** follows Rule 2 of Celebi workflow model
3. **Downstream Tasks** access outputs through dependency chain
4. **Data flow** is implicit through Task dependencies

### Sandbox Execution Model
**Important Clarification**:
- Tasks execute in isolated sandbox environments
- Outputs are written to `stageout/` within the sandbox
- Outputs are "collected on DIET" (Distributed Execution Environment)
- Local task directory won't contain `stageout/` folder
- Check `Output files (collected on DIET):` in `celebi-cli status` for output listing

**Output Access Pattern**:
```
Task execution → stageout/ in sandbox → DIET collection → Available to dependent tasks
```

## Task State Machine

**State Transitions**:
```
[new] → submit → [running] → success → [finished]
                ↓
         failure → check error-log
```

**State Definitions**:
- `[new]`: Task configured but not submitted
- `[running]`: Task submitted and executing
- `[finished]`: Task completed successfully
- `[waiting]`: Task waiting for dependencies to complete
- `[failed]`: Task execution failed (check `celebi-cli error-log`)

**State Management**:
- Use `celebi-cli status` to track state transitions
- Parameter changes via `celebi-cli add-parameter` mark task as `[new]`
- Tasks with `[new]` status must be resubmitted
- Failed tasks remain in failed state until resubmitted

**Workflow Implications**:
- Dependent tasks wait (`[waiting]`) until upstream tasks reach `[finished]`
- Failed tasks block downstream execution
- Use `celebi-cli tree` to visualize blocking dependencies

## Best Practices

### Before Execution
1. **Verify Task configuration**: Check `.celebi/config.json` for completeness
2. **Validate dependencies**: Ensure referenced Tasks exist
3. **Check Algorithm linkage**: Verify `code` alias points to valid Algorithm
4. **Confirm input availability**: All required inputs should be available
5. **Use `celebi-cli *` commands for configuration**: Avoid manual editing of `.celebi/config.json`; use appropriate commands for any configuration changes

### During Execution
1. **Monitor dependency resolution**: Watch for dependency satisfaction
2. **Check execution order**: Verify Tasks execute in correct sequence
3. **Track output generation**: Output files should appear as specified
4. **Watch for errors**: Failed Tasks may block dependent execution

### After Execution
1. **Verify outputs**: Check all expected output files exist
2. **Validate data quality**: Ensure outputs meet quality requirements
3. **Document execution**: Update README.md with execution details
4. **Clean up if needed**: Remove temporary files if appropriate

## Troubleshooting

### Common Execution Issues

1. **Missing Dependencies**:
   - **Symptom**: Task fails with missing input errors
   - **Solution**: Verify dependency Tasks exist and executed successfully

2. **Circular Dependencies**:
   - **Symptom**: Infinite loop or execution failure
   - **Solution**: Check dependency graph for cycles using `celebi-cli tree`

3. **Invalid Algorithm Link**:
   - **Symptom**: Task fails to execute Algorithm code
   - **Solution**: Verify `code` alias in Task configuration points to valid Algorithm

4. **Missing Input Files**:
   - **Symptom**: Task cannot find required inputs
   - **Solution**: Check dependency Task outputs exist

5. **Path Configuration Errors**:
   - **Symptom**: `FileNotFoundError` with duplicated paths (e.g., `inputdata/inputdata/stageout/file.root`)
   - **Root Cause**: Task parameters include alias prefix when algorithm already prepends it
   - **Solution**: Ensure task parameters specify paths relative to alias mount point
   - **Correct Pattern**: Task parameter: `stageout/file.root`, Algorithm: `f"inputdata/{parameter}"`

### Debugging Workflows

**Comprehensive Debugging Approach**:

1. **Check Error Logs**:
   ```bash
   celebi-cli error-log 0
   ```
   - Shows final error traceback for failed tasks
   - First step when task fails

2. **Visualize Dependency Graph**:
   ```bash
   celebi-cli tree
   ```
   - Understand task relationships
   - Identify circular dependencies

3. **Check Task Status and State**:
   ```bash
   celebi-cli status
   ```
   - Monitor state transitions: `[new]` → `[running]` → `[finished]`
   - Identify stuck tasks: `[waiting]` indicates dependency issues

4. **Verify Task Configuration** (read-only):
   ```bash
   cat [task]/.celebi/config.json
   ```
   - Check `code` (linked algorithm)
   - Verify `inputs` and `dependencies`
   - Confirm `default_runner` matches submission

5. **Test Individual Tasks**:
   - Submit tasks individually to isolate issues
   - Start with independent tasks first
   - Verify each task completes before testing dependent tasks

6. **Common Failure Patterns**:
   - **Path errors**: Check parameter vs. algorithm path construction
   - **Dependency issues**: Verify `celebi-cli add-input` was used correctly
   - **Environment mismatches**: Confirm `celebi-cli set-env` matches algorithm requirements
   - **Parameter problems**: Ensure task parameters align with algorithm expectations

7. **Sandbox Execution Notes**:
   - Tasks execute in isolated sandbox environments
   - Outputs written to `stageout/` within sandbox
   - Local task directory won't contain `stageout/` folder
   - Check `Output files (collected on DIET):` in `celebi-cli status` for output listing

## Advanced Workflow Patterns

### Pattern 1: Parallel Execution
- **Setup**: Tasks without dependencies can execute in parallel
- **Benefit**: Improved performance for independent computations
- **Implementation**: Design Tasks with minimal dependencies where possible

### Pattern 2: Conditional Execution
- **Setup**: Use Algorithm logic to conditionally produce outputs
- **Benefit**: Dynamic workflow paths based on data
- **Implementation**: Algorithm decides what to compute based on parameters

### Pattern 3: Iterative Workflows
- **Setup**: Tasks that produce inputs for next iteration
- **Benefit**: Multi-step iterative computations
- **Implementation**: Chain Tasks with incremental outputs

## Common User Requests

This skill should be used when users ask:
- "How do I submit a Celebi task for execution?"
- "How do Celebi workflow dependencies work?"
- "Why isn't my task executing?"
- "How do I check task execution status?"
- "How do I handle external data in Celebi?"
- "How do I troubleshoot workflow execution issues?"

## References

For detailed information about Celebi workflow model and DAG logic, see the Celebi documentation in the project's `.claude/CLAUDE.md` file or consult the specialized Celebi expert skills:

- **celebi-algorithm-expert** – Creating and modifying algorithms
- **celebi-task-expert** – Orchestrating and configuring tasks
- **celebi-amend-expert** – Diagnosing and debugging existing components

For other Celebi operations, consider these complementary skills:
- **celebi-navigation**: Explore project structure with `celebi-cli tree` and `celebi-cli ls`
- **celebi-object-creation**: Create tasks, algorithms, and directories using `celebi-cli create-*` commands
- **celebi-output-checker**: View task outputs, logs, and download result files after execution

## Skill Invocation Instructions

When working as `celebi-workflow-execution`, follow these instructions for invoking other Celebi skills:

1. **Creating algorithms**: If execution reveals a need to create or modify an algorithm, **invoke the `/celebi-algorithm-expert` skill**.
2. **Configuring tasks**: If execution reveals issues with task configuration, **invoke the `/celebi-task-expert` skill**.
3. **Debugging failures**: If tasks fail during execution, **invoke the `/celebi-amend-expert` skill** for diagnosis and repair.
4. **Exploring project structure**: If you need to understand dependencies or project hierarchy, **invoke the `/celebi-navigation` skill**.
5. **Creating objects**: If execution reveals a need for new objects, **invoke the `/celebi-object-creation` skill**.
6. **Maintaining patience during execution**: If tasks are running and need monitoring without premature debugging, **invoke the `/celebi-workflow-patience` skill**.
7. **Viewing outputs and results**: After tasks complete, if the user needs to view outputs, logs, or download results, **invoke the `/celebi-output-checker` skill**.

**Important**: Always use the Skill tool to invoke these skills. Do not proceed with manual fixes when specialized skills exist.
