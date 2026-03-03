---
name: celebi-workflow-patience
description: Use when working with Celebi physics analysis workflows to prevent common mistakes like premature debugging, configuration reversion, and inefficient command usage
---

# Celebi Workflow Patience Skill

## Overview

**Core Principle**: When working with Celebi physics analysis workflows, practice patience and trust in the system. Celebi manages complex dependency graphs and execution states - premature intervention often causes more problems than it solves.

## When to Use This Skill

Trigger this skill when you encounter any of these situations:

1. **Task shows `[running]` status** - This is normal operation, not an error
2. **User provides specific directory guidance** (e.g., "inputs should be in stageout directory")
3. **Working with Celebi configuration files** (`celebi.yaml`, `config.yaml`, etc.)
4. **User mentions Celebi workflow patterns** or physics analysis pipelines
5. **Encountering apparent "stalls" or "hangs"** in workflow execution

## Common Mistakes Table (Based on Real Experience)

| Failure Mode | Symptom | Root Cause | Solution |
|--------------|---------|------------|----------|
| **Configuration Reversion** | User says "inputs should be in stageout directory" but agent changes config back to default | Not trusting user's specific guidance | Maintain user's configuration guidance throughout execution |
| **Premature Debugging** | Agent tries to debug when task shows `[running]` status | Misinterpreting normal operation as error | Monitor, don't debug when status is `[running]` |
| **Inefficient Command Usage** | Using complex bash pipes instead of standard `celebi-cli` commands | Not leveraging Celebi's built-in utilities | Use standard command patterns: `celebi-cli status`, `celebi-cli logs`, etc. |
| **Modifying Configuration During Execution** | Changing config while task is running | Trying to "fix" a system that's working correctly | Never modify configuration while task is running |

## Rules for Each Failure Mode

### Rule 1: Maintain User Configuration Guidance
- **When**: User provides specific directory or configuration guidance
- **Do**: Preserve and use their guidance throughout the workflow
- **Don't**: Revert to defaults or "standard" patterns without explicit permission
- **Example**: If user says "inputs should be in stageout directory", keep inputs in stageout directory

### Rule 2: Trust `[running]` Status
- **When**: Task shows `[running]` status
- **Do**: Monitor progress with `celebi-cli status` and `celebi-cli logs`
- **Don't**: Attempt debugging, configuration changes, or restart attempts
- **Wait Time**: Minimum 2-3 minutes before considering any intervention

### Rule 3: Use Standard Command Patterns
- **Preferred**: `celebi-cli status` - Check workflow status
- **Preferred**: `celebi-cli logs <task_id>` - View task logs
- **Preferred**: `celebi-cli list` - List all tasks
- **Avoid**: Complex bash pipes (`grep`, `awk`, `sed`) when Celebi commands exist
- **Avoid**: Manual file inspection when `celebi-cli` provides the information

### Rule 4: Never Modify Running Configuration
- **When**: Any task is in `[running]` state
- **Do**: Monitor and wait for completion or failure
- **Don't**: Edit `celebi.yaml`, `config.yaml`, or any configuration files
- **Exception**: Only if user explicitly requests configuration changes

## Command Patterns Reference

### Efficient Monitoring Commands
```bash
# Basic status check
celebi-cli status

# Detailed task status
celebi-cli status --verbose

# View logs for specific task
celebi-cli logs <task_id>

# Follow logs in real-time
celebi-cli logs <task_id> --follow

# List all tasks with states
celebi-cli list --all
```

### Configuration Commands (Use Only When NOT Running)
```bash
# Validate configuration
celebi-cli validate

# Dry run (no execution)
celebi-cli run --dry-run

# Check dependencies
celebi-cli deps
```

## Red Flags (Rationalizations to Watch For)

1. **"It's taking too long"** → Celebi workflows often take minutes to hours
2. **"Maybe I should check the files manually"** → Use `celebi-cli logs` instead
3. **"The configuration might be wrong"** → Trust the running state
4. **"Let me restart it"** → This breaks dependency tracking
5. **"I'll use grep/awk to parse the output"** → Use `celebi-cli` commands instead

## Workflow Decision Diagram

```
Start Celebi Workflow
        ↓
celebi-cli run
        ↓
Check Status → [failed] → celebi-cli logs → Debug with user guidance
        ↓
     [running] → Wait 2-3 minutes → celebi-cli status
        ↓                               ↓
    [success] ←--------------------- Still [running]?
        ↓
    Continue to
    next analysis step
```

## Testing This Skill (TDD Approach)

### RED Phase Tests (Known Failures)
1. **Test**: User says "inputs in stageout directory" → agent maintains this
2. **Test**: `[running]` status → agent monitors, doesn't debug
3. **Test**: Uses `celebi-cli` commands instead of bash pipes
4. **Test**: Doesn't modify config during execution

### GREEN Phase Implementation
This skill document represents the minimal implementation to pass the above tests.

### REFACTOR Phase Considerations
- Add more specific Celebi workflow patterns
- Include common physics analysis pipeline examples
- Add troubleshooting flowcharts for specific error types

## Integration with Other Celebi Skills

This skill complements and integrates with:

### `celebi-workflow-execution`
- **This skill**: Focuses on patience, monitoring, and avoiding premature intervention
- **Workflow execution**: Focuses on submission, dependency resolution, and execution patterns
- **Use together**: Execute workflows with patience, monitor without premature debugging

### `celebi-master`
- **This skill**: Specific behavioral patterns for workflow patience
- **Master skill**: Comprehensive system overview and routing to specialized skills
- **Use together**: Get overview from master, apply patience patterns from this skill

### `celebi-task-expert`
- **This skill**: Behavioral guidance during execution
- **Task expert**: Task configuration and orchestration
- **Use together**: Configure tasks properly, then execute with patience

## Common User Requests

This skill should be used when users ask or encounter:
- "Why is my task taking so long?" (Answer: Monitor, don't debug)
- "Should I check the configuration?" (Answer: Not while running)
- "The task seems stuck" (Answer: Wait 2-3 minutes, then check status)
- "Let me restart it" (Answer: This breaks dependency tracking)
- "I'll modify the config to fix it" (Answer: Never modify during execution)

## Skill Invocation Instructions

When working as `celebi-workflow-patience`, follow these instructions for invoking other Celebi skills:

1. **Executing workflows**: When tasks are ready for submission after patient monitoring, **invoke the `/celebi-workflow-execution` skill** for submission and monitoring.
2. **Configuring tasks**: If patient monitoring reveals configuration issues that need fixing (after tasks have stopped), **invoke the `/celebi-task-expert` skill**.
3. **Creating algorithms**: If patient monitoring reveals algorithm issues that need fixing (after tasks have stopped), **invoke the `/celebi-algorithm-expert` skill**.
4. **Debugging failures**: If tasks fail and need diagnosis after patient monitoring, **invoke the `/celebi-amend-expert` skill**.
5. **Exploring project structure**: If you need to understand dependencies or project hierarchy, **invoke the `/celebi-navigation` skill**.
6. **Creating objects**: If patient monitoring reveals a need for new objects, **invoke the `/celebi-object-creation` skill**.
7. **Viewing outputs and results**: After tasks complete, if the user needs to view outputs, logs, or download results, **invoke the `/celebi-output-checker` skill**.

**Important**: Always use the Skill tool to invoke these skills. Do not proceed with manual fixes when specialized skills exist.

## References

For detailed information about Celebi workflow execution, see:
- `celebi-workflow-execution` - Workflow submission and monitoring
- `celebi-master` - Comprehensive system overview
- `celebi-task-expert` - Task configuration and orchestration

---
*Skill created based on TDD approach from real failure experiences in Celebi workflow management*