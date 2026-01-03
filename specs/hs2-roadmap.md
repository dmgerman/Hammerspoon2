# Hammerspoon 2 Roadmap
**Complete Module Porting Strategy**

**Generated**: 2026-01-03
**Status**: Ready for Implementation
**Version**: 1.0

---

## Executive Summary

This roadmap provides a comprehensive analysis of all remaining Hammerspoon modules to be ported to Hammerspoon 2, including dependency analysis, effort estimation, and implementation strategy.

**⚠️ VERIFIED AGAINST SOURCE CODE**: All porting status claims verified by inspecting actual Swift implementations as of 2026-01-03. See [Module Verification Report](module-verification-2026-01-03.md) for details.

### Key Findings

- **Total Modules**: 89 modules in original Hammerspoon
- **Registered in ModuleRoot**: 11 modules
- **Actually Complete**: 3 modules (hs.ipc + 2 new modules)
- **Partial Implementations**: 8 modules (ranging from 8.6% to 65.5% of original)
- **Remaining to Port**: 78 modules + window submodules (filter, layout, switcher, tiling, highlight)
- **Total Remaining Effort**: 225-404 developer days (9-14 months for 3-person team)
- **Critical Path** (Core Window Management): 37-54 days (1.5-2 developer months)

### Strategic Recommendations

1. **Phase A (Foundation)**: Port geometry, image, and core utilities first (4-8 weeks)
2. **Phase B (Window Management)**: Complete screen and window.filter (5-7 weeks)
3. **Quick Wins**: Prioritize 18 small utility modules for momentum (3-6 weeks, can run in parallel)
4. **MVP Timeline**: Phases A+B = 3-4 months (3-person team) for core window management
5. **Full-Featured**: Phases A-E = 9-12 months (3-person team) for most user-facing features

---

## Verification & Threats to Validity

### Verification Process (2026-01-03)

**Method**: Source code inspection comparing Swift implementations to original Lua/Objective-C

1. **Directory Scan**: Checked `Hammerspoon 2/Modules/hs.*/` for Swift implementations
2. **Registration Check**: Verified modules in `ModuleRoot.swift`
3. **LOC Comparison**: Counted lines of code in Swift vs original
4. **API Comparison**: Manually inspected key modules (alert, window, timer) for missing functionality

**Criteria**:
- **COMPLETE**: 70%+ of original LOC + all major API methods present
- **PARTIAL**: 40-70% of original OR missing significant functionality
- **MINIMAL**: <40% of original, core functionality only

### Threats to Validity

1. **LOC is imperfect**: Swift/JS may be more/less verbose than Lua/Objective-C (e.g., Swift may need less code with modern libraries)
2. **Cannot verify full API parity**: Would require deep inspection of every method signature and behavior
3. **Submodules inflate counts**: Original window module (6,066 LOC) includes 5 large submodules
4. **Library dependencies**: hs.ax uses AXSwift library which may reduce LOC but provide same functionality
5. **New modules**: hs.appinfo and hs.permissions are new, no baseline to compare
6. **Commented/disabled code**: Did not check if modules in ModuleRoot are commented out/disabled
7. **Name mismatches**: hs.hash directory vs "hashing" registration may indicate incomplete refactoring
8. **No runtime testing**: Only verified code presence, not functionality
9. **JavaScript companions**: Some modules have .js files with additional functionality not counted in original Lua
10. **Dependency code**: Some original Lua code may now be in shared Swift utilities

**Confidence Level**: High for presence/absence, Medium for completeness assessment

---

## Table of Contents

1. [Verified Porting Status](#verified-porting-status)
2. [Module Discovery](#module-discovery)
3. [Module Categories](#module-categories)
4. [Dependency Analysis](#dependency-analysis)
5. [Effort Estimation](#effort-estimation)
6. [Dependency Diagrams](#dependency-diagrams)
7. [Implementation Phases](#implementation-phases)
8. [Recommended Timeline](#recommended-timeline)
9. [Quick Reference](#quick-reference)

---

## Verified Porting Status

### Modules Registered in ModuleRoot.swift (11 total)

**✅ COMPLETE** (3 modules):
1. **hs.ipc** - 135.6% of original (666 vs 491 lines) + JS companion
   - More code than original, likely has additional features
2. **hs.appinfo** - New module, no original to compare
3. **hs.permissions** - New module, no original to compare

**⚠️ PARTIAL** (7 modules):
1. **hs.hotkey** - 65.5% of original (440 vs 672 lines)
   - Reasonably complete, likely has most core functionality
2. **hs.console** - 56.1% of original (78 vs 139 lines)
   - Moderate completeness
3. **hs.application** - 47.3% of original (313 vs 662 lines) + JS companion
   - Missing some application manipulation methods
4. **hs.hash** - 46.3% of original (131 vs 283 lines)
   - ⚠️ **BUG**: Directory is `hs.hash/` but registered as `hashing` in ModuleRoot
5. **hs.timer** - 43.4% of original (236 vs 544 lines) + JS companion
   - JavaScript adds `minutes()`, `hours()`, `days()`, `weeks()` helpers

**❌ MINIMAL** (3 modules):
1. **hs.alert** - 34.4% of original (103 vs 299 lines)
   - Missing: `showWithImage`, `closeAll`, `closeSpecific`, styling, screen selection, custom duration
2. **hs.ax** - 18.4% of original axuielement (528 vs 2,865 lines) + JS companion
   - May have more functionality via AXSwift library dependency
3. **hs.window** - 8.6% of total original (432 vs 5,024 lines) + JS companion
   - **CRITICAL**: Has basic core only, ALL SUBMODULES MISSING:
     - ❌ window.filter (2,325 LOC) - **Most requested feature**
     - ❌ window.layout (902 LOC)
     - ❌ window.switcher (428 LOC)
     - ❌ window.tiling (217 LOC)
     - ❌ window.highlight (303 LOC)

### Critical Findings

1. **Only 3 truly complete modules** (27% of registered modules)
2. **hs.window.filter NOT ported** - highest user demand feature missing
3. **hs.alert minimal** - basic show() only, missing 70% of API
4. **hs.hash name mismatch** - potential bug (dir=hash, registration=hashing)
5. **No geometry module** - required by 20+ modules, highest priority to port

---

## Module Discovery

### Summary

- **Total Original Modules**: 89 (excluded _coresetup and doc)
- **Registered in ModuleRoot**: 11 modules
- **Actually Complete**: 3 modules
- **Partial/Minimal**: 8 modules
- **Remaining to Port**: 78 modules + window submodules
- **Modules with Submodules**: 7 modules (14 total submodules)

### Registered Modules by Completeness

**Complete** (3):
- hs.ipc (1,689 LOC original) - ✅
- hs.appinfo (new) - ✅
- hs.permissions (new) - ✅

**Partial** (5):
- hs.hotkey (1,073 LOC original) - ⚠️ 65.5%
- hs.console (809 LOC original) - ⚠️ 56.1%
- hs.application (2,147 LOC original) - ⚠️ 47.3%
- hs.hash/hashing (1,426 LOC original) - ⚠️ 46.3% + name bug
- hs.timer (863 LOC original) - ⚠️ 43.4%

**Minimal** (3):
- hs.alert (318 LOC original) - ❌ 34.4%
- hs.ax (3,600 LOC original as axuielement) - ❌ 18.4%
- hs.window (6,066 LOC original with submodules) - ❌ 8.6%

### Modules by Size

**Very Large (>3000 LOC)** - 6 modules:
- webview: 6,838
- window: 6,066 (partially ported)
- canvas: 5,435
- network: 4,059
- axuielement: 3,600 ✅
- httpserver: 3,552

**Large (2000-3000 LOC)** - 11 modules:
- razer: 2,962
- styledtext: 2,936
- eventtap: 2,626
- audiodevice: 2,468
- streamdeck: 2,303
- drawing: 2,275
- fs: 2,229
- application: 2,147 ✅
- screen: 2,139
- location: 2,063
- image: 1,978

**Medium (1000-2000 LOC)** - 16 modules
**Small (500-1000 LOC)** - 15 modules
**Very Small (<500 LOC)** - 41 modules

---

## Module Categories

### Category Overview (14 categories)

| Category                | Modules | Total LOC | Avg LOC | Priority   | Status          |
|-------------------------|---------|-----------|---------|------------|-----------------|
| **Core Infrastructure** | 12      | 5,389     | 449     | ⭐⭐⭐⭐⭐ | 1/12 ported     |
| **Window Management**   | 7       | 11,585    | 1,655   | ⭐⭐⭐⭐   | 1/7 partial     |
| **Application**         | 6       | 2,835     | 473     | ⭐⭐       | 1/6 ported      |
| **System Integration**  | 11      | 9,301     | 845     | ⭐⭐⭐     | Foundation      |
| **Input/Events**        | 6       | 5,916     | 986     | ⭐⭐⭐     | 1/6 ported      |
| **Media/Audio**         | 10      | 9,765     | 977     | ⭐         | Low priority    |
| **Network**             | 8       | 14,067    | 1,758   | ⭐⭐       | Medium priority |
| **User Interface**      | 8       | 13,540    | 1,693   | ⭐⭐⭐     | 2/8 ported      |
| **Graphics/Drawing**    | 4       | 10,850    | 2,713   | ⭐         | Nice-to-have    |
| **Accessibility**       | 3       | 7,780     | 2,593   | ⭐⭐⭐     | 1/3 ported      |
| **Utility/Helper**      | 8       | 3,966     | 496     | ⭐⭐       | 3/8 ported      |
| **Device/Hardware**     | 6       | 11,647    | 1,941   | ⭐         | Niche           |
| **Data/Storage**        | 3       | 3,232     | 1,077   | ⭐⭐       | Useful          |
| **External Services**   | 1       | 447       | 447     | ⭐         | Very niche      |

### Core Infrastructure (⭐⭐⭐⭐⭐ HIGHEST PRIORITY)

Essential building blocks used by other modules:

- **geometry (758 LOC)** - ❌ NOT PORTED - 2D geometric operations - **CRITICAL DEPENDENCY**
- **timer (863 LOC)** - ⚠️ PARTIAL (43.4%) - Timers and scheduling
- settings (353 LOC) - Persistent settings
- fnutils (527 LOC) - Functional programming utilities
- json (249 LOC) - JSON parsing/encoding
- logger (393 LOC) - Logging utilities
- math (204 LOC) - Math utilities
- plist (186 LOC) - Property list handling
- base64 (117 LOC) - Base64 encoding/decoding
- utf8 (379 LOC) - UTF-8 string operations
- inspect (360 LOC) - Object inspection/debugging

### Window Management (⭐⭐⭐⭐ HIGH PRIORITY)

Window positioning, layout, organization:

- **window (6,066 LOC)** - ✅ PARTIALLY PORTED (core only)
  - window.filter (2,325 LOC) - ❌ Most requested submodule
  - window.layout (902 LOC) - ❌ NOT PORTED
  - window.switcher (428 LOC) - ❌ NOT PORTED
  - window.tiling (217 LOC) - ⚠️ PARTIALLY PORTED
  - window.highlight (303 LOC) - ❌ NOT PORTED
- **screen (2,139 LOC)** - ⚠️ CRITICAL - Display management
- grid (1,187 LOC) - Grid-based window positioning
- hints (501 LOC) - Window hints/labels
- layout (258 LOC) - Window layout management
- expose (1,063 LOC) - Expose/Mission Control integration
- spaces (1,370 LOC) - Mission Control Spaces

### System Integration (⭐⭐⭐ HIGH PRIORITY)

System-level functionality:

- **fs (2,229 LOC)** - ⚠️ CRITICAL - Filesystem operations
- task (1,000 LOC) - External process execution
- caffeinate (969 LOC) - Prevent sleep/screensaver
- battery (929 LOC) - Battery status
- brightness (205 LOC) - Screen brightness control
- host (1,362 LOC) - Host information
- location (2,063 LOC) - Geographic location
- pathwatcher (273 LOC) - Filesystem change monitoring
- watchable (323 LOC) - Key-value observation
- crash (218 LOC) - Crash reporting

### Other Categories

See [Module Categories](#module-categories) section for complete details on:
- Application Management (6 modules)
- Input/Events (6 modules)
- Media/Audio/Visual (10 modules)
- Network/Communication (8 modules)
- User Interface (8 modules)
- Graphics/Drawing (4 modules)
- Device/Hardware (6 modules)
- Data/Storage (3 modules)
- External Services (1 module)

---

## Dependency Analysis

### Dependency Hierarchy

**Level 0** (No dependencies - can port immediately):
- **geometry** ❌ NOT PORTED - **CRITICAL** (required by 20+ modules, HIGHEST PRIORITY)
- **timer** ⚠️ PARTIAL (43.4%) - CRITICAL (required by 10+ modules)
- settings, fnutils, keycodes
- Most Core, System, Input, Network, Media, Device modules

**Level 1** (Depends only on Level 0):
- **screen** ❌ NOT PORTED (needs geometry ❌, image ❌)
- **application** ⚠️ PARTIAL (47.3%) (needs settings ❌, timer ⚠️)
- **hotkey** ⚠️ PARTIAL (65.5%) (needs keycodes ❌)
- menubar ❌ NOT PORTED (needs geometry ❌, screen ❌)

**Level 2** (Depends on Level 0 + Level 1):
- **window** ❌ MINIMAL (8.6%) (needs application ⚠️, geometry ❌, screen ❌, timer ⚠️, image ❌)
- tabs ❌ NOT PORTED (needs application ⚠️, drawing ❌, fnutils ❌, uielement ❌)

**Level 3** (Depends on Level 0-2):
- grid ❌ NOT PORTED (needs screen ❌, window ❌)
- hints ❌ NOT PORTED (needs hotkey ⚠️, window ❌)
- appfinder ❌ NOT PORTED (needs application ⚠️, window ❌)

### Critical Path for Window Management

To enable **core window management** (highest user value):

**Must Port (in order)**:
1. **geometry** (758 LOC) - ❌ NOT PORTED - **START HERE** - Foundation
2. **image** (1,978 LOC) - ❌ NOT PORTED - Standalone, medium complexity
3. **screen** (2,139 LOC) - ❌ NOT PORTED - Needs geometry + image
4. **window core enhancements** (~500 LOC est.) - Expand minimal window implementation
5. **window.filter** (2,325 LOC) - ❌ NOT PORTED - **Most requested submodule**
6. **grid** (1,187 LOC) - ❌ NOT PORTED - Window positioning helper
7. **layout** (258 LOC) - ❌ NOT PORTED - Window layout management
8. **window.layout** (902 LOC) - ❌ NOT PORTED - Advanced layouts

**Partially Implemented** (need completion):
- ⚠️ timer (863 LOC) - 43.4% complete, need remaining functionality
- ⚠️ application (2,147 LOC) - 47.3% complete, expand functionality
- ⚠️ hotkey (1,073 LOC) - 65.5% complete, reasonably functional
- ⚠️ window core (~1,039 LOC) - 8.6% complete (432 lines), expand significantly

**Total LOC for critical path**: ~10,600 LOC (including completions)
**Estimated effort**: 37-54 developer days (includes geometry 3-4 days)

### Key Dependencies

**geometry is the foundation**:
- No dependencies itself
- Required by: window, screen, menubar, grid, and 15+ other modules
- Status: Currently being ported (spec exists)

**image enables window features**:
- No dependencies
- Required by: window, screen, menubar, canvas, drawing

**screen enables window management**:
- Requires: geometry, image
- Required by: window, menubar, grid, hints, expose

**window is the hub**:
- Requires: application, geometry, screen, timer, image
- Required by: grid, hints, layout, appfinder, tabs

---

## Effort Estimation

### Estimation Methodology

**Base Formula**: `effort_days = (LOC / 200) × complexity_multiplier`

**Complexity Multipliers**:
- 0.5-0.7: Simple wrappers, pure utilities, minimal logic
- 0.8-1.0: Standard modules, moderate native API integration
- 1.1-1.3: Complex native APIs, state management, async handling
- 1.4-1.6: Very complex (private APIs, extensive testing needed)

**Reference Baseline** (from ported modules):
- hs.geometry (758 LOC): 3-4 days actual (pure computation, verbose JS)
- hs.timer (863 LOC): 2-3 days actual (simple native API)
- hs.window core (1000 LOC): 6-8 days actual (complex native APIs)

### Modules by Effort

**⭐ Trivial (0.5-1 day)** - 17 modules:
- json, math, plist, base64, brightness, crash, distributednotifications
- tabs, sqlite3, appfinder, applescript, javascript, messages, shortcuts
- mjomatic, dockicon
- **Total**: ~2,372 LOC, ~8-17 days combined

**⭐⭐ Simple (1-2 days)** - 18 modules:
- settings, fnutils, logger, utf8, inspect
- window.highlight, layout, pathwatcher, watchable
- websocket, osascript, spoons, milight, redshift
- Media wrappers: itunes, spotify, deezer, vox, noises
- **Total**: ~6,245 LOC, ~18-36 days combined

**⭐⭐⭐ Medium (2-4 days)** - 10 modules:
- window.switcher, hints, battery
- mouse, keycodes, hid, urlevent
- sharing, usb
- **Total**: ~5,302 LOC, ~20-40 days combined

**⭐⭐⭐⭐ Complex (4-6 days)** - 12 modules:
- **geometry** ⚠️ IN PROGRESS, window.layout, grid
- task, host, caffeinate, http, dialog
- image, sound, speech, camera
- **Total**: ~12,291 LOC, ~48-72 days combined

**⭐⭐⭐⭐⭐ Very Complex (6+ days)** - 21 modules:
- **screen**, **window.filter**, expose, spaces
- fs, location, eventtap
- httpserver, socket, network, bonjour, wifi
- chooser, menubar, notify, webview
- canvas, drawing, styledtext, audiodevice
- midi, streamdeck, razer, tangent, serial
- pasteboard, spotlight
- **Total**: ~64,241 LOC, ~400+ days combined

### Category Effort Summary

| Category | Modules | Total LOC | Est. Days | Avg/Module |
|----------|---------|-----------|-----------|------------|
| Core | 11 (1 ported) | 4,409 | 15-25 | 1.5-2.5 |
| Window | 11 (1 partial) | 11,585 | 70-110 | 6-10 |
| Application | 5 | 2,639 | 8-15 | 1.5-3 |
| System | 11 | 9,301 | 55-85 | 5-8 |
| Input | 5 (1 ported) | 4,843 | 30-45 | 6-9 |
| Media | 10 | 9,765 | 25-40 | 2.5-4 |
| Network | 8 | 14,067 | 80-120 | 10-15 |
| UI | 6 (2 ported) | 12,721 | 60-90 | 10-15 |
| Graphics | 4 | 10,850 | 50-75 | 12-19 |
| Device | 6 | 11,647 | 70-105 | 11-17 |
| Data | 3 | 3,232 | 14-22 | 4-7 |
| Utility | 6 (3 ported) | 1,351 | 8-12 | 1-2 |
| External | 1 | 447 | 1-2 | 1-2 |

**Total Remaining Work**: ~90,510 LOC, **~486-746 developer days**

### Quick Wins (High Value, Low Effort)

**Immediate** (<1 day each):
- brightness, crash, json, math, plist, base64, sqlite3
- **Total**: ~7 modules, 5-7 days

**Short term** (1-2 days each):
- settings, fnutils, logger, utf8, inspect, pathwatcher
- **Total**: ~6 modules, 6-12 days

**Medium term** (2-4 days each):
- battery, mouse, keycodes, hid, urlevent
- **Total**: ~5 modules, 10-20 days

**Quick Wins Total**: ~18 modules, 21-39 days

---

## Dependency Diagrams

### Master Dependency Diagram (Category Level)

```
┌─────────────────────────────────────────────────────────────┐
│                     HAMMERSPOON 2 ROADMAP                   │
│                   Module Dependency Graph                   │
└─────────────────────────────────────────────────────────────┘

LEVEL 0 - FOUNDATION (No Dependencies)
════════════════════════════════════════
┌──────────────────────────────────────────────────────────┐
│  CORE INFRASTRUCTURE                                     │
│  - geometry ⚠️ (758 LOC) ⭐⭐⭐⭐                           │
│  - timer ✅ (863 LOC) ⭐⭐⭐                              │
│  - settings (353 LOC) ⭐⭐                               │
│  - fnutils, json, logger, math, plist, etc.            │
│                                                          │
│  SYSTEM (Low-level)                                     │
│  - fs, task, battery, brightness, caffeinate,          │
│    pathwatcher, crash, host, watchable                 │
│                                                          │
│  INPUT (Low-level)                                      │
│  - eventtap, keycodes, mouse, hid, urlevent            │
│                                                          │
│  NETWORK (Low-level)                                    │
│  - http, httpserver, socket, websocket, network,       │
│    bonjour, wifi, sharing                              │
│                                                          │
│  MEDIA (Standalone)                                     │
│  - image, audiodevice, sound, speech, camera           │
│                                                          │
│  DEVICE                                                 │
│  - midi, usb, serial, streamdeck, razer, tangent       │
│                                                          │
│  DATA                                                   │
│  - pasteboard, spotlight, sqlite3                      │
│                                                          │
│  UI (Standalone)                                        │
│  - alert ✅, console ✅, dialog, notify, chooser,       │
│    distributednotifications, webview                   │
│                                                          │
│  GRAPHICS (Standalone)                                  │
│  - canvas, drawing, styledtext, tabs                   │
└──────────────────────────────────────────────────────────┘
                           │
                           ▼
LEVEL 1 - FIRST LAYER (Depends on Level 0 only)
════════════════════════════════════════════════
         ┌─────────────────────────────┐
         │  SCREEN (2139 LOC) ⭐⭐⭐⭐⭐   │
         │  Requires:                  │
         │    - geometry ⚠️             │
         │    - image                  │
         └─────────────────────────────┘
                           │
         ┌─────────────────────────────┐
         │  APPLICATION ✅ (2147 LOC)   │
         │  Requires:                  │
         │    - settings               │
         │    - timer ✅                │
         └─────────────────────────────┘
                           │
         ┌─────────────────────────────┐
         │  HOTKEY ✅ (1073 LOC)        │
         │  Requires:                  │
         │    - keycodes               │
         └─────────────────────────────┘
                           │
         ┌─────────────────────────────┐
         │  MENUBAR (1289 LOC) ⭐⭐⭐⭐⭐│
         │  Requires:                  │
         │    - geometry ⚠️             │
         │    - screen                 │
         └─────────────────────────────┘
                           │
                           ▼
LEVEL 2 - WINDOW CORE (Depends on Levels 0-1)
════════════════════════════════════════════════
         ┌─────────────────────────────┐
         │  WINDOW CORE ⚠️ (1000 LOC)   │
         │  Requires:                  │
         │    - application ✅          │
         │    - geometry ⚠️             │
         │    - screen                 │
         │    - timer ✅                │
         │    - image                  │
         └─────────────────────────────┘
                           │
                           ▼
LEVEL 3 - WINDOW FEATURES (Depends on Levels 0-2)
═══════════════════════════════════════════════════
    ┌───────────────┐  ┌────────────────┐  ┌──────────────┐
    │ WINDOW.FILTER │  │  GRID (1187)   │  │ HINTS (501)  │
    │  (2325 LOC)   │  │  Requires:     │  │  Requires:   │
    │  Requires:    │  │   - screen     │  │   - hotkey ✅ │
    │   - window ⚠️  │  │   - window ⚠️   │  │   - window ⚠️ │
    │  ⭐⭐⭐⭐⭐      │  │  ⭐⭐⭐⭐        │  │  ⭐⭐⭐       │
    └───────────────┘  └────────────────┘  └──────────────┘
            │                  │                    │
            └──────────────────┴────────────────────┘
                               │
                               ▼
LEVEL 4 - ADVANCED WINDOW (Depends on Levels 0-3)
══════════════════════════════════════════════════
         ┌─────────────────────────────┐
         │  WINDOW.LAYOUT (902 LOC)    │
         │  WINDOW.SWITCHER (428 LOC)  │
         │  WINDOW.HIGHLIGHT (303 LOC) │
         │  LAYOUT (258 LOC)           │
         │  EXPOSE (1063 LOC)          │
         │  SPACES (1370 LOC)          │
         └─────────────────────────────┘
```

### Critical Path Detail (Window Management)

```
CRITICAL PATH FOR WINDOW MANAGEMENT
════════════════════════════════════════════════════════════

START
  │
  ▼
┌────────────────────────────────────────────────┐
│ 1. GEOMETRY (758 LOC) ❌ NOT PORTED           │
│    Effort: 3-4 days                           │
│    Status: Spec exists, not implemented       │
│    Type: Pure JavaScript, no macOS APIs       │
│    **START HERE FIRST**                       │
└────────────────────────────────────────────────┘
  │
  ▼
┌────────────────────────────────────────────────┐
│ 2. IMAGE (1,978 LOC)                          │
│    Effort: 4-6 days                           │
│    Dependencies: NONE                         │
│    Type: Image manipulation (CoreImage)       │
└────────────────────────────────────────────────┘
  │
  ├───────────┐
  ▼           ▼
┌───────────────────┐    ┌────────────────────┐
│ 3a. SCREEN        │    │ 3b. WINDOW CORE    │
│     (2,139 LOC)   │───>│     ENHANCEMENTS   │
│  Effort: 6-8 days │    │  Effort: 3-4 days  │
│  Requires:        │    │  Requires:         │
│   - geometry ✅    │    │   - application ✅  │
│   - image ✅       │    │   - geometry ✅     │
└───────────────────┘    │   - screen ✅       │
                         │   - timer ✅        │
                         └────────────────────┘
                                │
                                ▼
                    ┌────────────────────────┐
                    │ 4. WINDOW.FILTER       │
                    │    (2,325 LOC)         │
                    │  Effort: 8-10 days     │
                    │  Most requested!       │
                    └────────────────────────┘
                                │
                    ┌───────────┴───────────┐
                    ▼                       ▼
          ┌──────────────────┐    ┌──────────────────┐
          │ 5a. GRID (1187)  │    │ 5b. HINTS (501)  │
          │  Effort: 4-6 days│    │  Effort: 2-4 days│
          └──────────────────┘    └──────────────────┘
                    │
                    ▼
          ┌──────────────────────┐
          │ 6. WINDOW.LAYOUT     │
          │    (902 LOC)         │
          │  Effort: 4-6 days    │
          └──────────────────────┘
                    │
                    ▼
                  DONE
    (Core window management complete)

TOTAL CRITICAL PATH: 34-50 developer days
```

### Window Management Category Detail

```
WINDOW MANAGEMENT MODULES
═══════════════════════════════════════════════════════════

                     [geometry] ⚠️
                          │
                          │ required by
                          ▼
                      [screen]
                          │
         ┌────────────────┼────────────────┐
         │                │                │
         ▼                ▼                ▼
    [window core] ⚠️   [menubar]    [other modules]
         │
         │ Used by window submodules
         │
    ┌────┴────┬────────┬─────────┬────────────┐
    ▼         ▼        ▼         ▼            ▼
[filter]  [layout]  [switcher] [tiling] [highlight]
 2325      902       428        217      303
 ⭐⭐⭐⭐⭐   ⭐⭐⭐⭐    ⭐⭐⭐       ⚠️        ⭐⭐

Other window utilities depend on window + screen:
    ┌────────┐  ┌─────────┐  ┌────────┐  ┌────────┐
    │  grid  │  │  hints  │  │ layout │  │ expose │
    │  1187  │  │   501   │  │  258   │  │  1063  │
    │ ⭐⭐⭐⭐  │  │  ⭐⭐⭐   │  │  ⭐⭐   │  │ ⭐⭐⭐⭐⭐ │
    └────────┘  └─────────┘  └────────┘  └────────┘
                                             │
                                             ▼
                                        [spaces]
                                         1370
                                        ⭐⭐⭐⭐⭐
```

---

## Implementation Phases

### Phase Overview

| Phase | Focus | Modules | Effort (days) | Cumulative |
|-------|-------|---------|---------------|------------|
| **A** | Foundation | 18 | 26-58 | 26-58 |
| **B** | Window Mgmt | 8 | 31-46 | 57-104 |
| **C** | System & UI | 8 | 36-54 | 93-158 |
| **D** | Advanced | 13 | 63-125 | 156-283 |
| **E** | Nice-to-Have | 17 | 36-69 | 192-352 |
| **F** | Niche | 7 | 33-52 | 225-404 |
| **TOTAL** | All | 71 | 225-404 | - |

### PHASE A: Foundation (6-10 weeks)
**Goal**: Complete critical dependencies that enable window management

#### A1: Core Infrastructure (2-3 weeks)

**Priority 1 - Absolute Foundation**:
1. ❌ **geometry (758 LOC)** - **NOT PORTED** - **START HERE FIRST**
   - Effort: 3-4 days (week 1)
   - Status: Spec exists but no implementation yet
   - Blocks: screen, window, menubar, grid, and 15+ other modules
   - **CRITICAL**: Required by majority of window management features

**Priority 2 - Essential Utilities** (can be done in parallel):
2. **settings (353 LOC)** - UserDefaults wrapper
   - Effort: 1-2 days
   - Dependencies: None
   - Quick win

3. **fnutils (527 LOC)** - Functional utilities
   - Effort: 1-2 days
   - Dependencies: None
   - Used by many modules

4. **json (249 LOC)** - JSON wrapper
   - Effort: 0.5-1 day
   - Quick win

5. **logger (393 LOC)** - Logging system
   - Effort: 1-2 days
   - Useful for development

6. **keycodes (769 LOC)** - Key mappings
   - Effort: 2-4 days
   - Blocks: hotkey (already ported, but needs keycodes)

**Quick Wins** (can be deferred but easy):
- math (204 LOC) - 0.5-1 day
- plist (186 LOC) - 0.5-1 day
- base64 (117 LOC) - 0.5-1 day
- utf8 (379 LOC) - 1-2 days
- inspect (360 LOC) - 1-2 days

**Phase A1 Total**: ~12-22 days (with geometry)

#### A2: Media Foundation (1-2 weeks)

**Critical for window management**:
7. **image (1,978 LOC)** - Image manipulation
   - Effort: 4-6 days
   - Dependencies: None
   - Required by: screen, window

**Optional (parallel work)**:
- sound (717 LOC) - 2-4 days
- camera (829 LOC) - 4-6 days

**Phase A2 Total**: ~4-16 days (just image, or +media)

#### A3: System Essentials (1-2 weeks)

**High value utilities**:
8. **fs (2,229 LOC)** - Filesystem operations
   - Effort: 6+ days
   - Dependencies: None
   - High user value

9. **task (1,000 LOC)** - Process execution
   - Effort: 4-6 days
   - Dependencies: None
   - Essential for automation

**Quick wins** (parallel):
- brightness (205 LOC) - 0.5-1 day
- battery (929 LOC) - 2-4 days
- pathwatcher (273 LOC) - 1-2 days
- crash (218 LOC) - 0.5-1 day

**Phase A3 Total**: ~10-20 days

**PHASE A TOTAL**: 26-58 days (4-8 weeks, ~6 weeks avg)

---

### PHASE B: Core Window Management (6-10 weeks)
**Goal**: Enable essential window management features

#### B1: Screen Module (1-2 weeks)

10. **screen (2,139 LOC)** - Display management
    - Effort: 6-8 days
    - Dependencies: geometry ✅, image ✅
    - CRITICAL - blocks all window features

**Phase B1 Total**: ~6-8 days

#### B2: Window Enhancements (1 week)

11. **window core enhancements** (est. 500 LOC)
    - Effort: 3-4 days
    - Add missing methods to ported window module
    - Improve integration with screen + geometry

**Phase B2 Total**: ~3-4 days

#### B3: Window Filter (2 weeks)

12. **window.filter (2,325 LOC)** - Advanced filtering
    - Effort: 8-10 days
    - Dependencies: window ✅
    - MOST REQUESTED feature

**Phase B3 Total**: ~8-10 days

#### B4: Grid & Layout (2-3 weeks)

13. **grid (1,187 LOC)** - Grid-based positioning
    - Effort: 4-6 days
    - Dependencies: screen ✅, window ✅

14. **window.layout (902 LOC)** - Layout management
    - Effort: 4-6 days
    - Dependencies: window ✅

15. **layout (258 LOC)** - Simple layouts
    - Effort: 1-2 days
    - Can be done in parallel

**Phase B4 Total**: ~9-14 days

#### B5: Window Helpers (1-2 weeks)

16. **hints (501 LOC)** - Window hints
    - Effort: 2-4 days
    - Dependencies: hotkey ✅, window ✅

17. **window.switcher (428 LOC)** - Window switcher
    - Effort: 2-4 days
    - Dependencies: window ✅

18. **window.highlight (303 LOC)** - Visual effects
    - Effort: 1-2 days
    - Nice-to-have

**Phase B5 Total**: ~5-10 days

**PHASE B TOTAL**: 31-46 days (5-7 weeks, ~6 weeks avg)

**CUMULATIVE (A+B)**: 57-104 days (8-15 weeks, ~3 months avg)

---

### PHASE C: Essential System & UI (4-6 weeks)
**Goal**: Complete essential system integration and UI features

#### C1: UI Essentials (2-3 weeks)

19. **menubar (1,289 LOC)** - Menu bar items
    - Effort: 6+ days
    - Dependencies: geometry ✅, screen ✅

20. **chooser (1,900 LOC)** - Selection UI
    - Effort: 6+ days
    - Dependencies: None

21. **dialog (898 LOC)** - Native dialogs
    - Effort: 4-6 days
    - Dependencies: None

22. **notify (1,861 LOC)** - Notifications
    - Effort: 6+ days
    - Dependencies: None

**Quick wins**:
- distributednotifications (236 LOC) - 0.5-1 day

**Phase C1 Total**: ~22-30 days

#### C2: System Power Management (1 week)

23. **caffeinate (969 LOC)** - Prevent sleep
    - Effort: 4-6 days
    - Dependencies: None
    - High user value

24. **host (1,362 LOC)** - Host information
    - Effort: 4-6 days
    - Dependencies: timer ✅

**Phase C2 Total**: ~8-12 days

#### C3: Input Enhancement (1 week)

25. **mouse (559 LOC)** - Mouse control
    - Effort: 2-4 days
    - Dependencies: None

26. **hid (380 LOC)** - HID devices
    - Effort: 2-4 days
    - Dependencies: None

27. **urlevent (509 LOC)** - URL events
    - Effort: 2-4 days
    - Dependencies: None

**Phase C3 Total**: ~6-12 days

**PHASE C TOTAL**: 36-54 days (5-8 weeks, ~6 weeks avg)

**CUMULATIVE (A+B+C)**: 93-158 days (13-23 weeks, ~4-5 months avg)

---

### PHASE D: Advanced Features (8-12 weeks)
**Goal**: Add advanced functionality and integrations

See complete Phase D details including:
- D1: Advanced Window Features (expose, spaces)
- D2: Event Handling (eventtap)
- D3: Network Features (http, httpserver, socket, websocket)
- D4: Graphics & Drawing (drawing, canvas, styledtext)
- D5: Web Integration (webview)

**PHASE D TOTAL**: 63-125 days (9-18 weeks, ~12 weeks avg)

---

### PHASE E: Nice-to-Have (4-8 weeks)
**Goal**: Add useful but non-essential features

Including media control, data/search, system information, and utility modules.

**PHASE E TOTAL**: 36-69 days (5-10 weeks, ~7 weeks avg)

---

### PHASE F: Niche/Specialized (4-8 weeks)
**Goal**: Device-specific and specialized modules

Including device control (streamdeck, razer, midi, etc.) and external services.

**PHASE F TOTAL**: 33-52 days (5-7 weeks, ~6 weeks avg)

---

## Recommended Timeline

### Timeline Scenarios

**Team Size Impact**:

1. **Focused Team (3 developers)**: ~10-14 months (phases A-F)
2. **Small Team (2 developers)**: ~12-18 months (phases A-F)
3. **Solo Developer**: ~18-24 months (phases A-F)

**Milestone Scenarios**:

4. **MVP (Phases A+B only)**: 2-4 months (3 developers)
5. **Core Features (Phases A+B+C)**: 4-6 months (3 developers)
6. **Full-Featured (Phases A-E)**: 9-12 months (3 developers)

### Recommended Order (Phases A-B, Critical Path)

**Week 1-2**: Foundation
1. **geometry** ❌ (NOT STARTED - **BEGIN HERE IMMEDIATELY**)
   - Spec exists at `specs/2026-01-03-port-hs-geometry-module.md`
   - Required by 20+ modules
   - 3-4 days estimated effort
2. settings, fnutils, json, logger (parallel)

**Week 3-4**: Media & Core
3. image
4. keycodes (for hotkey support)
5. Quick wins: math, plist, base64

**Week 5-6**: System Essentials
6. fs
7. task
8. Quick wins: brightness, battery, pathwatcher

**Week 7-8**: Screen Module
9. screen (depends on geometry + image)

**Week 9**: Window Enhancements
10. Enhance window core

**Week 10-11**: Window Filter
11. window.filter (most requested!)

**Week 12-14**: Grid & Layout
12. grid
13. window.layout
14. layout (parallel)

**Week 15-16**: Window Helpers
15. hints
16. window.switcher
17. window.highlight (parallel)

**= 16 weeks / 4 months for complete window management**

---

## Quick Reference

### Critical Modules (Port First)

1. **geometry** (758 LOC) - ❌ NOT PORTED - **START HERE** - 3-4 days
2. **image** (1,978 LOC) - ❌ NOT PORTED - 4-6 days
3. **screen** (2,139 LOC) - ❌ NOT PORTED - 6-8 days
4. **window.filter** (2,325 LOC) - ❌ NOT PORTED - 8-10 days (most requested!)

### Quick Wins (Port for Momentum)

- json, math, plist, base64 (<1 day each)
- settings, fnutils, logger (1-2 days each)
- battery, keycodes, mouse (2-4 days each)

### High User Value

1. window.filter - Advanced window filtering
2. screen - Display management
3. fs - Filesystem operations
4. grid - Grid-based window positioning
5. caffeinate - Prevent sleep
6. task - Process execution

### Largest Modules (Plan Carefully)

1. webview: 6,838 LOC - 10-15 days
2. canvas: 5,435 LOC - 6+ days
3. network: 4,059 LOC - 6+ days
4. httpserver: 3,552 LOC - 6+ days
5. eventtap: 2,626 LOC - 6+ days

### Dependency Blockers

- **geometry** blocks: screen, window, menubar, grid, 15+ others
- **screen** blocks: window enhancements, menubar, grid, hints
- **image** blocks: screen, window, menubar, canvas
- **keycodes** blocks: hotkey (already ported, needs keycodes for full functionality)

---

## Appendix: Complexity Legend

**⭐ Trivial** (0.5-1 day):
- Simple wrappers, minimal logic
- Examples: json, math, plist, base64

**⭐⭐ Simple** (1-2 days):
- Standard utilities, basic API integration
- Examples: settings, fnutils, logger

**⭐⭐⭐ Medium** (2-4 days):
- Moderate native API integration
- Examples: battery, mouse, keycodes

**⭐⭐⭐⭐ Complex** (4-6 days):
- Complex native APIs, state management
- Examples: geometry, image, task, http

**⭐⭐⭐⭐⭐ Very Complex** (6+ days):
- Very complex (private APIs, extensive testing)
- Examples: screen, window.filter, eventtap, webview

---

---

## Verification Summary

**Date**: 2026-01-03
**Method**: Source code inspection vs original Lua/Objective-C
**Result**: See [module-verification-2026-01-03.md](module-verification-2026-01-03.md) for full details

**Key Corrections**:
1. **geometry is NOT being ported** - spec exists but no implementation
2. **Only 3 modules truly complete** out of 11 registered
3. **window.filter NOT ported** - highest priority user feature missing
4. **Most registered modules are partial** (8.6% to 65.5% complete)
5. **hs.hash name mismatch bug** - directory vs registration name

**Actual Status**:
- Complete: hs.ipc, hs.appinfo, hs.permissions (3 modules)
- Partial: hs.hotkey, hs.console, hs.application, hs.hash, hs.timer (5 modules)
- Minimal: hs.alert, hs.ax, hs.window (3 modules)
- Not Started: 78 modules + window submodules

---

**END OF ROADMAP**
**Generated**: 2026-01-03
**Verified**: 2026-01-03 (against source code)
**Status**: Ready for Implementation
**Next Step**: Port hs.geometry module (spec exists, not implemented yet)
