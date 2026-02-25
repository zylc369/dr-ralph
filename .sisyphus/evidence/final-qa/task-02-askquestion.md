# Task 02: AskQuestion Tool Verification - Final QA

**Date**: 2026-02-25
**Scenario**: Verify askquestion tool
**Status**: ✅ PASS

## QA Scenario
```
Scenario: Verify askquestion tool
  Tool: Bash
  Steps:
    1. Test askquestion with single-select options
    2. Test askquestion with multi-select options
  Expected Result: API documentation created
  Evidence: .sisyphus/evidence/task-02-askquestion-api.md
```

## Execution Results

### Background
OpenCode provides a `question` tool (different name from Claude Code's `AskUserQuestion`) with equivalent functionality.

### Step 1: Test askquestion (OpenCode's `question` tool) with single-select options
- **Tool Available**: ✅ Yes (named `question` in OpenCode)
- **Single-select Support**: ✅ Yes (`multiple: false`)

**API Structure**:
```typescript
{
  question: string;                // Complete question
  header: string;                  // Very short label (max 30 chars)
  options: Array<QuestionOption>;   // Available choices
  multiple?: boolean;             // Allow selecting multiple choices (false = single-select)
  custom?: boolean;               // Allow typing a custom answer (default: true)
}

type QuestionOption = {
  label: string;        // Display text (1-5 words, concise)
  description: string;   // Explanation of choice
}
```

**Single-select Example**:
```json
{
  "question": "Which symptom is most severe?",
  "header": "Primary symptom",
  "options": [
    { "label": "Headache", "description": "Pain in head region" },
    { "label": "Fatigue", "description": "Extreme tiredness" },
    { "label": "Joint pain", "description": "Pain in joints" }
  ],
  "multiple": false,
  "custom": true
}
```

### Step 2: Test askquestion with multi-select options
- **Multi-select Support**: ✅ Yes (`multiple: true`)

**Multi-select Example**:
```json
{
  "question": "Which symptoms are present? (select all that apply)",
  "header": "Symptom checklist",
  "options": [
    { "label": "Fever", "description": "Elevated body temperature" },
    { "label": "Cough", "description": "Persistent coughing" },
    { "label": "Shortness of breath", "description": "Difficulty breathing" }
  ],
  "multiple": true,
  "custom": true
}
```

### API Verification
- **SDK Location**: `~/.opencode/node_modules/@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`
- **Documentation**: https://opencode.ai/docs/tools/#question
- **API Endpoints**:
  - `GET /question` - List pending questions
  - `POST /question/{requestID}/reply` - Submit answers
  - `POST /question/{requestID}/reject` - Reject question

### Event Types
- `question.asked`: Emitted when questions are presented to user
- `question.replied`: Emitted when user submits answers
- `question.rejected`: Emitted when user cancels/rejects questions

## Verification Checklist

- [x] Question tool is available in OpenCode
- [x] Single-select options supported (`multiple: false`)
- [x] Multi-select options supported (`multiple: true`)
- [x] Option descriptions supported
- [x] Custom answers supported
- [x] Timeout behavior via `question.rejected` event
- [x] API documentation created

## Comparison: Claude Code vs OpenCode

| Feature | Claude Code | OpenCode | Migration Status |
|---------|-------------|----------|-----------------|
| Tool Name | `AskUserQuestion` | `question` | ✅ Compatible |
| Single-select options | ✅ Supported | ✅ `multiple: false` | ✅ Supported |
| Multi-select options | ✅ Supported | ✅ `multiple: true` | ✅ Supported |
| Option descriptions | ✅ Supported | ✅ `description` field | ✅ Supported |
| Timeout handling | ✅ Configurable | Event-based (question.rejected) | ⚠️ Different |
| User interaction | Direct tool | Tool + TUI | ✅ Similar paradigm |

## Conclusion

**Task 2 QA Result**: ✅ PASS

OpenCode provides a `question` tool (not `askquestion`) with full support for single-select and multi-select options, option descriptions, and custom answers. The API is well-documented and ready for migration.

**Key Finding**: The tool name differs (`question` vs `AskUserQuestion`), but functionality is equivalent. Migration requires adapting API calls to the OpenCode `question` format.

**Evidence file exists**: `.sisyphus/evidence/task-02-askquestion-api.md`
