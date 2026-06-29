# Ghostty Terminal Integration

## Role

Ghostty/libghostty should provide the terminal experience inside the native Mac app.

The goal is to get Ghostty-level terminal quality without making Muster a fork of Ghostty's full application.

## Current feasibility assumption

Ghostty's macOS app already proves the relevant technical path:

- Swift/AppKit/SwiftUI shell.
- Shared Ghostty core.
- Terminal emulation and rendering in native code.
- Metal-backed rendering.

The caveat is that libghostty should be treated as an evolving integration surface, not a stable third-party SDK.

## Integration options

### Option A: Embed libghostty directly

Best long-term UX, highest integration cost.

Pros:

- Best terminal fidelity.
- Native rendering performance.
- Consistent with Unpeel-style architecture.
- Gives control over terminal tabs, status, search, and inspector layout.

Cons:

- libghostty API stability risk.
- Requires Zig/C build integration.
- Requires careful packaging and signing.
- More complex crash/debug surface.

### Option B: Reuse Ghostty macOS components

Potentially faster if upstream internals are separable.

Pros:

- Less low-level renderer glue.
- More proven macOS behavior.

Cons:

- May be tightly coupled to Ghostty app architecture.
- Licensing and upstream drift need review.
- Could become harder to customize for Herdr attach streams.

### Option C: Temporary terminal view first

Best prototype path.

Pros:

- Lets the product validate Herdr workflows quickly.
- Reduces risk before investing in libghostty integration.
- Allows parallel work on UI/state/bridge.

Cons:

- Not the final quality bar.
- May require replacement once core workflow is proven.

Recommendation: start with Option C for phase 1, then move to Option A once the Herdr workflow is validated.

## Terminal data flow

```text
Herdr attach process/stream
  stdout/stderr/input
    -> GhosttyProcessAdapter
      -> GhosttyTerminalController
        -> GhosttyTerminalView
```

The rest of the app should not know whether the terminal bytes come from:

- A Herdr attach command.
- A PTY process.
- A mock fixture.
- A future direct Herdr stream.

## Swift boundary draft

```swift
final class TerminalSessionController: ObservableObject {
    let id: SessionID
    let title: String

    func connect() async throws
    func disconnect() async
    func resize(columns: Int, rows: Int) async
    func sendInput(_ data: Data) async
    func copySelection() -> String?
    func paste(_ text: String) async
}
```

## Required terminal features

MVP:

- Render Herdr attach output.
- Send keyboard input.
- Resize correctly.
- Copy and paste.
- Basic color/theme support.
- Detect connection loss.

Post-MVP:

- Search.
- Scrollback.
- Split panes.
- Font ligature settings.
- URL detection.
- Image protocol support if inherited safely.
- Terminal recording or snapshots.

## Packaging questions

Before implementation, answer:

- Can libghostty be consumed cleanly from a SwiftPM/Xcode build?
- Is a stable C header available for the required terminal embedding operations?
- What build artifacts need to be bundled and signed?
- How much of Ghostty's macOS app code can be reused without dragging in unwanted app behavior?
- How will crashes in renderer/native code be surfaced and reported?

