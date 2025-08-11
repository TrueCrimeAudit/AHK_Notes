# Topic: Conditional Mouse Button Blocking with Hooks

## Category

Pattern

## Overview

Similar to keyboard hooks, mouse hooks can intercept and conditionally block mouse button actions based on the state of other buttons or conditions. This technique allows creating custom mouse gestures and preventing default mouse behavior when specific button combinations are used.

## Key Points

- Mouse hooks work similarly to keyboard hooks with the same performance considerations
- The $ modifier forces hook usage and allows blocking the original mouse button function
- GetKeyState can check the current state of other mouse buttons or keyboard keys
- Blocked mouse actions must be explicitly re-sent if you want the normal behavior to occur
- Multiple mouse hooks from different applications can slow down mouse responsiveness
- Context-sensitive blocking allows different behavior in different applications

## Syntax and Parameters

```cpp
; Basic conditional blocking pattern
$MouseButton:: {
    if (GetKeyState("OtherButton", "P")) {
        ; Custom action - blocks original
        CustomAction()
        return
    }
    ; Send original action
    Send("{MouseButton}")
}

; Mouse button names
; LButton, RButton, MButton
; XButton1, XButton2 (side buttons)
; WheelUp, WheelDown, WheelLeft, WheelRight
```

## Code Examples

```cpp
; Example 1: Block right-click when middle button held
$RButton:: {
    if (GetKeyState("MButton", "P")) {
        ToolTip("Right-click blocked!")
        SetTimer(() => ToolTip(), -2000)
        return  ; Blocks the normal right-click
    }
    Send("{RButton}")  ; Allow normal right-click
}

; Example 2: Multiple button combination handling
$RButton:: {
    if (GetKeyState("MButton", "P")) {
        ShowCustomMenu()
        return
    } else if (GetKeyState("XButton1", "P")) {
        SpecialFunction()
        return
    }
    Send("{RButton}")
}

; Example 3: Keyboard + mouse combinations
$RButton:: {
    if (GetKeyState("Ctrl", "P")) {
        MsgBox("Ctrl + Right Click intercepted!")
        return
    }
    Send("{RButton}")
}

; Example 4: Time-based button sequence detection
LastMiddlePress := 0

MButton:: {
    global LastMiddlePress
    LastMiddlePress := A_TickCount
    Send("{MButton}")
}

$RButton:: {
    global LastMiddlePress
    if (A_TickCount - LastMiddlePress < 500) {
        ; Middle button was pressed within last 500ms
        MsgBox("Sequential button press detected!")
        return
    }
    Send("{RButton}")
}

; Example 5: Application-specific blocking
$RButton:: {
    ActiveApp := WinGetProcessName("A")
    
    if (GetKeyState("MButton", "P")) {
        if (ActiveApp = "notepad.exe") {
            ; Custom behavior in Notepad
            CustomNotepadAction()
            return
        }
    }
    Send("{RButton}")
}
```

## Implementation Notes

**Mouse Hook Behavior:**
- Mouse hooks are automatically installed when using $ modifier with mouse buttons
- Unlike keyboard hooks, mouse hooks typically have less impact on system performance
- Mouse wheel events can also be hooked and conditionally blocked
- Raw mouse input can still be captured by applications even when hooks block standard messages

**Button State Detection:**
- GetKeyState with "P" flag checks physical button state
- Mouse button states are updated in real-time unlike some keyboard states
- Side buttons (XButton1, XButton2) may not be available on all mice
- Button state checking works across different input methods (hardware, SendInput, etc.)

**Performance Considerations:**
- Multiple mouse hooks can create input lag, especially in games
- Keep hook callback functions as fast as possible
- Avoid heavy processing in mouse button handlers
- Consider using timers for delayed actions instead of blocking in the handler

**Common Use Cases:**
- Gaming: Custom mouse gestures for MMOs or strategy games
- CAD/Design: Modified right-click behavior in design applications
- Productivity: Context-sensitive mouse actions
- Accessibility: Alternative input methods for users with mobility issues

**Potential Issues:**
- Some games or applications may not work well with mouse hooks
- Anti-cheat software may flag mouse hook usage
- Context menus may not appear if right-click is blocked
- User confusion if standard mouse behavior is significantly altered

## Related AHK Concepts

- Keyboard hooks and the $ modifier
- GetKeyState for button state detection
- Mouse coordinate functions (MouseGetPos, etc.)
- Window detection for context-sensitive behavior
- Menu creation for custom context menus
- Timer functions for time-based conditions
- WinEvent hooks for window state changes

## Tags

#AutoHotkey #MouseHooks #InputHandling #Gestures #Conditional #ButtonBlocking #UserInterface