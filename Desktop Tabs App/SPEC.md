# Desktop Tabs: a tabbed workspace for desktop apps (macOS)

**Status:** Draft v0.1 · **Date:** 2026-10-14 · **Platform:** macOS 14 (Sonoma) and later

---

## 1. Problem

A typical workday runs across 8–12 desktop apps (Teams, Claude, Chrome, Notion, Photoshop, VS Code, Spotify, Outlook, PowerPoint, …). Each one has its own window, and the time goes into Cmd‑Tab, Mission Control, hunting for windows behind other windows, and windows ending up at different sizes and positions.

## 2. Goal

One window-like workspace with a **tab bar along the top**. Each tab is one of your apps. When you click a tab (or press a shortcut), that app fills the workspace and you use it as normal: typing, menus, drag and drop, GPU rendering, everything. The other apps stay out of the way.

### Success criteria
- Switching tabs takes **< 150 ms** from click to the app being usable.
- Every app keeps **full native functionality and performance** (Photoshop and VS Code included).
- Daily drivers (the apps above) work with **zero per-app configuration**.
- After a week of use, you rarely reach for Cmd‑Tab or Mission Control.

### Non-goals (v1)
- Windows or Linux versions (see §11).
- Changing what happens *inside* an app (no injecting UI into Photoshop etc.).
- Mac App Store distribution (not possible; see §8).

---

## 3. The key technical constraint (read this first)

**macOS doesn't let one app embed another app's window inside its own window.** Windows has `SetParent` and Linux/X11 has XEmbed, but macOS has nothing like them. The WindowServer keeps every process's windows separate. So "putting Photoshop inside my app" literally can't be done.

There are three ways to get the *experience* of it anyway:

| Approach | How it works | Full native features & speed | Works with every app | Feels "inside" | Verdict |
|---|---|---|---|---|---|
| **A. Window docking** | A tab-bar panel sits at the top of the screen. The Accessibility API moves and resizes the selected app's real window to fill the area under the tab bar, brings it to the front, and hides the other apps. | ✅ Yes, it's the real app | ✅ Any app with a window | 🟡 Very close, since windows snap into the exact frame | **Core of v1** |
| **B. Web tabs** | For apps with a web version (Teams, Outlook, Notion, Claude, Spotify, PowerPoint Online), load the website in an embedded `WKWebView` tab. Shift, Rambox and Station work this way. | 🟡 Web version, not desktop | ❌ No Photoshop, limited VS Code | ✅ Truly inside | **Optional tab type, v2** |
| **C. Screen mirroring** | Capture the app's window with ScreenCaptureKit and draw it inside our window, then forward mouse and keyboard input with synthetic events. | ❌ Lag, broken IME, drag and drop, right-click menus, popovers | 🟡 Fragile | ✅ Truly inside | **Rejected** (only used for tab hover previews) |

**Decision:** build **Approach A**. To the user, it behaves like the tabbed interface described, and every app stays the real, fully working desktop app. Approach B comes later as a tab type for people who prefer the web versions of their chat and mail apps.

Prior art to study: Rectangle and Amethyst (Accessibility window control), AltTab (window enumeration and previews), Hammerspoon (`hs.window`), Shift and Station (web-tab model), and the macOS-native "Merge All Windows" tabs, which only work within a single app.

---

## 4. User experience

### 4.1 Layout
```
┌────────────────────────────────────────────────────────────────────┐
│  macOS menu bar (belongs to the active tab's app, as normal)       │
├────────────────────────────────────────────────────────────────────┤
│ [Teams ●3] [Claude] [Chrome] [Notion] [Ps] [VS Code] [♫] [Outlook ●12] [+] │  ← Tab bar (our app)
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│                    STAGE: active app's window                      │
│                  snapped to exactly this rectangle                 │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```
- **Tab bar:** a thin (~36 px) always-on-top strip. Each tab shows the app icon, a name (custom or the window title), and an unread badge.
- **Stage:** the screen area below the tab bar, minus the Dock if it's visible. The active app's window is resized to fill it exactly.
- A nice side effect: the real **macOS menu bar** always matches the active tab, so app menus keep working with no extra work.

### 4.2 Core interactions
| Action | Behavior |
|---|---|
| Click a tab | Activate the app, snap its window to the stage, hide the other tabbed apps |
| `⌃Tab` / `⌃⇧Tab` | Next / previous tab |
| `⌥⌘1…9` | Jump to tab N |
| `⌥⌘T` | Quick switcher: type to search tabs or launch a new app |
| Drag a tab | Reorder |
| Right-click a tab | Rename, pin/unpin, "Tab per window" toggle, Quit app, Remove from tabs |
| Click `+` | Pick a running app or an installed app to add |
| Click a pinned tab whose app isn't running | Launch it, then dock its window when it appears |
| User Cmd‑Tabs or clicks the Dock | Tab bar follows: the matching tab highlights and the window snaps to the stage |
| Hover a tab (v1.1) | Live thumbnail preview (ScreenCaptureKit) |

### 4.3 Tabs per app vs. per window
- **Default: one tab per app.** If the app has several windows (two Chrome windows, two VS Code projects), the tab shows a small `▾` menu to choose between them.
- **Optional, per app: "Tab per window."** Each window gets its own tab. Good for VS Code projects or separate Chrome profiles.

### 4.4 Workspaces (v1.1)
Named tab sets, such as **Comms** (Teams, Outlook, Claude), **Design** (Photoshop, Chrome, Notion) and **Build** (VS Code, Chrome, Claude). Switching workspace swaps the tab bar. Apps that aren't in the current workspace are hidden, not quit.

### 4.5 Split view (v1.2)
Drag a tab onto the stage to split it 50/50 or 60/40 between two apps, for example VS Code next to Chrome. Same docking approach, just with two rectangles.

---

## 5. Functional requirements

### MVP (v1.0)
1. **Permission onboarding:** detect whether Accessibility access is granted. If not, guide the user to System Settings › Privacy & Security › Accessibility and keep re-checking until it's on.
2. **App discovery:** list running regular apps (`NSWorkspace.runningApplications`, `activationPolicy == .regular`) that have at least one standard window.
3. **Tabs:** add, remove, reorder, pin, and rename tabs. Pinned tabs persist across restarts.
4. **Docking:** on activation, set the window's `AXPosition` and `AXSize` to the stage rect, raise it (`AXRaise`), and activate the app. Hide every other tabbed app (`NSRunningApplication.hide()`).
5. **Restore on exit:** remember each window's original frame and restore it when the window is un-tabbed or Desktop Tabs quits.
6. **Follow external switching:** watch `NSWorkspace.didActivateApplicationNotification` so Cmd‑Tab and Dock clicks update the active tab.
7. **Window lifecycle:** watch `AXObserver` events (window created, destroyed, moved, resized, minimized). New windows of a tabbed app get docked, and windows the user moves get re-snapped (with a "float this window" escape hatch, see §6).
8. **Launch on demand:** clicking a pinned tab whose app isn't running launches it (`NSWorkspace.openApplication`), waits for its first window, then docks it.
9. **Global hotkeys:** as listed in §4.2, and configurable.
10. **Unread badges:** read each app's Dock badge through the Dock's accessibility tree (`AXStatusLabel` on Dock items) and show it on the tab. Poll every 2 s or use AX notifications.
11. **Settings:** tab bar position (top or bottom), height, whether to auto-hide the Dock, launch at login, hotkeys.
12. **Menu-bar icon** for quick access, pause/resume, and quit.

### v1.1
- Workspaces (§4.4)
- Hover thumbnails (needs Screen Recording permission, opt-in)
- Per-app rules: "always tab per window", "never dock" for utility apps, title-regex matching
- Import and export of the config

### v1.2+
- Split view (§4.5)
- Web tabs (Approach B), with separate cookie stores per tab for multiple accounts
- Multi-monitor: one tab bar per display, or a "send tab to display" option
- Session restore: relaunch the whole workspace (apps plus tab order) after a reboot

---

## 6. Edge cases to handle

| Case | Handling |
|---|---|
| **Photoshop floating panels / palettes** | Only dock the window with `AXSubrole == AXStandardWindow`. Leave panels, sheets, and dialogs alone, since they move with their parent. Recommend Photoshop's "Application Frame" mode. |
| **Dialogs, sheets, popovers, Save dialogs** | Never resize non-standard windows. |
| **Apps with a minimum window size** larger than the stage | Accept the app's clamped size and center it in the stage. |
| **Native full-screen apps** (green button) | Detect `AXFullScreen`. Offer to exit full screen, or leave the app in its own Space and show a "full screen" indicator on the tab. |
| **Stage Manager / Spaces** | Warn if Stage Manager is on, since the two conflict. v1 works on the current Space only. |
| **User drags a docked window** | Re-snap it on mouse-up. Holding ⌥ while dragging "floats" the window, so it's still tabbed but not snapped. |
| **App quits or crashes** | Unpinned tab: remove it. Pinned tab: grey it out, and relaunch on click. |
| **Electron apps** (Teams, VS Code, Notion, Claude, Spotify) | These generally respect AX resize. Test each one in the spike. |
| **Notifications** | Unchanged. macOS notifications still appear, and clicking one activates the app, so the tab bar follows. |
| **Multiple monitors** | v1: the tab bar and stage live on one chosen display, and windows on other displays aren't touched. |
| **Performance** | No polling for window state (event-driven through AXObserver). Badge polling stays lightweight. Target idle CPU < 1%, RAM < 80 MB. |

---

## 7. Architecture

**Stack:** Swift 6, AppKit for windows and the AX layer, SwiftUI for the tab bar and settings. No Electron, because it would cost RAM and add latency.

```
┌───────────────────────────── Desktop Tabs.app ─────────────────────────────┐
│                                                                            │
│  TabBarPanel (NSPanel, .floating, non-activating)  ← SwiftUI TabBarView      │
│        │                                                                   │
│  TabStore (ObservableObject) ── Persistence (JSON in ~/Library/App Support)  │
│        │                                                                   │
│  StageController ── computes stage rect per display, handles split layouts   │
│        │                                                                   │
│  WindowManager (AX layer)                                                  │
│     ├─ AXWindowRef wrapper (position, size, raise, minimized, subrole)     │
│     ├─ AXObserver per tabbed app (created/destroyed/moved/resized)         │
│     └─ FrameMemory (original frames for restore)                           │
│        │                                                                   │
│  AppMonitor ── NSWorkspace launch/terminate/activate notifications          │
│  HotkeyManager ── global shortcuts (Carbon RegisterEventHotKey / KeyboardShortcuts pkg) │
│  BadgeReader ── reads Dock AXStatusLabel per bundle ID                     │
│  PermissionManager ── AXIsProcessTrustedWithOptions, onboarding UI          │
│  ThumbnailService (v1.1) ── ScreenCaptureKit SCScreenshotManager            │
└────────────────────────────────────────────────────────────────────────────┘
```

### 7.1 Data model
```swift
struct Workspace: Codable, Identifiable {
    var id: UUID
    var name: String
    var tabs: [Tab]
    var activeTabID: UUID?
}

struct Tab: Codable, Identifiable {
    var id: UUID
    var bundleID: String              // e.g. "com.microsoft.teams2"
    var customName: String?
    var isPinned: Bool
    var mode: TabMode                 // .perApp | .perWindow
    var windowMatcher: WindowMatcher? // title regex or index, for perWindow tabs
    var behavior: DockBehavior        // .snap | .float | .ignore
}

struct Settings: Codable {
    var barEdge: Edge                 // .top | .bottom
    var barHeight: CGFloat
    var hideOtherApps: Bool           // hide vs. just send behind
    var autoHideDock: Bool
    var launchAtLogin: Bool
    var hotkeys: [Action: KeyCombo]
}
```

### 7.2 Tab switch sequence
1. User clicks tab *B* (currently on *A*).
2. `StageController` resolves *B*'s target window. If the app isn't running, it launches it and waits for `kAXWindowCreatedNotification`, with a 10 s timeout.
3. `WindowManager` sets *B*'s window frame to the stage rect, then calls `AXRaise` and `app.activate()`.
4. After *B* is frontmost, hide *A* (and any other visible tabbed apps). Doing it in this order avoids the desktop flashing through.
5. `TabStore.activeTabID = B` and the tab bar re-renders.

---

## 8. Permissions, security & distribution
- **Accessibility** (required): to move, resize, and raise other apps' windows.
- **Screen Recording** (optional, v1.1): only for hover thumbnails.
- **No App Sandbox**, because Accessibility control of other apps isn't allowed in the sandbox. That rules out the Mac App Store.
- **Distribution:** Developer ID–signed, notarized `.dmg`, with auto-update via Sparkle.
- **Privacy:** no network access in v1 apart from the update check. Window titles never leave the machine.

---

## 9. Milestones

| Milestone | Scope | Est. |
|---|---|---|
| **M0: Technical spike** | Command-line Swift prototype: list windows, then snap/raise/hide Teams, Chrome, VS Code, Photoshop and Spotify. Measure switch latency and note any app that misbehaves. | 3–5 days |
| **M1: MVP** | Tab bar panel, TabStore + persistence, docking, follow Cmd‑Tab, hotkeys, launch on demand, restore on quit, permission onboarding | 2–3 weeks |
| **M2: Daily-driver polish** | Badges, per-window tabs, edge cases from §6, settings UI, launch at login, signing and notarization | 2 weeks |
| **M3: v1.1** | Workspaces, hover thumbnails, per-app rules | 2 weeks |
| **M4: v1.2** | Split view, web tabs, multi-monitor | 3–4 weeks |

**Test matrix (each milestone):** Teams, Claude, Chrome, Notion, Photoshop, VS Code, Spotify, Outlook, PowerPoint, Finder. Check docking, multiple windows, dialogs, full screen, quit/relaunch, and Cmd‑Tab sync.

---

## 10. Risks
| Risk | Likelihood | Mitigation |
|---|---|---|
| An app ignores or fights AX resize | Medium | Per-app "float" behavior; center the window instead of resizing it |
| A brief flash when switching (one app hides before the next appears) | Medium | Raise the new app before hiding the old one; tune the ordering in M0 |
| Apple restricts the Accessibility APIs | Low | Rectangle, Magnet and others depend on the same APIs, so a change would be widely visible |
| Users expect *true* embedding | Medium | Explain the docking model in onboarding; web tabs (v1.2) cover the "truly inside" case for web-capable apps |

---

## 11. Future: Windows version
Windows makes this easier: `SetParent` can genuinely reparent another app's top-level window into our own window. A Windows port (C#/WinUI or Rust) could offer true embedding. It's out of scope for v1, but keeping `TabStore` and settings platform-neutral (shared JSON format) would make a port easier.

---

## 12. Open questions for you
1. **Tab per app or per window** by default? (The spec assumes per app, with a window picker.)
2. **Tab bar position:** top (like a browser) or bottom (near the Dock)?
3. **Hide the macOS Dock** while Desktop Tabs is running, since the tab bar replaces it?
4. **Multiple monitors:** do you work across more than one display? That decides whether multi-monitor moves into v1.
5. **Audience:** just for you, or something to share or sell? That affects polish, onboarding, and licensing.
6. **Web versions:** would you accept the *web* versions of Teams, Outlook and Notion if they lived truly inside the window, or must they be the desktop apps?
7. **Name:** "Desktop Tabs" is a placeholder.
