# Risks and Open Questions

## Critical risks

### libghostty stability

Ghostty's core is promising, but libghostty should not be assumed to be a stable public embedding SDK.

Mitigation:

- Isolate all Ghostty integration in GhosttyKit.
- Start with a temporary terminal implementation.
- Create a small embedding spike before committing the product architecture to libghostty details.

### Herdr API drift

Herdr may evolve its socket schema, CLI commands, or internal session model.

Mitigation:

- Keep HerdrKit as the only app boundary.
- Add version detection.
- Add fixture-based tests for socket messages and CLI outputs.
- Prefer structured API contracts over terminal scraping.

### Licensing

Herdr is not just an implementation detail if it ships with or is required by Muster.

Mitigation:

- Review Herdr's license before commercial distribution.
- Decide whether Muster requires users to install Herdr separately.
- Consider commercial licensing if building a closed-source commercial app.
- Track Ghostty license and any obligations from reused code.

### App sandboxing

Mac App Store sandboxing may conflict with shells, agents, project file access, sockets, worktrees, and helper processes.

Mitigation:

- Target Developer ID distribution first.
- Treat App Store support as a separate investigation.

### Security

The app coordinates shells and coding agents, so unsafe defaults could be harmful.

Mitigation:

- Keep MCP bridge permissions narrow by default.
- Make cross-session sends visible.
- Keep audit logs.
- Require opt-in for destructive operations.
- Avoid default raw-shell MCP tools.

### iPhone companion security

The iPhone companion app introduces a second control surface for Mac-hosted shells and agents.

Mitigation:

- Start read-only.
- Require explicit trusted-device pairing.
- Keep the companion API separate from the MCP bridge.
- Policy-check every state-changing phone action.
- Record phone-originated actions in the same audit log shown in the Mac app.
- Do not expose arbitrary shell execution from the phone.
- Validate App Store review constraints before committing to a shipping mobile feature set.

## Product risks

### Too much terminal, not enough workflow

If Muster only becomes another terminal wrapper, it will not justify itself.

Mitigation:

- Prioritize agent state, worktrees, sessions, notifications, and command routing.
- Keep the terminal excellent but not the only value proposition.

### Too much orchestration too early

Advanced multi-agent automation can distract from the basic attach/observe/control loop.

Mitigation:

- Build phases in order.
- Do not implement remote/team features before the local workflow is strong.

### Mobile app distracts from the Mac workflow

An iPhone app could pull the product toward remote control before the core Mac workflow is proven.

Mitigation:

- Treat iPhone as a companion app, not a replacement product.
- Build the Mac app and bridge policy first.
- Keep the first iPhone version focused on monitoring, notifications, and visible handoff messages.

## Open questions

- What is the exact Herdr object mapping from workspace/tab/pane/terminal/agent into Muster's Project/Session/Agent model?
- Which Herdr operations are socket-native today, and which require CLI fallback?
- Can libghostty be embedded cleanly without depending on private Ghostty app internals?
- Should Muster bundle Herdr or require the user to install it?
- Should the MCP bridge be a separate binary, an app helper, or an embedded service?
- What launch presets should ship first: Codex, Claude Code, OpenCode, Aider, shell?
- How should the app summarize agent state without relying on vendor-specific output formats?
- What is the minimum viable notification model?
- Should the app store any terminal scrollback, or should Herdr remain the only session source of truth?
- How should worktree cleanup be handled safely?
- Should the iPhone companion MVP be LAN-only, or should remote access be part of the first mobile design?
- What pairing model should authorize an iPhone to control a Mac-hosted Muster instance?
- Which iPhone actions are safe for the first release: read-only, message, launch preset, stop, archive, create worktree?
- Should phone-originated messages be inserted as terminal text, structured Herdr messages, or both?
- Should iPhone notifications be direct from the Mac, delivered through APNs, or deferred until a hosted relay exists?
- What happens when the paired Mac is asleep, offline, or on another network?
- What App Store review constraints apply to remote terminal control, local network discovery, and command execution?

## Decision log

### Initial architecture decision

Use a native Swift macOS app as the product shell, Herdr as the durable backend, and Ghostty/libghostty as the preferred terminal renderer.

Reasoning:

- Herdr already provides the terminal/session/agent backend primitives.
- Ghostty offers the best available path to a high-quality native terminal.
- Swift/AppKit/SwiftUI are the right fit for the Mac-first product shell.

### Initial name decision

Use **Muster** as the working name.

Reasoning:

- It communicates gathering agents for action.
- It is not tied too tightly to Herdr or Ghostty.
- It supports product concepts like roster, sessions, projects, and bridge.

### Initial iPhone scope decision

Treat the iPhone app as a companion controller for Mac-hosted sessions, not as a local iOS terminal/session backend.

Reasoning:

- The product's durable session model depends on Herdr running on the Mac or another host.
- The target terminal quality depends on a Mac-native Ghostty/libghostty integration.
- iPhone is best suited for monitoring, notifications, and lightweight control.
- Mobile actions need a tighter pairing, policy, and audit model than local Mac actions.
