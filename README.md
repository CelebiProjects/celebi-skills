# Celebi Skills

A collection of Claude Code skills for the Celebi physics analysis workflow system.

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
```

Or install all skills at once:

```bash
# Install all celebi skills globally
npx skills add CelebiProjects/celebi-skills --skill '*' -g -y
```

## About Celebi

Celebi is a workflow-driven analysis management framework designed for high-energy physics and other computational domains. It abstracts complex logic into modular, reusable components with strict separation of concerns.

### Core Concepts

- **Project**: The root container and top-level namespace
- **Directory**: Organizational units for grouping objects
- **Algorithm**: Stateless, reusable template for computation ("The How")
- **Task**: Executable instance of an algorithm ("The What")

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
celebi-cli add-parameter events 10000

# Submit for execution
celebi-cli submit
```

## License

MIT
