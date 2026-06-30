# Product Brief

## Vision

Muster is a native macOS app for running multiple coding agents with durable terminal sessions, visible status, and fast attach/detach workflows. A later iPhone companion app should provide mobile monitoring, notifications, and approved control actions for Mac-hosted sessions.

The target user is a developer who has several agents working across repositories, branches, or worktrees and needs a single place to observe progress, jump into terminals, send instructions, and coordinate follow-up work. The same user may also want to check whether an agent is blocked or waiting for input from an iPhone without turning the phone into the primary terminal environment.

## Product shape

Muster should not be a generic terminal emulator first. It should be an agent operations app with a high-quality embedded terminal.

The first screen should be the active work surface:

- Sidebar of projects, sessions, and agents.
- Main terminal for the selected session.
- Status strip showing current command, agent state, branch/worktree, dirty state, and last activity.
- Inspector for session metadata, logs, prompts, and actions.
- Command palette for launch, attach, send, split, stop, and archive actions.

The iPhone companion should have a different shape: session triage, notifications, status detail, and safe message sending. Full terminal control should remain a secondary escape hatch until the Mac workflow is proven.

## Target capabilities

- Launch a coding agent in a fresh Herdr-backed terminal.
- Attach to an existing Herdr terminal or agent session.
- Keep sessions alive when the app quits.
- Show agent state without parsing terminal pixels.
- Create and select worktrees for isolated agent work.
- Send text or commands into one or more sessions.
- Surface important events through native macOS notifications.
- Provide a local MCP server so agents can observe and coordinate controlled parts of the system.
- Pair an iPhone with a trusted Mac for read-only session monitoring and later approved control actions.
- Notify the iPhone when sessions are waiting, blocked, completed, or failed.

## Non-goals for the first version

- Reimplement Herdr in Swift.
- Reimplement Ghostty's terminal renderer.
- Build remote collaboration before the local single-user workflow is excellent.
- Ship through the Mac App Store if sandboxing blocks core terminal/agent workflows.
- Support every terminal emulator feature before proving the agent workflow.
- Run Herdr, arbitrary shells, or coding-agent CLIs locally on iPhone.
- Make mobile remote control the primary workflow before the local Mac workflow is excellent.

## Design principles

- Native first: keyboard shortcuts, menus, focus behavior, windowing, notifications, and accessibility should feel like a real Mac app.
- Backend second: Herdr should remain replaceable behind a narrow integration layer.
- Terminal quality matters: the terminal should feel fast, accurate, and familiar enough that users trust it for real work.
- Agent state should come from structured sources where possible, not fragile terminal scraping.
- Dangerous operations should be explicit, visible, and auditable.
- Mobile actions should be narrower than Mac actions and should go through pairing, policy, and audit logging.
