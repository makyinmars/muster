# System Architecture

## High-level architecture

```text
Muster.app
  SwiftUI/AppKit UI
  App state and commands
  HerdrKit
  GhosttyKit
  MCPBridgeClient

Muster Helper
  Launch agent / login item
  Supervises Herdr server
  Supervises optional MCP bridge

Herdr
  Persistent terminal/session backend
  Workspaces, tabs, panes
  Agent detection and state
  Worktree helpers
  Local socket API

Ghostty/libghostty
  Terminal emulation
  Font shaping
  Metal rendering
  Keyboard/mouse terminal input

Muster Bridge
  Local MCP server
  Controlled read/send/session APIs
  Policy and audit layer

Muster iPhone
  SwiftUI iOS companion UI
  Paired Mac client
  Shared schema models
  Notifications and handoff

Companion API
  Authenticated user-device control API
  Read-only status, notifications, and approved actions
  Shares policy and audit concepts with the bridge
```

## Layer responsibilities

### Muster.app

The app owns user-facing behavior:

- Window and scene management.
- Sidebar, terminal area, inspectors, menus, command palette, settings.
- Mapping Herdr state into native view models.
- Initiating launches, attaches, stops, archives, and sends.
- Showing notifications and activity indicators.

### HerdrKit

HerdrKit is the Swift boundary around Herdr.

It should hide whether an action is implemented through:

- Herdr socket API.
- Herdr CLI.
- A direct attach process.
- A temporary compatibility shim.

All other app layers should talk to HerdrKit, not to raw Herdr commands.

### GhosttyKit

GhosttyKit is the boundary around terminal rendering.

Its job is to expose a native terminal view that can connect to a PTY or attach stream. It should keep Ghostty-specific details out of the rest of the app.

Expected responsibilities:

- Own the embedded terminal view.
- Translate macOS keyboard/mouse/input events into terminal input.
- Render terminal output using libghostty where feasible.
- Support copy/paste, selection, search, font settings, colors, and resize.

### Muster Helper

The helper keeps background pieces alive without making the main app responsible for every lifecycle edge case.

Expected responsibilities:

- Start `herdr server` if needed.
- Detect server health.
- Restart local bridge services when configured.
- Expose simple status to the main app.

### Muster Bridge

The bridge is a local MCP server that exposes selected Muster/Herdr operations to coding agents.

It should be intentionally narrower than the full app API. Agents need structured awareness and safe coordination primitives, not unrestricted shell access by default.

### Muster iPhone

The iPhone app is a companion surface for the user, not a local Herdr host.

Expected responsibilities:

- Pair with a trusted Mac.
- Show project, session, and agent status.
- Show recent structured events.
- Receive notifications for waiting, blocked, completed, and failed sessions.
- Send visible messages or approved actions through policy-controlled APIs.
- Hand off to the Mac app for full terminal work.

### Companion API

The companion API is the user-device boundary between iPhone and the Mac-hosted Muster system.

It should not expose the raw MCP bridge directly to the phone. The phone and coding agents have different trust models, but they can share:

- Domain schemas.
- Policy decisions.
- Audit log storage.
- Session and agent status projections.

## Data flow

### Dashboard state

```text
Herdr server -> socket events -> HerdrKit -> AppState -> SwiftUI views
```

The UI should subscribe to Herdr events rather than poll aggressively.

### Terminal attach

```text
User selects session -> HerdrKit resolves terminal id -> attach process/stream -> GhosttyKit view
```

The initial implementation can use a simpler terminal view or external attach process. The target implementation should embed a Ghostty-backed terminal view.

### Agent command

```text
User action -> App command -> HerdrKit -> Herdr socket or CLI -> target terminal/agent
```

Every command should return structured success/failure information to the app so failures can be shown without guessing from terminal output.

### iPhone session monitoring

```text
Muster iPhone -> Companion API -> Muster Helper/App on Mac -> HerdrKit -> Herdr state
```

The phone should read structured state. It should not scrape terminal output or connect directly to Herdr internals.

### iPhone control action

```text
iPhone action -> Companion API -> policy check -> audit log -> HerdrKit or app command
```

State-changing phone actions should be narrower than local Mac actions until the pairing and audit model is proven.

## Domain model

```text
Project
  id
  name
  rootPath
  defaultBranch
  sessions[]

Session
  id
  title
  terminalId
  projectId
  worktreePath
  agentKind
  agentState
  lastActivityAt

Agent
  id
  sessionId
  kind
  status
  currentTask
  lastOutputSummary

Worktree
  id
  projectId
  path
  branch
  baseBranch
  dirtyState

Preset
  id
  name
  command
  environment
  workingDirectoryRule

PairedDevice
  id
  name
  platform
  trustLevel
  lastSeenAt
```

## Persistence

Herdr should remain the source of truth for sessions and terminal durability.

Muster should persist only app-specific preferences and metadata:

- Window layouts.
- Favorites and pinned sessions.
- Launch presets.
- Notification rules.
- Display names.
- Per-project app settings.
- Paired device metadata and policy.

Do not duplicate Herdr's session database unless there is a clear offline or migration need.
