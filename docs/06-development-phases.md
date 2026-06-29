# Development Phases

## Phase 0: Feasibility spike

Goal: prove the critical unknowns before building product surface area.

Deliverables:

- Minimal Swift macOS app shell.
- Herdr binary discovery.
- Start/check Herdr server.
- Connect to Herdr socket API.
- List active sessions/agents.
- Attach to one terminal through a basic process-backed view.
- Document libghostty build and embedding path.

Exit criteria:

- A user can see Herdr sessions in a native Mac window.
- A user can select one session and interact with it.
- The team knows whether direct libghostty embedding is practical for the next phase.

## Phase 1: Native Herdr client MVP

Goal: make Muster useful even before the final Ghostty terminal is complete.

Deliverables:

- Sidebar with projects/sessions/agents.
- Session creation from launch presets.
- Attach/detach selected session.
- Structured status display from Herdr state.
- Native notifications for session state changes.
- Basic settings for Herdr binary path and startup behavior.

Exit criteria:

- Muster can replace the basic Herdr TUI for day-to-day session monitoring.
- Sessions survive app quit/relaunch.
- Failure states are visible and understandable.

## Phase 2: Ghostty terminal integration

Goal: replace the temporary terminal with a production-quality embedded terminal.

Deliverables:

- GhosttyKit module.
- Embedded terminal view.
- Keyboard input, paste, resize, copy, selection.
- Theme/font settings.
- Terminal reconnect handling.
- Packaging/signing proof for Ghostty native artifacts.

Exit criteria:

- Terminal interaction feels good enough for real coding-agent work.
- No app-wide architecture depends on the temporary terminal implementation.
- Crash and failure modes are understood.

## Phase 3: Agent workflows

Goal: make the app more valuable than a terminal multiplexer.

Deliverables:

- Agent-aware session list.
- Launch presets for common coding agents.
- Worktree creation flow.
- Session inspector with agent state, branch/worktree, command, recent activity.
- Send message/action to one or more sessions.
- Archive completed sessions.

Exit criteria:

- A developer can run several agents across worktrees and keep track of them without losing context.
- Common actions are available through native UI and keyboard shortcuts.

## Phase 4: Muster Bridge

Goal: let agents coordinate through a controlled local MCP server.

Deliverables:

- Read-only MCP bridge.
- Session list/get tools.
- Message sending with audit log.
- Preset-based session creation.
- Per-project bridge permissions.
- Bridge status in app settings.

Exit criteria:

- An agent can safely inspect sibling sessions and send visible handoff messages.
- User can see what the bridge did and disable it per project.

## Phase 5: Product hardening

Goal: make the app shippable outside a prototype environment.

Deliverables:

- Error reporting and diagnostics bundle.
- Onboarding for Herdr install/version compatibility.
- Update strategy.
- Developer ID signing/notarization.
- Accessibility pass.
- Performance pass with many sessions.
- Documentation for security model.

Exit criteria:

- A new user can install, configure, run, and trust the app without developer intervention.

## Phase 6: Advanced workflows

Goal: extend beyond local session management once the local workflow is strong.

Possible deliverables:

- Saved project templates.
- Multi-agent task boards.
- Session summaries.
- Remote Herdr host support.
- Team-shared presets.
- Richer MCP tools.
- Deep GitHub integration.
- Timeline playback.

Exit criteria:

- Advanced features do not compromise the local native workflow.

