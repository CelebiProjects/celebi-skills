# Celebi Skills

A collection of agent skills for the Celebi physics analysis workflow system and
its Yuki execution and data service.

## Available Skills

| Skill | Description |
|-------|-------------|
| `celebi-master` | Master skill for system overview and routing to specialized skills |
| `celebi-navigation` | Explore project structure with `celebi-cli tree` and `celebi-cli ls` |
| `celebi-object-creation` | Create tasks, algorithms, and directories |
| `celebi-workflow-execution` | Submit and monitor workflow execution |
| `celebi-workflow-patience` | Prevent premature debugging during execution |
| `celebi-algorithm-expert` | Create and modify algorithms |
| `celebi-task-expert` | Configure and orchestrate tasks |
| `celebi-amend-expert` | Debug and fix existing components |
| `celebi-output-checker` | View logs and retrieve output files (PNG, ROOT, etc.) |
| `celebi-runner-management` | Configure runners, environment mappings, and REANA booking |
| `celebi-data-management` | Register, verify, locate, and transfer data |
| `celebi-impression-management` | Inspect provenance, publish bookkeeping, migrate/archive impressions, and scope cleanup |
| `yuki-service-management` | Deploy and diagnose Yuki, Celery, RabbitMQ, and storage |

## Installation

Install individual skills using the `npx skills` CLI:

```bash
# Install the master skill (recommended first)
npx skills add CelebiProjects/celebi-skills@celebi-master

# Install specific skills as needed
npx skills add CelebiProjects/celebi-skills@celebi-navigation
npx skills add CelebiProjects/celebi-skills@celebi-object-creation
npx skills add CelebiProjects/celebi-skills@celebi-workflow-execution
npx skills add CelebiProjects/celebi-skills@celebi-workflow-patience
npx skills add CelebiProjects/celebi-skills@celebi-algorithm-expert
npx skills add CelebiProjects/celebi-skills@celebi-task-expert
npx skills add CelebiProjects/celebi-skills@celebi-amend-expert
npx skills add CelebiProjects/celebi-skills@celebi-output-checker
npx skills add CelebiProjects/celebi-skills@celebi-runner-management
npx skills add CelebiProjects/celebi-skills@celebi-data-management
npx skills add CelebiProjects/celebi-skills@celebi-impression-management
npx skills add CelebiProjects/celebi-skills@yuki-service-management
```

Or install all skills at once:

```bash
# Install all celebi skills globally
npx skills add CelebiProjects/celebi-skills --skill '*' -g -y
```

To reinstall uncommitted local changes for Codex and Claude Code, use the local
checkout rather than downloading the published repository:

```bash
npx skills add /path/to/celebi-skills --skill '*' --agent codex claude-code -g -y
```

The master skill's declared name is `celebi`, so installers may use that directory
name even though its source folder is `celebi-master`. Back up older installations
before replacement and avoid retaining two stale copies with the same skill name.

## About Celebi

Celebi is a workflow-driven analysis management framework designed for high-energy physics and other computational domains. It abstracts complex logic into modular, reusable components with strict separation of concerns.

### Core Concepts

- **Project**: The root container and top-level namespace
- **Directory**: Organizational units for grouping objects, including nested directories
- **Algorithm**: Stateless, reusable template for computation ("The How")
- **Task**: Execution context using a linked algorithm or its own inline commands
- **Subtasks**: Optional positive repeat count on one Task, with an injected index
  and one shared impression; retained outputs are `stageout/<number>/...`
- **Data**: A task flavor representing raw data or a data list
- **Yuki**: Service coordinating runners, logs, result collection, and data caches

### Quick Start

```bash
# Create a new algorithm
celebi-cli create-algorithm my-algorithm

# Create a new task
celebi-cli create-task my-task

# Link algorithm to task
cd my-task
celebi-cli add-algorithm ../my-algorithm

# Set parameters
celebi-cli set-parameter events 10000

# New projects include .celebi-env/default.yaml; install it on the local runner.
celebi-cli install-environment local default

# Define commands/source in my-algorithm and configure the task environment first.
# Submit to the runner requested by this task (the argument is a runner name).
celebi-cli request-runner local
celebi-cli submit --runner local
celebi-cli status --wait
celebi-cli log 0

# Collect plots into Yuki storage and get the result-view URL
celebi-cli collect plots
celebi-cli view-url
```

The skills reflect the local Celebi and Yuki source trees reviewed on 2026-10-25,
including their uncommitted changes. The master folder retains its existing skill
name `celebi`; the other skill names match their folders. Companion skills are
optional and can be installed individually. Check the installed CLI's command-specific
help when working against an older deployment. The task skill documents the
current inline-code path differences between client tests and Yuki execution.
The task skill's `references/subtasks.md` covers indexed inputs, literal
`stageout/index/` instance outputs, logs, cache publication and retry limits,
including `Demo/demo-subtasks-01-gaussian-toys` (three Tasks, 100 fits).
The service and impression skills also cover the built-in local runner, immutable
impressions, bookkeeping publication, and implemented cleanup behavior.

## License

MIT
