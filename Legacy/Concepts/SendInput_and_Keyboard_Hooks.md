# Topic: Understanding SendInput and Keyboard Hooks

## Category

Concept

## Overview

SendInput is AutoHotkey's default send mode that provides fast, uninterruptible keystroke transmission by sending keys in batches. However, its effectiveness is significantly compromised when keyboard hooks are present, causing it to behave like SendEvent but with additional overhead. Understanding when keyboard hooks are installed and their impact on SendInput behavior is crucial for reliable script performance.

## Key Points

- SendInput removes and reinstalls AutoHotkey's keyboard hook during operation, which can cause key "leaking" during rapid input
- Any keyboard hook presence (from AHK or external programs) negates SendInput's advantages and may cause key interspersion
- SendInput automatically reverts to SendEvent when it detects other AutoHotkey keyboard hooks
- Many AutoHotkey features automatically install keyboard hooks: hotstrings, #HotIf, modifier symbols (~, *, $), custom combinations, InputHooks, and #InputLevel > 0
- SendEvent with appropriate key delays is often more reliable than SendInput when hooks are present
- GetKeyState's "physical" mode relies on keyboard hook data and can be unreliable when hooks are intermittently removed

## Syntax and Parameters

```cpp
; Send modes
SendMode "Input"    ; Default, fast but affected by hooks
SendMode "Event"    ; Slower but more reliable with hooks
SendMode "Play"     ; Alternative method

; Key delay settings for SendEvent
SetKeyDelay(PressDelay, ReleaseDuration)
SetKeyDelay(-1, 0)  ; Recommended for SendEvent

; Checking for keyboard hooks (v2.1+)
A_KeybdHookInstalled  ; >1 if another AHK script has hook

; Forcing keyboard hook installation
InstallKeybdHook(Install := true, LowLevel := false)
```

## Code Examples

```cpp
; Example 1: Key leaking problem with SendInput and hooks
$z:: {
    While GetKeyState("z", "P") {
        Send "{n Down}"     ; SendInput by default
        Sleep 200
        Send "{n Up}"
    }
}
; Problem: "z" can leak through between "n" characters

; Solution: Use SendEvent instead
SendMode "Event"
SetKeyDelay(-1, 0)
$z:: {
    While GetKeyState("z", "P") {
        Send "{n Down}"
        Sleep 200
        Send "{n Up}"
    }
}

; Example 2: SendInput vs SendEvent with hotkeys
; This triggers hotkey (RegisterHotkey implementation)
SendInput "{F1}"
F1::MsgBox("F1 triggered")

; This does NOT trigger hotkey (keyboard hook implementation)
SendInput "{F1}"
*F1::MsgBox("F1 triggered")

; This DOES trigger hotkey (SendEvent with SendLevel)
SendLevel 1
SendEvent "{F1}"
*F1::MsgBox("F1 triggered")

; Example 3: Hotstring behavior with different send modes
SendLevel 1
Send "abc "        ; Won't trigger hotstring with SendInput
::abc::123

; Uncomment to make it work:
; SendMode "Event"

; Example 4: Checking for external hooks (v2.1+)
if (A_KeybdHookInstalled > 1) {
    MsgBox("Another AHK script has keyboard hook installed")
    MsgBox("SendInput will revert to SendEvent")
}

; Example 5: Custom SendInput with LLKHF_INJECTED flag removal
InstallKeybdHook(1, 1)
KeyArray := [{sc: GetKeySC("a"), event: "Down"}]
SendInputEx(KeyArray)
MsgBox "Logical state: " GetKeyState("a") "`nPhysical state: " GetKeyState("a", "P")

SendInputEx(KeyArray) {
   static INPUT_KEYBOARD := 1, KEYEVENTF_KEYUP := 2, KEYEVENTF_SCANCODE := 8, InputSize := 16 + A_PtrSize*3
   INPUTS := Buffer(InputSize * KeyArray.Length, 0)
   offset := 0
   for k, v in KeyArray {
    NumPut("int", INPUT_KEYBOARD, "int", 0, "ushort", 0, "ushort", v.sc & 0xFF, "int", (v.event = "Up" ? KEYEVENTF_KEYUP : 0) | KEYEVENTF_SCANCODE | (v.sc >> 8), "int", 0, "int", 0, "int", 0xFFC3D44E, INPUTS, offset)
    offset += InputSize
   }
   DllCall("SendInput", "UInt", KeyArray.Length, "Ptr", INPUTS, "Int", InputSize)
}
```

## Implementation Notes

**When Keyboard Hooks Are Installed:**
- Hotstrings (always require hooks)
- Hotkeys with #HotIf conditions
- Hotkeys with modifiers ~, *, or $
- Custom combinations like a & b
- Hotkeys with Up option (key release detection)
- InputHook function usage
- SetCapsLockState/SetNumLockState/SetScrollLockState with AlwaysOn/AlwaysOff
- #InputLevel > 0
- Manual InstallKeybdHook(1) calls

**SendInput Behavior:**
- Uninstalls AHK's keyboard hook before sending, reinstalls after
- Reverts to SendEvent if other AHK keyboard hooks detected
- Cannot detect non-AHK keyboard hooks, leading to degraded performance
- Cannot trigger hotstrings or hook-based hotkeys in the same script
- Can trigger RegisterHotkey-based hotkeys

**Low Level Keyboard Hook Chain:**
- Windows calls hooks in reverse order of installation (most recent first)
- Each hook can block, allow, or pass through keystrokes
- Hook timeouts are controlled by LowLevelHooksTimeout registry value
- Multiple hooks slow down all keystroke processing

**Performance Considerations:**
- Multiple keyboard hooks slow down all keystroke processing
- Hook callback functions should execute as quickly as possible
- #HotIf conditions should resolve fast to avoid input lag
- Consider consolidating multiple scripts to reduce hook count

**Common Pitfalls:**
- Unexpected key leaking during rapid input with SendInput
- SendInput silently reverting to SendEvent without notification
- GetKeyState "physical" mode becoming unreliable when hooks are removed
- User input interspersing with SendInput when external hooks present
- Assuming SendInput is always faster - often SendEvent with proper delays is more reliable

**Keystroke Journey:**
1. Physical key press → kernel driver stack
2. Low-level keyboard hooks (WH_KEYBOARD_LL)
3. RegisterHotkey matching & WM_HOTKEY posting
4. Raw Input handling (WM_INPUT)
5. Standard keyboard message posting (WM_KEYDOWN/WM_KEYUP)
6. Application message loop & character translation
7. Dispatch to window procedure

## Related AHK Concepts

- SendLevel and #InputLevel for controlling hook triggering
- RegisterHotkey vs keyboard hook implementations
- GetKeyState logical vs physical states
- InputHook for custom input processing
- SetKeyDelay for controlling SendEvent timing
- Low-level vs standard keyboard hooks
- Windows keystroke processing pipeline
- ControlSend bypassing hooks and raw input
- AutoHotInterception for multi-keyboard handling

## Tags

#AutoHotkey #SendInput #KeyboardHooks #Performance #InputHandling #SendEvent #Hotkeys #Hotstrings #LowLevel #Windows