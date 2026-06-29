# Product Brief

## Vision

Muster is a native macOS app for running multiple coding agents with durable terminal sessions, visible status, and fast attach/detach workflows.

The target user is a developer who has several agents working across repositories, branches, or worktrees and needs a single place to observe progress, jump into terminals, send instructions, and coordinate follow-up work.

## Product shape

Muster should not be a generic terminal emulator first. It should be an agent operations app with a high-quality embedded terminal.

The first screen should be the active work surface:

- Sidebar of projects, sessions, and agents.
- Main terminal for the selected session.
- Status strip showing current command, agent state, branch/worktree, dirty state, and last activity.
- Inspector for session metadata, logs, prompts, and actions.
- Command palette for launch, attach, send, split, stop, and archive actions.

## Target capabilities

- Launch a coding agent in a fresh Herdr-backed terminal.
- Attach to an existing Herdr terminal or agent session.
- Keep sessions alive when the app quits.
- Show agent state without parsing terminal pixels.
- Create and select worktrees for isolated agent work.
- Send text or commands into one or more sessions.
- Surface important events through native macOS notifications.
- Provide a local MCP server so agents can observe and coordinate controlled parts of the system.

## Non-goals for the first version

- Reimplement Herdr in Swift.
- Reimplement Ghostty's terminal renderer.
- Build remote collaboration before the local single-user workflow is excellent.
- Ship through the Mac App Store if sandboxing blocks core terminal/agent workflows.
- Support every terminal emulator feature before proving the agent workflow.

## Design principles

- Native first: keyboard shortcuts, menus, focus behavior, windowing, notifications, and accessibility should feel like a real Mac app.
- Backend second: Herdr should remain replaceable behind a narrow integration layer.
- Terminal quality matters: the terminal should feel fast, accurate, and familiar enough that users trust it for real work.
- Agent state should come from structured sources where possible, not fragile terminal scraping.
- Dangerous operations should be explicit, visible, and auditable.

