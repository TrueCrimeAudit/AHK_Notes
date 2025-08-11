# Topic: GetKeyState Function - Input State Detection and Monitoring

## Category

Function

## Overview

The GetKeyState function retrieves the current state of keyboard keys, mouse buttons, and joystick controls. It's essential for real-time input monitoring, conditional automation, interactive applications, and any script requiring awareness of current input device states without blocking execution.

## Key Points

- Detects current state of keyboard keys, mouse buttons, and joystick controls in real-time
- Supports both physical state detection and logical state checking for comprehensive input monitoring
- Provides toggle state checking for keys like Caps Lock, Num Lock, and Scroll Lock
- Enables non-blocking input detection for responsive interactive applications
- Essential for conditional automation, gaming applications, and accessibility tools

## Syntax and Parameters

```cpp
State := GetKeyState(KeyName [, Mode])

; Parameters:
; KeyName - Name of key, mouse button, or joystick control to check
; Mode    - Optional mode specifier (default: logical state)

; Mode options:
; "P" or "Physical" - Physical state (hardware level, bypass software)
; "T" or "Toggle"   - Toggle state for toggle keys (Caps, Num, Scroll Lock)
; (no mode)         - Logical state (considers software key state)

; Key naming conventions:
; Keyboard: "a", "Space", "Enter", "Ctrl", "Alt", "Shift", "F1"-"F24"
; Mouse: "LButton", "RButton", "MButton", "XButton1", "XButton2"
; Joystick: "Joy1", "Joy2", etc. + button/axis specifiers

; Return Values:
; 1 (true)  - Key is pressed/down/on
; 0 (false) - Key is released/up/off
; For joystick axes: numeric position value
```

## Code Examples

```cpp
; Basic key state checking
if (GetKeyState("Ctrl")) {
    MsgBox("Ctrl key is currently pressed")
}

; Mouse button checking
if (GetKeyState("LButton")) {
    MsgBox("Left mouse button is down")
}

; Toggle key state
capsLockOn := GetKeyState("CapsLock", "T")
if (capsLockOn) {
    MsgBox("Caps Lock is ON")
}

; Physical vs logical state
logicalCtrl := GetKeyState("Ctrl")      ; Software state
physicalCtrl := GetKeyState("Ctrl", "P") ; Hardware state

if (logicalCtrl != physicalCtrl) {
    MsgBox("Ctrl key state differs between logical and physical levels")
}

; Real-time input monitoring
class InputMonitor {
    static isMonitoring := false
    static monitorTimer := 0
    static keyStates := Map()
    static callbacks := Map()
    
    static Start() {
        if (this.isMonitoring) return
        
        this.isMonitoring := true
        this.monitorTimer := SetTimer(this.CheckStates.Bind(this), 10) ; 10ms polling
        
        OutputDebug("Input monitoring started")
    }
    
    static Stop() {
        if (this.monitorTimer) {
            SetTimer(this.monitorTimer, 0)
            this.monitorTimer := 0
        }
        this.isMonitoring := false
        this.keyStates.Clear()
        
        OutputDebug("Input monitoring stopped")
    }
    
    static AddKey(keyName, callback) {
        this.callbacks[keyName] := callback
        this.keyStates[keyName] := GetKeyState(keyName)
    }
    
    static RemoveKey(keyName) {
        this.callbacks.Delete(keyName)
        this.keyStates.Delete(keyName)
    }
    
    static CheckStates() {
        for keyName, callback in this.callbacks {
            currentState := GetKeyState(keyName)
            previousState := this.keyStates.Get(keyName, false)
            
            ; Detect state changes
            if (currentState != previousState) {
                this.keyStates[keyName] := currentState
                
                ; Call registered callback with state change info
                try {
                    callback.Call({
                        key: keyName,
                        state: currentState,
                        pressed: currentState && !previousState,
                        released: !currentState && previousState,
                        timestamp: A_TickCount
                    })
                } catch Error as err {
                    OutputDebug("Callback error for " . keyName . ": " . err.Message)
                }
            }
        }
    }
}

; Modifier key combination detector
class ModifierDetector {
    static combinations := Map()
    static active := Map()
    
    static AddCombination(name, keys, callback) {
        this.combinations[name] := {
            keys: keys,
            callback: callback,
            wasActive: false
        }
    }
    
    static CheckCombinations() {
        for name, combo in this.combinations {
            allPressed := true
            
            ; Check if all keys in combination are pressed
            for key in combo.keys {
                if (!GetKeyState(key)) {
                    allPressed := false
                    break
                }
            }
            
            ; Detect activation/deactivation
            if (allPressed && !combo.wasActive) {
                ; Combination activated
                combo.wasActive := true
                try {
                    combo.callback.Call("activated", name, combo.keys)
                } catch Error as err {
                    OutputDebug("Combination callback error: " . err.Message)
                }
                
            } else if (!allPressed && combo.wasActive) {
                ; Combination deactivated
                combo.wasActive := false
                try {
                    combo.callback.Call("deactivated", name, combo.keys)
                } catch Error as err {
                    OutputDebug("Combination callback error: " . err.Message)
                }
            }
        }
    }
    
    static Start() {
        SetTimer(this.CheckCombinations.Bind(this), 20) ; Check every 20ms
    }
    
    static Stop() {
        SetTimer(this.CheckCombinations.Bind(this), 0)
    }
}

; Gaming input handler with precise timing
class GameInputHandler {
    static inputBuffer := []
    static maxBufferSize := 100
    static isRecording := false
    
    static StartRecording() {
        this.isRecording := true
        this.inputBuffer := []
        
        ; Monitor common gaming keys
        keys := ["w", "a", "s", "d", "Space", "LButton", "RButton", "Shift", "Ctrl"]
        
        for key in keys {
            InputMonitor.AddKey(key, this.RecordInput.Bind(this))
        }
        
        InputMonitor.Start()
    }
    
    static StopRecording() {
        this.isRecording := false
        InputMonitor.Stop()
        
        return this.GetInputSequence()
    }
    
    static RecordInput(event) {
        if (!this.isRecording) return
        
        ; Record only press events for cleaner sequence
        if (event.pressed) {
            inputEvent := {
                key: event.key,
                timestamp: event.timestamp,
                modifiers: this.GetCurrentModifiers()
            }
            
            this.inputBuffer.Push(inputEvent)
            
            ; Limit buffer size
            if (this.inputBuffer.Length > this.maxBufferSize) {
                this.inputBuffer.RemoveAt(1)
            }
        }
    }
    
    static GetCurrentModifiers() {
        modifiers := []
        
        if (GetKeyState("Ctrl")) modifiers.Push("Ctrl")
        if (GetKeyState("Shift")) modifiers.Push("Shift")
        if (GetKeyState("Alt")) modifiers.Push("Alt")
        if (GetKeyState("LWin") || GetKeyState("RWin")) modifiers.Push("Win")
        
        return modifiers
    }
    
    static GetInputSequence() {
        return this.inputBuffer.Clone()
    }
    
    static DetectCombo(sequence) {
        ; Define common gaming combos
        combos := Map(
            "wasd_movement", ["w", "a", "s", "d"],
            "jump_attack", ["Space", "LButton"],
            "sprint_jump", ["Shift", "Space"],
            "tactical_reload", ["r", "Shift"]
        )
        
        for comboName, comboKeys in combos {
            if (this.SequenceContains(sequence, comboKeys)) {
                return comboName
            }
        }
        
        return ""
    }
    
    static SequenceContains(sequence, targetKeys) {
        if (sequence.Length < targetKeys.Length) return false
        
        ; Check if target keys appear in sequence within reasonable timeframe
        targetIndex := 1
        timeWindow := 1000 ; 1 second window
        
        for event in sequence {
            if (event.key = targetKeys[targetIndex]) {
                targetIndex++
                if (targetIndex > targetKeys.Length) {
                    return true
                }
            }
        }
        
        return false
    }
}

; Accessibility key monitor for assistive features
class AccessibilityMonitor {
    static stickyKeys := false
    static slowKeys := false
    static toggleKeys := false
    static notifications := true
    
    static Start() {
        ; Monitor accessibility-related key states
        SetTimer(this.CheckAccessibilityFeatures.Bind(this), 100)
        
        ; Monitor for accessibility key combinations
        SetTimer(this.CheckAccessibilityCombos.Bind(this), 50)
    }
    
    static CheckAccessibilityFeatures() {
        ; Check toggle key states for accessibility notifications
        if (this.notifications) {
            this.CheckToggleKeyChanges()
        }
        
        ; Monitor for stuck keys (potential accessibility need)
        this.CheckForStuckKeys()
    }
    
    static CheckToggleKeyChanges() {
        static lastCapsState := GetKeyState("CapsLock", "T")
        static lastNumState := GetKeyState("NumLock", "T")
        static lastScrollState := GetKeyState("ScrollLock", "T")
        
        currentCaps := GetKeyState("CapsLock", "T")
        currentNum := GetKeyState("NumLock", "T")
        currentScroll := GetKeyState("ScrollLock", "T")
        
        if (currentCaps != lastCapsState) {
            this.NotifyToggleChange("Caps Lock", currentCaps)
            lastCapsState := currentCaps
        }
        
        if (currentNum != lastNumState) {
            this.NotifyToggleChange("Num Lock", currentNum)
            lastNumState := currentNum
        }
        
        if (currentScroll != lastScrollState) {
            this.NotifyToggleChange("Scroll Lock", currentScroll)
            lastScrollState := currentScroll
        }
    }
    
    static NotifyToggleChange(keyName, state) {
        message := keyName . " is now " . (state ? "ON" : "OFF")
        ; Could use ToolTip, TrayTip, or screen reader notifications
        ToolTip(message)
        SetTimer(() => ToolTip(), -2000) ; Hide after 2 seconds
    }
    
    static CheckForStuckKeys() {
        ; Check if any key has been held for an unusually long time
        static keyPressStart := Map()
        static stuckThreshold := 30000 ; 30 seconds
        
        commonKeys := ["Shift", "Ctrl", "Alt", "Space", "Enter", "LButton", "RButton"]
        
        for key in commonKeys {
            if (GetKeyState(key)) {
                if (!keyPressStart.Has(key)) {
                    keyPressStart[key] := A_TickCount
                } else if (A_TickCount - keyPressStart[key] > stuckThreshold) {
                    this.NotifyStuckKey(key)
                    keyPressStart[key] := A_TickCount ; Reset to avoid spam
                }
            } else {
                keyPressStart.Delete(key)
            }
        }
    }
    
    static NotifyStuckKey(key) {
        message := "Warning: " . key . " key appears to be stuck"
        MsgBox(message, "Accessibility Alert", "T5") ; 5 second timeout
    }
    
    static CheckAccessibilityCombos() {
        ; Sticky Keys activation (Shift pressed 5 times)
        static shiftPressCount := 0
        static lastShiftState := false
        
        currentShiftState := GetKeyState("Shift")
        
        if (currentShiftState && !lastShiftState) {
            shiftPressCount++
            if (shiftPressCount >= 5) {
                this.ToggleStickyKeys()
                shiftPressCount := 0
            }
            
            ; Reset counter after timeout
            SetTimer(() => shiftPressCount := 0, -2000)
        }
        
        lastShiftState := currentShiftState
    }
    
    static ToggleStickyKeys() {
        this.stickyKeys := !this.stickyKeys
        message := "Sticky Keys " . (this.stickyKeys ? "enabled" : "disabled")
        MsgBox(message, "Accessibility Feature")
    }
}

; Performance-optimized key state cache
class KeyStateCache {
    static cache := Map()
    static cacheTimeout := 50 ; Cache for 50ms
    
    static GetCachedState(key, mode := "") {
        cacheKey := key . "|" . mode
        currentTime := A_TickCount
        
        ; Check if cached value is still valid
        if (this.cache.Has(cacheKey)) {
            cached := this.cache[cacheKey]
            if (currentTime - cached.timestamp < this.cacheTimeout) {
                return cached.state
            }
        }
        
        ; Get fresh state and cache it
        state := GetKeyState(key, mode)
        this.cache[cacheKey] := {
            state: state,
            timestamp: currentTime
        }
        
        ; Clean old cache entries periodically
        if (this.cache.Count > 100) {
            this.CleanCache()
        }
        
        return state
    }
    
    static CleanCache() {
        currentTime := A_TickCount
        
        for cacheKey, data in this.cache {
            if (currentTime - data.timestamp > this.cacheTimeout * 2) {
                this.cache.Delete(cacheKey)
            }
        }
    }
    
    static ClearCache() {
        this.cache.Clear()
    }
}

; Example usage
; Start input monitoring
InputMonitor.Start()

; Monitor specific keys
InputMonitor.AddKey("F1", (event) => {
    if (event.pressed) {
        MsgBox("F1 was pressed at " . event.timestamp)
    }
})

; Set up modifier combinations
ModifierDetector.AddCombination("ctrl_shift", ["Ctrl", "Shift"], 
    (action, name, keys) => {
        if (action = "activated") {
            MsgBox("Ctrl+Shift combination activated")
        }
    })
ModifierDetector.Start()

; Start accessibility monitoring
AccessibilityMonitor.Start()

; Gaming input example
GameInputHandler.StartRecording()
; ... user plays game ...
; sequence := GameInputHandler.StopRecording()
; combo := GameInputHandler.DetectCombo(sequence)
```

## Implementation Notes

**Polling vs Event-Driven Detection:**
- GetKeyState requires polling; combine with SetTimer for real-time monitoring
- High-frequency polling (< 10ms) may impact performance on slower systems
- Consider caching recent results to reduce redundant GetKeyState calls
- Use appropriate polling intervals based on application responsiveness needs

**Physical vs Logical State Differences:**
- Physical state reflects actual hardware key position
- Logical state considers software modifications (key remapping, disabled keys)
- Use physical state for low-level input detection, logical for normal application logic
- Some security software may block physical state detection

**Toggle Key Behavior:**
- Toggle keys (Caps Lock, Num Lock, Scroll Lock) have persistent state
- Toggle state survives script restarts and system reboots
- Changes to toggle keys affect global system state
- Consider restoring original toggle states when script exits

**Performance Optimization:**
- Cache frequently checked key states to reduce API calls
- Use longer polling intervals for non-critical key monitoring
- Batch multiple key checks in single timer functions
- Consider using InputHook for high-performance input detection

**Cross-Platform Considerations:**
- Key names may vary between Windows versions
- Virtual key codes provide more consistent cross-version compatibility
- Some specialized keys may not be detectable on all hardware
- Joystick detection requires connected and recognized gaming devices

## Related AHK Concepts

- [InputHook](../../30_Built_In_Classes/04-Input_Classes/InputHook/inputhook.md) - Advanced input detection and interception
- [Hotkey](./hotkey-function.md) - Dynamic hotkey assignment based on conditions
- [Send](./send.md) - Input simulation complementing state detection
- [SetTimer](../03-Threading/settimer.md) - Timer-based polling for real-time monitoring
- [KeyWait](./keywait.md) - Waiting for specific key state changes

## Tags

#AutoHotkey #GetKeyState #InputDetection #KeyboardMonitoring #MouseState #Gaming #Accessibility #RealTime #StateMonitoring