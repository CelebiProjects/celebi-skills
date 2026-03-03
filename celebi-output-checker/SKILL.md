---
name: celebi-output-checker
description: Retrieve, view, and analyze Celebi task outputs including logs, console output, and generated files. Use when the user wants to (1) Check task execution logs or stdout, (2) View or export output files (PNG, ROOT, etc.), (3) Verify task completion status and results, (4) Analyze fit results or histograms from physics analysis workflows.
---

# Celebi Output Checker

## Quick Start

Check task results with these commands:

```bash
# View console output/log
cd <task_directory>
celebi-cli log 0

# Check task status
cd <task_directory>
celebi-cli status

# Get view URL for browser
cd <task_directory>
celebi-cli viewurl
```

## Viewing Logs

### Console Output (stdout)
```bash
cd <task_directory>
celebi-cli log 0
```

### Error Logs (if task failed)
```bash
cd <task_directory>
celebi-cli log 0  # Usually same as stdout for errors
```

### Collect Logs Locally
```bash
cd <task_directory>
celebi-cli collect-logs
```

## Retrieving Output Files

### Step 1: Get View URL
```bash
cd <task_directory>
celebi-cli viewurl
```

### Step 2: Fetch Output Files
Use `curl` to download files from the view URL:

```bash
# First, get the file paths from the view page
curl -s <view_url> | grep -E "href=|src="

# Download specific files (e.g., PNG, ROOT)
curl -s "http://127.0.0.1:3315/file-view/<path_to_file>" -o /tmp/output.png
```

### Step 3: View Downloaded Files
```bash
# For images
open /tmp/output.png

# For ROOT files, use ROOT
cd /tmp
root -l output.root
```

## Common Output Patterns

### Physics Analysis Tasks

**Generation Task Output:**
```
Generating 10000 events with Crystal Ball parameters:
  Mass: 3.0969 GeV/c^2
  Sigma: 0.05 GeV/c^2
Generated 10000 events in 37568 attempts
Output saved to: stageout/jpsi_distribution.root
Tree entries: 10000
Histogram mean: 3.0878 GeV/c^2
```

**Fit Task Output:**
```
==================================================
Gaussian Fit Results
==================================================
Amplitude: 364.56 ± 4.82
Mean:      3.0938 ± 0.0006 GeV/c²
Sigma:     0.0523 ± 0.0004 GeV/c²
Chi²/NDF:  313.34/69 = 4.54
==================================================
```

## Analyzing Results

### Check Fit Quality
- **χ²/NDF ~ 1**: Good fit
- **χ²/NDF > 3**: Poor fit (model may not describe data well)
- **χ²/NDF < 0.5**: Overfitting or errors overestimated

### Verify Outputs Exist
```bash
# Check if outputs were collected
cd <task_directory>
ls -la stageout/ 2>/dev/null || echo "No stageout yet - run collect-outputs"

# Collect outputs
cd <task_directory>
celebi-cli collect-outputs
```

## Troubleshooting

### "No such command 'log'"
Make sure you're in a task directory (not algorithm directory).

### View URL returns 404
The local server may not be running. Try:
```bash
cd <task_directory>
celebi-cli view
```

### Empty output files
Check if task actually finished:
```bash
cd <task_directory>
celebi-cli status
```
Look for `[coda][finished]` status.
