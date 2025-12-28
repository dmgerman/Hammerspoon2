# Hammerspoon 2 - LLM Developer Guide

## Project Overview

**Hammerspoon 2** is a complete modernization and rewrite of [Hammerspoon](https://github.com/Hammerspoon/hammerspoon), a powerful macOS automation tool. The original Hammerspoon uses Lua as its scripting language; Hammerspoon 2 replaces it with **JavaScript** while maintaining and extending the same philosophy: bridging native macOS APIs to a scripting language for desktop automation.

### Key Facts
- **Language**: Swift 5.x (application), JavaScript (user scripts)
- **JavaScript Engine**: JavaScriptCore (Apple's WebKit JS engine)
- **Platform**: macOS Sequoia+ (deployment target adjustable)
- **Architecture**: SwiftUI-based menu bar application
- **Config Location**: `~/.config/Hammerspoon2/init.js` (configurable)
- **Status**: Work in progress, actively developed
- **Reference Repository**: A clone of the original Hammerspoon repository is maintained at `hs_repo_old/` for API reference and comparison purposes

### Differences from Original Hammerspoon

| Aspect | Original Hammerspoon | Hammerspoon 2 |
|--------|---------------------|---------------|
| Scripting Language | Lua | JavaScript (ES6+) |
| Implementation | Objective-C (51%), Lua (25%), C (16%) | Swift (modern) |
| Engine | LuaSkin (Lua 5.3/5.4) | JavaScriptCore |
| UI Framework | Cocoa/AppKit | SwiftUI |
| Module System | Objective-C extensions | Swift classes with JSExport protocols |
| Type Bridge | Lua userdata | Swift types with JSConvertible protocol |
| Config File | `~/.hammerspoon/init.lua` | `~/.config/Hammerspoon2/init.js` |

## Architecture

### High-Level System Design

```
┌──────────────────────────────────────────────────────────┐
│                  User Configuration                      │
│             ~/.config/Hammerspoon2/init.js               │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│              JSEngine (JavaScriptCore)                   │
│  ┌────────────────────────────────────────────────────┐  │
│  │  JavaScript Global Namespace:                      │  │
│  │  • console - Logging functions                     │  │
│  │  • require() - CommonJS-style module loading       │  │
│  │  • EventEmitter - Event handling (from engine.js)  │  │
│  │  • hs - Root namespace for all modules             │  │
│  │  • Custom Types: HSRect, HSPoint, HSSize, etc.     │  │
│  └────────────────────────────────────────────────────┘  │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│           ModuleRoot (hs namespace manager)              │
│  Lazy-loads modules on first property access             │
│  hs.alert, hs.application, hs.hotkey, hs.timer, etc.     │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│         Individual Module Implementations                │
│  Swift classes conforming to HSModuleAPI protocol        │
│  Each module:                                            │
│  • Exposes @objc protocol inheriting from JSExport       │
│  • Implements native macOS API bridging                  │
│  • May have companion .js file for convenience methods   │
│  • Manages lifecycle (init/shutdown)                     │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│              Native macOS APIs                           │
│  • NSRunningApplication (app management)                 │
│  • Accessibility API via AXSwift                         │
│  • Carbon Event Manager (hotkeys)                        │
│  • NSTimer/RunLoop (timers)                              │
│  • CoreGraphics (window/screen geometry)                 │
└──────────────────────────────────────────────────────────┘
```

### Core Design Patterns

1. **Protocol-Oriented Programming**
   - Every module defines an `@objc protocol` inheriting from `JSExport`
   - Enables clean JavaScript API definition separate from implementation
   - Facilitates testing via protocol mocking

2. **Lazy Module Loading**
   - Modules instantiated only when first accessed via `hs.module_name`
   - Fast startup time - only loads what user scripts require
   - Implemented in `ModuleRoot.getOrCreate()`

3. **Dependency Injection**
   - Core managers use protocol types (`JSEngineProtocol`, `SettingsManagerProtocol`, `FileSystemProtocol`)
   - Enables unit testing with mock implementations
   - See `ManagerManager.swift` for example

4. **Hybrid Swift/JavaScript Architecture**
   - Performance-critical code in Swift (native API calls, observers, timers)
   - Convenience methods in JavaScript (time conversions, event multiplexing)
   - Example: `hs.timer.js` adds `minutes()`, `hours()` helpers to Swift timer module

5. **Type Bridging System**
   - Custom types implement `HSTypeAPI` protocol
   - Automatic conversion via `JSConvertible` protocol
   - Examples: `CGRect` ↔ `HSRect`, `CGPoint` ↔ `HSPoint`

## Project Structure

```
Hammerspoon2/
├── Hammerspoon 2/                   # Main application bundle
│   ├── Lifecycle/
│   │   └── Hammerspoon_2App.swift   # SwiftUI app entry point, AppDelegate
│   │
│   ├── Engine/                      # JavaScript engine core
│   │   ├── JSEngine.swift           # JavaScriptCore wrapper, context management
│   │   ├── ModuleRoot.swift         # hs.* namespace, lazy module loading
│   │   ├── engine.js                # EventEmitter, JS-side utilities
│   │   ├── InjectTypes.swift        # Registers custom types for JS construction
│   │   └── Types/                   # Bridgeable type definitions
│   │       ├── HSRect.swift         # Rectangle {x, y, w, h}
│   │       ├── HSPoint.swift        # Point {x, y}
│   │       ├── HSSize.swift         # Size {w, h}
│   │       └── HSFont.swift         # Font representation
│   │
│   ├── Managers/                    # Application lifecycle managers
│   │   ├── ManagerManager.swift     # Orchestrates boot/shutdown, config loading
│   │   └── SettingsManager.swift    # UserDefaults wrapper, config location
│   │
│   ├── Modules/                     # hs.* API implementations
│   │   ├── console/                 # Pre-hs logging (console.log/error/warn)
│   │   │   └── ConsoleModule.swift
│   │   ├── hs.alert/                # On-screen notifications
│   │   │   ├── AlertModule.swift
│   │   │   └── HSAlert.swift        # Alert configuration object
│   │   ├── hs.application/          # Application management
│   │   │   ├── ApplicationModule.swift
│   │   │   ├── hs.application.js    # JS-side event multiplexing
│   │   │   ├── HSApplication.swift  # Application object wrapper
│   │   │   └── ApplicationWatcher.swift
│   │   ├── hs.ax/                   # Accessibility API
│   │   │   ├── AXModule.swift
│   │   │   └── HSAXElement.swift    # AX element wrapper
│   │   ├── hs.hotkey/               # Global hotkey bindings
│   │   │   ├── HotkeyModule.swift
│   │   │   └── HSHotkey.swift       # Hotkey object
│   │   ├── hs.timer/                # Timer/scheduling
│   │   │   ├── TimerModule.swift
│   │   │   ├── hs.timer.js          # Convenience helpers
│   │   │   └── HSTimer.swift        # Timer object
│   │   ├── hs.window/               # Window management
│   │   │   ├── WindowModule.swift
│   │   │   └── HSWindow.swift       # Window object wrapper
│   │   └── [other modules]/
│   │
│   ├── Windows/                     # SwiftUI views
│   │   ├── ConsoleView.swift        # JavaScript REPL, log viewer
│   │   ├── SettingsView.swift       # Preferences UI
│   │   ├── AboutView.swift          # About dialog
│   │   └── ContentView.swift        # Placeholder main window
│   │
│   ├── Utilities/                   # Cross-cutting concerns
│   │   ├── Logging.swift            # HammerspoonLog, log levels
│   │   ├── Exceptions.swift         # HammerspoonError, error categories
│   │   └── AKLog.swift              # AKTrace, AKInfo, AKError helpers
│   │
│   ├── Extensions/                  # Swift standard library extensions
│   │   └── NSScreen+Ext.swift       # Screen utilities
│   │
│   └── Protocols/                   # Shared protocol definitions
│       ├── HSModuleAPI.swift        # Base protocol for all modules
│       ├── HSTypeAPI.swift          # Base protocol for bridged types
│       ├── JSEngineProtocol.swift   # Testable engine interface
│       ├── FileSystemProtocol.swift # Testable filesystem interface
│       └── SettingsManagerProtocol.swift
│
├── Hammerspoon 2Tests/              # Test suite
│   ├── Mocks/                       # Mock protocol implementations
│   ├── ManagerTests/                # ManagerManager tests
│   ├── IntegrationTests/            # End-to-end module tests
│   └── Helpers/                     # Test utilities
│
├── docs/                            # Documentation
│   └── TYPESCRIPT.md                # TypeScript setup guide
│
├── scripts/                         # Build/utility scripts
│
└── Hammerspoon 2.xcodeproj/         # Xcode project
```

## JavaScript Engine Integration

### JSEngine.swift

**Location**: `Hammerspoon 2/Engine/JSEngine.swift`

The core engine singleton that manages JavaScriptCore.

**Key Responsibilities**:
- Creates and manages `JSVirtualMachine` and `JSContext`
- Injects global functions and namespaces
- Loads and executes JavaScript code
- Manages engine lifecycle (create/reset/delete context)

**Initialization Sequence** (in `createContext()`):

```swift
1. Create JSVirtualMachine and JSContext
2. Assign unique ID to context: "Hammerspoon {UUID}"
3. Inject console namespace (ConsoleModule for logging)
4. Inject require() function (for loading external .js files)
5. Inject custom type bridges (HSRect, HSPoint, HSSize, HSFont, HSAlert)
6. Load and execute engine.js (provides EventEmitter class)
7. Create hs namespace (ModuleRoot instance)
```

**Public API** (via `JSEngineProtocol`):

```swift
subscript(key: String) -> Any?  // Get/set global JS variables
func eval(_ script: String) -> Any?  // Execute JS code
func evalFromURL(_ url: URL) throws -> Any?  // Execute JS file
func resetContext() throws  // Destroy and recreate JS context
func hasContext() -> Bool  // Check if context exists
```

### engine.js

**Location**: `Hammerspoon 2/Engine/engine.js`

JavaScript-side utilities loaded during engine initialization.

**Contents**:
- `EventEmitter` class - Node.js-style event emitter
  - Methods: `on(event, listener)`, `emit(event, ...args)`, `removeListener(event, listener)`
  - Used by modules like `hs.application` for event multiplexing

**Critical Note**: Code in `engine.js` cannot depend on any `hs.*` modules, as they haven't loaded yet when this executes.

### require() Function

**Location**: Implemented in `JSEngine.swift:injectRequire()`

Custom CommonJS-style module loader for external JavaScript files.

**Features**:
- Expands tilde in paths: `require("~/scripts/helpers.js")`
- Loads files relative to current working directory
- Returns result of script evaluation
- Logs errors if file not found or unreadable

**Example Usage**:
```javascript
// In init.js
require("~/hammerspoon-config/window-management.js");
require("./local-helpers.js");
```

## Module System

### Module Lifecycle

Every module follows this pattern:

1. **Definition** - Declare API protocol and implementation
2. **Registration** - Add property to `ModuleRoot`
3. **Lazy Loading** - Instantiated on first access
4. **Shutdown** - Cleanup when context destroyed

### Module Structure Template

```swift
// 1. Define JavaScript API
@objc protocol HSExampleModuleAPI: JSExport {
    @objc func doSomething(_ param: String) -> Bool
}

// 2. Implement module
@objc class HSExampleModule: NSObject, HSModuleAPI, HSExampleModuleAPI {
    var name = "hs.example"

    // Required initializer
    required override init() {
        super.init()
    }

    // Cleanup resources
    func shutdown() {
        // Remove observers, stop timers, etc.
    }

    // API implementation
    @objc func doSomething(_ param: String) -> Bool {
        // Native macOS API calls here
        return true
    }
}
```

```swift
// 3. Register in ModuleRoot.swift
@objc protocol ModuleRootAPI: JSExport {
    // ... other modules ...
    @objc var example: HSExampleModule { get }
}

@objc class ModuleRoot: NSObject, ModuleRootAPI {
    @objc var example: HSExampleModule {
        get { getOrCreate(name: "example", type: HSExampleModule.self) }
    }
}
```

### Module Loading Process

When user code accesses `hs.timer` for first time:

```
1. JavaScript accesses property: hs.timer
2. ModuleRoot property getter called
3. getOrCreate(name: "timer", type: HSTimerModule.self)
   a. Check if "timer" already in modules dictionary
   b. If not: instantiate HSTimerModule()
   c. Check if Bundle contains "hs.timer.js"
   d. If exists: evalFromURL(hs.timer.js) to load JS helpers
   e. Store in modules["timer"]
   f. Return module instance
4. Module available to JavaScript
```

### Companion JavaScript Files

Modules can have optional `.js` files providing JavaScript-side utilities.

**Example**: `hs.timer.js`
```javascript
hs.timer.minutes = function(n) { return n * 60; }
hs.timer.hours = function(n) { return n * 3600; }
hs.timer.days = function(n) { return n * 86400; }
hs.timer.weeks = function(n) { return n * 604800; }
```

**Benefits**:
- Keeps simple helpers in JavaScript (no Swift recompilation)
- User-friendly API extensions
- Loaded automatically with module

## Type System

### Type Bridge Architecture

Hammerspoon 2 bridges Swift value types to JavaScript objects.

**Base Protocol**: `HSTypeAPI`
```swift
@objc protocol HSTypeAPI: JSExport {
    @objc var typeName: String { get }
}
```

### Core Geometric Types

**HSRect** - Rectangle
```swift
@objc class HSRect: NSObject, HSTypeAPI {
    @objc var x: Double
    @objc var y: Double
    @objc var w: Double
    @objc var h: Double
    @objc var typeName: String { "HSRect" }
}
```

JavaScript usage:
```javascript
var rect = new HSRect();
rect.x = 100;
rect.y = 200;
rect.w = 800;
rect.h = 600;
```

**HSPoint** - 2D Point
```swift
@objc class HSPoint: NSObject, HSTypeAPI {
    @objc var x: Double
    @objc var y: Double
}
```

**HSSize** - Dimensions
```swift
@objc class HSSize: NSObject, HSTypeAPI {
    @objc var w: Double
    @objc var h: Double
}
```

### Type Conversion Protocol

**JSConvertible** enables automatic conversion between Swift and bridge types:

```swift
protocol JSConvertible {
    associatedtype BridgeType: NSObject
    init(from bridge: BridgeType)
    func toBridge() -> BridgeType
}

// Example: CGRect ↔ HSRect
extension CGRect: JSConvertible {
    typealias BridgeType = HSRect

    init(from bridge: HSRect) {
        self.init(x: bridge.x, y: bridge.y,
                  width: bridge.w, height: bridge.h)
    }

    func toBridge() -> HSRect {
        HSRect(x: origin.x, y: origin.y,
               w: size.width, h: size.height)
    }
}
```

### Object Types

Beyond geometric types, modules define their own object types:

- **HSApplication** (hs.application) - Running application wrapper
- **HSWindow** (hs.window) - Window object
- **HSTimer** (hs.timer) - Timer instance
- **HSHotkey** (hs.hotkey) - Hotkey binding
- **HSAXElement** (hs.ax) - Accessibility element
- **HSAlert** (hs.alert) - Alert configuration

These types typically:
- Hold references to native objects (NSRunningApplication, NSWindow, etc.)
- Expose methods to manipulate them
- Implement `HSTypeAPI` for type identification

## Application Lifecycle

### Startup Flow

```
1. SwiftUI App launches (Hammerspoon_2App)
2. NSApplicationDelegateAdaptor creates AppDelegate
3. AppDelegate.applicationDidFinishLaunching() called
4. ManagerManager.shared.boot() invoked
   a. JSEngine.resetContext() - Create fresh JS environment
   b. Change working directory to config directory
   c. Check config file exists
   d. JSEngine.evalFromURL(init.js) - Execute user config
5. User's init.js runs, accessing hs.* modules as needed
6. Modules lazy-load on first access
7. Event loops start (hotkeys, timers, watchers all active)
```

**Key Files**:
- `Hammerspoon 2/Lifecycle/Hammerspoon_2App.swift:15` - AppDelegate.applicationDidFinishLaunching
- `Hammerspoon 2/Managers/ManagerManager.swift:34` - boot() method

### Configuration Loading

**ManagerManager.boot()** process:

```swift
func boot() throws {
    // 1. Reset JavaScript context (destroys old modules)
    try engine.resetContext()

    // 2. Set working directory to config directory
    let configDir = settings.configLocation.deletingLastPathComponent()
    FileManager.default.changeCurrentDirectoryPath(configDir.path)

    // 3. Validate config file exists
    guard fileSystem.fileExists(atPath: settings.configLocation.path) else {
        throw error
    }

    // 4. Execute init.js
    try engine.evalFromURL(settings.configLocation)
}
```

**Default Config Path**: `~/.config/Hammerspoon2/init.js`

Users can change this in Settings UI (stored in UserDefaults).

### Config Reload

Two ways to reload configuration:

1. **Menu Bar** → "Reload Config"
2. **JavaScript** → `hs.reload()`

Both trigger `ManagerManager.boot()`, which:
- Shuts down all active modules
- Destroys JavaScript context
- Creates new context
- Re-executes init.js

This gives users a "fresh start" without restarting the app.

### Shutdown Flow

```
1. User selects "Quit" from menu bar
2. ManagerManager.shutdown() called
3. NSApp.terminate(self) initiated
4. JSEngine.deleteContext() called
   a. ModuleRoot.shutdown() - Iterates through all loaded modules
   b. Each module's shutdown() method called
      - Remove observers
      - Invalidate timers
      - Unregister hotkeys
   c. context = nil, vm = nil
5. Application terminates
```

## Implemented Modules

### hs.application

**File**: `Hammerspoon 2/Modules/hs.application/ApplicationModule.swift`

Application lifecycle and management.

**Key Functions**:
- `runningApplications()` - Get all running apps
- `find(bundleID)` / `find(name)` / `get(pid)` - Find applications
- `launchOrFocus(name)` - Launch/activate app
- `frontmostApplication()` - Get active app
- `menuBarOwningApplication()` - Get menu bar owner

**Application Objects** (HSApplication):
- `title()`, `bundleID()`, `pid()`, `path()`
- `activate()`, `kill()`, `hide()`, `unhide()`
- `windows()` - Get app's windows
- `mainWindow()` - Get main window

**Event Watching**:
```javascript
hs.application.addWatcher("didLaunch", function(event, app) {
    console.log("Launched:", app.title());
});

// Events: didLaunch, didTerminate, didActivate, didDeactivate, didHide, didUnhide
```

**Implementation Note**: `hs.application.js` provides event multiplexing - multiple listeners per event type using EventEmitter pattern.

### hs.window

**File**: `Hammerspoon 2/Modules/hs.window/WindowModule.swift`

Window discovery and manipulation.

**Discovery**:
- `focusedWindow()` - Currently focused window
- `allWindows()` - All windows from all apps
- `visibleWindows()` - Non-minimized windows
- `orderedWindows()` - Windows in Z-order (front to back)
- `windowForID(id)` - Get window by ID
- `windowsMatchingTitle(pattern)` - Regex search

**Window Objects** (HSWindow):
- `frame()` / `setFrame(rect)` - Get/set position and size
- `application()` - Get owning application
- `title()`, `role()`, `subrole()` - Window metadata
- `focus()`, `raise()`, `minimize()`, `unminimize()`
- `toggleFullScreen()`, `toggleZoom()`
- `close()` - Close window
- `isStandard()`, `isMinimized()`, `isVisible()`

**Example**:
```javascript
var win = hs.window.focusedWindow();
var frame = win.frame();
frame.x = 0;
frame.y = 0;
win.setFrame(frame); // Move to top-left corner
```

### hs.hotkey

**File**: `Hammerspoon 2/Modules/hs.hotkey/HotkeyModule.swift`

System-wide keyboard shortcut bindings using Carbon Event Manager.

**Key Function**:
```javascript
hs.hotkey.bind(modifiers, key, pressedCallback, releasedCallback)
```

**Parameters**:
- `modifiers`: Array of `["cmd", "ctrl", "alt", "shift"]` or any combination
- `key`: String like "a", "F1", "space", "return", "left", etc.
- `pressedCallback`: Function called on key down
- `releasedCallback`: Optional function called on key up

**Hotkey Objects** (HSHotkey):
- `enable()` / `disable()` - Toggle hotkey
- `delete()` - Permanently remove

**Example**:
```javascript
hs.hotkey.bind(["cmd", "alt"], "r", function() {
    hs.reload();
});

hs.hotkey.bind(["ctrl", "shift"], "left", function() {
    var win = hs.window.focusedWindow();
    // Move window left...
});
```

**Implementation Details**:
- Maps modifier names → Carbon modifier flags
- Maps key names → virtual key codes (see keyMap dictionary)
- Registers global event handlers
- Supports F1-F20, arrow keys, special keys

### hs.timer

**File**: `Hammerspoon 2/Modules/hs.timer/TimerModule.swift`

Timer and scheduling utilities.

**Timer Creation**:
```javascript
// One-shot timer
hs.timer.doAfter(5, function() {
    console.log("5 seconds elapsed");
});

// Repeating timer
var timer = hs.timer.doEvery(10, function() {
    console.log("Every 10 seconds");
});

// Timer at specific time of day
hs.timer.doAt(3600, 0, function() {
    console.log("It's 1 AM!");
});

// Create but don't start
var timer = hs.timer.create(5, function() { ... });
timer.start();
```

**Timer Objects** (HSTimer):
- `start()` / `stop()` - Control timer
- `running()` - Check if active
- `setNextTrigger(seconds)` - Reschedule
- `fire()` - Trigger immediately

**Time Utilities**:
```javascript
hs.timer.secondsSinceEpoch() // UNIX timestamp with milliseconds
hs.timer.absoluteTime()       // Nanoseconds since boot
hs.timer.localTime()          // Seconds since midnight

// From hs.timer.js:
hs.timer.minutes(5)  // Returns 300
hs.timer.hours(2)    // Returns 7200
```

**Advanced** (from hs.timer.js):
```javascript
// Wait until condition is true
hs.timer.waitUntil(
    function() { return hs.window.focusedWindow() != null; },
    function() { console.log("Window focused!"); }
);

// Do something while condition is true
hs.timer.doWhile(
    function() { return hs.application.find("Safari") != null; },
    function() { console.log("Safari still running"); }
);
```

### hs.ax

**File**: `Hammerspoon 2/Modules/hs.ax/AXModule.swift`

Accessibility API wrapper (requires Accessibility permission).

**Key Functions**:
- `systemWideElement()` - Get root AX element
- `elementAtPoint(x, y)` - Get element at screen position
- `applicationElement(app)` - Get element for application

**AX Elements** (HSAXElement):
- `attribute(name)` - Get attribute value
- `setAttribute(name, value)` - Set attribute
- `performAction(action)` - Trigger action
- `children()` - Get child elements
- `parent()` - Get parent element
- `setWatcher(event, callback)` - Watch for AX notifications

**Example**:
```javascript
var element = hs.ax.elementAtPoint(100, 100);
var role = element.attribute("AXRole");
console.log("Element role:", role);

// Watch for window creation
var appElement = hs.ax.applicationElement(hs.application.frontmostApplication());
appElement.setWatcher("AXWindowCreated", function(element, notif) {
    console.log("New window created!");
});
```

**Dependencies**: Uses [AXSwift](https://github.com/tmandry/AXSwift) library for Swift Accessibility API wrapper.

### hs.alert

**File**: `Hammerspoon 2/Modules/hs.alert/AlertModule.swift`

On-screen notification system.

**Usage**:
```javascript
// Simple alert
hs.alert.show("Hello World!");

// Alert with duration
hs.alert.show("This stays for 5 seconds", 5);

// Custom alert
var alert = new HSAlert("Custom Alert");
alert.textFont = "Helvetica";
alert.textSize = 36;
alert.fadeInDuration = 0.5;
alert.fadeOutDuration = 0.5;
hs.alert.showAlert(alert);
```

**HSAlert Properties**:
- `text` - Alert message
- `textFont` - Font name
- `textSize` - Font size
- `fadeInDuration` - Fade in time
- `fadeOutDuration` - Fade out time
- `radius` - Corner radius
- `padding` - Internal padding
- `strokeWidth` - Border width
- `fillColor`, `strokeColor`, `textColor` - Colors

### hs.ipc

**File**: `Hammerspoon 2/Modules/hs.ipc/IPCModule.swift`

Inter-process communication module enabling external processes to communicate with Hammerspoon 2 via CFMessagePort.

**Key Functions**:
- `localPort(name, callback)` - Create server message port
- `remotePort(name)` - Connect to remote port
- `cliInstall([path], [silent])` - Install hs2 command-line tool
- `cliUninstall([path], [silent])` - Remove hs2 symlinks
- `cliStatus([path], [silent])` - Check installation status

**Message Port Objects** (HSMessagePort):
- `name` - Port name string
- `isValid` - Port validity check
- `isRemote` - Local vs remote distinction
- `sendMessage(data, msgID, [timeout], [oneWay])` - Send IPC message
- `delete()` - Cleanup port

**Example**:
```javascript
// Create server port
const port = hs.ipc.localPort("MyService", function(port, msgID, data) {
    console.log("Received:", data);
    return "Response";
});

// From another process or CLI
const remote = hs.ipc.remotePort("MyService");
remote.sendMessage("Hello", 0);
```

**Default Port**: `Hammerspoon2` - Handles CLI communication protocol

**CLI Integration**: The hs.ipc module works seamlessly with the hs2 command-line tool to enable:
- Remote code execution from terminal
- Interactive REPL mode
- Script automation via shell integration
- Custom IPC services for third-party tools

See the hs2 CLI Tool section below for usage examples.

### Other Modules

**hs.console** - Enhanced console features
- `clearConsole()` - Clear console window
- `consoleCommandColor()` - Get/set command text color

**hs.hash** - Hashing and encoding
- `base64encode(str)` / `base64decode(str)`
- `MD5(str)`, `SHA1(str)`, `SHA256(str)`, `SHA512(str)`
- `hmacMD5(key, data)`, `hmacSHA256(key, data)`, etc.

**hs.permissions** - System permission management
- `hasAccessibility()` / `requestAccessibility()`
- `hasScreenRecording()` / `requestScreenRecording()`
- `hasCamera()` / `requestCamera()`
- `hasMicrophone()` / `requestMicrophone()`

**hs.appinfo** - Hammerspoon app metadata
- `version()` - App version string
- `build()` - Build number
- `bundleID()` - Bundle identifier
- `bundlePath()` - App bundle path

**console** (pre-hs namespace) - Basic logging
- `console.log()`, `console.error()`, `console.warn()`, `console.info()`
- Available immediately during engine initialization

## User Configuration Guide

### Example init.js

```javascript
// ~/.config/Hammerspoon2/init.js

console.log("Hammerspoon 2 loading...");

// Window management hotkeys
hs.hotkey.bind(["cmd", "alt"], "left", function() {
    var win = hs.window.focusedWindow();
    var frame = win.frame();
    var screen = win.screen().frame();
    frame.x = screen.x;
    frame.y = screen.y;
    frame.w = screen.w / 2;
    frame.h = screen.h;
    win.setFrame(frame);
});

// Application launcher
hs.hotkey.bind(["cmd", "alt"], "t", function() {
    hs.application.launchOrFocus("Terminal");
});

// Reload config
hs.hotkey.bind(["cmd", "alt"], "r", function() {
    hs.alert.show("Reloading config...");
    hs.reload();
});

// Watch for app launches
hs.application.addWatcher("didLaunch", function(eventName, app) {
    hs.alert.show(app.title() + " launched!");
});

// Periodic task
hs.timer.doEvery(3600, function() {
    hs.alert.show("Hourly reminder!");
});

console.log("Hammerspoon 2 ready!");
```

### Multi-File Configurations

Use `require()` to organize code:

**init.js**:
```javascript
require("./window-management.js");
require("./app-launchers.js");
require("./auto-actions.js");
```

**window-management.js**:
```javascript
// All window management hotkeys
hs.hotkey.bind(["cmd", "alt"], "f", function() {
    hs.window.focusedWindow().toggleFullScreen();
});
// ... more hotkeys
```

### TypeScript Support

See `docs/TYPESCRIPT.md` for complete TypeScript setup guide.

**Quick Start**:
```bash
cd ~/.config/Hammerspoon2
npm init -y
npm install --save-dev typescript
# Download hammerspoon.d.ts
npx tsc --init
```

Write config in TypeScript with full autocomplete and type checking, then compile to JavaScript.

### hs2 Command-Line Tool

The `hs2` CLI tool enables terminal access to Hammerspoon 2 for automation, scripting, and interactive development.

**Installation**:
```javascript
// In Hammerspoon 2 Console
hs.ipc.cliInstall("/usr/local");
```

**Usage Examples**:
```bash
# Execute single command
hs2 -c "hs.alert.show('Hello from terminal')"

# Interactive REPL
hs2 -i

# Execute JavaScript file
hs2 ~/scripts/automation.js

# Pipe input
echo "console.log(hs.application.frontmostApplication().title())" | hs2 -s

# With timeout
hs2 -t 10 -c "longRunningOperation()"

# Quiet mode (errors only)
hs2 -q -c "backgroundTask()"

# Console mirroring (see Hammerspoon console output in terminal)
hs2 -C

# As shebang script
#!/usr/local/bin/hs2
console.log("Running from file");
hs.application.launchOrFocus("Safari");
```

**Options**:
- `-A` - Auto-launch Hammerspoon 2 if not running
- `-a` - Exit with error if Hammerspoon 2 not running
- `-i` - Force interactive mode
- `-s` - Read from stdin
- `-c code` - Execute code (can be repeated)
- `-m name` - Connect to custom port (default: "Hammerspoon2")
- `-n` - Disable colors
- `-N` - Force colors
- `-C` - Enable console mirroring
- `-q` - Quiet mode
- `-t seconds` - Set timeout (default: 4.0)
- `-h` - Display help

**Tab Completion**: Available in interactive mode, completes against `hs.*` namespace (minimal in v1.0).

**Command History**: In-session history via up/down arrows (persistence deferred to v2.0).

**Special Variables in CLI Context**:
- `_cli.remote` - IPC message port for communication
- `_cli.quietMode` - Boolean indicating quiet mode
- `_cli.console` - Boolean indicating console mirroring
- `_cli.args` - Array of custom arguments (passed after `--`)
- `print(...)` - Enhanced print that outputs to terminal

## Development Guidelines

### Adding a New Module

1. **Create module directory**: `Hammerspoon 2/Modules/hs.newmodule/`

2. **Define API protocol**:
```swift
@objc protocol HSNewModuleAPI: JSExport {
    @objc func doSomething(_ param: String) -> String
}
```

3. **Implement module class**:
```swift
@objc class HSNewModule: NSObject, HSModuleAPI, HSNewModuleAPI {
    var name = "hs.newmodule"

    required override init() {
        super.init()
    }

    func shutdown() {
        // Cleanup
    }

    @objc func doSomething(_ param: String) -> String {
        return "Result: \(param)"
    }
}
```

4. **Register in ModuleRoot.swift**:
```swift
@objc protocol ModuleRootAPI: JSExport {
    // ...
    @objc var newmodule: HSNewModule { get }
}

@objc class ModuleRoot: NSObject, ModuleRootAPI {
    @objc var newmodule: HSNewModule {
        get { getOrCreate(name: "newmodule", type: HSNewModule.self) }
    }
}
```

5. **Optional: Add hs.newmodule.js** for JavaScript helpers

6. **Add to Xcode project** in appropriate groups

### Module Best Practices

- **Thread Safety**: Mark classes with `@MainActor` if they interact with UI/AppKit
- **Memory Management**:
  - Store active timers/observers in module instance variables
  - Clean up in `shutdown()` method
  - Add `deinit` with logging for leak detection
- **Error Handling**: Log errors via `AKError()`, don't crash
- **JavaScript API Design**:
  - Use clear, descriptive names
  - Return objects for complex data (not tuples/arrays)
  - Provide both synchronous and async variants where appropriate
- **Documentation**: Add doc comments to `@objc` protocol methods (appears in autocomplete)

### Type Bridge Best Practices

When creating custom types:

1. Inherit from `NSObject`
2. Conform to `HSTypeAPI`
3. Mark all properties `@objc`
4. Add to `InjectTypes.swift` for JavaScript construction
5. Implement `JSConvertible` if bridging from/to Swift value types

### Testing

**Test Structure**: `Hammerspoon 2Tests/`

**Mock Protocols**: Create mocks in `Mocks/` directory
```swift
class MockJSEngine: JSEngineProtocol {
    var evalCalled = false
    var lastScript: String?

    func eval(_ script: String) -> Any? {
        evalCalled = true
        lastScript = script
        return nil
    }
    // ... implement other methods
}
```

**Integration Tests**: Test modules with real JavaScriptCore context
```swift
class HSTimerIntegrationTests: XCTestCase {
    var harness: JSTestHarness!

    override func setUp() {
        harness = JSTestHarness()
    }

    func testTimerDoAfter() {
        let result = harness.eval("hs.timer.doAfter(1, function() {})")
        XCTAssertNotNil(result)
    }
}
```

**Test Helpers**: Use `JSTestHarness` for JS eval in tests

### Logging

Use the AK logging functions:

```swift
AKTrace("Detailed trace message")  // Verbose logging
AKInfo("Informational message")    // General info
AKError("Error message")           // Errors and warnings
```

Logs visible in:
- Console window (in-app)
- Xcode console
- macOS Console.app (search for "Hammerspoon")

### Code Style

- Use Swift 5+ modern features (async/await, actors where appropriate)
- Follow Swift naming conventions (camelCase, descriptive names)
- Use `@_documentation(visibility: private)` for internal classes
- Prefer `let` over `var`
- Use guard for early returns
- Add `// MARK: -` section comments

## Debugging Tips

### JavaScript Console

Open Console window: Menu Bar → Open Console

**Features**:
- View all logs (console.log, AKTrace, etc.)
- Filter by log level
- JavaScript REPL - evaluate code on-the-fly
- Command history (up/down arrows)

**REPL Usage**:
```javascript
> hs.application.frontmostApplication().title()
"Xcode"

> var win = hs.window.focusedWindow()
> win.frame()
{x: 0, y: 0, w: 1920, h: 1080}
```

### Common Issues

**Module not loading**:
- Check it's registered in `ModuleRoot`
- Verify protocol inherits from `JSExport`
- Ensure all methods are marked `@objc`

**Type conversion errors**:
- Verify type conforms to `HSTypeAPI`
- Check it's added to `InjectTypes.swift`
- Ensure proper `@objc` annotations

**Hotkeys not working**:
- Check Accessibility permission granted
- Verify key name is in `keyMap` dictionary
- Test with simple callback first

**Memory leaks**:
- Check `shutdown()` implementations
- Look for retain cycles (use `[weak self]` in closures)
- Watch for missing observer removals

### Breakpoint Strategies

- **Engine startup**: Breakpoint in `JSEngine.createContext()` line 60
- **Module loading**: Breakpoint in `ModuleRoot.getOrCreate()` line 33
- **Config loading**: Breakpoint in `ManagerManager.boot()` line 34
- **Module shutdown**: Breakpoint in specific module's `shutdown()` method

## Performance Considerations

### Lazy Loading

Modules only load when first accessed - keeps startup fast.

**Example**: If user never uses `hs.ax`, the entire Accessibility module never loads.

### JavaScript Execution

- JavaScriptCore is fast, but avoid long-running synchronous code
- Use timers for periodic tasks instead of loops
- Offload heavy computation to Swift when possible

### Observer Management

Modules that create observers (app watchers, AX watchers, etc.):
- Store observer references in instance variables
- Remove observers in `shutdown()`
- Use weak references where appropriate to avoid retain cycles

## Future Development Ideas

Based on code inspection, potential areas for expansion:

- **hs.screen** - Screen management (already has some support in NSScreen extensions)
- **hs.geometry** - Advanced geometric operations
- **hs.network** - Network monitoring and requests
- **hs.fs** - Filesystem operations
- **hs.audio** - Audio device management
- **hs.battery** - Battery status monitoring
- **hs.caffeinate** - Prevent sleep
- **hs.eventtap** - Low-level event monitoring
- **hs.pathwatcher** - Filesystem change notifications
- **hs.menubar** - Custom menu bar items
- **hs.canvas** - Drawing API
- **hs.json** - JSON parsing/encoding
- **hs.http** - HTTP client

Most of these exist in original Hammerspoon and could be ported.

## Key File Reference

### Critical Files for Understanding

| File | Purpose | LOC Range |
|------|---------|-----------|
| `Hammerspoon 2/Engine/JSEngine.swift` | JavaScript engine core | 60-91 (createContext) |
| `Hammerspoon 2/Engine/ModuleRoot.swift` | Module loading | 33-46 (getOrCreate) |
| `Hammerspoon 2/Lifecycle/Hammerspoon_2App.swift` | App entry point | 15-25 (launch sequence) |
| `Hammerspoon 2/Managers/ManagerManager.swift` | Boot/shutdown orchestration | 34-53 (boot method) |
| `Hammerspoon 2/Engine/engine.js` | EventEmitter implementation | 14-49 (EventEmitter class) |
| `Hammerspoon 2/Modules/hs.application/ApplicationModule.swift` | Application API | Full file (complex module example) |
| `Hammerspoon 2/Modules/hs.timer/TimerModule.swift` | Timer API | 84-99 (timer constructors) |

### Protocol Definitions

- `Hammerspoon 2/Protocols/JSEngineProtocol.swift` - Engine abstraction
- `Hammerspoon 2/Protocols/FileSystemProtocol.swift` - Filesystem abstraction
- `Hammerspoon 2/Protocols/SettingsManagerProtocol.swift` - Settings abstraction

All modules implicitly implement:
- `HSModuleAPI` - Base module protocol (name, init, shutdown)
- `JSExport` - JavaScriptCore bridging (via specific module API protocol)

### Type Definitions

- `Hammerspoon 2/Engine/Types/HSRect.swift` - Rectangle type
- `Hammerspoon 2/Engine/Types/HSPoint.swift` - Point type
- `Hammerspoon 2/Engine/Types/HSSize.swift` - Size type
- `Hammerspoon 2/Modules/hs.alert/HSAlert.swift` - Alert configuration object

### Logging & Utilities

- `Hammerspoon 2/Utilities/Logging.swift` - HammerspoonLog observable class
- `Hammerspoon 2/Utilities/AKLog.swift` - AKTrace/AKInfo/AKError functions
- `Hammerspoon 2/Utilities/Exceptions.swift` - HammerspoonError enum

## External Dependencies

From `Hammerspoon 2.xcodeproj` analysis:

1. **Sparkle** - Auto-update framework
   - Used in `Hammerspoon_2App.swift` for app updates
   - `SPUStandardUpdaterController` manages update checking

2. **AXSwift** - Swift wrapper for Accessibility API
   - Used by `hs.ax` module
   - Provides cleaner Swift API over raw C Accessibility API

## Conclusion

Hammerspoon 2 is a well-architected, modern rewrite bringing JavaScript to macOS automation. Its protocol-oriented design, lazy loading, and hybrid Swift/JS architecture provide a solid foundation for extensive automation capabilities.

**Key Strengths**:
- Clean separation of concerns (protocols, modules, engine)
- Testable via dependency injection
- Fast startup via lazy loading
- Extensible module system
- Modern Swift codebase
- Strong typing with seamless JS bridge

**Development Status**: Work in progress - core infrastructure solid, module library growing.

For questions or contributions, refer to the original [Hammerspoon documentation](https://www.hammerspoon.org/docs/) for API inspiration, as Hammerspoon 2 aims to maintain API compatibility where sensible while modernizing the implementation.
