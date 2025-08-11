# Topic: WinWait Function - Window Synchronization and Timing Control

## Category

Function

## Overview

The WinWait function pauses script execution until a specified window appears, providing essential synchronization capabilities for automation scripts that interact with external applications. It's crucial for coordinating with application launch sequences, dialog handling, process synchronization, and any scenario requiring precise timing coordination with window appearance events.

## Key Points

- Pauses script execution until specified window appears with configurable timeout limits
- Supports flexible window identification using titles, classes, processes, and advanced criteria
- Provides timeout handling for robust error recovery and fallback behavior implementation
- Enables sophisticated window state monitoring with optional exclusion criteria
- Essential for application automation, installation scripts, process coordination, and multi-application workflows

## Syntax and Parameters

```cpp
WinWait(WinTitle [, WinText, Timeout, ExcludeTitle, ExcludeText])

; Parameters:
; WinTitle     - Window title or identifier criteria
; WinText      - Text contained within the window (optional)
; Timeout      - Maximum wait time in seconds (optional, 0 = infinite wait)
; ExcludeTitle - Windows to exclude from matching (optional)
; ExcludeText  - Text in windows to exclude (optional)

; WinTitle formats:
; "Exact Title"           - Exact window title match
; "ahk_class ClassName"   - Window class identification
; "ahk_exe ProcessName"   - Process executable name
; "ahk_pid ProcessID"     - Process ID number
; "ahk_id WindowHandle"   - Window handle (HWND)
; "Partial"               - Partial title matching (default behavior)

; Timeout behavior:
; 0 or omitted - Wait indefinitely until window appears
; Positive value - Wait specified seconds, then continue
; Throws TimeoutError if window doesn't appear within timeout

; Return value:
; Returns window handle (HWND) of found window
; Throws exception if timeout occurs and no window found
```

## Code Examples

```cpp
; Basic window waiting
WinWait("Notepad")  ; Wait for Notepad window

; Wait with timeout
try {
    WinWait("Calculator", , 5)  ; Wait up to 5 seconds
    ; Window appeared
} catch TimeoutError {
    ; Handle timeout
    MsgBox("Calculator didn't appear within 5 seconds")
}

; Wait for specific window class
WinWait("ahk_class Notepad")

; Wait for process-based window
WinWait("ahk_exe chrome.exe")

; Advanced window synchronization system
class WindowSynchronizer {
    static pendingWaits := Map()
    static callbacks := Map()
    static defaultTimeout := 30
    static maxConcurrentWaits := 10
    static monitoringEnabled := false
    
    static Initialize() {
        this.pendingWaits.Clear()
        this.callbacks.Clear()
        this.StartMonitoring()
    }
    
    static WaitForWindow(winTitle, options := {}) {
        ; Parse options
        timeout := options.HasProp("timeout") ? options.timeout : this.defaultTimeout
        winText := options.HasProp("winText") ? options.winText : ""
        excludeTitle := options.HasProp("excludeTitle") ? options.excludeTitle : ""
        excludeText := options.HasProp("excludeText") ? options.excludeText : ""
        callback := options.HasProp("callback") ? options.callback : ""
        async := options.HasProp("async") ? options.async : false
        
        ; Generate unique wait ID
        waitId := this.GenerateWaitId()
        
        ; Store wait information
        waitInfo := {
            winTitle: winTitle,
            winText: winText,
            timeout: timeout,
            excludeTitle: excludeTitle,
            excludeText: excludeText,
            callback: callback,
            startTime: A_TickCount,
            waitId: waitId,
            completed: false,
            result: ""
        }
        
        this.pendingWaits[waitId] := waitInfo
        
        if (async) {
            ; Start asynchronous wait
            SetTimer(() => this.CheckAsyncWait(waitId), 250)
            return waitId
        } else {
            ; Synchronous wait
            return this.ExecuteWait(waitInfo)
        }
    }
    
    static ExecuteWait(waitInfo) {
        try {
            windowHandle := WinWait(waitInfo.winTitle, 
                                   waitInfo.winText, 
                                   waitInfo.timeout, 
                                   waitInfo.excludeTitle, 
                                   waitInfo.excludeText)
            
            waitInfo.completed := true
            waitInfo.result := windowHandle
            
            ; Execute callback if provided
            if (waitInfo.callback && IsFunc(waitInfo.callback)) {
                waitInfo.callback.Call(windowHandle, waitInfo.waitId)
            }
            
            return windowHandle
            
        } catch TimeoutError as err {
            waitInfo.completed := true
            waitInfo.result := "timeout"
            
            if (waitInfo.callback && IsFunc(waitInfo.callback)) {
                waitInfo.callback.Call("", waitInfo.waitId, err)
            }
            
            throw err
        } finally {
            ; Clean up
            this.pendingWaits.Delete(waitInfo.waitId)
        }
    }
    
    static CheckAsyncWait(waitId) {
        if (!this.pendingWaits.Has(waitId)) {
            SetTimer(() => this.CheckAsyncWait(waitId), 0)  ; Stop timer
            return
        }
        
        waitInfo := this.pendingWaits[waitId]
        
        ; Check if window exists
        if (WinExist(waitInfo.winTitle, waitInfo.winText, waitInfo.excludeTitle, waitInfo.excludeText)) {
            ; Window found
            windowHandle := WinExist(waitInfo.winTitle, waitInfo.winText, waitInfo.excludeTitle, waitInfo.excludeText)
            waitInfo.completed := true
            waitInfo.result := windowHandle
            
            if (waitInfo.callback && IsFunc(waitInfo.callback)) {
                waitInfo.callback.Call(windowHandle, waitId)
            }
            
            this.pendingWaits.Delete(waitId)
            SetTimer(() => this.CheckAsyncWait(waitId), 0)  ; Stop timer
            
        } else if (waitInfo.timeout > 0 && A_TickCount - waitInfo.startTime > waitInfo.timeout * 1000) {
            ; Timeout occurred
            waitInfo.completed := true
            waitInfo.result := "timeout"
            
            if (waitInfo.callback && IsFunc(waitInfo.callback)) {
                waitInfo.callback.Call("", waitId, TimeoutError("Window wait timeout"))
            }
            
            this.pendingWaits.Delete(waitId)
            SetTimer(() => this.CheckAsyncWait(waitId), 0)  ; Stop timer
        }
    }
    
    static WaitForMultipleWindows(windowList, mode := "any") {
        ; Wait for multiple windows based on mode
        ; mode: "any" = wait for any window, "all" = wait for all windows
        
        startTime := A_TickCount
        foundWindows := Map()
        
        switch mode {
            case "any":
                ; Wait for any of the specified windows
                while true {
                    for winSpec in windowList {
                        if (WinExist(winSpec.title, winSpec.HasProp("text") ? winSpec.text : "")) {
                            return {
                                window: winSpec,
                                handle: WinExist(winSpec.title, winSpec.HasProp("text") ? winSpec.text : ""),
                                waitTime: A_TickCount - startTime
                            }
                        }
                    }
                    
                    ; Check for timeout if specified
                    if (windowList[1].HasProp("timeout") && windowList[1].timeout > 0) {
                        if (A_TickCount - startTime > windowList[1].timeout * 1000) {
                            throw TimeoutError("None of the specified windows appeared within timeout")
                        }
                    }
                    
                    Sleep(100)
                }
                
            case "all":
                ; Wait for all specified windows
                while foundWindows.Count < windowList.Length {
                    for i, winSpec in windowList {
                        if (!foundWindows.Has(i) && WinExist(winSpec.title, winSpec.HasProp("text") ? winSpec.text : "")) {
                            foundWindows[i] := {
                                window: winSpec,
                                handle: WinExist(winSpec.title, winSpec.HasProp("text") ? winSpec.text : ""),
                                foundAt: A_TickCount - startTime
                            }
                        }
                    }
                    
                    ; Check for timeout
                    if (windowList[1].HasProp("timeout") && windowList[1].timeout > 0) {
                        if (A_TickCount - startTime > windowList[1].timeout * 1000) {
                            throw TimeoutError("Not all windows appeared within timeout")
                        }
                    }
                    
                    Sleep(100)
                }
                
                return foundWindows
        }
    }
    
    static WaitForWindowChain(windowChain, chainTimeout := 0) {
        ; Wait for a sequence of windows to appear in order
        results := []
        totalStartTime := A_TickCount
        
        for i, winSpec in windowChain {
            stepStartTime := A_TickCount
            timeout := winSpec.HasProp("timeout") ? winSpec.timeout : this.defaultTimeout
            
            try {
                windowHandle := WinWait(winSpec.title, 
                                       winSpec.HasProp("text") ? winSpec.text : "",
                                       timeout)
                
                results.Push({
                    step: i,
                    window: winSpec,
                    handle: windowHandle,
                    stepTime: A_TickCount - stepStartTime,
                    totalTime: A_TickCount - totalStartTime
                })
                
                ; Optional step delay
                if (winSpec.HasProp("delay")) {
                    Sleep(winSpec.delay)
                }
                
            } catch TimeoutError {
                throw TimeoutError("Window chain broke at step " . i . ": " . winSpec.title)
            }
            
            ; Check overall chain timeout
            if (chainTimeout > 0 && A_TickCount - totalStartTime > chainTimeout * 1000) {
                throw TimeoutError("Window chain timeout exceeded")
            }
        }
        
        return results
    }
    
    static CancelWait(waitId) {
        if (this.pendingWaits.Has(waitId)) {
            this.pendingWaits.Delete(waitId)
            return true
        }
        return false
    }
    
    static GetWaitStatus(waitId) {
        return this.pendingWaits.Has(waitId) ? this.pendingWaits[waitId] : {}
    }
    
    static GetActiveWaits() {
        return this.pendingWaits.Clone()
    }
    
    static GenerateWaitId() {
        static nextId := 1
        return "wait_" . (nextId++)
    }
    
    static StartMonitoring() {
        if (this.monitoringEnabled) return
        
        this.monitoringEnabled := true
        SetTimer(this.MonitorWaits.Bind(this), 1000)
    }
    
    static MonitorWaits() {
        if (!this.monitoringEnabled) return
        
        ; Clean up completed waits and check for timeouts
        for waitId, waitInfo in this.pendingWaits {
            if (waitInfo.timeout > 0 && A_TickCount - waitInfo.startTime > waitInfo.timeout * 1000) {
                ; Timeout cleanup would be handled by individual async checks
            }
        }
    }
    
    static StopMonitoring() {
        this.monitoringEnabled := false
        SetTimer(this.MonitorWaits.Bind(this), 0)
    }
}

; Application launch coordination system
class AppLauncher {
    static launchQueue := []
    static launchHistory := []
    static retryAttempts := 3
    static retryDelay := 2000
    
    static LaunchAndWait(appPath, windowCriteria, options := {}) {
        ; Launch application and wait for its window
        maxRetries := options.HasProp("retries") ? options.retries : this.retryAttempts
        timeout := options.HasProp("timeout") ? options.timeout : 30
        workingDir := options.HasProp("workingDir") ? options.workingDir : ""
        runOptions := options.HasProp("runOptions") ? options.runOptions : ""
        
        for attempt in Range(1, maxRetries + 1) {
            try {
                ; Launch the application
                if (workingDir) {
                    processId := Run(appPath, workingDir, runOptions)
                } else {
                    processId := Run(appPath, , runOptions)
                }
                
                ; Wait for window to appear
                windowHandle := WinWait(windowCriteria.title, 
                                       windowCriteria.HasProp("text") ? windowCriteria.text : "",
                                       timeout)
                
                ; Record successful launch
                launchRecord := {
                    appPath: appPath,
                    processId: processId,
                    windowHandle: windowHandle,
                    launchTime: A_TickCount,
                    attempts: attempt,
                    success: true
                }
                
                this.launchHistory.Push(launchRecord)
                return launchRecord
                
            } catch Error as err {
                if (attempt < maxRetries) {
                    ; Wait before retry
                    Sleep(this.retryDelay)
                    continue
                } else {
                    ; Final attempt failed
                    launchRecord := {
                        appPath: appPath,
                        processId: 0,
                        windowHandle: 0,
                        launchTime: A_TickCount,
                        attempts: attempt,
                        success: false,
                        error: err.Message
                    }
                    
                    this.launchHistory.Push(launchRecord)
                    throw err
                }
            }
        }
    }
    
    static LaunchSequence(appSequence) {
        ; Launch multiple applications in sequence
        results := []
        
        for i, appSpec in appSequence {
            try {
                result := this.LaunchAndWait(appSpec.path, appSpec.windowCriteria, appSpec)
                results.Push(result)
                
                ; Optional delay between launches
                if (appSpec.HasProp("delay")) {
                    Sleep(appSpec.delay)
                }
                
            } catch Error as err {
                results.Push({
                    appPath: appSpec.path,
                    success: false,
                    error: err.Message,
                    sequencePosition: i
                })
                
                ; Stop sequence on failure unless continueOnFailure is set
                if (!appSpec.HasProp("continueOnFailure") || !appSpec.continueOnFailure) {
                    break
                }
            }
        }
        
        return results
    }
    
    static WaitForAppReady(processId, readinessCriteria) {
        ; Wait for application to be fully ready based on various criteria
        startTime := A_TickCount
        timeout := readinessCriteria.HasProp("timeout") ? readinessCriteria.timeout : 60
        
        ; Wait for main window
        if (readinessCriteria.HasProp("mainWindow")) {
            WinWait(readinessCriteria.mainWindow.title,
                   readinessCriteria.mainWindow.HasProp("text") ? readinessCriteria.mainWindow.text : "",
                   timeout)
        }
        
        ; Wait for specific controls to appear
        if (readinessCriteria.HasProp("controls")) {
            for control in readinessCriteria.controls {
                ControlWait(control.class, control.window, , timeout)
            }
        }
        
        ; Wait for file system activity to settle
        if (readinessCriteria.HasProp("fileActivity")) {
            this.WaitForFileSystemSettle(processId, readinessCriteria.fileActivity)
        }
        
        ; Wait for CPU usage to stabilize
        if (readinessCriteria.HasProp("cpuSettle")) {
            this.WaitForCpuSettle(processId, readinessCriteria.cpuSettle)
        }
        
        return {
            processId: processId,
            readyTime: A_TickCount - startTime,
            allCriteriaMet: true
        }
    }
    
    static WaitForFileSystemSettle(processId, criteria) {
        ; Monitor file system activity for the process
        ; This is a simplified implementation
        settleTime := criteria.HasProp("settleTime") ? criteria.settleTime : 2000
        checkInterval := criteria.HasProp("checkInterval") ? criteria.checkInterval : 500
        
        lastActivityTime := A_TickCount
        
        while A_TickCount - lastActivityTime < settleTime {
            ; In a real implementation, this would monitor actual file system activity
            ; For demonstration, we'll use a simple delay
            Sleep(checkInterval)
            
            ; Check if process still exists
            if (!ProcessExist(processId)) {
                throw Error("Process " . processId . " terminated during file system settle wait")
            }
        }
    }
    
    static WaitForCpuSettle(processId, criteria) {
        ; Wait for CPU usage to stabilize
        maxCpuPercent := criteria.HasProp("maxCpu") ? criteria.maxCpu : 10
        settleTime := criteria.HasProp("settleTime") ? criteria.settleTime : 3000
        checkInterval := criteria.HasProp("checkInterval") ? criteria.checkInterval : 1000
        
        settleStartTime := 0
        
        while true {
            ; Get CPU usage (simplified - real implementation would use performance counters)
            cpuUsage := this.GetProcessCpuUsage(processId)
            
            if (cpuUsage <= maxCpuPercent) {
                if (settleStartTime = 0) {
                    settleStartTime := A_TickCount
                } else if (A_TickCount - settleStartTime >= settleTime) {
                    break  ; CPU has been stable for required time
                }
            } else {
                settleStartTime := 0  ; Reset settle timer
            }
            
            Sleep(checkInterval)
            
            ; Check if process still exists
            if (!ProcessExist(processId)) {
                throw Error("Process " . processId . " terminated during CPU settle wait")
            }
        }
    }
    
    static GetProcessCpuUsage(processId) {
        ; Simplified CPU usage calculation
        ; Real implementation would use performance counters
        return Random(0, 15)  ; Placeholder
    }
    
    static GetLaunchHistory() {
        return this.launchHistory.Clone()
    }
    
    static ClearHistory() {
        this.launchHistory := []
    }
}

; Installation and setup synchronization
class InstallationSynchronizer {
    static installSteps := []
    static currentStep := 0
    static installationTimeout := 300  ; 5 minutes default
    
    static DefineInstallation(steps) {
        this.installSteps := steps
        this.currentStep := 0
    }
    
    static ExecuteInstallation() {
        results := []
        startTime := A_TickCount
        
        for i, step in this.installSteps {
            this.currentStep := i
            stepStartTime := A_TickCount
            
            try {
                result := this.ExecuteInstallStep(step)
                result.stepNumber := i
                result.stepDuration := A_TickCount - stepStartTime
                results.Push(result)
                
                ; Check overall timeout
                if (A_TickCount - startTime > this.installationTimeout * 1000) {
                    throw TimeoutError("Installation timeout exceeded")
                }
                
            } catch Error as err {
                stepResult := {
                    stepNumber: i,
                    stepName: step.name,
                    success: false,
                    error: err.Message,
                    stepDuration: A_TickCount - stepStartTime
                }
                
                results.Push(stepResult)
                
                ; Stop on critical errors unless continueOnError is set
                if (!step.HasProp("continueOnError") || !step.continueOnError) {
                    break
                }
            }
        }
        
        return {
            steps: results,
            totalDuration: A_TickCount - startTime,
            completed: this.currentStep >= this.installSteps.Length,
            success: results.Length > 0 && results[-1].success
        }
    }
    
    static ExecuteInstallStep(step) {
        stepResult := {
            stepName: step.name,
            success: false,
            details: {}
        }
        
        ; Execute step action
        if (step.HasProp("action")) {
            switch step.action.type {
                case "launch":
                    result := AppLauncher.LaunchAndWait(step.action.path, step.action.windowCriteria, step.action)
                    stepResult.details.processId := result.processId
                    stepResult.details.windowHandle := result.windowHandle
                    
                case "waitWindow":
                    windowHandle := WinWait(step.action.title,
                                           step.action.HasProp("text") ? step.action.text : "",
                                           step.action.HasProp("timeout") ? step.action.timeout : 60)
                    stepResult.details.windowHandle := windowHandle
                    
                case "waitFile":
                    this.WaitForFile(step.action.filePath, step.action.HasProp("timeout") ? step.action.timeout : 30)
                    stepResult.details.filePath := step.action.filePath
                    
                case "custom":
                    if (IsFunc(step.action.function)) {
                        stepResult.details := step.action.function.Call(step)
                    }
            }
        }
        
        stepResult.success := true
        return stepResult
    }
    
    static WaitForFile(filePath, timeout := 30) {
        startTime := A_TickCount
        
        while !FileExist(filePath) {
            if (A_TickCount - startTime > timeout * 1000) {
                throw TimeoutError("File " . filePath . " did not appear within timeout")
            }
            Sleep(500)
        }
        
        return filePath
    }
    
    static GetCurrentStep() {
        return this.currentStep
    }
    
    static GetStepProgress() {
        return {
            current: this.currentStep,
            total: this.installSteps.Length,
            percentage: this.installSteps.Length > 0 ? (this.currentStep / this.installSteps.Length) * 100 : 0
        }
    }
}

; Example usage and demonstrations
; Initialize synchronization systems
WindowSynchronizer.Initialize()

; Basic window waiting
try {
    ; Wait for Notepad to appear
    windowHandle := WinWait("Notepad", , 10)
    MsgBox("Notepad appeared! Handle: " . windowHandle)
} catch TimeoutError {
    MsgBox("Notepad didn't appear within 10 seconds")
}

; Asynchronous window waiting
waitId := WindowSynchronizer.WaitForWindow("Calculator", {
    timeout: 15,
    async: true,
    callback: (handle, id, error?) => {
        if (error) {
            MsgBox("Calculator wait failed: " . error.Message)
        } else {
            MsgBox("Calculator appeared asynchronously! Handle: " . handle)
        }
    }
})

; Wait for multiple windows (any)
windowList := [
    {title: "Notepad"},
    {title: "Calculator"},
    {title: "Paint"}
]

try {
    result := WindowSynchronizer.WaitForMultipleWindows(windowList, "any")
    MsgBox("First window appeared: " . result.window.title)
} catch TimeoutError {
    MsgBox("None of the specified windows appeared")
}

; Application launch coordination
try {
    launchResult := AppLauncher.LaunchAndWait("notepad.exe", {title: "Notepad"}, {
        timeout: 15,
        retries: 3
    })
    
    MsgBox("Notepad launched successfully!")
    
    ; Wait for application to be fully ready
    AppLauncher.WaitForAppReady(launchResult.processId, {
        mainWindow: {title: "Notepad"},
        timeout: 30
    })
    
} catch Error as err {
    MsgBox("Failed to launch Notepad: " . err.Message)
}

; Window chain waiting
windowChain := [
    {title: "Setup Wizard", timeout: 30},
    {title: "License Agreement", timeout: 15, delay: 1000},
    {title: "Installation Complete", timeout: 120}
]

try {
    chainResults := WindowSynchronizer.WaitForWindowChain(windowChain, 300)
    MsgBox("Installation wizard completed successfully!")
} catch TimeoutError as err {
    MsgBox("Installation wizard failed: " . err.Message)
}

; Installation step definition and execution
InstallationSynchronizer.DefineInstallation([
    {
        name: "Launch Installer",
        action: {
            type: "launch",
            path: "setup.exe",
            windowCriteria: {title: "Setup Wizard"}
        }
    },
    {
        name: "Wait for Welcome Screen",
        action: {
            type: "waitWindow",
            title: "Welcome",
            timeout: 30
        }
    },
    {
        name: "Wait for Installation Complete",
        action: {
            type: "waitFile",
            filePath: "C:\\Program Files\\MyApp\\installed.flag",
            timeout: 180
        }
    }
])

; Execute installation
installResult := InstallationSynchronizer.ExecuteInstallation()

if (installResult.success) {
    MsgBox("Installation completed successfully!")
} else {
    MsgBox("Installation failed or incomplete")
}

; Advanced window waiting with exclusions
WinWait("ahk_class Notepad", , 10, "Save As")  ; Wait for Notepad but exclude Save As dialog

; Wait for specific process window
WinWait("ahk_exe chrome.exe", , 15)  ; Wait for any Chrome window

; Wait for window with specific text
WinWait("Error", "Access Denied", 5)  ; Wait for error dialog containing "Access Denied"
```

## Implementation Notes

**Timeout Handling and Performance:**
- Infinite waits (timeout 0) can cause script hang if window never appears
- Use reasonable timeouts to maintain script responsiveness and user experience
- Consider implementing user cancellation mechanisms for long waits
- Monitor system resource usage during extended wait operations

**Window Identification Strategies:**
- Window titles can change dynamically, use class names or process identification for reliability
- Some applications create temporary windows before main window, consider exclusion criteria
- Multi-monitor setups may affect window appearance timing and detection
- Hidden or minimized windows are detected by WinWait but may not be user-visible

**Error Recovery and Fallback:**
- Implement comprehensive timeout handling for production automation scripts
- Provide meaningful error messages that guide users toward resolution
- Consider alternative window identification methods when primary criteria fail
- Log window waiting events for debugging complex automation sequences

**Synchronization Patterns:**
- Chain multiple WinWait calls for complex application startup sequences
- Use asynchronous waiting for monitoring multiple applications simultaneously
- Coordinate with process monitoring for complete application lifecycle management
- Implement retry mechanisms for unreliable application launch scenarios

**System Integration Considerations:**
- Antivirus software may delay application launches and window appearance
- System performance affects application startup timing and window creation
- User account permissions can impact window visibility and accessibility
- Remote desktop scenarios may have different window behavior and timing characteristics

## Related AHK Concepts

- [WinExist](./winexist.md) - Checking for window existence without waiting
- [WinActivate](./winactivate.md) - Bringing windows to foreground after detection
- [WinClose](./winclose.md) - Closing windows after interaction completion
- [Run](./run.md) - Launching applications that create windows to wait for
- [ProcessWait](../../40_Advanced_Features/05-System_Integration/processwait.md) - Process-level synchronization

## Tags

#AutoHotkey #WinWait #WindowSynchronization #Automation #ProcessCoordination #ApplicationLaunch #Timing #SystemIntegration #WindowManagement #Scripting
