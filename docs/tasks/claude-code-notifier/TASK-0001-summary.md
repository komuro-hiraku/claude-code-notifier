# TASK-0001: Quick Reference Summary

## Log File Locations
- **Primary**: `~/.claude/debug/latest` (symlink to current session)
- **Session History**: `~/.claude/projects/<project>/<session-id>.jsonl`
- **Global History**: `~/.claude/history.jsonl`
- **Session Metadata**: `~/.claude/sessions/<pid>.json`

## State Detection Patterns (Regex)

### Idle (Waiting for User Input)
```regex
query: idle_prompt
```
Pattern: `Getting matching hook commands for Notification with query: idle_prompt`

### Permission Required (Waiting for User Permission)
```regex
query: permission_prompt
```
Pattern: `Getting matching hook commands for Notification with query: permission_prompt`

### User Submitted Input
```regex
user_prompt
```
Pattern: `[3P telemetry] Event dropped.*: user_prompt`

### Tool Execution (Active Processing)
```regex
executePreToolHooks called for tool: (\w+)
```
Pattern: `executePreToolHooks called for tool: <ToolName>`

### Assistant Responding
```regex
Forked agent \[.*\] received message: type=assistant
```

### Task/Subagent Completion
```regex
Forked agent \[.*\] finished: (\d+) messages
```

## State Mapping for Notifications

| Log Pattern | State | Notification Action |
|------------|-------|---------------------|
| `query: idle_prompt` | Idle | "Claude is waiting for input" |
| `query: permission_prompt` | Permission | "Claude needs permission" |
| `user_prompt` | Active | Clear idle notification |
| `executePreToolHooks` | Processing | "Claude is working..." |
| `finished:` | Complete | "Task completed" |

## Recommended Monitoring Strategy

1. **Monitor**: `~/.claude/debug/latest` (symlink to active session)
2. **Method**: Tail file for new lines
3. **Parse**: Apply regex patterns to detect state changes
4. **Track**: State transitions per session
5. **Notify**: On state change matching notification criteria

## Implementation Priority

1. ✅ **High**: `idle_prompt` detection (waiting for user)
2. ✅ **High**: `permission_prompt` detection (waiting for permission)
3. ✅ **Medium**: `user_prompt` detection (clear notifications)
4. ⚠️ **Medium**: Task completion detection (needs more investigation)
5. 🔴 **Low**: Tool execution tracking (nice to have)

## Key Files for TASK-0007

- Monitor file: `~/.claude/debug/latest`
- Log format: Plain text with timestamps
- Line format: `<timestamp> [<level>] [<component>] <message>`
- Update frequency: Real-time as events occur

## Sample Log Lines

```
2026-03-04T07:30:14.704Z [DEBUG] Getting matching hook commands for Notification with query: idle_prompt
2026-03-04T07:17:31.086Z [DEBUG] Getting matching hook commands for Notification with query: permission_prompt
2026-03-04T07:13:59.462Z [WARN] [3P telemetry] Event dropped (no event logger initialized): user_prompt
2026-03-04T07:14:07.586Z [DEBUG] executePreToolHooks called for tool: Grep
2026-03-04T07:15:22.952Z [DEBUG] Forked agent [prompt_suggestion] finished: 2 messages
```

---

**Status**: ✅ Investigation Complete
**Next**: TASK-0007 (Log Parser Implementation)
**Full Report**: [TASK-0001-investigation-report.md](TASK-0001-investigation-report.md)
