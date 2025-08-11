# Topic: Click Function - Mouse Input Automation

## Category

Function

## Overview

The Click function performs mouse clicks at specified screen coordinates or relative to the current mouse position. It's the primary method for automating mouse input in AutoHotkey v2, supporting various button types, click modes, and coordinates systems for precise mouse control.

## Key Points

- Supports absolute and relative coordinates for precise positioning
- Handles all mouse buttons (left, right, middle, X1, X2) with customizable click counts
- Provides multiple coordinate modes (Screen, Window, Client) for different targeting needs
- Includes optional delays and speed settings for natural mouse movement simulation
- Can perform both clicks and mouse movement without clicking

## Syntax and Parameters

```cpp
Click([X, Y, WhichButton, ClickCount, Speed, Relative])

; Parameters:
; X, Y        - Target coordinates (optional, defaults to current position)
; WhichButton - "Left", "Right", "Middle", "X1", "X2" (default: "Left")
; ClickCount  - Number of clicks to perform (default: 1)
; Speed       - Movement speed 0-100, 0=instant (default: uses SetDefaultMouseSpeed)
; Relative    - "Rel" for relative movement (default: absolute)
```

## Code Examples

```cpp
; Basic left click at current position
Click()

; Click at specific coordinates
Click(100, 200)

; Right-click with multiple clicks
Click(300, 400, "Right", 2)

; Move mouse without clicking (0 clicks)
Click(500, 600, , 0)

; Relative movement from current position
Click(50, -30, , 1, , "Rel")

; Slow movement for natural animation
Click(800, 600, "Left", 1, 25)

; Complex example: Triple-click with custom speed
Click(400, 300, "Left", 3, 50)

; Mouse wheel simulation using different buttons
Click( , , "WheelUp", 3)      ; Scroll up 3 times
Click( , , "WheelDown", 5)    ; Scroll down 5 times

; Advanced coordinate targeting
CoordMode("Mouse", "Window")   ; Set coordinate mode
Click(100, 50)                 ; Click relative to active window
```

## Implementation Notes

**Coordinate Systems:**
- Default uses screen coordinates (absolute positioning)
- Use CoordMode() to change between Screen, Window, and Client coordinates
- Relative movement adds to current mouse position

**Performance Considerations:**
- Speed parameter affects execution time - use 0 for instant movement
- Higher click counts may need SendMode adjustment for reliability
- Mouse movement respects system mouse acceleration settings

**Common Pitfalls:**
- Forgetting to account for window positioning in Window/Client modes
- Not handling screen resolution differences between systems
- Rapid clicking may trigger system double-click detection

**Compatibility Notes:**
- Works with all mouse button types including extended buttons (X1, X2)
- Wheel operations require specific button names (WheelUp, WheelDown, WheelLeft, WheelRight)
- Movement speed interacts with SetDefaultMouseSpeed global setting

## Related AHK Concepts

- [Send](../send.md) - Alternative method using Send command syntax
- [MouseMove](../../10_Language_Core/01-Functions/Built_In_Functions/mousemove.md) - Pure movement without clicking
- [CoordMode](../../10_Language_Core/01-Functions/Built_In_Functions/coordmode.md) - Coordinate system configuration
- [SetDefaultMouseSpeed](../../10_Language_Core/01-Functions/Built_In_Functions/setdefaultmousespeed.md) - Global speed settings
- [MouseClick](../../10_Language_Core/01-Functions/Built_In_Functions/mouseclick.md) - Legacy v1 equivalent

## Tags

#AutoHotkey #Mouse #Input #Automation #Coordinates #GUI #UserInterface