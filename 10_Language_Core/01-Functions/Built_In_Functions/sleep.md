# Function: Sleep

## Category

Built-in Function

## Overview

Sleep pauses script execution for a specified amount of time, providing essential timing control for automation scripts. It's fundamental for managing script pacing, waiting for system responses, and coordinating time-sensitive operations in AutoHotkey applications.

## Syntax

```cpp
Sleep(Delay)
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Delay | Integer | Yes | Time to pause in milliseconds. Use -1 to sleep indefinitely until interrupted |

## Return Value

| Type | Description |
|------|-------------|
| None | Sleep does not return a value; it performs the timing delay operation |

## Timing Behavior

### Delay Values
| Delay Value | Behavior | Use Case |
|-------------|----------|----------|
| Positive (1+) | Pause for specified milliseconds | Normal timing delays |
| Zero (0) | Yield CPU briefly to other processes | Allow system processing |
| Negative (-1) | Sleep indefinitely until interrupted | Waiting for external events |

### Interruption Conditions
- Hotkey activation
- Script termination
- System shutdown
- Thread interruption

## Exceptions

| Exception Type | When Thrown |
|----------------|-------------|
| TypeError | When Delay parameter is not a valid integer |
| ValueError | When Delay parameter is invalid (rare) |

## Basic Examples

```cpp
; Basic timing delays
MsgBox("Starting process...")
Sleep(1000)          ; Wait 1 second
MsgBox("Process step 1 complete")

Sleep(500)           ; Wait half a second
MsgBox("Process step 2 complete")

; Quick system yield
Sleep(0)             ; Let other processes run briefly

; File operation timing
FileWrite("data", "temp.txt")
Sleep(100)           ; Brief pause for file system
if (FileExist("temp.txt")) {
    MsgBox("File created successfully")
}

; Application startup timing
Run("notepad.exe")
Sleep(2000)          ; Wait for application to load
WinActivate("Notepad")
Send("Hello World")

; Network operation delays
Send("^o")           ; Open dialog
Sleep(300)           ; Wait for dialog to appear
Send("\\server\file.txt")
Sleep(1000)          ; Wait for network access
Send("{Enter}")
```

## Advanced Examples

```cpp
; Advanced timing manager with adaptive delays
class TimingManager {
    static defaultDelays := Map(
        "ui_response", 250,
        "file_operation", 100,
        "network_timeout", 5000,
        "application_launch", 3000,
        "dialog_appear", 500,
        "key_repeat", 50
    )
    
    static currentProfile := "normal"
    static profiles := Map(
        "fast", 0.5,        ; 50% of normal delays
        "normal", 1.0,      ; Standard delays
        "slow", 2.0,        ; 200% of normal delays  
        "debug", 3.0        ; 300% for debugging
    )
    
    static SetProfile(profileName) {
        if (this.profiles.Has(profileName)) {
            this.currentProfile := profileName
            OutputDebug("Timing profile set to: " . profileName)
            return true
        } else {
            throw ValueError("Unknown timing profile: " . profileName)
        }
    }
    
    static Sleep(delayType, customDelay := 0) {
        local baseDelay := customDelay > 0 ? customDelay : this.GetDelay(delayType)
        local adjustedDelay := Round(baseDelay * this.profiles[this.currentProfile])
        
        ; Minimum delay of 1ms except for profile "fast" with very small delays
        if (adjustedDelay < 1 && this.currentProfile != "fast") {
            adjustedDelay := 1
        }
        
        ; Log timing for debugging
        if (this.currentProfile == "debug") {
            OutputDebug("Sleep: " . delayType . " = " . adjustedDelay . "ms")
        }
        
        if (adjustedDelay > 0) {
            Sleep(adjustedDelay)
        }
        
        return adjustedDelay
    }
    
    static GetDelay(delayType) {
        return this.defaultDelays.Has(delayType) ? this.defaultDelays[delayType] : 250
    }
    
    static SmartSleep(condition, maxWait := 5000, checkInterval := 100) {
        ; Sleep until condition is met or timeout
        local startTime := A_TickCount
        local elapsed := 0
        
        while (elapsed < maxWait) {
            ; Check condition
            try {
                if (condition.Call()) {
                    return {success: true, elapsed: elapsed, reason: "condition_met"}
                }
            } catch Error as err {
                return {success: false, elapsed: elapsed, reason: "condition_error", error: err.Message}
            }
            
            ; Sleep for check interval
            Sleep(checkInterval)
            elapsed := A_TickCount - startTime
        }
        
        return {success: false, elapsed: elapsed, reason: "timeout"}
    }
    
    static ProgressiveSleep(baseDelay, maxDelay, increment, steps) {
        ; Gradually increase delay over multiple steps
        local currentDelay := baseDelay
        
        for step in Range(1, steps) {
            OutputDebug("Progressive sleep step " . step . ": " . currentDelay . "ms")
            Sleep(currentDelay)
            
            currentDelay := Min(currentDelay + increment, maxDelay)
        }
        
        return currentDelay
    }
    
    static InterruptibleSleep(delay, interruptCondition) {
        ; Sleep that can be interrupted by a condition
        local slept := 0
        local sleepChunk := 50  ; Check every 50ms
        
        while (slept < delay) {
            local remainingTime := delay - slept
            local nextSleep := Min(sleepChunk, remainingTime)
            
            Sleep(nextSleep)
            slept += nextSleep
            
            ; Check interrupt condition
            try {
                if (interruptCondition.Call()) {
                    return {interrupted: true, slept: slept, remaining: delay - slept}
                }
            } catch {
                ; Ignore condition errors and continue sleeping
            }
        }
        
        return {interrupted: false, slept: slept, remaining: 0}
    }
}

; Usage examples
TimingManager.SetProfile("slow")  ; Use slower timing for debugging

; Use typed delays
TimingManager.Sleep("ui_response")     ; Wait for UI response
TimingManager.Sleep("file_operation")  ; Wait for file operation

; Smart waiting for conditions
result := TimingManager.SmartSleep(() => WinExist("Calculator"), 3000)
if (result.success) {
    MsgBox("Calculator appeared after " . result.elapsed . "ms")
}

; Progressive delay for retries
TimingManager.ProgressiveSleep(100, 1000, 200, 5)  ; 100, 300, 500, 700, 900ms

; Interruptible sleep
interruptResult := TimingManager.InterruptibleSleep(5000, () => GetKeyState("Escape", "P"))
if (interruptResult.interrupted) {
    MsgBox("Sleep interrupted by Escape key after " . interruptResult.slept . "ms")
}
```

```cpp
; Application automation with intelligent timing
class ApplicationAutomator {
    static launchTimes := Map(
        "notepad.exe", 1000,
        "calc.exe", 800,
        "mspaint.exe", 1500,
        "chrome.exe", 3000,
        "OUTLOOK.EXE", 8000
    )
    
    static LaunchAndWait(executable, windowTitle := "", maxWait := 10000) {
        local startTime := A_TickCount
        local launched := false
        local activated := false
        
        try {
            ; Check if already running
            if (windowTitle != "" && WinExist(windowTitle)) {
                WinActivate(windowTitle)
                if (WinWaitActive(windowTitle, , 3)) {
                    return {
                        success: true,
                        action: "activated_existing",
                        elapsed: A_TickCount - startTime
                    }
                }
            }
            
            ; Launch application
            Run(executable)
            launched := true
            
            ; Wait for window with progressive checking
            local checkDelay := 200
            local totalWaited := 0
            
            while (totalWaited < maxWait) {
                Sleep(checkDelay)
                totalWaited += checkDelay
                
                ; Check if window exists
                if (windowTitle != "" && WinExist(windowTitle)) {
                    ; Try to activate
                    WinActivate(windowTitle)
                    if (WinWaitActive(windowTitle, , 1)) {
                        activated := true
                        break
                    }
                } else if (windowTitle == "") {
                    ; If no specific window title, just wait for estimated launch time
                    local estimatedTime := this.launchTimes.Has(executable) ? this.launchTimes[executable] : 2000
                    if (totalWaited >= estimatedTime) {
                        activated := true
                        break
                    }
                }
                
                ; Progressive delay increase for slow applications
                if (totalWaited > 5000) {
                    checkDelay := 500  ; Check less frequently for slow apps
                }
            }
            
            local result := {
                success: activated,
                action: launched ? "launched" : "failed_to_launch",
                elapsed: A_TickCount - startTime,
                totalWaited: totalWaited
            }
            
            if (!activated && launched) {
                result.action := "launched_but_no_window"
            }
            
            return result
            
        } catch Error as err {
            return {
                success: false,
                action: "error",
                error: err.Message,
                elapsed: A_TickCount - startTime
            }
        }
    }
    
    static PerformActionSequence(actions, pauseBetween := 500) {
        local results := Array()
        local totalTime := 0
        local startTime := A_TickCount
        
        for index, action in actions {
            local actionStart := A_TickCount
            local actionResult := ""
            
            try {
                ; Show progress
                ToolTip("Executing action " . index . " of " . actions.Length . ": " . action.description)
                
                ; Execute action based on type
                switch action.type {
                    case "launch":
                        actionResult := this.LaunchAndWait(action.executable, action.window, action.timeout ?? 10000)
                    case "activate":
                        WinActivate(action.window)
                        actionResult := {success: WinWaitActive(action.window, , 3), action: "activate"}
                    case "send":
                        Send(action.keys)
                        actionResult := {success: true, action: "send"}
                    case "wait":
                        Sleep(action.duration)
                        actionResult := {success: true, action: "wait", duration: action.duration}
                    case "custom":
                        actionResult := action.function.Call()
                    default:
                        throw ValueError("Unknown action type: " . action.type)
                }
                
                actionResult.actionIndex := index
                actionResult.actionTime := A_TickCount - actionStart
                actionResult.description := action.description
                
            } catch Error as err {
                actionResult := {
                    success: false,
                    action: "error",
                    error: err.Message,
                    actionIndex: index,
                    actionTime: A_TickCount - actionStart,
                    description: action.description
                }
            }
            
            results.Push(actionResult)
            
            ; Stop on failure if specified
            if (!actionResult.success && action.hasOwnProp("stopOnFailure") && action.stopOnFailure) {
                ToolTip("Action sequence stopped due to failure at step " . index)
                Sleep(2000)
                break
            }
            
            ; Pause between actions (except last one)
            if (index < actions.Length) {
                local actionPause := action.hasOwnProp("pauseAfter") ? action.pauseAfter : pauseBetween
                if (actionPause > 0) {
                    Sleep(actionPause)
                }
            }
        }
        
        ToolTip()  ; Clear tooltip
        totalTime := A_TickCount - startTime
        
        return {
            success: true,
            results: results,
            totalTime: totalTime,
            completedActions: results.Length
        }
    }
    
    static CreateWorkSession(sessionName, applications) {
        ; Create a predefined work session by launching multiple applications
        local actions := Array()
        
        ; Convert applications to action sequence
        for index, app in applications {
            actions.Push({
                type: "launch",
                executable: app.executable,
                window: app.window,
                description: "Launch " . app.name,
                timeout: app.hasOwnProp("timeout") ? app.timeout : 10000,
                pauseAfter: app.hasOwnProp("pauseAfter") ? app.pauseAfter : 1000
            })
        }
        
        MsgBox("Starting work session: " . sessionName, "Session Manager", "OK T3")
        
        return this.PerformActionSequence(actions, 800)
    }
}

; Usage examples
result := ApplicationAutomator.LaunchAndWait("calc.exe", "Calculator")
if (result.success) {
    MsgBox("Calculator launched in " . result.elapsed . "ms")
}

; Complex action sequence
actionSequence := [
    {type: "launch", executable: "notepad.exe", window: "Notepad", description: "Open Notepad"},
    {type: "wait", duration: 500, description: "Wait for Notepad to stabilize"},
    {type: "send", keys: "Hello World{Enter}", description: "Type greeting"},
    {type: "launch", executable: "calc.exe", window: "Calculator", description: "Open Calculator"},
    {type: "activate", window: "Notepad", description: "Return to Notepad"}
]

sequenceResult := ApplicationAutomator.PerformActionSequence(actionSequence)
MsgBox("Sequence completed: " . sequenceResult.completedActions . " actions in " . sequenceResult.totalTime . "ms")

; Work session example
workApps := [
    {name: "Text Editor", executable: "notepad.exe", window: "Notepad"},
    {name: "Calculator", executable: "calc.exe", window: "Calculator", pauseAfter: 500},
    {name: "File Explorer", executable: "explorer.exe", window: "ahk_class CabinetWClass"}
]

sessionResult := ApplicationAutomator.CreateWorkSession("Development Setup", workApps)
```

## Real-World Example

```cpp
; Network operation timing manager with retry logic
class NetworkOperationManager {
    static defaultTimeouts := Map(
        "connection", 5000,
        "transfer", 30000,
        "response", 10000
    )
    
    static retryDelays := [500, 1000, 2000, 5000, 10000]  ; Progressive retry delays
    
    static PerformNetworkOperation(operation, maxRetries := 3) {
        local attempt := 0
        local lastError := ""
        local startTime := A_TickCount
        
        while (attempt < maxRetries) {
            attempt++
            local attemptStart := A_TickCount
            
            try {
                ; Show operation progress
                ToolTip("Network operation attempt " . attempt . " of " . maxRetries)
                
                ; Perform the operation
                local result := this.ExecuteOperation(operation, attempt)
                
                if (result.success) {
                    ToolTip()
                    return {
                        success: true,
                        result: result.data,
                        attempt: attempt,
                        totalTime: A_TickCount - startTime,
                        attemptTime: A_TickCount - attemptStart
                    }
                } else {
                    lastError := result.error
                    throw Error(result.error)
                }
                
            } catch Error as err {
                lastError := err.Message
                OutputDebug("Network operation attempt " . attempt . " failed: " . err.Message)
                
                ; Don't wait after the last attempt
                if (attempt < maxRetries) {
                    local retryDelay := this.GetRetryDelay(attempt, operation)
                    
                    ToolTip("Retrying in " . Round(retryDelay/1000, 1) . " seconds... (Attempt " . (attempt + 1) . ")")
                    
                    ; Interruptible retry delay
                    local sleepResult := this.InterruptibleRetryDelay(retryDelay)
                    if (sleepResult.interrupted) {
                        ToolTip()
                        return {
                            success: false,
                            error: "Operation cancelled by user",
                            attempt: attempt,
                            totalTime: A_TickCount - startTime
                        }
                    }
                }
            }
        }
        
        ToolTip()
        return {
            success: false,
            error: "Operation failed after " . maxRetries . " attempts. Last error: " . lastError,
            attempt: attempt,
            totalTime: A_TickCount - startTime
        }
    }
    
    static ExecuteOperation(operation, attempt) {
        ; Simulate network operation with realistic timing
        local operationType := operation.type
        local timeout := operation.hasOwnProp("timeout") ? operation.timeout : this.defaultTimeouts[operationType]
        
        ; Simulate operation progress
        local steps := ["Connecting...", "Authenticating...", "Transferring data...", "Completing..."]
        local stepTime := timeout / steps.Length
        
        for index, step in steps {
            ToolTip(step . " (Attempt " . attempt . ")")
            
            ; Simulate variable step timing
            local actualStepTime := stepTime * Random(0.5, 1.5)
            Sleep(actualStepTime)
            
            ; Simulate random failures (more likely on first attempts)
            local failureChance := operation.hasOwnProp("failureRate") ? operation.failureRate : 0.2
            failureChance *= (1.0 / attempt)  ; Less likely to fail on later attempts
            
            if (Random() < failureChance) {
                local errorMessages := [
                    "Connection timeout",
                    "Authentication failed", 
                    "Network unreachable",
                    "Server error",
                    "Transfer interrupted"
                ]
                throw Error(errorMessages[Random(1, errorMessages.Length)])
            }
        }
        
        ; Operation succeeded
        return {
            success: true,
            data: "Operation completed successfully",
            duration: timeout
        }
    }
    
    static GetRetryDelay(attempt, operation) {
        ; Get appropriate retry delay based on attempt number and operation type
        local baseDelay := (attempt <= this.retryDelays.Length) ? 
            this.retryDelays[attempt] : 
            this.retryDelays[this.retryDelays.Length]
        
        ; Add jitter to avoid thundering herd
        local jitter := Random(0.8, 1.2)
        local finalDelay := Round(baseDelay * jitter)
        
        ; Custom delay modifiers based on operation type
        switch operation.type {
            case "connection":
                finalDelay *= 0.5  ; Shorter delays for connection retries
            case "transfer":
                finalDelay *= 1.5  ; Longer delays for transfer retries
        }
        
        return finalDelay
    }
    
    static InterruptibleRetryDelay(delay) {
        ; Allow user to interrupt retry delay
        local elapsed := 0
        local checkInterval := 100
        
        while (elapsed < delay) {
            ; Check for escape key to cancel
            if (GetKeyState("Escape", "P")) {
                return {interrupted: true, elapsed: elapsed}
            }
            
            local sleepTime := Min(checkInterval, delay - elapsed)
            Sleep(sleepTime)
            elapsed += sleepTime
        }
        
        return {interrupted: false, elapsed: elapsed}
    }
    
    static SimulateFileDownload(url, fileSize := 1000000) {
        ; Simulate file download with progress and timing
        local operation := {
            type: "transfer",
            timeout: 15000,
            failureRate: 0.15,
            url: url,
            size: fileSize
        }
        
        return this.PerformNetworkOperation(operation, 4)
    }
    
    static SimulateAPICall(endpoint, timeout := 5000) {
        ; Simulate API call with appropriate timing
        local operation := {
            type: "response",
            timeout: timeout,
            failureRate: 0.1,
            endpoint: endpoint
        }
        
        return this.PerformNetworkOperation(operation, 3)
    }
}

; Usage examples
downloadResult := NetworkOperationManager.SimulateFileDownload("https://example.com/file.zip")
if (downloadResult.success) {
    MsgBox("Download completed in " . Round(downloadResult.totalTime/1000, 1) . " seconds after " . downloadResult.attempt . " attempts")
} else {
    MsgBox("Download failed: " . downloadResult.error)
}

apiResult := NetworkOperationManager.SimulateAPICall("/api/users", 3000)
if (apiResult.success) {
    MsgBox("API call successful")
} else {
    MsgBox("API call failed: " . apiResult.error)
}
```

## Common Use Cases

- **Application Timing**: Pausing between application interactions to allow for response
- **File Operation Delays**: Waiting for file system operations to complete
- **Network Timeouts**: Managing timing for network-dependent operations
- **Animation Pacing**: Controlling the speed of automated sequences
- **System Resource Management**: Yielding CPU time to other processes
- **Retry Logic**: Implementing delays between retry attempts

## Performance Notes

- Sleep accuracy is typically within 10-15ms on modern Windows systems
- Very short sleeps (< 10ms) may be less accurate due to system timer resolution
- Sleep(0) is optimized for CPU yielding without significant delay
- Long sleeps (> 1 hour) should be avoided; use SetTimer for extended delays

## Common Pitfalls

### Pitfall 1: Excessive Sleep Usage
**Problem**: Using fixed delays instead of waiting for actual conditions
**Solution**: Use condition-based waiting when possible
```cpp
; Problematic - fixed delay assumption
Run("notepad.exe")
Sleep(3000)  ; Assumes Notepad takes 3 seconds to load
WinActivate("Notepad")

; Better - wait for actual condition
Run("notepad.exe")
if (WinWait("Notepad", , 10)) {  ; Wait up to 10 seconds
    WinActivate("Notepad")
} else {
    MsgBox("Notepad failed to start")
}
```

### Pitfall 2: Blocking Script with Long Sleeps
**Problem**: Using very long sleeps that make script unresponsive
**Solution**: Use interruptible delays or SetTimer for long delays
```cpp
; Problematic - blocks script for long time
Sleep(60000)  ; 1 minute - script is unresponsive

; Better - use timer for long delays
SetTimer(() => ContinueOperation(), 60000)  ; Non-blocking
return

; Or interruptible sleep
InterruptibleSleep(60000, () => GetKeyState("Escape", "P"))
```

### Pitfall 3: Not Adjusting for System Performance
**Problem**: Using fixed delays that don't account for system differences
**Solution**: Use adaptive delays based on system response
```cpp
; Problematic - same delay for all systems
WinActivate("Application")
Sleep(1000)  ; Fixed delay

; Better - adaptive delay based on actual activation
WinActivate("Application")
startTime := A_TickCount
if (WinWaitActive("Application", , 5)) {
    actualTime := A_TickCount - startTime
    ; Adjust future delays based on system performance
    OptimalDelay := Max(actualTime * 1.2, 500)  ; 20% buffer, min 500ms
}
```

## Version History

- **v2.0**: Improved timing precision and better thread handling
- **v2.0-a**: Enhanced interrupt handling for indefinite sleeps
- **v2.1**: Better integration with system power management

## Related Functions

- [SetTimer](../settimer.md) - Non-blocking timed operations as alternative to Sleep
- [WinWait](../winwait.md) - Condition-based waiting instead of fixed delays
- [WinWaitActive](../winwaitactive.md) - Wait for window activation
- [KeyWait](../../../40_Advanced_Features/00-Hotkeys_and_Input/keywait.md) - Wait for key events

## Related Concepts

- [Timing Patterns](../../../50_Ecosystem/00-Design_Patterns/timing-patterns.md) - Advanced timing control strategies
- [Automation Reliability](../../../50_Ecosystem/01-Best_Practices/automation-reliability.md) - Building reliable automated workflows
- [Performance Optimization](../../../50_Ecosystem/02-Performance_Optimization/timing-optimization.md) - Optimizing script timing

## See Also

- [System Automation Guide](../../../Index/learning-paths.md#system-automator-path) - Complete automation development
- [Timing Control Best Practices](../../../50_Ecosystem/01-Best_Practices/timing-control.md) - Effective timing strategies
- [Application Integration](../../../50_Ecosystem/02-Performance_Optimization/app-integration.md) - Optimizing application automation

## Tags

#AutoHotkey #Function #Sleep #Timing #Delay #SystemControl #Automation #Essential #BuiltIn #Performance #Synchronization