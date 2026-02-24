# Task 02: AskQuestion Tool API Verification

**Date**: 2026-02-24
**Status**: ✅ Verified
**Finding**: OpenCode DOES provide a `question` tool (different name from Claude Code's `AskUserQuestion`)

## Summary

After exhaustive investigation of OpenCode and Oh-My-OpenCode ecosystems, **no `askquestion` tool exists**, but **OpenCode DOES provide a `question` tool** with similar functionality. The `AskUserQuestion` tool from Claude Code has an equivalent in OpenCode named `question`.

## Investigation Methods

1. ✅ Searched OpenCode SDK type definitions (`@opencode-ai/sdk/dist/gen/types.gen.d.ts`)
2. ✅ Examined OpenCode plugin API (`@opencode-ai/plugin/dist/index.d.ts`)
3. ✅ Reviewed Oh-My-OpenCode configuration schema (`oh-my-opencode.schema.json`)
4. ✅ Analyzed Oh-My-OpenCode features documentation
5. ✅ Checked available hooks and tool definitions
6. ✅ Searched Oh-My-OpenCode repository for user interaction tools

## Findings

### 1. Claude Code's AskUserQuestion (What We're Migrating From)

The dr-ralph plugin currently uses `AskUserQuestion` from Claude Code:

**Location**: `commands/diagnose.md:4`
```yaml
allowed-tools: ["Bash(...)", "AskUserQuestion", "Write", "Read", ...]
```

**Expected behavior** (from diagnose.md):
- Comprehensive medical intake questions
- Single-select or multi-select options
- User prompts with option descriptions

**However**: This is a Claude Code-specific tool, not available in OpenCode.

### 2. OpenCode's Tool Ecosystem

OpenCode provides tools through these mechanisms:

**SDK Tools**:
- File operations: `read`, `write`, `edit`, `bash`, `glob`, `grep`
- LSP tools: `lsp_goto_definition`, `lsp_find_references`, `lsp_symbols`, `lsp_diagnostics`, `lsp_rename`
- AST tools: `ast_grep_search`, `ast_grep_replace`
- Web tools: `websearch_web_search_exa`, `webfetch`
- Session tools: `session_list`, `session_read`, `session_search`, `session_info`
- Background tools: `background_output`, `background_cancel`, `call_omo_agent`
- Media tools: `look_at`
- MCP tools: `skill_mcp`

**Hooks** (via `@opencode-ai/plugin`):
- Event hooks: `event`, `config`, `chat.message`, `chat.params`, `chat.headers`
- Permission hooks: `permission.ask`, `tool.execute.before`, `tool.execute.after`
- Shell hooks: `shell.env`
- Experimental hooks: `experimental.session.compacting`, `experimental.text.complete`, etc.

**OpenCode provides a `question` tool**: See section 4 below for detailed API documentation.
### 4. OpenCode's `question` Tool API

**OpenCode DOES have a `question` tool** for user interaction.

**Location in SDK**: `@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`

**Documentation**: https://opencode.ai/docs/tools/#question

#### Type Definitions

**QuestionOption**:
```typescript
{
  label: string;        // Display text (1-5 words, concise)
  description: string;   // Explanation of choice
}
```

**QuestionInfo**:
```typescript
{
  question: string;                // Complete question
  header: string;                  // Very short label (max 30 chars)
  options: Array<QuestionOption>;   // Available choices
  multiple?: boolean;             // Allow selecting multiple choices
  custom?: boolean;               // Allow typing a custom answer (default: true)
}
```

**QuestionRequest**:
```typescript
{
  id: string;
  sessionID: string;
  questions: Array<QuestionInfo>;  // Questions to ask
  tool?: {
    messageID: string;
    callID: string;
  };
}
```

**QuestionAnswer**: `Array<string>` // Selected option labels or custom answers

#### API Endpoints

**List pending questions**:
- **Endpoint**: `GET /question`
- **Response**: `200: Array<QuestionRequest>`

**Submit answers**:
- **Endpoint**: `POST /question/{requestID}/reply`
- **Body**: `{ answers: Array<QuestionAnswer> }`
- **Response**: `200: boolean`
- **Note**: User answers in order of questions (each answer is an array of selected labels)

**Reject question**:
- **Endpoint**: `POST /question/{requestID}/reject`
- **Response**: `200: boolean`

#### Features

- ✅ **Single-select**: Set `multiple: false` (or omit)
- ✅ **Multi-select**: Set `multiple: true`
- ✅ **Option labels**: Short, concise display text (1-5 words)
- ✅ **Option descriptions**: Detailed explanation of each choice
- ✅ **Custom answers**: Users can type custom responses (configurable)
- ✅ **Multiple questions**: Can ask several questions in one request
- ✅ **Short headers**: Max 30 chars for question section labels

#### Event Types

- `question.asked`: Emitted when questions are presented to user
- `question.replied`: Emitted when user submits answers
- `question.rejected`: Emitted when user cancels/rejects questions

#### Usage Example

**Single-select question**:
, "{", 
question": "Which symptom is most severe?",
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

**Multi-select question**:
, "{", 
question": "Which symptoms are present? (select all that apply)",
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

### 3. Oh-My-OpenCode's Approach

Oh-My-OpenCode provides agent orchestration but does not add a user question tool:

**Agents**:
- Sisyphus (orchestrator)
- Prometheus (interview-based planning)
- Oracle (architecture consultant)
- Librarian (documentation search)
- Explore (codebase grep)
- Hephaestus (deep worker)
- And others...

**Prometheus Interview Mode**:
- Can ask users questions during planning phase
- This is **not a tool** but an agent behavior
- Uses natural language conversation, not structured prompts
- Accessible via Tab key or `@plan` command

**Built-in Commands**:
- `/init-deep`, `/ralph-loop`, `/ulw-loop`, `/refactor`, `/start-work`, etc.
- None provide user question functionality

## Comparison: Claude Code vs OpenCode

| Feature | Claude Code | OpenCode | Migration Status |
|----------|-------------|-----------|-----------------|
| Tool Name | `AskUserQuestion` | `question` | ✅ Compatible |
| Single-select options | ✅ Supported | ✅ `multiple: false` | ✅ Supported |
| Multi-select options | ✅ Supported | ✅ `multiple: true` | ✅ Supported |
| Option descriptions | ✅ Supported | ✅ `description` field | ✅ Supported |
| Timeout handling | ✅ Configurable | Event-based (question.rejected) | ⚠️ Different |
| User interaction | Direct tool | Tool + TUI | ✅ Similar paradigm |

## Alternative Approaches

OpenCode **DOES** have a `question` tool with similar functionality to Claude Code's `AskUserQuestion`. Migration is **STRAIGHTFORWARD** by adapting to the `question` tool API:

### Option 1: Use OpenCode's Built-in `question` Tool (Recommended)
- Directly use OpenCode's `question` tool
- Adapt API calls from `AskUserQuestion` to `question`
- Map single-select/multi-select using `multiple` field
- **Pros**: Built-in, officially supported, no extra development
- **Cons**: API differences (different field names, structure)

### Option 2: Natural Language Questions (Fallback)
- Use Prometheus agent for interview phase
- Natural language interview
- Captures intent and clarifications
- **Pros**: Designed for interviewing
- **Cons**: Requires switching agents, not a tool

### Option 3: Prometheus Interview Mode
- Write questions to a file
- Ask user to edit file with answers
- Read file back for responses
- **Pros**: Fully supported
- **Cons**: Awkward workflow, poor UX

### Option 4: Build a Custom Tool Wrapper (Optional)
- Create a wrapper around `question` tool that mimics `AskUserQuestion` API
- Use OpenCode's plugin or MCP integration
- Translate `AskUserQuestion` format to `question` format automatically
- **Pros**: No changes needed to existing dr-ralph commands
- **Cons**: Additional layer of abstraction, maintenance overhead

## Migration Status: ✅ BLOCKER RESOLVED

**OpenCode's `question` tool provides equivalent functionality to Claude Code's `AskUserQuestion`. The dr-ralph migration can proceed with minimal API adaptation.**

The diagnose command's structured user questions ARE supported:
- Medical records intake
- Symptom collection
- File handling decisions
- Multi-step interview process

With the `question` tool, the workflow will be:
- Equally user-friendly with proper TUI support
- Structured and type-safe (TypeScript types defined)
- Robust with event-based feedback
- Similar user experience to Claude Code (single/multi-select, options, descriptions)

## Migration Recommendations

1. **Immediate**: Update dr-ralph diagnose command to use `question` tool instead of `AskUserQuestion`
2. **Short-term**: Adapt existing question data structures to `QuestionInfo` format
3. **Optional**: Create a compatibility wrapper if keeping `AskUserQuestion` API is preferred

## References

- OpenCode SDK v2: `~/.opencode/node_modules/@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`
- OpenCode Tools Documentation: https://opencode.ai/docs/tools/#question
- OpenCode GitHub: https://github.com/anomalyco/opencode
- Oh-My-OpenCode Repo: https://github.com/code-yeongyu/oh-my-opencode
- Schema: https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json

## Verification Status

- ✅ OpenCode SDK v2 explored - `question` tool found
- ✅ Question tool type definitions documented (QuestionOption, QuestionInfo, QuestionRequest)
- ✅ API endpoints documented (GET /question, POST /question/{id}/reply, POST /question/{id}/reject)
- ✅ Features verified: single-select, multi-select, option labels/descriptions, custom answers
- ✅ Event types documented (question.asked, question.replied, question.rejected)

**Updated Conclusion**: OpenCode **DOES** provide a `question` tool (not `askquestion`) with equivalent functionality to Claude Code's `AskUserQuestion`. Migration is **STRAIGHTFORWARD** - requires adapting API calls from `AskUserQuestion` to OpenCode's `question` tool format. No blocker exists for the dr-ralph migration.
