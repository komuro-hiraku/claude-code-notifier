# TASK-0001: Claude Code Log File Investigation Report

## Investigation Summary

**Date**: 2026-03-26
**Task**: TASK-0001 - Claude Code Log File Investigation
**Status**: Completed
**Investigation Time**: ~1 hour

## Executive Summary

Claude Code maintains comprehensive logging infrastructure in `~/.claude/` directory. The investigation successfully identified:
- Primary log location: `~/.claude/debug/` directory with session-based text logs
- Session conversation history: `~/.claude/projects/` with JSONL format
- Global interaction history: `~/.claude/history.jsonl`
- Three distinct state patterns: `idle_prompt`, `permission_prompt`, and `user_prompt`

## 1. Log File Locations

### 1.1 Primary Debug Logs
**Location**: `~/.claude/debug/`

**Characteristics**:
- Format: Plain text with timestamped entries
- Naming: UUID-based session identifiers (e.g., `462d394e-a6cb-4a2f-9616-be7973b8309a.txt`)
- Size: Ranges from ~6KB to ~9.8MB depending on session length
- Rotation: New file per session
- Symlink: `latest` points to most recent session

**Example entries**:
```
2026-03-04T07:13:59.462Z [WARN] [3P telemetry] Event dropped (no event logger initialized): user_prompt
2026-03-04T07:14:07.586Z [DEBUG] executePreToolHooks called for tool: Grep
2026-03-04T07:30:14.704Z [DEBUG] Getting matching hook commands for Notification with query: idle_prompt
```

### 1.2 Session Conversation History
**Location**: `~/.claude/projects/-Users-hiraku-komuro-Develop-<project>/<session-id>.jsonl`

**Characteristics**:
- Format: JSONL (JSON Lines)
- One JSON object per line
- Contains complete conversation history with:
  - User messages
  - Assistant responses (including thinking blocks)
  - Tool calls and results
  - Timestamp and session metadata
  - Token usage statistics

**Key fields**:
- `type`: "user" | "assistant" | "file-history-snapshot"
- `message.role`: "user" | "assistant"
- `message.content`: Array of content blocks
- `timestamp`: ISO 8601 format
- `sessionId`: UUID
- `cwd`: Current working directory
- `parentUuid`: Links messages in conversation chain

### 1.3 Global History
**Location**: `~/.claude/history.jsonl`

**Characteristics**:
- Format: JSONL
- Size: ~208KB (50+ entries shown)
- Contains brief display text and metadata for all sessions
- Tracks project paths and session IDs

### 1.4 Session Metadata
**Location**: `~/.claude/sessions/<pid>.json`

**Example content**:
```json
{
  "pid": 90424,
  "sessionId": "2bf27e38-eda4-4689-b41f-85babd0c3d49",
  "cwd": "/Users/hiraku.komuro/Develop/claude/notification-claude",
  "startedAt": 1774509217573,
  "kind": "interactive",
  "entrypoint": "cli"
}
```

### 1.5 File History
**Location**: `~/.claude/file-history/<session-id>/`

**Characteristics**:
- Tracks file modifications during session
- Hash-based file naming (e.g., `08267893c9cabd41@v2`)
- Used for file versioning and rollback

## 2. Log Format Analysis

### 2.1 Debug Log Format
**Pattern**: `<timestamp> [<level>] [<component>] <message>`

**Components**:
- Timestamp: ISO 8601 with milliseconds
- Level: DEBUG, INFO, WARN, ERROR
- Component: Optional tags like `[API:auth]`, `[3P telemetry]`, `[REPL:mount]`
- Message: Free-form text

### 2.2 Session JSONL Format
Each line is a complete JSON object with message history:

**User Message Structure**:
```json
{
  "type": "user",
  "message": {
    "role": "user",
    "content": "user query text"
  },
  "uuid": "message-uuid",
  "timestamp": "2026-03-26T06:08:49.398Z",
  "sessionId": "session-uuid",
  "cwd": "/path/to/project"
}
```

**Assistant Message Structure**:
```json
{
  "type": "assistant",
  "message": {
    "role": "assistant",
    "model": "claude-sonnet-4-5-20250929",
    "content": [
      {
        "type": "thinking",
        "thinking": "internal reasoning..."
      },
      {
        "type": "text",
        "text": "response text"
      },
      {
        "type": "tool_use",
        "id": "tool-call-id",
        "name": "Read",
        "input": {...}
      }
    ],
    "usage": {
      "input_tokens": 10,
      "output_tokens": 7,
      "cache_read_input_tokens": 0
    }
  },
  "parentUuid": "parent-message-uuid",
  "timestamp": "2026-03-26T06:08:58.972Z"
}
```

## 3. State Detection Patterns

### 3.1 Idle Prompt State (Waiting for User Input)
**Detection Pattern**:
```
Getting matching hook commands for Notification with query: idle_prompt
```

**Frequency**: Appears regularly when Claude is waiting for user input after completing a response

**Log Examples**:
```
2026-03-04T07:30:14.704Z [DEBUG] Getting matching hook commands for Notification with query: idle_prompt
2026-03-04T04:02:42.124Z [DEBUG] Getting matching hook commands for Notification with query: idle_prompt
2026-02-26T04:12:14.027Z [DEBUG] Getting matching hook commands for Notification with query: idle_prompt
```

**Reliability**: High (🔵) - Appears consistently in debug logs

### 3.2 Permission Prompt State (Waiting for User Permission)
**Detection Pattern**:
```
Getting matching hook commands for Notification with query: permission_prompt
```

**Frequency**: Appears when Claude needs user permission to execute commands or access resources

**Log Examples**:
```
2026-03-04T07:17:31.086Z [DEBUG] Getting matching hook commands for Notification with query: permission_prompt
2026-02-26T03:48:07.970Z [DEBUG] Getting matching hook commands for Notification with query: permission_prompt
```

**Reliability**: High (🔵) - Clear indicator of waiting for user action

### 3.3 User Prompt State (User Submitting Input)
**Detection Pattern in Debug Logs**:
```
[WARN] [3P telemetry] Event dropped (no event logger initialized): user_prompt
```

**Detection Pattern in JSONL**:
```json
{
  "type": "user",
  "message": {
    "role": "user",
    "content": "..."
  }
}
```

**Reliability**: High (🔵) - Marks start of new user interaction

### 3.4 Tool Execution State (Claude Executing Commands)
**Detection Pattern**:
```
executePreToolHooks called for tool: <ToolName>
```

**Examples**:
```
2026-03-04T07:14:07.586Z [DEBUG] executePreToolHooks called for tool: Grep
2026-03-04T07:14:12.667Z [DEBUG] executePreToolHooks called for tool: Read
2026-03-04T07:14:17.412Z [DEBUG] executePreToolHooks called for tool: Glob
```

**Reliability**: High (🔵) - Clear indicator of active processing

### 3.5 Assistant Thinking/Responding
**Detection Pattern in JSONL**:
```json
{
  "type": "assistant",
  "message": {
    "content": [
      {"type": "thinking", ...},
      {"type": "text", ...}
    ]
  }
}
```

**Detection Pattern in Debug Logs**:
```
Forked agent [prompt_suggestion] received message: type=assistant
```

**Reliability**: High (🔵) - Indicates active response generation

### 3.6 Task/Session Completion
**Detection Patterns**:

1. **Session End** (Not yet observed - need more investigation)
2. **Subagent Completion**:
```
Forked agent [prompt_suggestion] finished: 2 messages, types=[assistant, assistant], totalUsage: input=0 output=670
```

3. **REPL Mount State**:
```
[REPL:mount] REPL mounted, disabled=false
```

**Reliability**: Medium (🟡) - Need more investigation for definitive completion signals

## 4. Log Rotation and Lifecycle

### 4.1 Debug Logs
- **Creation**: New file created per interactive session
- **Naming**: UUID-based session identifier
- **Retention**: Observed logs from February-March 2026 (multiple months)
- **Cleanup**: No automatic cleanup observed; appears to accumulate
- **Size Management**: Individual files range from 6KB to 9.8MB

### 4.2 Session JSONL Files
- **Creation**: One file per conversation session
- **Location**: Project-specific subdirectories
- **Subagents**: Nested in `/subagents/` directory when spawned
- **Retention**: Persistent across sessions

## 5. Monitoring Strategy Recommendations

### 5.1 Primary Approach: Tail Debug Logs
**File**: `~/.claude/debug/latest` (symlink to current session)

**Pros**:
- Real-time monitoring
- Low latency (immediate state changes)
- Clear state indicators

**Cons**:
- Text parsing required
- Log format may change between Claude Code versions

**Implementation**:
```swift
// Monitor: ~/.claude/debug/latest
// Watch for patterns:
// - "query: idle_prompt" -> Waiting for user input
// - "query: permission_prompt" -> Waiting for permission
// - "executePreToolHooks called" -> Active processing
```

### 5.2 Alternative: Parse Session JSONL
**File**: `~/.claude/projects/<project>/<session-id>.jsonl`

**Pros**:
- Structured data (JSON)
- Complete conversation context
- Definitive message types

**Cons**:
- Need to discover session file dynamically
- Higher parsing overhead
- May have write delay

**Implementation**:
```swift
// 1. Monitor ~/.claude/sessions/ for active session
// 2. Derive project path from session metadata
// 3. Tail corresponding JSONL file
// 4. Parse last line for message type
```

### 5.3 Hybrid Approach (Recommended)
1. **Primary**: Monitor debug log for real-time state changes
2. **Validation**: Cross-reference with session metadata
3. **Context**: Parse JSONL for detailed task information

## 6. Detected State Transition Diagram

```
[Session Start]
    ↓
[REPL Mounted] → [Idle Prompt] ← ─ ─ ─ ─ ─ ┐
    ↓                                        │
[User Prompt] → [Assistant Thinking]         │
    ↓                                        │
[Tool Execution] ──────────────────────────  │
    ↓                                        │
[Permission Prompt]? ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │
    ↓                                        │
[Assistant Response] ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │
    ↓                                        │
[Idle Prompt] ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

## 7. Regular Expression Patterns

### 7.1 State Detection Patterns
```swift
// Idle (Waiting for Input)
let idlePattern = #"Getting matching hook commands for Notification with query: idle_prompt"#

// Permission Required
let permissionPattern = #"Getting matching hook commands for Notification with query: permission_prompt"#

// User Input Submitted
let userPromptPattern = #"\[3P telemetry\] Event dropped.*: user_prompt"#

// Tool Execution
let toolExecutionPattern = #"executePreToolHooks called for tool: (\w+)"#

// Assistant Response
let assistantPattern = #"Forked agent \[prompt_suggestion\] received message: type=assistant"#

// Subagent Completion
let completionPattern = #"Forked agent \[.*\] finished: (\d+) messages"#
```

### 7.2 Session Detection
```swift
// Active session file
let sessionPattern = #"~/.claude/sessions/(\d+)\.json"#

// Extract session ID
let sessionIdPattern = #"\"sessionId\":\s*\"([a-f0-9-]+)\""#
```

## 8. Key Findings Summary

### 8.1 Confirmed (🔵)
1. **Log Location**: `~/.claude/debug/` with UUID-based session files
2. **Log Format**: Timestamped text with component tags
3. **Idle State Pattern**: `query: idle_prompt` in debug logs
4. **Permission State Pattern**: `query: permission_prompt` in debug logs
5. **User Prompt Pattern**: `user_prompt` telemetry event
6. **JSONL Structure**: Complete conversation history in structured format
7. **Session Metadata**: Active sessions tracked in `~/.claude/sessions/`

### 8.2 Needs Verification (🟡)
1. **Task Completion Pattern**: Need to observe complete session lifecycle
2. **Log Rotation**: Exact rotation mechanism and triggers
3. **Error States**: Patterns for Claude errors or crashes
4. **Multi-session Behavior**: How multiple concurrent sessions are handled

### 8.3 Open Questions (🔴)
1. **Version Compatibility**: Will these patterns remain stable across Claude Code updates?
2. **Performance Impact**: CPU/memory overhead of file monitoring
3. **Edge Cases**: Behavior during network issues or API errors

## 9. Next Steps

### 9.1 Immediate (TASK-0007 Implementation)
- [ ] Implement file monitor for `~/.claude/debug/latest`
- [ ] Create regex patterns for state detection
- [ ] Build state machine for transition tracking

### 9.2 Testing Requirements
- [ ] Test with multiple concurrent Claude Code instances
- [ ] Verify pattern detection across different task types
- [ ] Monitor for false positives/negatives

### 9.3 Future Enhancements
- [ ] Investigate JSONL parsing for richer context
- [ ] Add support for session-specific notifications
- [ ] Monitor for long-running tasks (timeout detection)

## 10. Implementation Code Snippets

### 10.1 Swift File Monitor (Concept)
```swift
import Foundation

class ClaudeLogMonitor {
    private let logPath = FileManager.default.homeDirectoryForCurrentUser
        .appendingPathComponent(".claude/debug/latest")

    private let idlePattern = try! NSRegularExpression(
        pattern: "query: idle_prompt"
    )

    private let permissionPattern = try! NSRegularExpression(
        pattern: "query: permission_prompt"
    )

    func monitorLog() {
        // Tail file implementation
        // Parse new lines
        // Match patterns
        // Emit state change events
    }
}
```

### 10.2 State Detection Logic
```swift
enum ClaudeState {
    case idle           // Waiting for user input
    case permission     // Waiting for permission
    case thinking       // Processing user request
    case executing      // Running tools
    case responding     // Generating response
}

func detectState(from logLine: String) -> ClaudeState? {
    if logLine.contains("query: idle_prompt") {
        return .idle
    }
    if logLine.contains("query: permission_prompt") {
        return .permission
    }
    if logLine.contains("executePreToolHooks called") {
        return .executing
    }
    // ... additional patterns
    return nil
}
```

## Appendix A: Sample Log Files

### A.1 Debug Log Sample (Condensed)
```
2026-03-04T07:13:17.932Z [DEBUG] [REPL:mount] REPL mounted, disabled=false
2026-03-04T07:13:59.462Z [WARN] [3P telemetry] Event dropped: user_prompt
2026-03-04T07:14:01.091Z [DEBUG] [API:auth] OAuth token check complete
2026-03-04T07:14:07.586Z [DEBUG] executePreToolHooks called for tool: Grep
2026-03-04T07:14:08.128Z [DEBUG] Auto tool search disabled: 289 tokens
2026-03-04T07:15:22.952Z [DEBUG] Forked agent finished: 2 messages
2026-03-04T07:30:14.704Z [DEBUG] Getting matching hook commands for Notification with query: idle_prompt
```

### A.2 Session JSONL Sample (Simplified)
```jsonl
{"type":"user","message":{"role":"user","content":"implement feature"},"timestamp":"2026-03-26T06:08:49.398Z"}
{"type":"assistant","message":{"role":"assistant","content":[{"type":"thinking"},{"type":"text"}]},"timestamp":"2026-03-26T06:08:58.972Z"}
```

---

**Report Completed**: 2026-03-26
**Next Task**: TASK-0007 (Log Parser Implementation)
