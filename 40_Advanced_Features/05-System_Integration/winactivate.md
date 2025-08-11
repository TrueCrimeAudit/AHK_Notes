# Function: WinActivate

## Category

Built-in Function

## Overview

WinActivate brings a window to the foreground and gives it focus, making it the active window. This function is essential for window management in automation scripts, allowing precise control over which application receives input and user attention.

## Syntax

```cpp
WinActivate([WinTitle, WinText, ExcludeTitle, ExcludeText])
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| WinTitle | String | No | Window title or criteria to identify the target window |
| WinText | String | No | Text contained within the window to help identify it |
| ExcludeTitle | String | No | Windows with this title will be excluded from consideration |
| ExcludeText | String | No | Windows containing this text will be excluded |

## Window Identification Criteria

### WinTitle Options
| Criteria | Example | Description |
|----------|---------|-------------|
| Exact Title | `"Notepad"` | Matches exact window title |
| Partial Title | `"ahk_class Notepad"` | Uses window class name |
| Process Name | `"ahk_exe notepad.exe"` | Matches by executable name |
| Process ID | `"ahk_pid 1234"` | Matches specific process ID |
| Window ID | `"ahk_id 0x12345"` | Matches specific window handle |
| Active Window | `"A"` | Refers to currently active window |

## Return Value

| Type | Description |
|------|-------------|
| None | WinActivate does not return a value; it performs the activation operation |

## Exceptions

| Exception Type | When Thrown |
|----------------|-------------|
| TargetError | When the specified window cannot be found or activated |
| OSError | When system prevents window activation (rare security restrictions) |

## Basic Examples

```cpp
; Activate window by title
WinActivate("Calculator")

; Activate window by executable name
WinActivate("ahk_exe notepad.exe")

; Activate window with partial title matching
WinActivate("Microsoft Word")

; Activate window by class name
WinActivate("ahk_class Chrome_WidgetWin_1")

; Activate with additional text criteria
WinActivate("Notepad", "readme.txt")

; Activate while excluding certain windows
WinActivate("Chrome", , "Private")  ; Exclude private browsing windows
```

## Advanced Examples

```cpp
; Advanced window management with error handling and validation
ActivateWindowSafely(windowCriteria, timeout := 3000) {
    local startTime := A_TickCount
    local activated := false
    
    try {
        ; Check if window exists first
        if (!WinExist(windowCriteria)) {
            throw TargetError("Window not found: " . windowCriteria)
        }
        
        ; Get window details before activation
        local originalWinID := WinGetID("A")  ; Current active window
        local targetWinID := WinGetID(windowCriteria)
        
        ; Attempt activation
        WinActivate(windowCriteria)
        
        ; Wait for activation with timeout
        while (A_TickCount - startTime < timeout) {
            if (WinActive(windowCriteria)) {
                activated := true
                break
            }
            Sleep(50)
        }
        
        if (!activated) {
            throw TimeoutError("Window activation timed out after " . timeout . "ms")
        }
        
        ; Log successful activation
        local windowTitle := WinGetTitle(windowCriteria)
        OutputDebug("Successfully activated: " . windowTitle)
        
        return {
            success: true,
            windowID: targetWinID,
            windowTitle: windowTitle,
            previousWindow: originalWinID,
            activationTime: A_TickCount - startTime
        }
        
    } catch Error as err {
        OutputDebug("Window activation failed: " . err.Message)
        return {
            success: false,
            error: err.Message,
            errorType: Type(err),
            criteria: windowCriteria
        }
    }
}

; Usage examples
result := ActivateWindowSafely("ahk_exe firefox.exe")
if (result.success) {
    MsgBox("Firefox activated in " . result.activationTime . "ms")
} else {
    MsgBox("Failed to activate Firefox: " . result.error)
}
```

```cpp
; Window management utility class
class WindowManager {
    static activeWindows := Map()
    static windowHistory := Array()
    
    static ActivateApplication(appName, createIfNotExists := false) {
        local windowCriteria := this.GetWindowCriteria(appName)
        local result := {success: false, action: "", details: ""}
        
        try {
            ; Check if window exists
            if (WinExist(windowCriteria)) {
                ; Window exists, activate it
                WinActivate(windowCriteria)
                
                ; Wait for activation
                if (WinWaitActive(windowCriteria, , 3)) {
                    result.success := true
                    result.action := "activated"
                    result.details := "Existing window activated"
                    
                    ; Track activation
                    this.TrackWindowActivation(appName, windowCriteria)
                } else {
                    result.action := "timeout"
                    result.details := "Window activation timed out"
                }
            } else if (createIfNotExists) {
                ; Window doesn't exist, try to launch it
                local launchResult := this.LaunchApplication(appName)
                if (launchResult.success) {
                    result.success := true
                    result.action := "launched"
                    result.details := "Application launched and activated"
                } else {
                    result.action := "launch_failed"
                    result.details := launchResult.error
                }
            } else {
                result.action := "not_found"
                result.details := "Window not found and createIfNotExists is false"
            }
            
        } catch Error as err {
            result.action := "error"
            result.details := err.Message
        }
        
        return result
    }
    
    static GetWindowCriteria(appName) {
        ; Define window criteria for common applications
        local criteria := Map()
        criteria["notepad"] := "ahk_exe notepad.exe"
        criteria["calculator"] := "ahk_exe calc.exe"
        criteria["chrome"] := "ahk_exe chrome.exe"
        criteria["firefox"] := "ahk_exe firefox.exe"
        criteria["explorer"] := "ahk_class CabinetWClass"
        criteria["cmd"] := "ahk_exe cmd.exe"
        criteria["powershell"] := "ahk_exe powershell.exe"
        criteria["vscode"] := "ahk_exe Code.exe"
        
        return criteria.Has(appName) ? criteria[appName] : appName
    }
    
    static LaunchApplication(appName) {
        local launchCommands := Map()
        launchCommands["notepad"] := "notepad.exe"
        launchCommands["calculator"] := "calc.exe"
        launchCommands["chrome"] := "chrome.exe"
        launchCommands["firefox"] := "firefox.exe"
        launchCommands["cmd"] := "cmd.exe"
        launchCommands["explorer"] := "explorer.exe"
        
        try {
            if (launchCommands.Has(appName)) {
                Run(launchCommands[appName])
                
                ; Wait for window to appear
                local windowCriteria := this.GetWindowCriteria(appName)
                if (WinWait(windowCriteria, , 10)) {
                    WinActivate(windowCriteria)
                    return {success: true, action: "launched"}
                } else {
                    return {success: false, error: "Application launched but window not detected"}
                }
            } else {
                return {success: false, error: "Unknown application: " . appName}
            }
        } catch Error as err {
            return {success: false, error: "Launch failed: " . err.Message}
        }
    }
    
    static TrackWindowActivation(appName, windowCriteria) {
        local windowID := WinGetID(windowCriteria)
        local windowTitle := WinGetTitle(windowCriteria)
        
        ; Update active windows tracking
        this.activeWindows[appName] := {
            id: windowID,
            title: windowTitle,
            criteria: windowCriteria,
            lastActivated: A_Now
        }
        
        ; Add to history
        this.windowHistory.Push({
            app: appName,
            title: windowTitle,
            timestamp: A_Now,
            action: "activated"
        })
        
        ; Limit history size
        if (this.windowHistory.Length > 50) {
            this.windowHistory.RemoveAt(1)
        }
    }
    
    static GetRecentWindows(count := 5) {
        local recent := Array()
        local historyLength := this.windowHistory.Length
        local startIndex := Max(1, historyLength - count + 1)
        
        for i in Range(historyLength, startIndex, -1) {
            recent.Push(this.windowHistory[i])
        }
        
        return recent
    }
    
    static SwitchToLastWindow() {
        ; Activate the previously active window
        if (this.windowHistory.Length >= 2) {
            local lastWindow := this.windowHistory[this.windowHistory.Length - 1]
            return this.ActivateApplication(lastWindow.app)
        }
        return {success: false, action: "no_history", details: "No previous window in history"}
    }
}

; Usage examples
WindowManager.ActivateApplication("chrome", true)  ; Launch Chrome if not running
WindowManager.ActivateApplication("notepad")       ; Just activate if exists

; Get recent window activity
recentWindows := WindowManager.GetRecentWindows(3)
for index, window in recentWindows {
    MsgBox("Recent: " . window.app . " - " . window.title)
}
```

## Real-World Example

```cpp
; Multi-application workflow automation system
class WorkflowAutomator {
    static workflows := Map()
    static currentWorkflow := ""
    
    static DefineWorkflow(name, steps) {
        ; Define a workflow with window activation steps
        this.workflows[name] := {
            name: name,
            steps: steps,
            lastRun: "",
            runCount: 0
        }
    }
    
    static ExecuteWorkflow(workflowName, pauseBetweenSteps := 1000) {
        if (!this.workflows.Has(workflowName)) {
            throw ValueError("Workflow not found: " . workflowName)
        }
        
        local workflow := this.workflows[workflowName]
        local results := Array()
        local startTime := A_TickCount
        
        this.currentWorkflow := workflowName
        
        try {
            MsgBox("Starting workflow: " . workflow.name, "Workflow Automation", "OK T2")
            
            for stepIndex, step in workflow.steps {
                local stepResult := this.ExecuteWorkflowStep(step, stepIndex)
                results.Push(stepResult)
                
                if (!stepResult.success) {
                    ; Ask user if they want to continue on error
                    local continueMsg := "Step " . stepIndex . " failed: " . stepResult.error
                    continueMsg .= "\n\nContinue with remaining steps?"
                    
                    if (MsgBox(continueMsg, "Workflow Error", "YesNo + Error") != "Yes") {
                        break
                    }
                }
                
                ; Pause between steps
                if (stepIndex < workflow.steps.Length && pauseBetweenSteps > 0) {
                    Sleep(pauseBetweenSteps)
                }
            }
            
            ; Update workflow statistics
            workflow.lastRun := A_Now
            workflow.runCount++
            
            ; Generate workflow report
            local report := this.GenerateWorkflowReport(workflow.name, results, A_TickCount - startTime)
            MsgBox(report, "Workflow Complete", "OK + Information")
            
            return {
                success: true,
                workflow: workflowName,
                steps: results,
                duration: A_TickCount - startTime
            }
            
        } catch Error as err {
            return {
                success: false,
                workflow: workflowName,
                error: err.Message,
                steps: results
            }
        } finally {
            this.currentWorkflow := ""
        }
    }
    
    static ExecuteWorkflowStep(step, stepIndex) {
        try {
            switch step.type {
                case "activate":
                    return this.ExecuteActivateStep(step, stepIndex)
                case "wait":
                    return this.ExecuteWaitStep(step, stepIndex)
                case "send":
                    return this.ExecuteSendStep(step, stepIndex)
                case "custom":
                    return this.ExecuteCustomStep(step, stepIndex)
                default:
                    throw ValueError("Unknown step type: " . step.type)
            }
        } catch Error as err {
            return {
                success: false,
                step: stepIndex,
                type: step.type,
                error: err.Message
            }
        }
    }
    
    static ExecuteActivateStep(step, stepIndex) {
        local windowCriteria := step.window
        local timeout := step.hasOwnProp("timeout") ? step.timeout : 5000
        
        ; Show step progress
        ToolTip("Step " . stepIndex . ": Activating " . windowCriteria)
        
        ; Check if window exists
        if (!WinExist(windowCriteria)) {
            if (step.hasOwnProp("launch") && step.launch) {
                ; Try to launch the application
                Run(step.launch)
                if (!WinWait(windowCriteria, , 10)) {
                    throw TargetError("Window not found and launch failed: " . windowCriteria)
                }
            } else {
                throw TargetError("Window not found: " . windowCriteria)
            }
        }
        
        ; Activate the window
        WinActivate(windowCriteria)
        
        ; Wait for activation
        if (!WinWaitActive(windowCriteria, , timeout/1000)) {
            throw TimeoutError("Window activation timed out: " . windowCriteria)
        }
        
        ; Execute any post-activation commands
        if (step.hasOwnProp("postAction")) {
            Sleep(200)  ; Brief pause after activation
            Send(step.postAction)
        }
        
        ToolTip()  ; Clear tooltip
        
        return {
            success: true,
            step: stepIndex,
            type: "activate",
            window: windowCriteria,
            title: WinGetTitle(windowCriteria)
        }
    }
    
    static ExecuteWaitStep(step, stepIndex) {
        local duration := step.duration
        ToolTip("Step " . stepIndex . ": Waiting " . duration . "ms")
        Sleep(duration)
        ToolTip()
        
        return {
            success: true,
            step: stepIndex,
            type: "wait",
            duration: duration
        }
    }
    
    static GenerateWorkflowReport(workflowName, results, duration) {
        local report := "Workflow: " . workflowName . "\n"
        report .= "Duration: " . Round(duration/1000, 1) . " seconds\n"
        report .= "Steps: " . results.Length . "\n\n"
        
        local successful := 0
        local failed := 0
        
        for index, result in results {
            if (result.success) {
                successful++
                report .= "✓ Step " . index . ": " . result.type
                if (result.hasOwnProp("window")) {
                    report .= " (" . result.window . ")"
                }
                report .= "\n"
            } else {
                failed++
                report .= "✗ Step " . index . ": " . result.type . " - " . result.error . "\n"
            }
        }
        
        report .= "\nSummary: " . successful . " successful, " . failed . " failed"
        return report
    }
    
    static CreateSampleWorkflows() {
        ; Development workflow
        this.DefineWorkflow("development", [
            {type: "activate", window: "ahk_exe Code.exe", launch: "code.exe"},
            {type: "wait", duration: 1000},
            {type: "activate", window: "ahk_exe chrome.exe", launch: "chrome.exe"},
            {type: "activate", window: "ahk_exe cmd.exe", launch: "cmd.exe"},
            {type: "send", keys: "cd C:\\Projects{Enter}"}
        ])
        
        ; Daily work workflow
        this.DefineWorkflow("daily_work", [
            {type: "activate", window: "ahk_exe OUTLOOK.EXE"},
            {type: "activate", window: "ahk_exe chrome.exe"},
            {type: "activate", window: "ahk_exe Teams.exe"},
            {type: "wait", duration: 2000},
            {type: "activate", window: "ahk_class CabinetWClass", launch: "explorer.exe"}
        ])
        
        ; Research workflow
        this.DefineWorkflow("research", [
            {type: "activate", window: "ahk_exe firefox.exe", launch: "firefox.exe"},
            {type: "activate", window: "ahk_exe notepad.exe", launch: "notepad.exe"},
            {type: "activate", window: "Calculator", launch: "calc.exe"}
        ])
    }
}

; Usage examples
WorkflowAutomator.CreateSampleWorkflows()
WorkflowAutomator.ExecuteWorkflow("development", 1500)

; Custom workflow creation
customSteps := [
    {type: "activate", window: "ahk_exe notepad.exe", launch: "notepad.exe"},
    {type: "wait", duration: 500},
    {type: "activate", window: "Calculator", launch: "calc.exe"}
]
WorkflowAutomator.DefineWorkflow("custom_workflow", customSteps)
WorkflowAutomator.ExecuteWorkflow("custom_workflow")
```

## Common Use Cases

- **Application Switching**: Quickly switch between different applications during automation
- **Workflow Automation**: Activate specific windows as part of multi-step processes
- **Focus Management**: Ensure input goes to the correct window before sending keystrokes
- **Window Organization**: Arrange and activate windows in specific sequences
- **Error Recovery**: Reactivate windows that may have lost focus during automation
- **User Interface Control**: Manage window focus in custom applications and scripts

## Performance Notes

- WinActivate is generally fast but may be delayed by system animations or slow applications
- Some applications may resist activation due to security restrictions or modal dialogs
- Multiple rapid activations may cause focus conflicts - add small delays between calls
- Performance varies significantly based on system load and window responsiveness

## Common Pitfalls

### Pitfall 1: Window Not Found
**Problem**: Attempting to activate a window that doesn't exist
**Solution**: Always check if window exists before attempting activation
```cpp
; Problematic - assumes window exists
WinActivate("Calculator")  ; May throw error if Calculator not running

; Better - check existence first
if (WinExist("Calculator")) {
    WinActivate("Calculator")
} else {
    MsgBox("Calculator not found")
}

; Best - comprehensive error handling
try {
    if (WinExist("Calculator")) {
        WinActivate("Calculator")
        WinWaitActive("Calculator", , 3)  ; Wait for activation
    } else {
        Run("calc.exe")  ; Launch if not found
        WinWait("Calculator", , 5)
        WinActivate("Calculator")
    }
} catch Error as err {
    MsgBox("Failed to activate Calculator: " . err.Message)
}
```

### Pitfall 2: Activation Without Verification
**Problem**: Not verifying that window actually became active
**Solution**: Use WinWaitActive to confirm successful activation
```cpp
; Problematic - no verification
WinActivate("Notepad")
Send("Hello World")  ; May send to wrong window if activation failed

; Better - verify activation
WinActivate("Notepad")
if (WinWaitActive("Notepad", , 3)) {
    Send("Hello World")
} else {
    MsgBox("Failed to activate Notepad")
}
```

### Pitfall 3: Imprecise Window Identification
**Problem**: Using ambiguous window titles that may match multiple windows
**Solution**: Use more specific criteria like process name or window class
```cpp
; Problematic - ambiguous title
WinActivate("Document")  ; Could match many different applications

; Better - use process name
WinActivate("ahk_exe WINWORD.EXE")  ; Specifically targets Microsoft Word

; Best - combine criteria for precision
WinActivate("Document", "", "", "ahk_exe notepad.exe")  ; Word document, not Notepad
```

## Version History

- **v2.0**: Enhanced window detection and improved error handling
- **v2.0-a**: Added support for additional window identification criteria
- **v2.1**: Performance improvements for window activation timing

## Related Functions

- [WinWaitActive](../winwaitactive.md) - Wait for window to become active after activation
- [WinExist](../winexist.md) - Check if window exists before attempting activation
- [WinGetTitle](../wingettitle.md) - Get window title for identification and verification
- [Run](../run.md) - Launch applications when windows don't exist

## Related Concepts

- [Window Management](../../../50_Ecosystem/00-Design_Patterns/window-management.md) - Advanced window management patterns
- [Application Automation](../../../50_Ecosystem/00-Design_Patterns/automation-patterns.md) - Integrating window activation in automation workflows
- [Error Handling](../../../00_Fundamentals/05-Error_Basics/error-handling.md) - Managing window activation failures

## See Also

- [System Automation Guide](../../../Index/learning-paths.md#system-automator-path) - Complete automation development
- [Window Control Best Practices](../../../50_Ecosystem/01-Best_Practices/window-control.md) - Effective window management
- [Application Integration](../../../50_Ecosystem/02-Performance_Optimization/app-integration.md) - Optimizing application automation

## Tags

#AutoHotkey #Function #Window #WindowManagement #Activation #Focus #SystemIntegration #Automation #Essential #BuiltIn #ApplicationControl