# Dr. Ralph - OpenCode Plugin

> **OpenCode Version:** Migrated from Claude Code with Oh My OpenCode integration
> 
> **AI-assisted medical diagnostics with comprehensive symptom analysis and research-backed treatment plans.**

> **Disclaimer:** This software is for informational and educational purposes only. It is NOT a substitute for professional medical advice. See [full disclaimer](#disclaimer).

---

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Installation](#installation)
  - [Prerequisites](#prerequisites)
  - [Install Oh My OpenCode](#install-oh-my-opencode)
  - [Install Dr. Ralph Plugin](#install-dr-ralph-plugin)
  - [Verify Installation](#verify-installation)
- [Commands](#commands)
  - [/dr-ralph:diagnose](#dr-ralphdiagnose)
  - [/dr-ralph:cancel](#dr-ralphcancel)
  - [/cancel-ralph](#cancel-ralph)
  - [/ralph-loop](#ralph-loop)
- [Diagnostic Workflow](#diagnostic-workflow)
  - [5-Phase Process](#5-phase-process)
  - [Patient Case Management](#patient-case-management)
  - [SOAP Report Format](#soap-report-format)
- [Usage Tips](#usage-tips)
- [When to Use Dr. Ralph](#when-to-use-dr-ralph)
- [Architecture](#architecture)
- [Migration Notes](#migration-notes)
  - [Differences from Claude Code Version](#differences-from-claude-code-version)
  - [Tool Mapping](#tool-mapping)
  - [Loop Mechanism Changes](#loop-mechanism-changes)
- [Disclaimer](#disclaimer)
- [License](#license)
- [Learn More](#learn-more)

---

## Overview

Dr. Ralph provides a structured 5-phase diagnostic workflow:

**Interview → Research → Differential Diagnosis → Treatment Plan → SOAP Report**

It uses Oh My OpenCode's built-in `ralph-loop` command to iterate through phases until reaching a convincing diagnosis with research-backed treatment recommendations.

### Key Features

- **Self-referential AI Loop:** Continuous iteration through diagnostic phases using OpenCode's continuation system
- **5-Phase Workflow:** Interview, Research, Differential Diagnosis, Treatment Plan, and SOAP Report generation
- **Patient Tracking:** Persistent patient notes across sessions
- **Research-Backed:** Integrated web search for medical literature
- **SOAP Reports:** Professional medical documentation format

---

## Quick Start

```bash
/dr-ralph:diagnose "Persistent fatigue and joint pain" --patient "John Doe"
```

---

## Installation

### Prerequisites

**Required:**
- Oh My OpenCode framework
- OpenCode environment
- Internet connection for web search tools

**Recommended:**
- Basic familiarity with medical terminology
- Understanding of SOAP documentation format

### Install Oh My OpenCode

Dr. Ralph requires **Oh My OpenCode** framework. Install it first:

```bash
# Clone Oh My OpenCode
git clone https://github.com/code-yeongyu/oh-my-opencode.git ~/.opencode

# Or follow official installation guide at:
# https://github.com/code-yeongyu/oh-my-opencode
```

### Install Dr. Ralph Plugin

**Option 1: Local Development**

```bash
# Clone the repository
git clone git@github.com:blencorp/dr-ralph.git

# Navigate to the project
cd dr-ralph

# The OpenCode plugin files are in the `opencode/` directory
# No additional installation needed for local use
```

**Option 2: Configuration in OpenCode Settings**

Configure OpenCode to recognize the plugin:

```json
{
  "plugins": {
    "dr-ralph": {
      "enabled": true,
      "path": "/path/to/dr-ralph/opencode"
    }
  }
}
```

### Verify Installation

```bash
# Check if commands are available
/dr-ralph:help

# Or test the loop mechanism
/ralph-loop "Test basic loop functionality"
```

### Configure Ralph Loop

The loop control is configured in `opencode/.opencode/oh-my-opencode.json`:

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "ralph_loop": {
    "enabled": true,
    "default_max_iterations": 100,
    "state_dir": ".sisyphus"
  }
}
```

---

## Commands

### /dr-ralph:diagnose

Full diagnostic workflow with 5 phases.

```bash
/dr-ralph:diagnose "I've been having back pain for 3 months" --patient "John Doe"
/dr-ralph:diagnose "Headaches and dizziness" --questions 20
```

**Options:**

| Flag | Default | Description |
|------|---------|-------------|
| `--patient <name>` | anonymous | Patient name for case file |
| `--questions <n>` | 15 | Maximum interview questions |
| `--output <dir>` | @notes/ | Directory for case files |
| `--max-iterations <n>` | unlimited | Max iterations before auto-stop |

**Output Files:**
- `@notes/[patient].md` - Running notes (persists across sessions)
- `@notes/[patient]-report-[timestamp].md` - Final SOAP report

**Completion:**
- The workflow completes when the agent outputs `<promise>DONE</promise>`
- Use `/cancel-ralph` to manually stop an active diagnostic

### /dr-ralph:cancel

Cancel an active diagnostic session.

```bash
/dr-ralph:cancel
```

This is an alias for `/cancel-ralph`.

### /cancel-ralph

Cancel the active Dr. Ralph loop.

```bash
/cancel-ralph
```

Stops any active diagnostic workflow and cleans up state files.

### /ralph-loop

Built-in OpenCode command for self-referential loops.

```bash
/ralph-loop "Task description" --max-iterations=50
```

**Options:**
- `--max-iterations <n>`: Maximum loop iterations (default: 100)

**How it works:**
1. Agent works continuously on the task
2. Loop stops when `<promise>DONE</promise>` is detected
3. State is automatically saved in `.sisyphus/` directory

---

## Diagnostic Workflow

### 5-Phase Process

| Phase | Description | Primary Tools |
|-------|-------------|---------------|
| 1. Interview | Medical records intake first, then comprehensive symptom questions | `question`, `Read` |
| 2. Research | Web search for literature, iterative refinement | `websearch_web_search_exa`, `webfetch` |
| 3. Differential | Confidence-based diagnosis (>80% = single, else top 3-5) | Internal analysis |
| 4. Treatment | Action plan with urgency levels and follow-up schedule | Internal analysis |
| 5. Report | SOAP format with executive summary | `Write` |

**Medical Records Handling:**
- Records requested FIRST, before symptom questions
- Files processed one-by-one (never all at once)
- Size check before reading - files >3MB trigger warning
- Tip: Use Adobe Acrobat to split large PDFs into sections <3MB

**Diagnosis Output:**
- **>80% confident:** Single most likely diagnosis
- **Uncertain:** Top 3-5 differential diagnoses ranked by likelihood
- **Transparency:** States uncertainty, distinguishing features, recommended tests

### Patient Case Management

```
@notes/
├── john-doe.md              # Patient notes (grows over time)
├── john-doe-report-20240105.md  # SOAP report
├── jane-smith.md
└── anonymous.md             # Default if no --patient specified
```

- **Persistence:** Notes are regular markdown files, persist across sessions
- **Continuity:** Previous notes auto-read at session start
- **Multiple patients:** Switch cases with `--patient` flag

### SOAP Report Format

```markdown
# Patient Report: John Doe
## Date: 2024-01-05

## Executive Summary
[2-3 paragraph overview]

## Subjective
[Patient's reported symptoms]

## Objective
[Research findings with citations]

## Assessment
[Diagnosis or differential with confidence]

## Plan
[Treatment plan with urgency levels]

## Detailed Findings
[Full interview, research, reasoning]

## References
[Cited sources]
```

---

## Usage Tips

### Safety Limits

Use `--max-iterations` as a safety net:

```bash
/dr-ralph:diagnose "Symptoms" --max-iterations 10
```

### Patient Tracking

Use `--patient` to maintain continuity across sessions:

```bash
/dr-ralph:diagnose "Follow-up on headaches" --patient "John Doe"
```

### Loop Control

- **Complete automatically:** Agent outputs `<promise>DONE</promise>` when finished
- **Cancel manually:** Use `/cancel-ralph` to stop mid-diagnosis
- **Check state:** State files are in `.sisyphus/` directory

---

## When to Use Dr. Ralph

**Good for:**
- Comprehensive medical symptom analysis
- Research-backed diagnostic workflows
- Structured patient intake interviews
- Generating SOAP-format documentation
- Tracking patient cases over multiple sessions

**Not good for:**
- Emergency medical situations (call 911)
- Replacing professional medical advice
- Quick one-off health questions

---

## Architecture

### OpenCode Project Structure

```
dr-ralph/
├── opencode/                          # OpenCode plugin files
│   ├── .opencode/
│   │   └── oh-my-opencode.json       # Ralph loop configuration
│   ├── commands/
│   │   ├── diagnose.md               # /dr-ralph:diagnose
│   │   ├── cancel.md                 # /dr-ralph:cancel
│   │   └── help.md                   # /dr-ralph:help
│   ├── docs/
│   │   └── diagnose-spec.md           # Full diagnostic specification
│   ├── skills/                       # OpenCode skill definitions
│   │   ├── dr-ralph-diagnose.md
│   │   ├── dr-ralph-cancel.md
│   │   └── dr-ralph-help.md
│   └── AGENTS.md                     # Agent knowledge base
├── .sisyphus/                        # OpenCode state directory
│   ├── plans/
│   ├── evidence/
│   └── notepads/
└── README.md                          # Claude Code version (legacy)
```

### How It Works

1. **User Command:** `/dr-ralph:diagnose` initiates the workflow
2. **Loop Control:** OpenCode's `ralph-loop` command manages iteration
3. **State Management:** Automatic state saving in `.sisyphus/`
4. **Completion Detection:** `<promise>DONE</promise>` tag signals completion
5. **Cleanup:** State automatically cleaned up when loop completes

---

## Migration Notes

This section documents the differences between the original Claude Code version and the OpenCode version.

### Differences from Claude Code Version

| Aspect | Claude Code Version | OpenCode Version |
|--------|---------------------|------------------|
| **Framework** | Claude Code | Oh My OpenCode |
| **Installation** | Plugin marketplace | Manual/local setup |
| **Loop Mechanism** | Stop Hook (`hooks/stop-hook.sh`) | Built-in `ralph-loop` command |
| **State Files** | `.claude/dr-ralph-loop.local.md` | `.sisyphus/` (automatic) |
| **Hook System** | Custom bash scripts | OpenCode continuation hooks |
| **Completion Detection** | Parse transcript JSONL | Promise tags in output |
| **Tool Names** | AskUserQuestion, WebSearch, WebFetch | question, websearch_web_search_exa, webfetch |

### Tool Mapping

| Claude Code Tool | OpenCode Tool | Notes |
|------------------|---------------|-------|
| `AskUserQuestion` | `question` | Same functionality, different name |
| `WebSearch` | `websearch_web_search_exa` | Uses Exa search API |
| `WebFetch` | `webfetch` | Same functionality |
| `Read` | `Read` | Identical |
| `Write` | `Write` | Identical |

### Loop Mechanism Changes

**Claude Code (Original):**
- Uses `hooks/stop-hook.sh` bash script
- Registered in `hooks/hooks.json`
- Parses transcript JSONL for promise tags
- Feeds prompt back via JSON response

**OpenCode (New):**
- Uses built-in `/ralph-loop` command
- Configuration in `.opencode/oh-my-opencode.json`
- Detects promise tags in agent output
- Automatic state management via continuation hooks

**Benefits of OpenCode Approach:**
- No custom bash scripts needed
- Cleaner, configuration-based setup
- Better integration with OpenCode ecosystem
- Automatic state management
- Platform-independent

---

## Disclaimer

**THIS SOFTWARE IS FOR INFORMATIONAL AND EDUCATIONAL PURPOSES ONLY.**

Dr. Ralph is an AI-assisted tool and is **NOT** a substitute for professional medical advice, diagnosis, or treatment. Always seek the advice of a qualified healthcare provider with any questions regarding a medical condition. Never disregard professional medical advice or delay seeking it because of something generated by this tool.

**BLEN and the authors of this software assume no responsibility or liability for any errors, omissions, or outcomes arising from the use of this tool.** Use at your own risk.

**Red Flag Handling:** Emergency symptoms (chest pain + SOB, sudden severe headache, etc.) are flagged prominently but the workflow continues to gather complete information.

**Data Privacy:** Remind patients to redact sensitive information from files and answers.

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Learn More

- [Original Ralph technique](https://ghuntley.com/ralph/)
- [Ralph Orchestrator](https://github.com/mikeyobrien/ralph-orchestrator)
- [Ralph Wiggum Plugin](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/ralph-wiggum)
- [Oh My OpenCode](https://github.com/code-yeongyu/oh-my-opencode)
- [OpenCode Documentation](https://opencode.ai/docs/)

---

Run `/dr-ralph:help` for detailed command reference.

---

Built with ❤️ by [BLEN, Inc](https://www.blencorp.com).

## About BLEN

BLEN, Inc is a digital services company that provides Emerging Technology (ML/AI, RPA), Digital Modernization (Legacy to Cloud), and Human-Centered Web/Mobile Design and Development.

## OpenCode Migration

This version of Dr. Ralph has been migrated from Claude Code to OpenCode using the Oh My OpenCode framework. The migration preserves all core functionality while leveraging OpenCode's built-in continuation system and ralph-loop command.

For migration details and comparison with the original version, see the "Migration Notes" section above.
