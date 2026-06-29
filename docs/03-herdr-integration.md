# Herdr Integration

## Role

Herdr should be the durable terminal and agent backend.

Muster should not try to replace Herdr's core responsibilities:

- Persistent terminal sessions.
- Workspaces, tabs, and panes.
- Agent detection.
- Session state.
- Worktree management.
- Attach/detach semantics.

## Integration surfaces

### Socket API

Use the socket API for structured app state and control where available.

Expected usage:

- List workspaces, tabs, panes, terminals, and agents.
- Subscribe to session and agent events.
- Send input to a target session.
- Read structured output or recent activity.
- Create or inspect worktrees.
- Stop or archive sessions.

The socket API should be the preferred path for dashboard state because it gives the app structured data.

### CLI

Use the CLI as a fallback or for capabilities not yet exposed cleanly through the socket API.

Expected usage:

- Start or verify `herdr server`.
- Attach to a specific terminal.
- Attach to a specific agent.
- Run compatibility flows during early prototypes.

### Attach streams

Use direct attach for the visible terminal.

Target behavior:

```text
Muster selected session -> Herdr terminal id -> attach stream -> terminal renderer
```

Early prototypes can shell out to `herdr terminal attach`. The mature implementation should hide this behind `HerdrAttachSession`.

## HerdrKit API draft

```swift
protocol HerdrClient {
    func serverStatus() async throws -> HerdrServerStatus
    func startServerIfNeeded() async throws

    func listProjects() async throws -> [MusterProject]
    func listSessions(projectID: ProjectID?) async throws -> [MusterSession]
    func listAgents() async throws -> [MusterAgent]

    func events() async throws -> AsyncThrowingStream<HerdrEvent, Error>

    func createSession(_ request: CreateSessionRequest) async throws -> MusterSession
    func attach(to sessionID: SessionID) async throws -> HerdrAttachSession
    func send(_ input: String, to sessionID: SessionID) async throws
    func stop(sessionID: SessionID) async throws
    func archive(sessionID: SessionID) async throws

    func createWorktree(_ request: CreateWorktreeRequest) async throws -> Worktree
}
```

## Session mapping

```text
Herdr workspace -> Muster project
Herdr tab       -> optional group inside project
Herdr pane      -> Muster session candidate
Herdr terminal  -> visible attach target
Herdr agent     -> enriched session state
```

The exact mapping should be validated against real Herdr state. The app should keep this mapping contained in HerdrKit so it can change without rewriting the UI.

## Agent state

Muster should prefer Herdr's agent detection over terminal scraping.

Useful states:

- Starting.
- Idle.
- Thinking.
- Running command.
- Waiting for user input.
- Blocked.
- Completed.
- Failed.
- Unknown.

If Herdr does not expose a state directly, HerdrKit should mark it as `unknown` rather than guessing too aggressively.

## Supervisor behavior

Muster should not assume Herdr is already running.

Startup sequence:

1. Check configured Herdr binary path.
2. Check Herdr version.
3. Check server health.
4. Start server if allowed by user settings.
5. Connect socket client.
6. Subscribe to events.
7. Hydrate initial app state.

## Versioning

HerdrKit should perform a version check and expose compatibility status.

Compatibility states:

- Supported.
- Older but usable.
- Unsupported older version.
- Newer untested version.
- Missing binary.

This avoids confusing UI failures when Herdr changes its socket schema or CLI behavior.

