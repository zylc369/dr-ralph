---
description: "Cancel active Dr. Ralph session"
argument-hint: ""
tools:
  Read: true
  Bash: true
hide-from-slash-command-tool: "true"
---

# Cancel Dr. Ralph

To cancel the Dr. Ralph diagnostic session:

1. Check if `.sisyphus/ralph-loop-state.md` exists using Bash: `test -f .sisyphus/ralph-loop-state.md && echo "EXISTS" || echo "NOT_FOUND"`

2. **If NOT_FOUND**: Say "No active Dr. Ralph session found."

3. **If EXISTS**:
   - Use `/cancel-ralph` command to stop the active Ralph Loop
   - Remove the state file using Bash: `rm .sisyphus/ralph-loop-state.md`
   - Report: "Cancelled Dr. Ralph diagnostic session"
