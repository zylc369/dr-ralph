# Dr. Ralph OpenCode Plugin

## Project Overview

Dr. Ralph is an AI-assisted medical diagnostic tool that implements a structured 5-phase diagnostic workflow using OpenCode's self-referential AI loop pattern.

### 5-Phase Diagnostic Workflow

| Phase | Description | Primary Focus |
|-------|-------------|---------------|
| 1. Interview | Medical records intake + comprehensive symptom questions | Data collection via structured interview |
| 2. Research | Literature search and treatment guidelines validation | Evidence gathering from trusted sources |
| 3. Differential Diagnosis | Analyze symptoms to determine most likely diagnosis | Confidence-based diagnostic reasoning |
| 4. Treatment Plan | Develop research-backed action plan | Evidence-based treatment recommendations |
| 5. Report Generation | Create SOAP format documentation | Structured patient case documentation |

### Core Concepts

- **Self-referential loop**: AI iterates through diagnostic phases until completion criteria are met
- **Completion promise**: Use `<promise>DONE</promise>` to signal workflow completion
- **Confidence threshold**: Single diagnosis if >80% confident; otherwise provide 3-5 differential diagnoses
- **Research-backed**: All treatment plans must cite medical literature sources

---

## Directory Structure

```
dr-ralph/
├── .opencode/              # OpenCode plugin root
│   ├── commands/           # Slash command definitions
│   │   ├── diagnose.md     # Main diagnostic workflow command
│   │   ├── cancel.md       # Cancel active diagnostic session
│   │   └── help.md         # Help documentation
│   ├── skills/             # OpenCode skills (optional)
│   │   └── (skill files)   # Reusable skill components
│   ├── config.json         # OpenCode configuration
│   └── prompts/            # Prompt templates (optional)
├── .sisyphus/              # State management directory
│   ├── plans/              # Development plans (READ-ONLY)
│   │   └── dr-ralph-opencode-migration.md
│   ├── notepads/           # Learning and issue tracking
│   │   └── dr-ralph-opencode-migration/
│   │       ├── learnings.md
│   │       ├── issues.md
│   │       ├── decisions.md
│   │       └── problems.md
│   └── evidence/           # QA verification outputs
└── notes/                  # Patient case files (persists across sessions)
    ├── john-doe.md
    └── john-doe-report-20240105.md
```

---

## OpenCode-Specific Conventions

### Commands

- **Location**: `.opencode/commands/`
- **Format**: Markdown with YAML frontmatter
- **File naming**: Lowercase, hyphen-separated (e.g., `diagnose.md`)

**Command template:**
```yaml
---
description: "Command description"
argument-hint: "usage hint [optional]"
allowed-tools: ["Tool1", "Tool2"]
hide-from-slash-command-tool: "true"
---

# Command Title

Detailed instructions...
```

### Skills

- **Location**: `.opencode/skills/`
- **Purpose**: Reusable diagnostic components
- **Usage**: Import skills into commands or other skills
- **Naming convention**: Descriptive name with hyphens (e.g., `medical-research.md`)

### State Files

- **Location**: `.sisyphus/`
- **Plan files**: `.sisyphus/plans/` - READ-ONLY, managed by orchestrator
- **Notepads**: `.sisyphus/notepads/` - Learning, issues, decisions, problems
- **Evidence**: `.sisyphus/evidence/` - QA verification outputs

**IMPORTANT**: Never modify plan files (`.sisyphus/plans/*.md`). These are read-only and managed exclusively by the orchestrator.

---

## Tool Usage

### Available Tools

| Tool | Purpose | Usage in Dr. Ralph |
|------|---------|-------------------|
| `question` | Interactive user questioning | Structured symptom interview, medical record confirmation |
| `websearch_web_search_exa` | Web search for current information | Medical literature research, treatment guidelines |
| `webfetch` | Fetch content from URLs | Retrieve specific medical articles, guidelines |
| `Read` | Read files from filesystem | Process patient medical records, previous notes |
| `Write` | Write files to filesystem | Generate SOAP reports, update patient notes |

### Tool Conventions

1. **Question tool**: Use for all patient interactions
   - Always ask one question at a time
   - Clarify ambiguous responses before proceeding
   - Request medical records BEFORE symptom questions

2. **websearch_web_search_exa**: Use for research phase
   - Search for peer-reviewed sources
   - Prioritize medical guidelines and consensus statements
   - Use specific, medical-focused queries

3. **webfetch**: Use to retrieve full articles
   - Follow up search results with targeted fetching
   - Capture citations and references
   - Verify source credibility

4. **Read**: Process files sequentially
   - Check file size before reading (>3MB triggers warning)
   - Process one file at a time, not all simultaneously
   - Suggest splitting large PDFs with Adobe Acrobat

5. **Write**: Generate structured output
   - SOAP format for patient reports
   - Markdown format for notes and reports
   - Include timestamps and patient identifiers

---

## Ralph Loop Integration

### Loop Control

Dr. Ralph uses OpenCode's continuation system to iterate through diagnostic phases:

```
User Request → Process Phase → Check Completion → Continue/Exit
```

**Completion Detection:**
- Primary: `<promise>DONE</promise>` in output
- Secondary: Maximum iteration count (safety limit)
- Manual: User cancels with `/dr-ralph:cancel`

### Loop State

The loop maintains state through:
- **Phase tracking**: Current diagnostic phase (1-5)
- **Question count**: Track interview progress
- **Confidence level**: Track diagnostic certainty
- **Iteration counter**: Safety mechanism for unlimited loops

### State Management

All persistent state is stored in:
- **Patient notes**: `notes/[patient].md` - Cross-session persistence
- **Session state**: Managed by OpenCode continuation
- **Configuration**: Set via command flags at workflow start

---

## Diagnostic Rules

### Medical Records Handling

1. **Priority**: Always request medical records FIRST, before symptom questions
2. **Size limit**: Warn before reading files >3MB
3. **Sequential processing**: Process one file at a time
4. **Redaction**: Remind patients to redact sensitive information

### Interview Phase

- Ask one question at a time
- Clarify ambiguous responses
- Document all responses in patient notes
- Target 15 questions by default (configurable)

### Research Phase

- Search for peer-reviewed sources
- Prioritize recent guidelines (last 5-10 years)
- Verify source credibility
- Capture complete citations

### Differential Diagnosis

- **Confidence >80%**: Output single most likely diagnosis
- **Confidence <80%**: Output 3-5 differential diagnoses, ranked by likelihood
- Include reasoning and distinguishing features
- State uncertainty transparently

### Treatment Planning

- Evidence-based recommendations with urgency levels
- Specific follow-up schedule
- Recommended diagnostic tests if needed
- Lifestyle recommendations when applicable

### Report Format (SOAP)

```markdown
# Patient Report: [Patient Name]
## Date: [Timestamp]

## Executive Summary
[2-3 paragraph overview]

---

## Subjective
[Patient-reported symptoms and history]

## Objective
[File analysis and research findings]

## Assessment
[Diagnosis or differential with confidence]

## Plan
[Structured treatment plan]
[Follow-up schedule]

---

## Detailed Findings
[Full interview record]
[Research analysis]
[Differential reasoning]

## References
[Cited sources]
```

---

## Safety and Limitations

### What Dr. Ralph Does

- Comprehensive symptom analysis
- Research-backed diagnostic workflows
- Structured patient documentation
- Evidence-based treatment recommendations

### What Dr. Ralph Does NOT Do

- Provide emergency medical advice
- Replace professional medical judgment
- Make definitive diagnoses
- Prescribe medications or treatments

### Red Flag Handling

Flag emergency symptoms prominently but continue gathering complete information:
- Chest pain + shortness of breath
- Sudden severe headache
- Neurological deficits
- Severe abdominal pain

---

## Development Workflow

When working on Dr. Ralph:

1. **Read the plan**: Check `.sisyphus/plans/` for task assignments
2. **Follow conventions**: Adhere to directory structure and naming rules
3. **Document learnings**: Append to `.sisyphus/notepads/*/learnings.md`
4. **Track issues**: Append to `.sisyphus/notepads/*/issues.md`
5. **Record decisions**: Append to `.sisyphus/notepads/*/decisions.md`
6. **Verify work**: Run QA scenarios, save evidence to `.sisyphus/evidence/`

**IMPORTANT**: Never modify plan files. The orchestrator manages plan state exclusively.
