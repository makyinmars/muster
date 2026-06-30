# iPhone Companion App

## Product position

Muster for iPhone should start as a companion controller for a Mac-hosted Muster system, not as a full local replacement for the Mac app.

The Mac app remains responsible for:

- Running Herdr.
- Owning terminal attach streams.
- Rendering the production terminal surface.
- Managing local project paths, worktrees, shells, and coding-agent CLIs.
- Supervising the local MCP bridge.

The iPhone app is responsible for:

- Monitoring active projects, sessions, and agents.
- Receiving important notifications.
- Reading structured session status and recent events.
- Sending visible messages or approved commands to selected sessions.
- Starting sessions only from approved presets once policy and audit logging exist.
- Handing off to the Mac app for full terminal interaction when needed.

This keeps the first mobile version useful while avoiding a fragile attempt to reproduce the Mac terminal and process environment on iOS.

## Why companion-first

The core Muster backend depends on capabilities that are natural on macOS and awkward or unavailable on iOS:

- Launching arbitrary local shells and coding-agent CLIs.
- Accessing developer project directories and Git worktrees.
- Keeping durable terminal sessions alive.
- Embedding the target Ghostty/libghostty terminal experience.
- Coordinating local socket APIs and helper processes.

An iPhone app can still be valuable if it treats those pieces as remote state owned by the user's Mac.

## MVP scope

The first iPhone build should be narrow:

- Pair with one trusted Mac.
- Show the active session list grouped by project.
- Show agent status, branch/worktree, last activity, and current task when available.
- Show recent structured events for a selected session.
- Send a visible handoff message into a session.
- Trigger native push or local notifications for states like waiting for input, blocked, completed, and failed.
- Open a deep link or continuity handoff back to the Mac app for full terminal control.

The first iPhone build should not:

- Run Herdr locally.
- Embed a full interactive terminal as the main product surface.
- Offer arbitrary shell execution from the phone.
- Create or delete worktrees before the policy model is proven.
- Require cloud sync before a local trusted-device workflow is validated.

## Architecture option

```text
Muster iPhone
  SwiftUI iOS UI
  PairedMacClient
  Shared schema models
  Notification handling

Secure pairing channel
  Local network first
  Authenticated device identity
  Explicit per-Mac trust

Muster.app / Muster Helper on Mac
  Companion API
  HerdrKit
  Bridge policy and audit log
  Notification publisher

Herdr
  Durable sessions and agent state
```

The companion API should be separate from the MCP bridge. Agents and phones have different trust models:

- Agents need narrow coordination tools.
- The iPhone app needs user-facing monitoring and approved control actions.
- Both should share schemas and audit logging where possible.

## iOS UI shape

The iPhone app should be optimized for triage, not terminal immersion:

- Tab 1: Active sessions.
- Tab 2: Waiting for me.
- Tab 3: Notifications or timeline.
- Tab 4: Settings and paired Macs.

Session detail should prioritize:

- Agent state.
- Current command or task.
- Branch and worktree.
- Recent events.
- Message composer.
- Safe actions such as stop, archive, or open on Mac, gated by policy.

If an interactive terminal is added later, it should be a secondary escape hatch for short checks, not the main value proposition.

## Pairing and security

The pairing model needs to be decided before implementation.

Baseline requirements:

- User-initiated pairing from both devices.
- Per-device revocation.
- Encrypted transport.
- Short-lived session tokens.
- No unauthenticated LAN control endpoint.
- Explicit policy for which phone actions are allowed.
- Audit log entries for every state-changing phone action.

Suggested initial policy levels:

- Read-only: view sessions and statuses.
- Message-only: send visible messages to selected sessions.
- Preset-only: launch approved presets.
- Admin: stop, archive, create worktrees, and adjust settings.

## Notification model

The Mac app should decide which events matter. The phone should display them.

Initial event types:

- Agent waiting for input.
- Agent blocked.
- Agent completed.
- Agent failed.
- Long-running command exceeded threshold.
- Session requested review.

Open choice:

- Local-network notifications are simpler but only work nearby.
- Push notifications require an account or relay service, but they make the phone useful away from the Mac.

## Questions to answer before building

- Should the iPhone MVP be LAN-only, or should remote access be part of the first mobile design?
- Does the app need user accounts, or can trusted-device pairing carry the first version?
- Which actions are safe from the phone in the MVP: view-only, message, launch preset, stop, archive, create worktree?
- Should phone-originated messages appear as typed text in the terminal, as structured Herdr messages, or both?
- What App Store review constraints apply to remote terminal control and command execution?
- How much schema should be shared between Mac, iPhone, bridge, and future remote hosts?
- Should iPhone notifications come directly from the Mac, through APNs, or through a future hosted relay?
- What happens when the Mac is asleep, offline, or on a different network?
- Is a watchOS notification-only surface useful later, or a distraction?

## Recommended path

1. Build the Mac app and local Herdr workflow first.
2. Add bridge policy and audit logging before allowing phone-originated state changes.
3. Define a small companion API using the same domain model as HerdrKit/MusterShared.
4. Spike local trusted-device pairing.
5. Build a read-only iPhone prototype.
6. Add message sending only after audit logging is visible in the Mac app.
7. Decide whether remote push/relay belongs in the product before expanding mobile actions.
