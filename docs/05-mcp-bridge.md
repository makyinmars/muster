# MCP Bridge

## Purpose

Muster Bridge is a local MCP server that lets coding agents understand and coordinate the local agent workspace.

The bridge should expose carefully scoped operations. It is not a general shell backdoor.

## Why it matters

Without a bridge, each agent only knows its own terminal context. With a bridge, an agent can ask controlled questions such as:

- What other sessions are active for this project?
- Which agent is working on which worktree?
- Is another agent blocked?
- Can I send a short handoff note to another session?
- What branch or worktree should I avoid touching?

## Architecture

```text
Coding agent
  -> MCP client
    -> Muster Bridge
      -> HerdrKit-compatible adapter
        -> Herdr socket API / CLI
```

The bridge can start as a small standalone process. Later, it can share code with HerdrKit through a common schema package.

## Initial tools

### `muster.list_sessions`

Returns active sessions with project, branch/worktree, agent kind, status, and last activity.

### `muster.get_session`

Returns detail for one session, including safe metadata and recent structured events.

### `muster.send_message`

Sends text to a target agent/session.

This should require explicit policy controls. A bridge caller should not be able to invisibly inject arbitrary destructive commands into unrelated terminals.

### `muster.create_session`

Creates a new session from an approved preset.

### `muster.create_worktree`

Creates a worktree for a project using an approved base branch and naming rule.

### `muster.claim_task`

Marks a session as responsible for a task, branch, or file area.

This can start as Muster-only metadata.

## Policy model

Bridge permissions should be explicit.

Suggested levels:

- Read-only: list sessions, inspect status, read metadata.
- Message-only: send natural-language handoff messages.
- Preset-only: create sessions from approved commands.
- Full local: advanced mode for trusted local agents.

The default should be read-only or message-only.

## Audit log

Every bridge action that changes state should be recorded:

- Timestamp.
- Calling agent/session if known.
- Tool name.
- Target session/project.
- Arguments summary.
- Result.

The app should show this log in the inspector so users can understand cross-agent behavior.

## Safety rules

- Never expose raw shell execution as a default MCP tool.
- Never allow silent cross-session command injection by default.
- Prefer sending visible text into target terminals over hidden control paths.
- Require user opt-in for destructive worktree, branch, or process operations.
- Keep the bridge local-only unless remote access becomes an explicit product area.

## Implementation path

1. Implement read-only bridge backed by Herdr socket state.
2. Add message sending to selected sessions.
3. Add preset-based session creation.
4. Add worktree creation.
5. Add audit log in the Mac app.
6. Add per-project policy settings.

