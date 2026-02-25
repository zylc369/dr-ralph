# Task 08: AGENTS.md File Verification - Final QA

**Date**: 2026-02-25
**Scenario**: Verify AGENTS.md file
**Status**: ✅ PASS

## QA Scenario
```
Scenario: Verify AGENTS.md file
  Tool: Bash
  Steps:
    1. test -f opencode/AGENTS.md
  Expected Result: File exists
  Evidence: .sisyphus/evidence/task-08-agents.md
```

## Execution Results

### Step 1: test -f opencode/AGENTS.md
- **Result**: ✅ PASS
- **Command**: `test -f opencode/AGENTS.md`
- **Output**: File exists

## File Overview

The AGENTS.md file is the knowledge base for the OpenCode plugin. It contains:

- ✅ Project Overview
- ✅ 5-Phase Diagnostic Workflow
- ✅ Core Concepts (self-referential loop, completion promise, confidence threshold, research-backed)
- ✅ Directory Structure
- ✅ OpenCode-Specific Conventions (commands, skills, state files)
- ✅ Tool Usage (available tools, tool conventions)
- ✅ Ralph Loop Integration (loop control, loop state, state management)
- ✅ Diagnostic Rules (medical records handling, interview phase, research phase, differential diagnosis, treatment planning, report format)
- ✅ Safety and Limitations
- ✅ Development Workflow

## Key Sections

### Project Overview
- Dr. Ralph OpenCode Plugin description
- 5-phase diagnostic workflow table
- Core concepts explanation

### Directory Structure
```
dr-ralph/
├── .opencode/              # OpenCode plugin root
│   ├── commands/           # Slash command definitions
│   ├── skills/             # OpenCode skills
│   ├── config.json         # OpenCode configuration
│   └── prompts/            # Prompt templates (optional)
├── .sisyphus/              # State management directory
│   ├── plans/              # Development plans (READ-ONLY)
│   ├── notepads/           # Learning and issue tracking
│   └── evidence/           # QA verification outputs
└── notes/                  # Patient case files
```

### OpenCode-Specific Conventions
- Commands format and location
- Skills usage and location
- State files locations and rules
- IMPORTANT: Never modify plan files

### Tool Usage
- Available tools table (question, websearch_web_search_exa, webfetch, Read, Write)
- Tool conventions (question tool usage, websearch usage, webfetch usage, Read usage, Write usage)

### Ralph Loop Integration
- Loop control description
- Completion detection (promise tags, iteration limits, manual cancellation)
- Loop state (phase tracking, question count, confidence level, iteration counter)
- State management (patient notes, session state, configuration)

### Diagnostic Rules
- Medical Records Handling (priority, size limit, sequential processing, redaction)
- Interview Phase (one question at a time, clarify responses, document all)
- Research Phase (peer-reviewed sources, recent guidelines, verify credibility)
- Differential Diagnosis (confidence thresholds, transparency)
- Treatment Planning (evidence-based, urgency levels, follow-up schedule)
- Report Format (SOAP structure)

### Safety and Limitations
- What Dr. Ralph Does
- What Dr. Ralph Does NOT Do
- Red Flag Handling

### Development Workflow
1. Read the plan
2. Follow conventions
3. Document learnings
4. Track issues
5. Record decisions
6. Verify work

**IMPORTANT**: Never modify plan files. The orchestrator manages plan state exclusively.

## Verification Checklist

- [x] File `opencode/AGENTS.md` exists
- [x] Contains project overview
- [x] Contains 5-phase diagnostic workflow
- [x] Contains directory structure
- [x] Contains OpenCode-specific conventions
- [x] Contains tool usage documentation
- [x] Contains Ralph loop integration details
- [x] Contains diagnostic rules
- [x] Contains safety and limitations
- [x] Contains development workflow instructions

## Conclusion

**Task 8 QA Result**: ✅ PASS (1/1 checks passed)

The AGENTS.md file exists and provides comprehensive documentation for the Dr. Ralph OpenCode plugin. It serves as the agent knowledge base, covering all aspects of the project including the 5-phase workflow, conventions, tool usage, loop integration, and development workflow.

**Key Features**:
- Complete project documentation
- Clear directory structure
- OpenCode-specific conventions
- Comprehensive tool usage guide
- Ralph loop integration details
- Diagnostic rules and safety guidelines
- Development workflow instructions
- Important reminders about plan file management
