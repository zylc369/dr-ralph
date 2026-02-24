# Task 01: Ralph Loop API Documentation

**Date**: 2026-02-24
**Status**: ✅ Verified

## Summary

The `/ralph-loop` command is **available** in Oh My OpenCode as a **built-in slash command** (not a hook). It provides a self-referential development loop that runs continuously until task completion.

## Command Syntax

```bash
/ralph-loop "Build a REST API with authentication"
/ralph-loop "Refactor the payment module" --max-iterations=50
```

## Configuration Schema

Configuration is in `oh-my-opencode.json` under the `ralph_loop` section:

```json
{
  "ralph_loop": {
    "enabled": false,                    // Enable/disable ralph-loop (default: false)
    "default_max_iterations": 100,         // Max loop iterations (1-1000, default: 100)
    "state_dir": "/path/to/state"        // Optional custom state directory
  }
}
```

## State File Location

**Default state directory**: `.sisyphus/` (Oh My OpenCode convention)

**NOTES**:
- The state file path is configurable via the `state_dir` parameter
- This differs from Claude Code's `.claude/dr-ralph-loop.local.md`
- Oh My OpenCode uses `.sisyphus/` as its standard state directory

## Completion Detection Mechanism

The loop detects completion using **promise tags** in the agent's output:

- **Detection**: `<promise>DONE</promise>`
- **Behavior**: Agent works continuously until this tag is detected

**Termination Conditions**:
1. Completion detected via `<promise>DONE</promise>`
2. Max iterations reached (default 100)
3. Explicit cancellation via `/cancel-ralph`
4. `/stop-continuation` command

## Behavior

- Agent works continuously toward the goal
- Auto-continues if agent stops without completion signal
- State is persisted across sessions
- Integration with continuation hooks for automatic resumption

## Related Commands

- `/ulw-loop`: Same as `/ralph-loop` but with ultrawork mode active (parallel agents, background tasks, aggressive exploration)
- `/cancel-ralph`: Cancel active Ralph Loop
- `/stop-continuation`: Stop all continuation mechanisms (ralph-loop, todo-continuation, boulder)

## Verification Steps Performed

1. ✅ Checked for Oh My OpenCode configuration files
   - Found: `~/.config/opencode/oh-my-opencode.json`
   - No project-specific `.oh-my-opencode.json` found

2. ✅ Searched for ralph-loop references
   - Schema confirmed: `ralph_loop` hook in oh-my-opencode schema
   - Documentation verified in features.md

3. ✅ Verified command availability
   - Listed in Built-in Commands table in documentation
   - Scope: `builtin` (not a plugin feature)

## API Summary

| Property | Type | Default | Range | Description |
|----------|-------|---------|--------|-------------|
| `enabled` | boolean | `false` | - | Enable/disable ralph-loop |
| `default_max_iterations` | number | `100` | 1-1000 | Maximum loop iterations |
| `state_dir` | string | `.sisyphus/` | - | Custom state directory path |

## References

- Oh My OpenCode Schema: `assets/oh-my-opencode.schema.json`
- Features Documentation: `docs/reference/features.md` - Commands section
- Repository: https://github.com/code-yeongyu/oh-my-opencode

## Comparison: Claude Code vs Oh My OpenCode

| Feature | Claude Code | Oh My OpenCode |
|----------|-------------|----------------|
| Command | `/dr-ralph-loop` | `/ralph-loop` |
| State Location | `.claude/dr-ralph-loop.local.md` | `.sisyphus/` (configurable) |
| Completion | Stop Hook intercepts exit | Promise tags (`<promise>DONE</promise>`) |
| Hook Type | Stop Hook (Claude Code) | Built-in command + continuation hooks |
| State Persistence | File-based | File-based in `.sisyphus/` |

## Notes for Migration

When migrating from Claude Code's Dr. Ralph Loop to Oh My OpenCode:

1. Update state file references from `.claude/` to `.sisyphus/`
2. Use promise tags (`<promise>DONE</promise>`) instead of Stop Hook JSON responses
3. Enable ralph-loop in config: `{"ralph_loop": {"enabled": true}}`
4. Consider using `/ulw-loop` for ultrawork mode (parallel agents)
