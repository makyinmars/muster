# macOS App Structure

## Project layout

The eventual repository can use this shape:

```text
muster/
  README.md
  docs/
  apps/
    mac/
      Muster.xcodeproj or Package.swift
      Sources/
        MusterApp/
        HerdrKit/
        GhosttyKit/
        MusterBridgeClient/
        MusterShared/
      Tests/
        HerdrKitTests/
        MusterAppTests/
    ios/
      MusterMobile.xcodeproj or Package.swift
      Sources/
        MusterMobileApp/
        PairedMacClient/
        MusterShared/
      Tests/
        PairedMacClientTests/
        MusterMobileAppTests/
  bridge/
    muster-mcp/
  scripts/
  fixtures/
```

This docs-only skeleton does not create the app code yet. The structure above is the intended implementation direction.

## Swift modules

### MusterApp

The product UI and app lifecycle.

Recommended contents:

- `MusterApp.swift`
- `AppModel.swift`
- `ProjectListView.swift`
- `SessionListView.swift`
- `TerminalSceneView.swift`
- `SessionInspectorView.swift`
- `CommandPaletteView.swift`
- `SettingsView.swift`
- `NotificationController.swift`

### HerdrKit

Typed Swift client for Herdr.

Recommended contents:

- `HerdrClient.swift`
- `HerdrSocketClient.swift`
- `HerdrCLI.swift`
- `HerdrModels.swift`
- `HerdrEventStream.swift`
- `HerdrSupervisor.swift`
- `HerdrAttachSession.swift`

### GhosttyKit

Terminal rendering boundary.

Recommended contents:

- `GhosttyTerminalView.swift`
- `GhosttyTerminalController.swift`
- `GhosttyProcessAdapter.swift`
- `TerminalInputMapper.swift`
- `TerminalTheme.swift`
- `TerminalSearchController.swift`

### MusterBridgeClient

App-side control of the local MCP bridge.

Recommended contents:

- `BridgeSupervisor.swift`
- `BridgeHealth.swift`
- `BridgeSettings.swift`
- `BridgeAuditLog.swift`

### MusterShared

Types shared across app modules.

Recommended contents:

- `ProjectID.swift`
- `SessionID.swift`
- `AgentKind.swift`
- `AgentStatus.swift`
- `CommandResult.swift`
- `MusterError.swift`
- `PairedDevice.swift`
- `CompanionAction.swift`

MusterShared should avoid AppKit, SwiftUI, and Ghostty dependencies so the same domain types can be reused by the Mac app, iPhone app, bridge, and tests.

### MusterMobileApp

The iPhone companion UI.

Recommended contents:

- `MusterMobileApp.swift`
- `PairedMacsView.swift`
- `MobileSessionListView.swift`
- `MobileSessionDetailView.swift`
- `WaitingForMeView.swift`
- `MobileTimelineView.swift`
- `MobileSettingsView.swift`

### PairedMacClient

Typed client for the companion API exposed by the Mac app/helper.

Recommended contents:

- `PairingClient.swift`
- `CompanionAPIClient.swift`
- `CompanionEventsStream.swift`
- `DeviceTrustStore.swift`
- `MobileNotificationRouter.swift`

## UI structure

The app should use a dense, operational layout rather than a marketing-style interface.

```text
Window
  Toolbar
    New Session
    Attach
    Send
    Split
    Search
    Command Palette

  NavigationSplitView
    Sidebar
      Projects
      Active Agents
      Recent Sessions
      Archived

    Content
      Terminal tab strip
      Ghostty terminal view
      Status bar

    Inspector
      Agent state
      Worktree state
      Recent events
      Actions
```

## Native Mac behavior

Required first-class behaviors:

- `Command-N`: new session.
- `Command-T`: new terminal/session tab.
- `Command-W`: close selected tab or window according to context.
- `Command-K`: clear terminal.
- `Command-F`: terminal search.
- `Command-Shift-P`: command palette.
- Native menu commands for session, project, terminal, and bridge actions.
- Standard copy/paste/select-all behavior inside the terminal.
- Proper focus handoff between sidebar, terminal, inspector, and command palette.

## App distribution

Developer ID distribution is the likely first path.

Mac App Store distribution is risky because the app needs to:

- Launch shells and coding-agent CLIs.
- Access arbitrary project directories.
- Manage worktrees.
- Communicate over local sockets.
- Potentially run a local MCP bridge.

Sandbox viability should be investigated later, but it should not shape the MVP.

## iPhone app structure

The iPhone app should use a compact, triage-first layout rather than copying the Mac terminal workspace.

```text
TabView
  Sessions
    Project/session list
    Agent status badges

  Waiting
    Sessions needing user input
    Blocked sessions

  Timeline
    Recent important events

  Settings
    Paired Macs
    Notification preferences
    Allowed actions
```

The iPhone app can share Swift domain models with the Mac app, but it should not depend on HerdrKit or GhosttyKit directly.
