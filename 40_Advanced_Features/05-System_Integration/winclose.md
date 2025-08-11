# Topic: WinClose Function - Window Termination and Application Control

## Category

Function

## Overview

The WinClose function closes windows by sending a close message to the target application, allowing for graceful window termination. It's essential for automation scripts, application management, system cleanup, and any scenario requiring controlled closure of windows or applications.

## Key Points

- Sends WM_CLOSE message for graceful window termination allowing applications to save data
- Supports multiple window targeting methods including title, class, process, and handle
- Provides optional timeout control for responsive vs unresponsive application handling
- Enables batch window closure and automated application management workflows
- Essential for system cleanup, automation scripting, and application lifecycle management

## Syntax and Parameters

```cpp
WinClose([WinTitle, WinText, SecondsToWait, ExcludeTitle, ExcludeText])

; Parameters:
; WinTitle      - Window title or criteria (optional, defaults to active window)
; WinText       - Text contained in window (optional)
; SecondsToWait - Time to wait for window to close (optional, default: 0)
; ExcludeTitle  - Windows to exclude by title (optional)
; ExcludeText   - Windows to exclude by text (optional)

; WinTitle formats:
; "Window Title"           - Exact or partial title match
; "ahk_class ClassName"    - Window class name
; "ahk_exe ProcessName"    - Executable file name
; "ahk_pid ProcessID"      - Process ID
; "ahk_id WindowID"        - Window handle (HWND)

; Return Value:
; None (function completes when window closes or timeout occurs)
```

## Code Examples

```cpp
; Basic window closing
WinClose("Notepad")

; Close with timeout
WinClose("Calculator", , 5)  ; Wait up to 5 seconds

; Close by executable name
WinClose("ahk_exe chrome.exe")

; Close specific window by handle
windowId := WinExist("Untitled - Notepad")
if (windowId) {
    WinClose("ahk_id " . windowId)
}

; Close with exclusions
WinClose("Document", , , "Print Preview")  ; Close documents but not print preview

; Application lifecycle manager
class ApplicationManager {
    static managedApps := Map()
    static closeTimeout := 10
    static forceCloseDelay := 5
    
    static RegisterApplication(name, executable, windowTitle := "") {
        this.managedApps[name] := {
            exe: executable,
            title: windowTitle,
            isRunning: false,
            lastHandle: 0,
            startTime: 0
        }
    }
    
    static LaunchApplication(name, workingDir := "", windowState := "") {
        if (!this.managedApps.Has(name)) {
            throw ValueError("Application '" . name . "' not registered")
        }
        
        app := this.managedApps[name]
        
        ; Check if already running
        if (this.IsApplicationRunning(name)) {
            return app.lastHandle
        }
        
        try {
            ; Launch application
            if (workingDir && windowState) {
                Run(app.exe, workingDir, windowState, &pid)
            } else if (workingDir) {
                Run(app.exe, workingDir, , &pid)
            } else {
                Run(app.exe, , , &pid)
            }
            
            ; Wait for window to appear
            targetWindow := app.title ? app.title : "ahk_exe " . app.exe
            WinWait(targetWindow, , 10)
            
            app.lastHandle := WinExist(targetWindow)
            app.isRunning := true
            app.startTime := A_TickCount
            
            return app.lastHandle
            
        } catch Error as err {
            throw OSError("Failed to launch " . name . ": " . err.Message)
        }
    }
    
    static CloseApplication(name, graceful := true) {
        if (!this.managedApps.Has(name)) {
            throw ValueError("Application '" . name . "' not registered")
        }
        
        app := this.managedApps[name]
        
        if (!this.IsApplicationRunning(name)) {
            return true  ; Already closed
        }
        
        targetWindow := app.title ? app.title : "ahk_exe " . app.exe
        
        if (graceful) {
            return this.GracefulClose(targetWindow, app)
        } else {
            return this.ForceClose(targetWindow, app)
        }
    }
    
    static GracefulClose(targetWindow, app) {
        try {
            ; Attempt graceful close
            WinClose(targetWindow, , this.closeTimeout)
            
            ; Verify window closed
            Sleep(500)  ; Give time for close to complete
            
            if (!WinExist(targetWindow)) {
                app.isRunning := false
                app.lastHandle := 0
                return true
            }
            
            ; If still exists, try force close
            OutputDebug("Graceful close failed, attempting force close")
            return this.ForceClose(targetWindow, app)
            
        } catch Error as err {
            OutputDebug("Error during graceful close: " . err.Message)
            return this.ForceClose(targetWindow, app)
        }
    }
    
    static ForceClose(targetWindow, app) {
        try {
            ; Get process ID and terminate
            pid := WinGetPID(targetWindow)
            ProcessClose(pid)
            
            ; Wait for process to terminate
            ProcessWait(pid, this.forceCloseDelay)
            
            app.isRunning := false
            app.lastHandle := 0
            return true
            
        } catch Error as err {
            OutputDebug("Force close failed: " . err.Message)
            return false
        }
    }
    
    static IsApplicationRunning(name) {
        if (!this.managedApps.Has(name)) {
            return false
        }
        
        app := this.managedApps[name]
        targetWindow := app.title ? app.title : "ahk_exe " . app.exe
        
        if (WinExist(targetWindow)) {
            app.isRunning := true
            app.lastHandle := WinExist(targetWindow)
            return true
        } else {
            app.isRunning := false
            app.lastHandle := 0
            return false
        }
    }
    
    static CloseAllManagedApps() {
        results := Map()
        
        for name, app in this.managedApps {
            if (this.IsApplicationRunning(name)) {
                results[name] := this.CloseApplication(name)
            } else {
                results[name] := true  ; Already closed
            }
        }
        
        return results
    }
    
    static GetApplicationStatus() {
        status := Map()
        
        for name, app in this.managedApps {
            isRunning := this.IsApplicationRunning(name)
            uptime := isRunning && app.startTime ? A_TickCount - app.startTime : 0
            
            status[name] := {
                running: isRunning,
                handle: app.lastHandle,
                uptime: uptime,
                executable: app.exe
            }
        }
        
        return status
    }
}

; Session cleanup manager
class SessionManager {
    static sessionApps := []
    static cleanupOnExit := true
    static cleanupTimeout := 30
    
    static Initialize() {
        ; Register cleanup on script exit
        OnExit(this.PerformCleanup.Bind(this))
        
        ; Track commonly used applications
        this.sessionApps := [
            "notepad.exe",
            "calc.exe",
            "mspaint.exe",
            "explorer.exe"
        ]
    }
    
    static AddSessionApp(executable) {
        if (this.sessionApps.IndexOf(executable) = 0) {
            this.sessionApps.Push(executable)
        }
    }
    
    static CleanupSession(appList := "") {
        if (!appList) {
            appList := this.sessionApps
        }
        
        cleanupResults := Map()
        
        for app in appList {
            try {
                windows := this.FindWindowsByExecutable(app)
                cleanupResults[app] := this.CloseWindowList(windows)
            } catch Error as err {
                cleanupResults[app] := {success: false, error: err.Message}
            }
        }
        
        return cleanupResults
    }
    
    static FindWindowsByExecutable(executable) {
        windows := []
        
        ; Get all windows for this executable
        try {
            ; Use window enumeration to find all windows
            DetectHiddenWindows(true)
            
            ; Get list of all windows
            allWindows := WinGetList()
            
            for hwnd in allWindows {
                try {
                    winExe := WinGetProcessName("ahk_id " . hwnd)
                    if (StrLower(winExe) = StrLower(executable)) {
                        windows.Push({
                            handle: hwnd,
                            title: WinGetTitle("ahk_id " . hwnd),
                            pid: WinGetPID("ahk_id " . hwnd)
                        })
                    }
                } catch {
                    ; Skip windows that can't be queried
                    continue
                }
            }
            
            DetectHiddenWindows(false)
            
        } catch Error as err {
            throw OSError("Failed to enumerate windows: " . err.Message)
        }
        
        return windows
    }
    
    static CloseWindowList(windows) {
        results := {
            attempted: windows.Length,
            successful: 0,
            failed: [],
            forced: 0
        }
        
        for window in windows {
            try {
                ; Attempt graceful close
                WinClose("ahk_id " . window.handle, , 5)
                
                ; Check if closed
                Sleep(1000)
                if (!WinExist("ahk_id " . window.handle)) {
                    results.successful++
                } else {
                    ; Force close if still exists
                    try {
                        ProcessClose(window.pid)
                        results.forced++
                    } catch Error as forceErr {
                        results.failed.Push({
                            window: window,
                            error: forceErr.Message
                        })
                    }
                }
                
            } catch Error as err {
                results.failed.Push({
                    window: window,
                    error: err.Message
                })
            }
        }
        
        return results
    }
    
    static PerformCleanup(exitReason, exitCode) {
        if (!this.cleanupOnExit) return
        
        OutputDebug("Performing session cleanup...")
        
        try {
            results := this.CleanupSession()
            
            totalClosed := 0
            for app, result in results {
                if (Type(result) = "Object" && result.HasProp("successful")) {
                    totalClosed += result.successful + result.forced
                }
            }
            
            OutputDebug("Session cleanup completed. Closed " . totalClosed . " windows.")
            
        } catch Error as err {
            OutputDebug("Session cleanup error: " . err.Message)
        }
    }
    
    static SetCleanupTimeout(seconds) {
        this.cleanupTimeout := seconds
    }
    
    static EnableCleanupOnExit() {
        this.cleanupOnExit := true
    }
    
    static DisableCleanupOnExit() {
        this.cleanupOnExit := false
    }
}

; Window batch operations
class WindowBatchOps {
    static CloseWindowsByPattern(titlePattern, timeout := 5) {
        windows := []
        results := {
            found: 0,
            closed: 0,
            failed: []
        }
        
        ; Find all matching windows
        allWindows := WinGetList()
        
        for hwnd in allWindows {
            try {
                title := WinGetTitle("ahk_id " . hwnd)
                
                if (RegExMatch(title, titlePattern)) {
                    windows.Push({
                        handle: hwnd,
                        title: title
                    })
                }
            } catch {
                continue
            }
        }
        
        results.found := windows.Length
        
        ; Close each matching window
        for window in windows {
            try {
                WinClose("ahk_id " . window.handle, , timeout)
                
                ; Verify closure
                Sleep(500)
                if (!WinExist("ahk_id " . window.handle)) {
                    results.closed++
                } else {
                    results.failed.Push({
                        handle: window.handle,
                        title: window.title,
                        reason: "Failed to close within timeout"
                    })
                }
                
            } catch Error as err {
                results.failed.Push({
                    handle: window.handle,
                    title: window.title,
                    reason: err.Message
                })
            }
        }
        
        return results
    }
    
    static CloseAllExcept(keepList) {
        allWindows := WinGetList()
        closed := 0
        
        for hwnd in allWindows {
            try {
                title := WinGetTitle("ahk_id " . hwnd)
                exe := WinGetProcessName("ahk_id " . hwnd)
                
                ; Check if window should be kept
                shouldKeep := false
                for keepPattern in keepList {
                    if (RegExMatch(title, keepPattern) || RegExMatch(exe, keepPattern)) {
                        shouldKeep := true
                        break
                    }
                }
                
                if (!shouldKeep) {
                    ; Check if it's a system window
                    if (!this.IsSystemWindow(hwnd)) {
                        WinClose("ahk_id " . hwnd, , 3)
                        closed++
                    }
                }
                
            } catch {
                continue
            }
        }
        
        return closed
    }
    
    static IsSystemWindow(hwnd) {
        try {
            ; Get window style
            style := WinGetStyle("ahk_id " . hwnd)
            exStyle := WinGetExStyle("ahk_id " . hwnd)
            
            ; Check for system window characteristics
            if (exStyle & 0x80) {  ; WS_EX_TOOLWINDOW
                return true
            }
            
            ; Check process name for system processes
            processName := WinGetProcessName("ahk_id " . hwnd)
            systemProcesses := ["dwm.exe", "winlogon.exe", "csrss.exe", "smss.exe", "explorer.exe"]
            
            for sysProc in systemProcesses {
                if (StrLower(processName) = StrLower(sysProc)) {
                    return true
                }
            }
            
            return false
            
        } catch {
            return true  ; Assume system window if can't determine
        }
    }
}

; Smart window closing with data protection
class SmartWindowCloser {
    static unsavedPatterns := ["*", "Untitled", "New Document", "unnamed"]
    static savePromptTimeout := 30
    
    static CloseWithDataProtection(winTitle, allowSavePrompt := true) {
        if (!WinExist(winTitle)) {
            return {success: true, reason: "Window not found"}
        }
        
        windowInfo := {
            handle: WinExist(winTitle),
            title: WinGetTitle(winTitle),
            process: WinGetProcessName(winTitle)
        }
        
        ; Check if window might have unsaved data
        hasUnsavedData := this.CheckForUnsavedData(windowInfo.title)
        
        if (hasUnsavedData && allowSavePrompt) {
            return this.CloseWithSavePrompt(winTitle, windowInfo)
        } else {
            return this.DirectClose(winTitle, windowInfo)
        }
    }
    
    static CheckForUnsavedData(title) {
        for pattern in this.unsavedPatterns {
            if (InStr(title, pattern)) {
                return true
            }
        }
        return false
    }
    
    static CloseWithSavePrompt(winTitle, windowInfo) {
        ; Send close request
        WinClose(winTitle, , 1)  ; Short timeout for initial close
        
        ; Check if save dialog appeared
        saveDialogs := [
            "Save As",
            "Save Document",
            "Do you want to save",
            "Save changes"
        ]
        
        dialogFound := false
        for dialogPattern in saveDialogs {
            if (WinWait(dialogPattern, , 2)) {
                dialogFound := true
                break
            }
        }
        
        if (dialogFound) {
            return this.HandleSaveDialog()
        } else {
            ; Window closed without save prompt
            return {
                success: true,
                reason: "Closed without save prompt",
                windowInfo: windowInfo
            }
        }
    }
    
    static HandleSaveDialog() {
        ; Present options to user
        response := MsgBox("A save dialog has appeared. How would you like to proceed?`n`n" .
                          "Yes: Save and close`n" .
                          "No: Close without saving`n" .
                          "Cancel: Cancel close operation", 
                          "Save Prompt Detected", "YesNoCancel")
        
        switch response {
            case "Yes":
                ; Send save command (typically Enter or Alt+S)
                Send("{Enter}")
                WinWaitClose(A_LastFoundWindow, , this.savePromptTimeout)
                return {success: true, reason: "Saved and closed", action: "saved"}
                
            case "No":
                ; Send don't save command (typically Alt+N or similar)
                Send("{Alt down}n{Alt up}")
                WinWaitClose(A_LastFoundWindow, , 5)
                return {success: true, reason: "Closed without saving", action: "discarded"}
                
            case "Cancel":
                ; Cancel the dialog
                Send("{Escape}")
                return {success: false, reason: "User cancelled close operation", action: "cancelled"}
        }
    }
    
    static DirectClose(winTitle, windowInfo) {
        try {
            WinClose(winTitle, , 10)
            
            ; Verify closure
            Sleep(1000)
            if (!WinExist("ahk_id " . windowInfo.handle)) {
                return {
                    success: true,
                    reason: "Closed successfully",
                    windowInfo: windowInfo
                }
            } else {
                return {
                    success: false,
                    reason: "Failed to close within timeout",
                    windowInfo: windowInfo
                }
            }
            
        } catch Error as err {
            return {
                success: false,
                reason: "Error during close: " . err.Message,
                windowInfo: windowInfo
            }
        }
    }
}

; Example usage
; Register applications for management
ApplicationManager.RegisterApplication("TextEditor", "notepad.exe", "Notepad")
ApplicationManager.RegisterApplication("Calculator", "calc.exe", "Calculator")

; Launch and later close applications
ApplicationManager.LaunchApplication("TextEditor")
Sleep(5000)  ; Do some work
ApplicationManager.CloseApplication("TextEditor")

; Initialize session cleanup
SessionManager.Initialize()
SessionManager.AddSessionApp("chrome.exe")

; Batch close operations
result := WindowBatchOps.CloseWindowsByPattern(".*Document.*")
MsgBox("Closed " . result.closed . " of " . result.found . " document windows")

; Smart close with data protection
result := SmartWindowCloser.CloseWithDataProtection("Untitled - Notepad")
if (result.success) {
    MsgBox("Window closed: " . result.reason)
}
```

## Implementation Notes

**Graceful vs Forceful Closing:**
- WinClose sends WM_CLOSE message allowing applications to save data and cleanup
- Applications can prompt user for save confirmation or deny close request
- Use ProcessClose() for forceful termination when graceful close fails
- Always prefer graceful close to prevent data loss

**Timeout Behavior:**
- SecondsToWait parameter determines how long to wait for window closure
- Zero timeout returns immediately without waiting for closure confirmation
- Non-zero timeout blocks execution until window closes or timeout expires
- Use appropriate timeouts based on application response characteristics

**Window Targeting:**
- Multiple targeting methods provide flexibility for different scenarios
- Process-based targeting (ahk_exe) more reliable than title-based
- Handle-based targeting (ahk_id) most precise for specific windows
- Class-based targeting useful for windows with dynamic titles

**Error Handling:**
- WinClose does not throw exceptions but may not actually close window
- Always verify closure using WinExist() after WinClose operation
- Implement fallback mechanisms for unresponsive applications
- Consider user intervention for applications requiring manual save

**Application Behavior Variations:**
- Some applications ignore WM_CLOSE message (security software, system tools)
- Document editors often show save prompts before closing
- Games and full-screen applications may require special handling
- System windows should generally not be closed to maintain stability

## Related AHK Concepts

- [WinExist](./winexist.md) - Window detection and validation before closing
- [WinWait](./winwait.md) - Waiting for windows to appear or disappear
- [ProcessClose](../../30_Built_In_Classes/03-System_Classes/Process/processclose.md) - Forceful process termination
- [WinActivate](./winactivate.md) - Window activation before closing
- [OnExit](../../40_Advanced_Features/05-System_Integration/onexit.md) - Script cleanup and window management

## Tags

#AutoHotkey #WinClose #WindowManagement #ApplicationControl #Cleanup #SessionManagement #GracefulShutdown #Automation