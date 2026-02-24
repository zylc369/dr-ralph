# Task 10: Create opencode/README.md

**Date**: 2026-02-24
**Status**: ✅ Completed

## Summary

Created `opencode/README.md` with comprehensive OpenCode plugin documentation. The README includes installation instructions, usage examples, dependency requirements, and migration notes documenting differences from the Claude Code version.

## Implementation Details

### File Created

**Location**: `opencode/README.md`
**Size**: 456 lines
**Format**: Markdown with complete documentation structure

### Content Structure

The README includes the following sections:

1. **Title and Badge** - OpenCode version identification
2. **Quick Start** - Immediate usage example
3. **Installation**
   - Prerequisites (Oh My OpenCode)
   - Install Oh My OpenCode
   - Install Dr. Ralph Plugin
   - Verify Installation
4. **Commands**
   - /dr-ralph:diagnose
   - /dr-ralph:cancel
   - /cancel-ralph
   - /ralph-loop
5. **Diagnostic Workflow**
   - 5-Phase Process
   - Patient Case Management
   - SOAP Report Format
6. **Usage Tips**
7. **When to Use Dr. Ralph**
8. **Architecture**
9. **Migration Notes**
   - Differences from Claude Code Version
   - Tool Mapping
   - Loop Mechanism Changes
10. **Disclaimer**
11. **License**
12. **Learn More**

### Key Adaptations from Claude Code Version

| Category | Claude Code | OpenCode |
|----------|-------------|----------|
| **Framework** | Claude Code | Oh My OpenCode |
| **Loop Control** | Stop Hook (stop-hook.sh) | Built-in ralph-loop command |
| **State Files** | `.claude/dr-ralph-loop.local.md` | `.sisyphus/` (automatic) |
| **Installation** | Plugin marketplace | Manual/local with Oh My OpenCode |
| **Tool Names** | AskUserQuestion, WebSearch, WebFetch | question, websearch_web_search_exa, webfetch |

### Installation Instructions

**Prerequisites:**
- Oh My OpenCode framework
- OpenCode environment
- Internet connection for web search tools

**Install Oh My OpenCode:**
```bash
git clone https://github.com/code-yeongyu/oh-my-opencode.git ~/.opencode
```

**Configuration:**
```json
{
  "ralph_loop": {
    "enabled": true,
    "default_max_iterations": 100,
    "state_dir": ".sisyphus"
  }
}
```

### Usage Examples

**Quick Start:**
```bash
/dr-ralph:diagnose "Persistent fatigue and joint pain" --patient "John Doe"
```

**With Options:**
```bash
/dr-ralph:diagnose "I've been having back pain for 3 months" --patient "John Doe"
/dr-ralph:diagnose "Headaches and dizziness" --questions 20
/dr-ralph:diagnose "Symptoms" --max-iterations 10
```

**Cancel:**
```bash
/cancel-ralph
```

**Direct Loop:**
```bash
/ralph-loop "Task description" --max-iterations=50
```

### Dependency Requirements

Documented dependencies:
1. **Oh My OpenCode Framework** - Required for ralph-loop command
2. **OpenCode Environment** - Runtime environment
3. **Web Search Tools** - For medical literature research

### Migration Notes Section

The "Migration Notes" section provides:

1. **Differences from Claude Code Version** - Comparison table
2. **Tool Mapping** - Mapping between Claude Code and OpenCode tools
3. **Loop Mechanism Changes** - Technical differences explained
4. **Benefits of OpenCode Approach** - Advantages listed

## Verification

### QA Verification

```bash
# Test 1: File exists
$ test -f opencode/README.md && echo "File exists: YES" || echo "File exists: NO"
File exists: YES

# Test 2: Contains installation instructions
$ grep -q "Install" opencode/README.md && echo "Contains 'Install': YES" || echo "Contains 'Install': NO"
Contains 'Install': YES

# Test 3: Key sections present
$ grep -E "(Oh My OpenCode|dr-ralph:diagnose|Migration Notes)" opencode/README.md
> **OpenCode Version:** Migrated from Claude Code with Oh My OpenCode integration
- [Install Oh My OpenCode](#install-oh-my-opencode)
- [/dr-ralph:diagnose](#dr-ralphdiagnose)
- [Migration Notes](#migration-notes)
...
```

✅ All verification tests passed

### Content Verification

- ✅ Installation instructions present (Oh My OpenCode + Dr. Ralph)
- ✅ Usage examples provided (Quick Start + Commands)
- ✅ Dependency requirements documented
- ✅ Differences from Claude Code version explained
- ✅ Tool mapping provided (AskUserQuestion → question, etc.)
- ✅ Loop mechanism changes documented
- ✅ Architecture section updated for OpenCode
- ✅ Command references updated (dr-ralph:diagnose, dr-ralph:cancel)
- ✅ Tool differences noted (websearch_web_search_exa, webfetch)
- ✅ Existing features preserved (5-phase workflow, SOAP format, patient notes)

## References

- **Original README**: `README.md` - Claude Code version source
- **Learning Documentation**: `LEARNING-CN.md` - Migration notes and technical details
- **Task 9 Evidence**: `.sisyphus/evidence/task-09-loop-control.md` - Loop control implementation
- **Oh My OpenCode**: https://github.com/code-yeongyu/oh-my-opencode

## Acceptance Criteria

- [x] File `opencode/README.md` exists
- [x] Contains installation instructions
- [x] Contains usage examples
- [x] Contains dependency requirements
- [x] Documents differences from Claude Code version

## Task Status: ✅ COMPLETE

The `opencode/README.md` has been successfully created with comprehensive documentation for the OpenCode version of Dr. Ralph, including all required sections and migration notes.
