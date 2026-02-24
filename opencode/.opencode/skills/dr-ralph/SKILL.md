---
name: dr-ralph
description: AI-assisted medical diagnostic workflow with 5-phase structured analysis (Interview → Research → Differential Diagnosis → Treatment → Report) using Claude Code's Ralph technique for iterative self-referential loops.
---

# What I Do

Dr. Ralph provides comprehensive medical diagnostic workflows through structured 5-phase analysis:

1. **Medical Records Intake** - Request and analyze existing medical documentation first
2. **Symptom Interview** - Interactive questioning using AskUserQuestion tool (max 15 questions)
3. **Research** - Web search for literature, evidence, and treatment protocols with iterative refinement
4. **Differential Diagnosis** - Confidence-based analysis (>80% = single diagnosis, else top 3-5 differentials)
5. **Treatment & Reporting** - Research-backed action plan in SOAP format with executive summary

# When to Use Me

Use Dr. Ralph when you need:

- Comprehensive medical symptom analysis with research backing
- Structured patient intake interviews
- Differential diagnosis development with confidence levels
- SOAP-format documentation generation
- Tracking patient cases across multiple sessions

**Do NOT use for:**
- Emergency medical situations (call 911)
- Quick one-off health questions
- Replacing professional medical advice

# 5-Phase Workflow Overview

```
Interview (Phase 1)  →  Research (Phase 2)  →  Differential (Phase 3)  →  Treatment (Phase 4)  →  Report (Phase 5)
     ↓                          ↓                        ↓                        ↓                       ↓
Medical Records +    Web search for      Analyze symptoms      Research-backed        SOAP format with
Symptom Questions    literature &         and research to       action plan with        executive summary
                     evidence              determine diagnosis    urgency levels
```

I continue iterating through phases until reaching a convincing diagnosis with complete treatment plan.

# Key Features

## Ralph Loop Integration

Built on the Ralph technique using Claude Code's Stop hook system:
- Automatic iteration until diagnostic completion detected
- State tracking across iterations
- Completion via `<promise>` phrase or max-iteration limit

## Question Tool

Structured medical interviews using `AskUserQuestion`:
- Clear, targeted questions with multiple-choice or open-ended responses
- Medical records requested FIRST before symptom questions
- Context-dependent sensitive topics (mental health, substances, sexual health)
- Auto-read previous patient notes from `@notes/` directory

## Web Tools

Research capabilities for evidence-based diagnostics:
- AI judges source relevance (not restricted to specific sites)
- Iterative refinement: Search → Analyze → Refine → Search again
- Hypothesis-driven searches to confirm/rule out differential diagnoses
- Inline citations: "According to Mayo Clinic [link]..."

## Medical Records Handling

- Records processed ONE BY ONE (never all at once)
- File size check before reading
- Large file handling (>3MB): warn user, offer to skip
- Tip: Use Adobe Acrobat to split large PDFs into sections under 3MB

## Patient Case Management

- File-based persistence in `@notes/` directory
- Single file per patient grows over time
- Previous notes auto-read at session start
- Multiple concurrent cases supported

## Confidence-Based Output

- **>80% confident:** Single most likely diagnosis
- **Uncertain:** Top 3-5 differential diagnoses ranked by likelihood
- Full transparency: states uncertainty, distinguishing features, recommended tests

## SOAP Report Format

- Executive summary (2-3 paragraphs)
- Subjective: Patient's reported symptoms
- Objective: Research findings with citations
- Assessment: Diagnosis or differential with confidence
- Plan: Treatment plan with urgency levels and follow-up schedule
- Detailed Findings: Full interview, research, and reasoning
- References: Cited sources

# Usage

```
/dr-ralph:diagnose "Symptom description" --patient "Patient Name"
```

The complete diagnostic logic is implemented in the `/dr-ralph:diagnose` command.

# Important Notes

**Required Disclaimers:**
- "This is an AI-assisted tool, not a substitute for professional medical advice"
- "Always consult a qualified healthcare provider for medical concerns"
- "Remind user to redact sensitive information from files and answers"

**Red Flag Protocol:**
- Emergency symptoms flagged prominently
- Continue interview but note concerns prominently in report
