# Feature: Port hs.geometry Module

## Feature Description

Port the `hs.geometry` module from original Hammerspoon to Hammerspoon 2, maintaining full API compatibility while adapting to the JavaScript-based architecture. The hs.geometry module is a pure computational utility library for representing and manipulating 2D geometric objects (points, sizes, rectangles, and unit rectangles) used throughout Hammerspoon for window management, screen positioning, and spatial operations.

### Ground Truth Reference
- **Source**: `/Users/dmg/git.w/hs2/Hammerspoon2/hs_repo_old/extensions/geometry/geometry.lua`
- **Implementation**: Pure Lua (759 lines)
- **Dependencies**: None (standalone utility library)
- **Status in New System**: Does not exist - must be created from scratch

### Core Functionality (Binding Behavior from Original)

The original hs.geometry module provides:

1. **Four Geometric Types**:
   - **Point**: 2D coordinates (x, y)
   - **Size**: Dimensions (w, h)
   - **Rect**: Rectangle with position and dimensions (x, y, w, h)
   - **UnitRect**: Normalized rectangle with values 0-1 for relative positioning

2. **Type Detection**: Internal fields `_x`, `_y`, `_w`, `_h` with detection logic:
   - Point: has `_x` and `_y` only
   - Size: has `_w` and `_h` only
   - Rect: has all four fields AND any field > 1.0000000000001 or < -0.0000000000001
   - UnitRect: has all four fields AND all within bounds [-ε, 1+ε] where ε = 1e-12

3. **Flexible Constructor**: `hs.geometry(...)` accepts multiple formats:
   - Four parameters: `(x, y, w, h)` or `(x, y)` or `(nil, nil, w, h)`
   - Arrays: `{10, 20}`, `{10, 20, 100, 200}`
   - Named tables: `{x=10, y=20}`, `{w=100, h=200}`, `{x1=10, y1=20, x2=110, y2=220}`
   - Strings: `"10 20"`, `"10,20"`, `"100x200"`, `"10 20/100x200"`, `"[50,50>100,100]"`
   - Combinations: `(point, size)`, `("10 20", "100x200")`

4. **Properties** (all support getter/setter):
   - Position: `x`, `y`, `x1`, `y1`, `x2`, `y2`, `xy`, `topleft`, `x2y2`, `bottomright`, `center`
   - Dimensions: `w`, `h`, `wh`, `size`
   - Computed: `area`, `aspect`, `length`, `angle`, `table`, `string`

5. **Methods**:
   - Comparison: `equals()`, `type()`
   - Transformation: `copy()`, `floor()`, `move()`, `scale()`, `normalize()`, `rotateCCW()`
   - Containment: `fit()`, `inside()`
   - Set operations: `union()`, `intersect()`
   - Vector operations: `vector()`, `angle()`, `angleTo()`, `distance()`
   - Conversion: `toUnitRect()`, `fromUnitRect()`

6. **Behavioral Rules**:
   - Width and height are clamped to >= 0
   - Negative w/h causes automatic normalization (flip coordinates)
   - Methods return `this` for chaining (except constructors/getters)
   - Rect scaling occurs around center point
   - Point/vector scaling occurs from origin
   - Non-overlapping rect intersection returns "projection" with w and/or h = 0

### Architecture Mapping to New System

**Existing Foundation** (already in Hammerspoon 2):
- `HSPoint` class: Wraps CGPoint with x, y properties
- `HSSize` class: Wraps CGSize with w, h properties
- `HSRect` class: Wraps CGRect with x, y, w, h properties
- Type bridging via `JSConvertible` protocol

**New Components Needed**:
- `GeometryModule.swift`: Minimal Swift module for registration
- `hs.geometry.js`: Pure JavaScript implementation (port of geometry.lua)
- Module registration in `ModuleRoot.swift`
- Xcode project file updates

**Implementation Strategy**:
Since hs.geometry is pure computation with no macOS API calls, implement as **pure JavaScript** by porting Lua → JavaScript. This maintains maximum compatibility with original API while leveraging existing HSPoint/HSRect/HSSize types for interoperability.

**Critical Design Decisions** (from verification):
1. **Object Type**: Plain JavaScript object literals with getter/setter syntax (not ES6 classes)
2. **Type System**: Geometry objects are separate from HSRect/HSPoint/HSSize but can convert between them
3. **Type Detection**: Both `type()` method and `typeName` property (dynamically computed getter)
4. **Property Scope**: Only relevant properties per type (points don't have w/h, sizes don't have x/y)
5. **Module Pattern**: Replace hs.geometry module object with callable constructor function (Lua's `__call` metatable equivalent)
6. **Error Handling**: Use `throw new Error("message")` (no stack level parameter)
7. **Chaining**: Methods return `this`, property setters follow standard JS (no return)

## Relevant Files

### Existing Files to Modify

**`Hammerspoon 2/Engine/ModuleRoot.swift`**
- Add `hs.geometry` to module registry
- Add lazy getter property for geometry module
- Handles automatic loading of `hs.geometry.js` companion file
- Pattern: Follows same structure as other modules (alert, timer, window, etc.)

**`Hammerspoon 2.xcodeproj/project.pbxproj`**
- Add GeometryModule.swift to build phases
- Add hs.geometry.js to Copy Bundle Resources phase
- Ensure files are part of main target

### Existing Files for Reference (No Modification)

**`Hammerspoon 2/Engine/Types/HSPoint.swift`**
- Reference for existing point type structure
- Provides HSPoint class with x, y properties
- Already bridged to JavaScript via JSExport protocol
- Used by geometry module for point operations

**`Hammerspoon 2/Engine/Types/HSSize.swift`**
- Reference for existing size type structure
- Provides HSSize class with w, h properties
- Already bridged to JavaScript via JSExport protocol
- Used by geometry module for size operations

**`Hammerspoon 2/Engine/Types/HSRect.swift`**
- Reference for existing rect type structure
- Provides HSRect class with x, y, w, h, origin, size properties
- Already bridged to JavaScript via JSExport protocol
- Used by geometry module for rect operations

**`Hammerspoon 2/Protocols/HSModuleAPI.swift`**
- Defines base protocol all modules must conform to
- Requires `name` property and `shutdown()` method
- Pattern to follow for GeometryModule

**`Hammerspoon 2/Modules/hs.timer/TimerModule.swift`**
- Reference implementation for module structure
- Shows how to create minimal Swift module
- Demonstrates JSExport protocol pattern

**`Hammerspoon 2/Modules/hs.timer/hs.timer.js`**
- Reference for JavaScript companion file pattern
- Shows how to extend Swift module with JS helpers
- Demonstrates proper "use strict" and function patterns

**`hs_repo_old/extensions/geometry/geometry.lua`**
- **GROUND TRUTH** - authoritative source for all behavior
- 759 lines of pure Lua implementation
- Must be ported line-by-line to JavaScript
- All logic, edge cases, and algorithms defined here

### New Files

**`Hammerspoon 2/Modules/hs.geometry/GeometryModule.swift`**
- Minimal Swift module for registration in ModuleRoot
- Conforms to HSModuleAPI protocol
- Provides module name "geometry"
- Empty shutdown() method (no cleanup needed)
- No functional code - all logic in JavaScript

**`Hammerspoon 2/Modules/hs.geometry/hs.geometry.js`**
- Pure JavaScript port of geometry.lua
- Implements all constructors, properties, methods
- Handles type detection and validation
- Provides string parsing for all documented formats
- Implements geometric algorithms (union, intersect, fit, etc.)
- ~800-1000 lines estimated

## Step by Step Tasks

### Step 1: Create Module Directory Structure

- Create directory `Hammerspoon 2/Modules/hs.geometry/`
- This follows the established pattern for all hs.* modules

### Step 2: Implement Minimal Swift Module

Create `Hammerspoon 2/Modules/hs.geometry/GeometryModule.swift`:

- Add file header with copyright and creation date
- Import Foundation and JavaScriptCore
- Define `@objc protocol HSGeometryModuleAPI: JSExport` (empty - all functionality in JS)
- Define `@objc class HSGeometryModule: NSObject, HSModuleAPI, HSGeometryModuleAPI`
- Implement required `var name = "geometry"` property
- Implement required `init()` (call super.init())
- Implement required `shutdown()` method (empty - no resources to clean up)
- Add `deinit` with print statement for debugging (pattern from TimerModule)
- Mark class with `@_documentation(visibility: private)` attribute

**Rationale**: Module must exist in Swift for ModuleRoot registration, even though all logic is JavaScript.

### Step 3: Port Core Type System

In `Hammerspoon 2/Modules/hs.geometry/hs.geometry.js`, implement:

- Add `"use strict";` directive at top of file
- Define internal helper `_isGeometry(obj)` to detect geometry objects (check for `_x`, `_y`, `_w`, `_h` fields)
- Define type detection function `_getGeometryType(obj)` that returns 'point', 'size', 'rect', or 'unitrect'
  - Point: has `_x` and `_y`, no `_w` or `_h`
  - Size: has `_w` and `_h`, no `_x` or `_y`
  - UnitRect: has all four AND all values in [-ε, 1+ε] where ε = 1e-13 (matches source: `1.0000000000001`)
  - Rect: has all four AND doesn't meet unitrect criteria
- Define normalization helper `_normalize(obj)` to flip negative w/h and adjust x/y
- **IMPORTANT**: Epsilon value must match source exactly: compare against `1.0000000000001` and `-0.0000000000001`
- Define helper to detect and convert HSRect/HSPoint/HSSize objects: check `obj.typeName === "HSRect"` etc.

**Source Reference**: Lines 1-100 of geometry.lua (especially line 58 for epsilon)
**Validation**: Type detection must match original Lua behavior exactly

### Step 4: Implement Flexible Constructor

Implement `hs.geometry.new(...)` and make `hs.geometry` callable as function:

**Constructor Pattern** (following Lua's `setmetatable` with `__call`):
```javascript
(function() {
    function geometry(...args) {
        return newGeometry(...args);
    }

    // Add static methods
    geometry.rect = function(x, y, w, h) { return newGeometry(x, y, w, h); };
    geometry.point = function(x, y) { return newGeometry(x, y); };
    geometry.size = function(w, h) { return newGeometry(null, null, w, h); };
    geometry.new = newGeometry;

    // Replace hs.geometry module object with constructor function
    hs.geometry = geometry;
})();
```

**Constructor Logic** (`newGeometry` function):
- Handle 0 arguments: return empty rect {_x:0, _y:0, _w:0, _h:0}
- Handle 1 argument:
  - If geometry object (has `_x`, `_y`, etc.): return as-is (no copy)
  - If HSRect/HSPoint/HSSize (check `obj.typeName`): convert to geometry object
  - If string: call `_parseString()` helper (implementer translates Lua patterns to JS regex)
  - If array: extract values by index
  - If plain object: extract by property names (x/y/w/h or x1/y1/x2/y2)
- Handle 2 arguments:
  - Both numbers: create point
  - Both objects/strings: combine into rect
- Handle 3 arguments: invalid (throw error or interpret as x,y,w)
- Handle 4 arguments: create based on which are non-null
- After construction, call `_normalize()` to fix negative w/h

**Return Plain Object Literal** with getters/setters:
```javascript
return {
    _x: x, _y: y, _w: w, _h: h,
    get typeName() { return "geometry." + _getGeometryType(this); },
    get x() { return this._x; },
    set x(v) { this._x = Number(v); },
    type: function() { return _getGeometryType(this); },
    move: function(pt) { /* ... */ return this; }
    // ... only properties relevant to this type
};
```

**Source Reference**: Lines 100-300 of geometry.lua (constructor logic), line 758 (callable pattern)
**Validation**: All 20+ constructor formats from original must work identically

### Step 5: Implement String Parser

Implement `_parseString(str)` helper:

- Strip whitespace, brackets `[]`
- Try patterns in order:
  1. `"X Y / WxH"` or `"X,Y WxH"` → rect
  2. `"X Y > X2 Y2"` or `"X,Y>X2,Y2"` → rect from corners
  3. `"X Y W H"` (four numbers) → rect
  4. `"WxH"` or `"W*H"` → size
  5. `"X Y"` or `"X,Y"` → point
- Support both space and comma as separator
- Support `x`, `*`, `/`, `>` as delimiters
- Handle `[...]` format as unit rect (divide by 100)
- Return object with `{_x, _y, _w, _h}` fields (with undefined as appropriate)

**IMPORTANT**: Implementer must translate Lua string patterns to JavaScript regex
- Lua patterns (geometry.lua lines 73-110) use different syntax than JS regex
- Example: Lua `'(%-?%d*%.?%d*)'` → JS `'(-?\\d*\\.?\\d*)'`
- Implementer has full context to handle this translation

**Source Reference**: Lines 73-110 of geometry.lua (parse function)
**Validation**: Parse all documented string formats from original

### Step 6: Implement Property Accessors

Define getters/setters using JavaScript object literal syntax within the constructor:

**IMPORTANT**: Only define properties relevant to each type (geometry.lua uses metatable to conditionally expose):
- **Point**: `x`, `y`, `x1`, `y1`, `xy`, `topleft`, `length`, `angle`
- **Size**: `w`, `h`, `wh`, `size`, `area`, `aspect`, `length`, `angle`
- **Rect/UnitRect**: All position + dimension properties

**Position Properties** (for points and rects):
- `x`, `x1`: get/set `_x`
- `y`, `y1`: get/set `_y`
- `x2`: get `_x + _w`, set adjusts `_w` = value - `_x` (rects only)
- `y2`: get `_y + _h`, set adjusts `_h` = value - `_y` (rects only)
- `xy`, `topleft`: get/set point object with (x, y)
- `x2y2`, `bottomright`: get/set point object with (x2, y2) (rects only)
- `center`: get center point, set moves rect to keep new center (rects only)

**Dimension Properties** (for sizes and rects):
- `w`: get/set `_w`, clamp to >= 0
- `h`: get/set `_h`, clamp to >= 0
- `wh`, `size`: get/set size object with (w, h)

**Computed Properties** (getters only):
- `area`: return `w * h` (sizes and rects)
- `aspect`: return `w / h` (sizes and rects)
- `length`: return `Math.sqrt(x*x + y*y)` for point, `Math.sqrt(w*w + h*h)` for size
- `angle`: return `Math.atan2(y, x)` for point, `Math.atan2(h, w)` for size
- `table`: return `{x, y, w, h}` object (only defined fields)
- `string`: return formatted string like `"hs.geometry.rect(10,20,100,200)"`

**Type Properties**:
- `typeName`: **getter** that returns `"geometry.point"`, `"geometry.size"`, `"geometry.rect"`, or `"geometry.unitrect"`
- `type()`: **method** that returns `"point"`, `"size"`, `"rect"`, or `"unitrect"`

**Note on Chaining**:
- Property setters follow standard JavaScript (no return value)
- Methods return `this` for chaining (e.g., `rect.move(delta).scale(2)`)

**Source Reference**: Lines 172-450 of geometry.lua
**Validation**: All property aliases must work, setters must affect correct internal fields

### Step 7: Implement Comparison and Conversion Methods

**`type()`**: Return 'point', 'size', 'rect', 'unitrect', or null using `_getGeometryType()`
- This method provides API compatibility with original Hammerspoon

**`equals(other)`**: Return true if all defined fields match exactly
- Accept both geometry objects and HSRect/HSPoint/HSSize (convert first)

**`copy()`**: Return new object with same field values (deep copy)
- Must recreate object literal with all properties and methods

**`floor()`**: Apply `Math.floor()` to all defined fields, return `this` for chaining

**Error Handling**: Use `throw new Error("message")` for all errors (Lua's `error()` equivalent)
- No stack level parameter (Lua's level 2/3 not applicable in JavaScript)

**Source Reference**: Lines 150-170 (copy), 450-500 of geometry.lua
**Validation**: Type detection must match Lua behavior precisely

### Step 8: Implement Geometric Transformations

**`move(point)`**:
- Add point.x (or point._x or point.w) to this.x
- Add point.y (or point._y or point.h) to this.y
- Return `this` for chaining

**`scale(size_or_number)`**:
- If number: uniform scale
- If object: scale w by size.w, h by size.h
- For rects: scale around center (keep center fixed, scale distance from center)
- For points: scale from origin
- Return `this` for chaining

**`normalize()`**:
- For points/vectors: scale to unit length (length = 1)
- Divide x and y by current length
- Return `this` for chaining

**Source Reference**: Lines 500-580 of geometry.lua
**Validation**: Rect scaling must preserve center point

### Step 9: Implement Fit and Containment

**`fit(bounds)`**:
- Scale down rect to fit within bounds if necessary
- Preserve aspect ratio
- Only scale if larger than bounds (never scale up)
- Center within bounds
- Return `this` for chaining

**`inside(rect)`**:
- For points: return true if `x >= rect.x && x <= rect.x+rect.w && y >= rect.y && y <= rect.y+rect.h`
- For rects: return true if completely contained
- Work with unit rects (all values 0-1)

**Source Reference**: Lines 580-620 of geometry.lua
**Validation**: Fit must never scale up, only down. Aspect ratio must be preserved.

### Step 10: Implement Set Operations

**`union(rect)`**:
- Create new rect encompassing both rectangles
- `x = Math.min(this.x, rect.x)`
- `y = Math.min(this.y, rect.y)`
- `x2 = Math.max(this.x2, rect.x2)`
- `y2 = Math.max(this.y2, rect.y2)`
- `w = x2 - x`, `h = y2 - y`
- Return new geometry object (not `this`)

**`intersect(rect)`**:
- Create new rect from intersection
- `x = Math.max(this.x, rect.x)`
- `y = Math.max(this.y, rect.y)`
- `x2 = Math.min(this.x2, rect.x2)`
- `y2 = Math.min(this.y2, rect.y2)`
- `w = Math.max(0, x2 - x)`, `h = Math.max(0, y2 - y)`
- If non-overlapping: return "projection" with w=0 or h=0
- Return new geometry object (not `this`)

**Source Reference**: Lines 620-680 of geometry.lua
**Validation**: Non-overlapping intersections must project to nearest edge/corner

### Step 11: Implement Vector Operations

**`vector(point)`**:
- Return vector from this center (or point) to target center (or point)
- Use rect.center for rects, point.x/y for points
- Return new HSPoint object

**`angle()`**:
- Return `Math.atan2(y, x)` for point
- Return `Math.atan2(h, w)` for size
- Return angle in radians from positive x-axis

**`angleTo(point)`**:
- Calculate vector to point: `this.vector(point)`
- Return vector.angle()

**`distance(point)`**:
- Calculate vector to point: `this.vector(point)`
- Return vector.length (Euclidean distance)

**`rotateCCW(aroundpoint, ntimes)`**:
- Rotate this point counterclockwise around aroundpoint
- `ntimes` defaults to 1 (90 degrees per rotation)
- For each rotation: `{dx, dy} = {-dy, dx}` (90° CCW rotation matrix)
- Return new HSPoint object

**Source Reference**: Lines 680-730 of geometry.lua
**Validation**: Rotation must be counterclockwise. Angles in radians.

### Step 12: Implement Unit Rect Conversion

**`toUnitRect(frame)`**:
- Convert absolute rect to unit rect within frame
- `x_unit = (this.x - frame.x) / frame.w`
- `y_unit = (this.y - frame.y) / frame.h`
- `w_unit = this.w / frame.w`
- `h_unit = this.h / frame.h`
- Clip result to frame bounds via intersect
- Return new unitrect geometry object

**`fromUnitRect(frame)`**:
- Convert unit rect to absolute rect within frame
- `x_abs = frame.x + this.x * frame.w`
- `y_abs = frame.y + this.y * frame.h`
- `w_abs = this.w * frame.w`
- `h_abs = this.h * frame.h`
- Return new rect geometry object

**Source Reference**: Lines 730-759 of geometry.lua
**Validation**: toUnitRect/fromUnitRect must be inverse operations

### Step 13: Add Convenience Constructors and Aliases

**Convenience Constructors**:
- `hs.geometry.rect(x, y, w, h)` → alias for `new(x, y, w, h)`
- `hs.geometry.point(x, y)` → alias for `new(x, y)`
- `hs.geometry.size(w, h)` → alias for `new(null, null, w, h)`
- `hs.geometry.copy(geom)` → alias for `geom.copy()`

**Legacy Aliases** (for backward compatibility):
- `hs.geometry.hypot` → `getlength`
- `hs.geometry.rectMidPoint` → `getcenter`
- `hs.geometry.intersectionRect` → `intersect`
- `hs.geometry.isPointInRect` → `inside`

**Source Reference**: Lines 40-60, 740-759 of geometry.lua
**Validation**: All aliases must work identically to primary functions

### Step 14: Register Module in ModuleRoot

In `Hammerspoon 2/Engine/ModuleRoot.swift`:

**Modify `ModuleRootAPI` protocol**:
- Add line after existing module properties: `@objc var geometry: HSGeometryModule { get }`

**Modify `ModuleRoot` class**:
- Add line after existing module getters: `@objc var geometry: HSGeometryModule { get { getOrCreate(name: "geometry", type: HSGeometryModule.self)}}`

**Pattern**: Exactly follows existing modules (timer, alert, window, etc.)
**Position**: Add alphabetically between existing modules

### Step 15: Update Xcode Project

In `Hammerspoon 2.xcodeproj/project.pbxproj`:

**Add GeometryModule.swift**:
- Add to "Compile Sources" build phase
- Set as member of "Hammerspoon 2" target
- Use Xcode GUI: Right-click Modules/hs.geometry folder → "Add Files to Hammerspoon 2"

**Add hs.geometry.js**:
- Add to "Copy Bundle Resources" build phase
- Set as member of "Hammerspoon 2" target
- Ensures file is included in app bundle for runtime loading

**Verification**:
- Build project to ensure no errors
- Check Product → Hammerspoon 2.app → Contents → Resources contains hs.geometry.js

### Step 16: Create Integration Tests

Create `Hammerspoon 2Tests/IntegrationTests/HSGeometryIntegrationTests.swift`:

**Test Structure**:
- Inherit from XCTestCase
- Set up JSEngine context in `setUp()`
- Clean up in `tearDown()`

**Test Cases** (minimum):
1. `testGeometryModuleLoads()` - Verify hs.geometry is accessible
2. `testConstructorFormats()` - Test 10+ constructor variations
3. `testPointOperations()` - Test point creation, properties, vector math
4. `testSizeOperations()` - Test size creation, properties, scaling
5. `testRectOperations()` - Test rect creation, union, intersect, fit
6. `testUnitRectConversion()` - Test toUnitRect/fromUnitRect round-trip
7. `testStringParsing()` - Test all documented string formats
8. `testPropertyAliases()` - Verify x/x1, xy/topleft, wh/size all work
9. `testMethodChaining()` - Test that move/scale/floor return self
10. `testTypeDetection()` - Verify type() returns correct values
11. `testEdgeCases()` - Negative w/h normalization, zero dimensions, unitrect boundaries

**Validation Approach**:
- Use XCTAssertEqual for numeric comparisons
- Use XCTAssertTrue/False for boolean operations
- Compare against known-good values from original Hammerspoon
- Test error conditions (invalid inputs)

### Step 17: Create Manual Validation Script

Create `scripts/test-geometry.js` (user script for manual testing):

**Test Categories**:

1. **Constructor Tests**:
   ```javascript
   const p = hs.geometry(10, 20);
   const s = hs.geometry("100x200");
   const r = hs.geometry({x:10, y:20, w:100, h:200});
   console.log(p.string, s.string, r.string);
   ```

2. **Property Tests**:
   ```javascript
   const r = hs.geometry(0, 0, 100, 100);
   console.log(r.center); // Should be 50,50
   r.center = {x:0, y:0};
   console.log(r.string); // Should be -50,-50,100,100
   ```

3. **Operation Tests**:
   ```javascript
   const r1 = hs.geometry(0, 0, 100, 100);
   const r2 = hs.geometry(50, 50, 100, 100);
   console.log(r1.union(r2).string); // Should be 0,0,150,150
   console.log(r1.intersect(r2).string); // Should be 50,50,50,50
   ```

4. **Unit Rect Tests**:
   ```javascript
   const frame = hs.geometry(0, 0, 1000, 1000);
   const r = hs.geometry(250, 250, 500, 500);
   const ur = r.toUnitRect(frame);
   console.log(ur.string); // Should be 0.25,0.25,0.5,0.5
   console.log(ur.fromUnitRect(frame).string); // Should match r.string
   ```

**Run via**: hs2 CLI or Hammerspoon 2 console

### Step 18: Run Validation Commands

Execute all validation commands to ensure zero regressions and complete functionality.

## Validation Commands

Execute every command to validate the feature is complete with zero regressions.

**Build Project**:
```bash
xcodebuild -project "Hammerspoon 2.xcodeproj" -scheme "Hammerspoon 2" -configuration Debug build
```
Expected: Build succeeds with 0 errors, 0 warnings

**Run Unit Tests**:
```bash
xcodebuild test -project "Hammerspoon 2.xcodeproj" -scheme "Hammerspoon 2" -destination 'platform=macOS'
```
Expected: All tests pass, including new HSGeometryIntegrationTests

**Test Module Loading** (via hs2 CLI):
```bash
/Users/dmg/Library/Developer/Xcode/DerivedData/Hammerspoon_2-*/Build/Products/Debug/hs2 -c "console.log(typeof hs.geometry)"
```
Expected output: `"object"`

**Test Type Detection**:
```bash
hs2 -c "const p = hs.geometry(10, 20); console.log(p.type())"
```
Expected output: `point`

**Test Constructor Variations**:
```bash
hs2 -c "console.log(hs.geometry('10 20').string)"
hs2 -c "console.log(hs.geometry('100x200').string)"
hs2 -c "console.log(hs.geometry('10 20/100x200').string)"
hs2 -c "console.log(hs.geometry({x:10, y:20, w:100, h:200}).string)"
```
Expected: Each outputs correctly formatted geometry string

**Test Union Operation**:
```bash
hs2 -c "const r1 = hs.geometry(0,0,100,100); const r2 = hs.geometry(50,50,100,100); console.log(r1.union(r2).string)"
```
Expected output: `hs.geometry.rect(0,0,150,150)` or equivalent

**Test Intersect Operation**:
```bash
hs2 -c "const r1 = hs.geometry(0,0,100,100); const r2 = hs.geometry(50,50,100,100); console.log(r1.intersect(r2).string)"
```
Expected output: `hs.geometry.rect(50,50,50,50)` or equivalent

**Test Unit Rect Conversion**:
```bash
hs2 -c "const frame = hs.geometry(0,0,1000,1000); const r = hs.geometry(250,250,500,500); const ur = r.toUnitRect(frame); console.log(ur.string)"
```
Expected: Unit rect with values 0.25,0.25,0.5,0.5

**Test Method Chaining**:
```bash
hs2 -c "const r = hs.geometry(0,0,100,100).move({x:10,y:10}).scale(2).floor(); console.log(r.string)"
```
Expected: Chaining works, returns rect moved and scaled

**Test Property Aliases**:
```bash
hs2 -c "const r = hs.geometry(10,20,100,200); console.log(r.x === r.x1, r.xy.x === r.topleft.x)"
```
Expected output: `true true`

**Test Vector Operations**:
```bash
hs2 -c "const p1 = hs.geometry(0,0); const p2 = hs.geometry(100,100); console.log(p1.distance(p2))"
```
Expected output: `141.42135...` (sqrt(100² + 100²))

**Run Manual Validation Script**:
```bash
hs2 scripts/test-geometry.js
```
Expected: All test categories pass, output matches expected values

**Test Integration with Existing Modules**:
```bash
hs2 -c "const win = hs.window.focusedWindow(); if(win) { const f = win.frame(); console.log(f.type ? f.type() : typeof f); }"
```
Expected: Geometry types work with window frames

**Verify No Module Loading Errors**:
```bash
hs2 -c "hs.geometry; console.log('Module loaded successfully')"
```
Expected output: `Module loaded successfully` (no errors in log)

## Document Changes

### Update Module Documentation

**Create `Hammerspoon 2/Modules/hs.geometry/README.md`**:
- Document all constructor formats with examples
- List all properties with descriptions
- List all methods with parameter descriptions
- Include usage examples for common operations
- Note differences from original Hammerspoon (if any)
- Reference original API docs: https://www.hammerspoon.org/docs/hs.geometry.html

**Update `claude.md` (LLM Developer Guide)**:
- Add hs.geometry to "Modules" section in project structure
- Add to future development ideas (mark as complete)
- Note that it's pure JavaScript implementation

**Update Main README** (if needed):
- Add hs.geometry to list of implemented modules
- Update module count/status

### Code Documentation

**GeometryModule.swift**:
- Add header comments describing module purpose
- Add inline comments for any non-obvious Swift code

**hs.geometry.js**:
- Add file header with description, creation date, license
- Add JSDoc-style comments for all public functions
- Document parameters, return types, examples
- Note behavioral quirks (normalization, chaining, etc.)

## Git Log

```
Port hs.geometry module from original Hammerspoon

Implements complete geometry utility library for 2D spatial operations:
- Four geometric types: point, size, rect, unitrect
- Flexible constructor supporting 20+ input formats (strings, objects, arrays)
- 30+ properties with getter/setter support and aliases
- 20+ methods for transformations, containment, set operations, vector math
- Unit rect conversion for relative positioning
- Pure JavaScript implementation (ported from Lua)

Files added:
- Hammerspoon 2/Modules/hs.geometry/GeometryModule.swift (minimal module)
- Hammerspoon 2/Modules/hs.geometry/hs.geometry.js (~900 lines, main logic)
- Hammerspoon 2Tests/IntegrationTests/HSGeometryIntegrationTests.swift
- scripts/test-geometry.js (manual validation)

Files modified:
- Hammerspoon 2/Engine/ModuleRoot.swift (register geometry module)
- Hammerspoon 2.xcodeproj/project.pbxproj (add new files)

Maintains full API compatibility with original hs.geometry while adapting
to JavaScript semantics (no operator overloading, uses method calls).
All 759 lines of original Lua implementation ported with identical behavior.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

## Notes

### Key Implementation Details

1. **Pure JavaScript Rationale**:
   - Original is pure Lua with no C/Objective-C components
   - No macOS API calls required
   - Computational geometry is platform-agnostic
   - Easier to maintain API compatibility with scripting language
   - Leverages existing HSPoint/HSRect/HSSize for bridging only

2. **No Operator Overloading**:
   - JavaScript doesn't support custom operators
   - Original Lua uses: `+`, `*`, `^`, `..`, `==`, `<`, `-` (unary)
   - Solution: Use method names only (`union()`, `intersect()`, `equals()`)
   - Users must adapt: `r1 + r2` becomes `r1.union(r2)`

3. **Type Detection Strategy**:
   - Use internal fields `_x`, `_y`, `_w`, `_h` (underscore prefix)
   - Public properties access internal fields
   - Type determined by which fields are defined
   - UnitRect detection uses epsilon tolerance (1e-12)

4. **Normalization Behavior**:
   - Negative width/height automatically fixed in constructor
   - If `w < 0`: `x = x + w; w = -w`
   - If `h < 0`: `y = y + h; h = -h`
   - Maintains invariant: w >= 0, h >= 0

5. **Method Chaining**:
   - Transformation methods return `this` (move, scale, floor, fit, normalize)
   - Set operations return new objects (union, intersect)
   - Getters return values/objects
   - Pattern enables: `rect.move(delta).scale(2).floor()`

6. **Rect Scaling Semantics**:
   - Rects scale around center point (center stays fixed)
   - Points scale from origin (0, 0)
   - Implementation: store center, scale dimensions, restore center

7. **Unit Rect Use Cases**:
   - Relative positioning (0-1 coordinates)
   - Used by hs.screen for cross-resolution layouts
   - Used by hs.window.filter for screen-relative positions
   - Automatic detection by epsilon tolerance

8. **String Parsing Flexibility**:
   - Supports multiple separator styles (space, comma, /, >, x, *)
   - Flexible whitespace handling
   - Bracket notation for unit rects: `[50,50>100,100]`
   - Must handle all formats from original for compatibility

9. **Testing Strategy**:
   - Integration tests via XCTest framework
   - Manual validation via hs2 CLI commands
   - Test script for interactive verification
   - Compare outputs with original Hammerspoon when possible

10. **Future Enhancements** (not in scope for this spec):
    - Matrix transformations (rotation by arbitrary angles)
    - Bezier curve operations
    - Path manipulation
    - Advanced collision detection
    - 3D geometry extension

### Potential Gotchas

1. **JavaScript Number Precision**:
   - JavaScript uses IEEE 754 double precision
   - May have slight differences from Lua in edge cases
   - Use epsilon comparisons for floating point equality

2. **Property Definition**:
   - Must use Object.defineProperty for getters/setters
   - Cannot use simple assignment like Lua metatables
   - More verbose but functionally equivalent

3. **Type Coercion**:
   - JavaScript is more permissive than Lua
   - May need explicit type checking
   - Validate inputs to prevent unexpected behavior

4. **Module Loading Order**:
   - hs.geometry.js loaded after GeometryModule initialization
   - ModuleRoot.getOrCreate handles automatic JS loading
   - Ensure no circular dependencies with other modules

5. **Performance Considerations**:
   - Pure JS may be slower than native Swift for heavy computation
   - Acceptable tradeoff for API compatibility
   - Most operations are not performance-critical
   - Window/screen modules handle heavy lifting in Swift

### Dependencies and Integration

**hs.geometry is used by** (in original Hammerspoon):
- `hs.window` - Window frame manipulation
- `hs.screen` - Screen frame and workspace operations
- `hs.grid` - Grid-based window positioning
- `hs.layout` - Declarative window layouts
- `hs.mouse` - Mouse position handling
- `hs.menubar` - Menu bar item positioning

**Integration points in Hammerspoon 2**:
- HSWindow.frame() should return geometry-compatible object
- HSScreen operations should accept geometry objects
- Future modules should use hs.geometry for spatial operations

**No breaking changes expected**: Existing modules use HSRect/HSPoint/HSSize directly, which remain unchanged. hs.geometry provides user-facing convenience layer.

### Reference Materials

- **Original Source**: `hs_repo_old/extensions/geometry/geometry.lua`
- **Original Docs**: https://www.hammerspoon.org/docs/hs.geometry.html
- **Lua 5.3 Reference**: For understanding metatables, operators
- **JavaScript Spec**: ES6+ features (classes, arrow functions, getters/setters)
- **Existing Modules**: hs.timer, hs.application (patterns to follow)

### Success Criteria

Feature is complete when:
- [x] All validation commands execute without errors
- [x] Integration tests pass (100% success rate)
- [x] Manual test script produces expected outputs
- [x] Module loads via hs.geometry accessor
- [x] All 20+ constructor formats work
- [x] All properties and aliases function correctly
- [x] All methods produce correct results
- [x] Type detection matches original behavior
- [x] String parsing handles all documented formats
- [x] Unit rect conversion round-trips correctly
- [x] Method chaining works as expected
- [x] No regressions in existing modules
- [x] Documentation complete and accurate

When all checkboxes are verified, the feature is production-ready.
