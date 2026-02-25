# QA Evidence: Task 08 - Create AGENTS.md

## Test Scenario: Verify AGENTS.md file

### Test Steps

```bash
# Step 1: Verify file exists
test -f opencode/AGENTS.md
```

**Result**: ✓ PASS - File exists

### Verification Checklist

- [x] File created: `opencode/AGENTS.md`
- [x] Contains project structure description
- [x] Contains conventions and rules
- [x] Project title: Dr. Ralph OpenCode Plugin
- [x] Project overview (5-phase diagnostic workflow)
- [x] Directory structure (.opencode/ layout)
- [x] OpenCode-specific conventions
  - [x] Commands location: .opencode/commands/
  - [x] Skills location: .opencode/skills/
  - [x] State files: .sisyphus/
- [x] Tool usage (question, websearch_web_search_exa, webfetch)
- [x] Ralph loop integration
- [x] No rules unrelated to diagnosis

### Section Headers Found

- Project Overview
- Directory Structure
- OpenCode-Specific Conventions
  - Commands
  - Skills
  - State Files
- Tool Usage
  - Available Tools
  - Tool Conventions
- Ralph Loop Integration
  - Loop Control
  - Loop State
  - State Management
- Diagnostic Rules
  - Medical Records Handling
  - Interview Phase
  - Research Phase
  - Differential Diagnosis
  - Treatment Planning
  - Report Format (SOAP)
- Safety and Limitations
- Development Workflow

### File Statistics

- Lines: 281
- Size: ~13KB
- Created: 2025-02-24

### Conclusion

All requirements met. AGENTS.md file successfully created with comprehensive OpenCode project rules for Dr. Ralph diagnostic workflow.
