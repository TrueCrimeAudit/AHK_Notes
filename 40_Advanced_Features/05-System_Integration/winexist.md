# Topic: WinExist Function - Window Detection and Validation

## Category

Function

## Overview

The WinExist function checks for the existence of windows matching specified criteria and returns the unique window ID (HWND) if found. It's essential for window management, automation scripts, and applications that need to interact with specific windows or verify their presence before performing operations.

## Key Points

- Detects windows using title, class, process, or custom criteria for precise targeting
- Returns unique window ID (HWND) for successful matches, 0 if no window found
- Supports partial matching, regular expressions, and multiple search criteria
- Provides foundation for window automation and inter-application communication
- Essential for robust automation scripts that depend on specific application states

## Syntax and Parameters

```cpp
HWND := WinExist([WinTitle, WinText, ExcludeTitle, ExcludeText])

; Parameters:
; WinTitle     - Window title or criteria (optional, defaults to active window)
; WinText      - Text contained in window (optional)
; ExcludeTitle - Windows to exclude by title (optional)
; ExcludeText  - Windows to exclude by text (optional)

; WinTitle formats:
; "Window Title"           - Exact or partial title match
; "ahk_class ClassName"    - Window class name
; "ahk_exe ProcessName"    - Executable file name
; "ahk_pid ProcessID"      - Process ID
; "ahk_id WindowID"        - Window handle (HWND)

; Return Value:
; Returns HWND (window handle) if window exists, 0 if not found
```

## Code Examples

```cpp
; Basic window existence check
if (WinExist("Notepad")) {
    MsgBox("Notepad is running")
} else {
    MsgBox("Notepad is not found")
}

; Check by executable name
if (WinExist("ahk_exe chrome.exe")) {
    MsgBox("Chrome browser is open")
}

; Check by window class
calcWindow := WinExist("ahk_class CalcFrame")
if (calcWindow) {
    MsgBox("Calculator found with handle: " . calcWindow)
}

; Multiple criteria matching
targetWindow := WinExist("Document", "Microsoft Word", "Print Preview")
if (targetWindow) {
    MsgBox("Found Word document, excluding print preview")
}

; Wait for window with timeout
function WaitForWindow(title, timeout := 30) {
    startTime := A_TickCount
    
    while (A_TickCount - startTime < timeout * 1000) {
        if (WinExist(title)) {
            return WinExist(title)  ; Return window handle
        }
        Sleep(100)  ; Check every 100ms
    }
    
    return 0  ; Timeout reached
}

; Window monitoring and management
class WindowMonitor {
    watchedWindows := Map()
    monitorTimer := 0
    
    __New() {
        ; Start monitoring timer
        this.monitorTimer := SetTimer(this.CheckWindows.Bind(this), 1000)
    }
    
    AddWindow(title, callback) {
        this.watchedWindows[title] := {
            callback: callback,
            lastSeen: 0,
            wasPresent: false
        }
    }
    
    RemoveWindow(title) {
        this.watchedWindows.Delete(title)
    }
    
    CheckWindows() {
        currentTime := A_TickCount
        
        for title, info in this.watchedWindows {
            windowExists := WinExist(title) != 0
            
            ; Detect state changes
            if (windowExists && !info.wasPresent) {
                ; Window appeared
                info.callback.Call("appeared", title, WinExist(title))
                info.wasPresent := true
                info.lastSeen := currentTime
                
            } else if (!windowExists && info.wasPresent) {
                ; Window disappeared
                info.callback.Call("disappeared", title, 0)
                info.wasPresent := false
                
            } else if (windowExists) {
                ; Window still present
                info.lastSeen := currentTime
            }
        }
    }
    
    Stop() {
        if (this.monitorTimer) {
            SetTimer(this.monitorTimer, 0)
            this.monitorTimer := 0
        }
    }
}

; Application state management
class ApplicationManager {
    requiredApps := Map()
    
    AddRequirement(appName, executable, title := "") {
        this.requiredApps[appName] := {
            exe: executable,
            title: title,
            isRunning: false,
            handle: 0
        }
    }
    
    CheckAllRequirements() {
        allRunning := true
        status := Map()
        
        for appName, info in this.requiredApps {
            ; Check by executable first
            handle := WinExist("ahk_exe " . info.exe)
            
            ; If specific title required, check that too
            if (handle && info.title) {
                handle := WinExist(info.title . " ahk_exe " . info.exe)
            }
            
            info.isRunning := handle != 0
            info.handle := handle
            status[appName] := info.isRunning
            
            if (!info.isRunning) {
                allRunning := false
            }
        }
        
        return {allRunning: allRunning, status: status}
    }
    
    LaunchMissing() {
        for appName, info in this.requiredApps {
            if (!info.isRunning) {
                try {
                    Run(info.exe)
                    ; Wait for application to start
                    if (WaitForWindow("ahk_exe " . info.exe, 10)) {
                        info.isRunning := true
                        info.handle := WinExist("ahk_exe " . info.exe)
                    }
                } catch Error as err {
                    MsgBox("Failed to launch " . appName . ": " . err.Message)
                }
            }
        }
    }
}

; Window validation and safety checks
function SafeWindowOperation(winTitle, operation) {
    ; Validate window exists before operation
    windowHandle := WinExist(winTitle)
    if (!windowHandle) {
        throw Error("Window not found: " . winTitle)
    }
    
    ; Store original active window for restoration
    originalWindow := WinExist("A")
    
    try {
        ; Perform operation with error handling
        result := operation.Call(windowHandle)
        return result
        
    } catch Error as err {
        ; Log error and restore state
        Logger.Error("Window operation failed: " . err.Message)
        throw err
        
    } finally {
        ; Restore original active window if it still exists
        if (originalWindow && WinExist("ahk_id " . originalWindow)) {
            WinActivate("ahk_id " . originalWindow)
        }
    }
}

; Advanced window search with regex
function FindWindowByPattern(titlePattern, classPattern := "") {
    ; Get list of all windows
    windows := WinGetList()
    matches := []
    
    for index, hwnd in windows {
        try {
            title := WinGetTitle("ahk_id " . hwnd)
            className := WinGetClass("ahk_id " . hwnd)
            
            ; Check title pattern
            titleMatch := titlePattern ? RegExMatch(title, titlePattern) : true
            
            ; Check class pattern if specified
            classMatch := classPattern ? RegExMatch(className, classPattern) : true
            
            if (titleMatch && classMatch) {
                matches.Push({
                    handle: hwnd,
                    title: title,
                    class: className
                })
            }
            
        } catch Error {
            ; Skip windows that can't be queried
            continue
        }
    }
    
    return matches
}

; Process-based window management
function GetWindowsByProcess(processName) {
    windows := []
    
    ; Get all windows for the process
    try {
        loop {
            hwnd := WinExist("ahk_exe " . processName)
            if (!hwnd) break
            
            ; Add to collection
            windows.Push({
                handle: hwnd,
                title: WinGetTitle("ahk_id " . hwnd),
                pid: WinGetPID("ahk_id " . hwnd)
            })
            
            ; Move to next window by excluding current one
            WinHide("ahk_id " . hwnd)
        }
        
        ; Restore all windows
        for index, window in windows {
            WinShow("ahk_id " . window.handle)
        }
        
    } catch Error as err {
        ; Ensure windows are restored even on error
        for index, window in windows {
            try {
                WinShow("ahk_id " . window.handle)
            } catch {
                ; Ignore restoration errors
            }
        }
        throw err
    }
    
    return windows
}

; Example usage
monitor := WindowMonitor()

; Monitor for application events
monitor.AddWindow("Notepad", (event, title, handle) => {
    if (event = "appeared") {
        MsgBox("Notepad opened with handle: " . handle)
    } else if (event = "disappeared") {
        MsgBox("Notepad was closed")
    }
})

; Application dependency management
appManager := ApplicationManager()
appManager.AddRequirement("Browser", "chrome.exe")
appManager.AddRequirement("Editor", "notepad.exe")

status := appManager.CheckAllRequirements()
if (!status.allRunning) {
    appManager.LaunchMissing()
}
```

## Implementation Notes

**Window Matching Behavior:**
- Partial title matching is default behavior (contains, not exact match)
- Case-insensitive matching for titles and window text
- Multiple criteria are combined with AND logic
- Last found window becomes the default for subsequent operations

**Handle Management:**
- Returned HWND values are unique system identifiers
- Handles remain valid until window is destroyed
- Can be used with other Win functions for consistent targeting
- Store handles for repeated operations to improve performance

**Search Performance:**
- Simple title searches are fastest
- Process-based searches are more reliable than title-based
- Class-based searches are most specific but require knowledge of internal window structure
- Avoid frequent polling; use event-driven approaches when possible

**Error Handling:**
- Function returns 0 (falsy) when no window is found
- Does not throw exceptions for missing windows
- Hidden or minimized windows are still detected
- System windows and invisible windows may appear in searches

**Platform Considerations:**
- Window enumeration behavior varies slightly between Windows versions
- DPI scaling affects window positioning but not detection
- Security software may hide certain windows from detection
- Virtual desktop behavior may affect window visibility

## Related AHK Concepts

- [WinActivate](./winactivate.md) - Activating windows once found
- [WinWait](./winwait.md) - Waiting for windows to appear
- [WinGetTitle](./wingettitle.md) - Retrieving window properties
- [WinGetList](./wingetlist.md) - Enumerating all windows
- [SetTitleMatchMode](../../../40_Advanced_Features/05-System_Integration/settitlematchmode.md) - Configuring match behavior
- [ProcessExist](../../../30_Built_In_Classes/03-System_Classes/Process/processexist.md) - Process-level existence checking

## Tags

#AutoHotkey #WinExist #WindowManagement #Detection #HWND #Automation #ProcessManagement #WindowValidation