# Topic: ControlClick Function - Advanced GUI Control Automation and Interaction

## Category

Function

## Overview

The ControlClick function performs automated clicks on specific GUI controls within windows, providing precise control interaction without relying on screen coordinates. It's essential for automated testing, legacy application integration, GUI validation, accessibility automation, and any scenario requiring reliable interaction with specific controls independent of window position or visual appearance.

## Key Points

- Clicks specific controls within windows using control identifiers rather than screen coordinates
- Supports multiple mouse button interactions with customizable click timing and modifiers
- Provides robust automation that works regardless of window position, size, or visual state
- Enables sophisticated GUI testing frameworks and legacy application automation systems
- Essential for automated testing, accessibility tools, legacy system integration, and reliable GUI automation

## Syntax and Parameters

```cpp
ControlClick([Control, WinTitle, WinText, WhichButton, ClickCount, Options, ExcludeTitle, ExcludeText])

; Parameters:
; Control      - Control identifier (ClassNN, text, or control object)
; WinTitle     - Window title or identifier
; WinText      - Text contained in the window (optional)
; WhichButton  - Mouse button to click (Left, Right, Middle, default: Left)
; ClickCount   - Number of clicks (default: 1)
; Options      - Additional options (Pos, NA, X##, Y##)
; ExcludeTitle - Windows to exclude from matching (optional)
; ExcludeText  - Text in windows to exclude (optional)

; Control identification methods:
; "Button1"           - ClassNN identifier (class name + instance number)
; "OK"                - Control text content
; "ahk_id " . hwnd    - Control handle (HWND)
; controlObj          - Control object reference

; Window identification:
; "Window Title"      - Exact or partial window title
; "ahk_class Class"   - Window class name
; "ahk_exe Process"   - Process executable name
; "ahk_pid PID"       - Process ID

; Mouse buttons:
; "Left" (default)    - Left mouse button
; "Right"             - Right mouse button  
; "Middle"            - Middle mouse button
; "X1", "X2"          - Extended mouse buttons

; Options:
; "Pos"               - Use control's position for click coordinates
; "NA"                - Don't activate window before clicking
; "X##" "Y##"         - Click at specific offset within control

; Error handling:
; Throws exception if control not found
; Throws exception if window not found
; May fail silently if control is disabled or hidden
```

## Code Examples

```cpp
; Basic control clicking
ControlClick("Button1", "Calculator")  ; Click first button in Calculator
ControlClick("OK", "Save As")          ; Click OK button by text
ControlClick("Edit1", "Notepad", , "Right", 2)  ; Right double-click on edit control

; Advanced GUI automation and testing framework
class GUIAutomator {
    static automationMode := "safe"
    static clickDelay := 50
    static verificationEnabled := true
    static retryAttempts := 3
    static retryDelay := 500
    static screenshotOnFailure := true
    static logAllActions := false
    static actionLog := []
    static errorThreshold := 5
    static errorCount := 0
    
    static Initialize() {
        this.actionLog := []
        this.errorCount := 0
        this.SetupErrorHandling()
    }
    
    static ClickControl(controlSpec, windowSpec, options := {}) {
        ; Advanced control clicking with comprehensive error handling
        button := options.HasProp("button") ? options.button : "Left"
        clickCount := options.HasProp("clickCount") ? options.clickCount : 1
        waitBefore := options.HasProp("waitBefore") ? options.waitBefore : 0
        waitAfter := options.HasProp("waitAfter") ? options.waitAfter : this.clickDelay
        verifyClick := options.HasProp("verify") ? options.verify : this.verificationEnabled
        retries := options.HasProp("retries") ? options.retries : this.retryAttempts
        
        ; Create action record
        action := {
            type: "click",
            control: controlSpec,
            window: windowSpec,
            button: button,
            clickCount: clickCount,
            timestamp: A_TickCount,
            success: false,
            error: "",
            attempts: 0
        }
        
        ; Wait before action if specified
        if (waitBefore > 0) {
            Sleep(waitBefore)
        }
        
        ; Attempt click with retries
        for attempt in Range(1, retries + 1) {
            action.attempts := attempt
            
            try {
                ; Verify window exists
                if (!WinExist(windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")) {
                    throw Error("Window not found: " . windowSpec.title)
                }
                
                ; Verify control exists
                if (!ControlGetHwnd(controlSpec, windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")) {
                    throw Error("Control not found: " . controlSpec)
                }
                
                ; Perform the click
                this.ExecuteControlClick(controlSpec, windowSpec, button, clickCount, options)
                
                ; Verify click if enabled
                if (verifyClick) {
                    this.VerifyClickSuccess(controlSpec, windowSpec, options)
                }
                
                ; Success
                action.success := true
                action.error := ""
                break
                
            } catch Error as err {
                action.error := err.Message
                
                if (attempt < retries) {
                    ; Log retry attempt
                    if (this.logAllActions) {
                        OutputDebug("Click attempt " . attempt . " failed: " . err.Message . ", retrying...")
                    }
                    
                    Sleep(this.retryDelay)
                    continue
                } else {
                    ; Final attempt failed
                    this.HandleClickFailure(action, err)
                    throw err
                }
            }
        }
        
        ; Wait after action
        if (waitAfter > 0) {
            Sleep(waitAfter)
        }
        
        ; Log action
        this.LogAction(action)
        
        return action
    }
    
    static ExecuteControlClick(controlSpec, windowSpec, button, clickCount, options) {
        ; Execute the actual control click
        clickOptions := ""
        
        ; Build options string
        if (options.HasProp("usePosition") && options.usePosition) {
            clickOptions .= "Pos "
        }
        
        if (options.HasProp("noActivate") && options.noActivate) {
            clickOptions .= "NA "
        }
        
        if (options.HasProp("offsetX")) {
            clickOptions .= "X" . options.offsetX . " "
        }
        
        if (options.HasProp("offsetY")) {
            clickOptions .= "Y" . options.offsetY . " "
        }
        
        ; Perform the click
        ControlClick(controlSpec, 
                    windowSpec.title, 
                    windowSpec.HasProp("text") ? windowSpec.text : "",
                    button,
                    clickCount,
                    Trim(clickOptions),
                    windowSpec.HasProp("excludeTitle") ? windowSpec.excludeTitle : "",
                    windowSpec.HasProp("excludeText") ? windowSpec.excludeText : "")
    }
    
    static VerifyClickSuccess(controlSpec, windowSpec, options) {
        ; Verify that the click had the expected effect
        verifyMethod := options.HasProp("verifyMethod") ? options.verifyMethod : "focus"
        
        switch verifyMethod {
            case "focus":
                ; Verify control received focus
                Sleep(100)  ; Allow time for focus change
                focusedControl := ControlGetFocus(windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")
                if (focusedControl != controlSpec) {
                    ; Try to get control HWND for comparison
                    try {
                        controlHwnd := ControlGetHwnd(controlSpec, windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")
                        focusedHwnd := ControlGetHwnd(focusedControl, windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")
                        if (controlHwnd != focusedHwnd) {
                            throw Error("Click verification failed: control did not receive focus")
                        }
                    } catch {
                        throw Error("Click verification failed: unable to verify focus")
                    }
                }
                
            case "enabled":
                ; Verify control is still enabled (for toggle controls)
                if (!ControlGetEnabled(controlSpec, windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")) {
                    throw Error("Click verification failed: control became disabled")
                }
                
            case "text":
                ; Verify control text changed (if expected)
                if (options.HasProp("expectedText")) {
                    Sleep(200)  ; Allow time for text change
                    actualText := ControlGetText(controlSpec, windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")
                    if (actualText != options.expectedText) {
                        throw Error("Click verification failed: text mismatch")
                    }
                }
                
            case "custom":
                ; Custom verification function
                if (options.HasProp("verifyFunction") && IsFunc(options.verifyFunction)) {
                    if (!options.verifyFunction.Call(controlSpec, windowSpec)) {
                        throw Error("Click verification failed: custom verification")
                    }
                }
        }
    }
    
    static ClickSequence(clickSequence, windowSpec, options := {}) {
        ; Execute a sequence of control clicks
        sequenceDelay := options.HasProp("sequenceDelay") ? options.sequenceDelay : 100
        stopOnError := options.HasProp("stopOnError") ? options.stopOnError : true
        
        results := []
        
        for i, clickSpec in clickSequence {
            try {
                ; Merge sequence options with individual click options
                clickOptions := options.Clone()
                if (clickSpec.HasProp("options")) {
                    for prop, value in clickSpec.options.OwnProps() {
                        clickOptions.%prop% := value
                    }
                }
                
                ; Execute click
                result := this.ClickControl(clickSpec.control, windowSpec, clickOptions)
                results.Push(result)
                
                ; Delay between clicks
                if (i < clickSequence.Length && sequenceDelay > 0) {
                    Sleep(sequenceDelay)
                }
                
            } catch Error as err {
                result := {
                    type: "click",
                    control: clickSpec.control,
                    success: false,
                    error: err.Message,
                    attempts: 1
                }
                
                results.Push(result)
                
                if (stopOnError) {
                    throw Error("Click sequence failed at step " . i . ": " . err.Message)
                }
            }
        }
        
        return results
    }
    
    static FindAndClickByText(searchText, windowSpec, options := {}) {
        ; Find and click control containing specific text
        searchMethod := options.HasProp("searchMethod") ? options.searchMethod : "exact"
        controlTypes := options.HasProp("controlTypes") ? options.controlTypes : ["Button", "Edit", "Static", "ListBox"]
        
        for controlType in controlTypes {
            try {
                ; Enumerate controls of this type
                controlList := WinGetControls(windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")
                
                for controlId in controlList {
                    if (InStr(controlId, controlType)) {
                        try {
                            controlText := ControlGetText(controlId, windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")
                            
                            found := false
                            switch searchMethod {
                                case "exact":
                                    found := (controlText = searchText)
                                case "contains":
                                    found := InStr(controlText, searchText)
                                case "startswith":
                                    found := (SubStr(controlText, 1, StrLen(searchText)) = searchText)
                                case "regex":
                                    found := RegExMatch(controlText, searchText)
                            }
                            
                            if (found) {
                                return this.ClickControl(controlId, windowSpec, options)
                            }
                            
                        } catch {
                            ; Continue searching other controls
                        }
                    }
                }
                
            } catch {
                ; Continue with next control type
            }
        }
        
        throw Error("Control with text '" . searchText . "' not found")
    }
    
    static WaitAndClick(controlSpec, windowSpec, timeout := 10000, options := {}) {
        ; Wait for control to become available, then click
        startTime := A_TickCount
        checkInterval := options.HasProp("checkInterval") ? options.checkInterval : 250
        
        while (A_TickCount - startTime < timeout) {
            try {
                ; Check if window exists
                if (!WinExist(windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")) {
                    Sleep(checkInterval)
                    continue
                }
                
                ; Check if control exists and is enabled
                if (ControlGetHwnd(controlSpec, windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")) {
                    if (ControlGetEnabled(controlSpec, windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")) {
                        ; Control is available, perform click
                        return this.ClickControl(controlSpec, windowSpec, options)
                    }
                }
                
            } catch {
                ; Control not ready yet
            }
            
            Sleep(checkInterval)
        }
        
        throw TimeoutError("Control '" . controlSpec . "' did not become available within timeout")
    }
    
    static HandleClickFailure(action, error) {
        ; Handle click failure with error reporting and recovery
        this.errorCount++
        
        ; Take screenshot if enabled
        if (this.screenshotOnFailure) {
            this.TakeFailureScreenshot(action)
        }
        
        ; Log detailed error information
        errorDetails := {
            action: action,
            error: error.Message,
            timestamp: FormatTime(),
            windowExists: WinExist(action.window.title),
            errorCount: this.errorCount
        }
        
        this.LogError(errorDetails)
        
        ; Check error threshold
        if (this.errorCount >= this.errorThreshold) {
            this.HandleErrorThresholdExceeded()
        }
    }
    
    static TakeFailureScreenshot(action) {
        ; Take screenshot for debugging failed clicks
        try {
            screenshotPath := A_ScriptDir . "\screenshots\failure_" . FormatTime(, "yyyyMMdd_HHmmss") . ".png"
            
            ; Create screenshots directory if it doesn't exist
            if (!DirExist(A_ScriptDir . "\screenshots")) {
                DirCreate(A_ScriptDir . "\screenshots")
            }
            
            ; Take screenshot (implementation would depend on available screenshot functionality)
            ; ImagePut.ToFile(screenshotPath)  ; Example using ImagePut library
            
        } catch {
            ; Screenshot failed, continue without
        }
    }
    
    static LogAction(action) {
        ; Log action for debugging and analysis
        this.actionLog.Push(action)
        
        ; Limit log size
        if (this.actionLog.Length > 1000) {
            this.actionLog.RemoveAt(1)
        }
        
        ; Debug output if enabled
        if (this.logAllActions) {
            status := action.success ? "SUCCESS" : "FAILED"
            OutputDebug("GUI Action [" . status . "]: " . action.type . " on " . action.control . " (attempts: " . action.attempts . ")")
        }
    }
    
    static LogError(errorDetails) {
        ; Log error details
        errorMessage := "GUI Automation Error: " . errorDetails.error . 
                      " | Action: " . errorDetails.action.type . 
                      " | Control: " . errorDetails.action.control . 
                      " | Time: " . errorDetails.timestamp
        
        OutputDebug(errorMessage)
        
        ; Could also write to error log file
        try {
            FileAppend(errorMessage . "`n", A_ScriptDir . "\automation_errors.log")
        } catch {
            ; Continue if logging fails
        }
    }
    
    static SetupErrorHandling() {
        ; Set up global error handling for GUI automation
        ; This would be called during initialization
    }
    
    static HandleErrorThresholdExceeded() {
        ; Handle when error threshold is exceeded
        OutputDebug("GUI Automation error threshold exceeded: " . this.errorCount . " errors")
        
        ; Could implement circuit breaker pattern
        ; Could send notifications
        ; Could switch to safe mode
    }
    
    static GetActionStatistics() {
        ; Get statistics about performed actions
        totalActions := this.actionLog.Length
        successfulActions := 0
        failedActions := 0
        
        for action in this.actionLog {
            if (action.success) {
                successfulActions++
            } else {
                failedActions++
            }
        }
        
        return {
            total: totalActions,
            successful: successfulActions,
            failed: failedActions,
            successRate: totalActions > 0 ? Round((successfulActions / totalActions) * 100, 1) : 0,
            errorCount: this.errorCount
        }
    }
    
    static GenerateAutomationReport() {
        ; Generate comprehensive automation report
        stats := this.GetActionStatistics()
        
        report := "GUI Automation Report`n"
        report .= "=====================`n`n"
        
        report .= "Summary:`n"
        report .= "Total Actions: " . stats.total . "`n"
        report .= "Successful: " . stats.successful . "`n"
        report .= "Failed: " . stats.failed . "`n"
        report .= "Success Rate: " . stats.successRate . "%`n"
        report .= "Error Count: " . stats.errorCount . "`n`n"
        
        report .= "Configuration:`n"
        report .= "Mode: " . this.automationMode . "`n"
        report .= "Click Delay: " . this.clickDelay . "ms`n"
        report .= "Verification: " . (this.verificationEnabled ? "Enabled" : "Disabled") . "`n"
        report .= "Retry Attempts: " . this.retryAttempts . "`n"
        report .= "Screenshot on Failure: " . (this.screenshotOnFailure ? "Enabled" : "Disabled") . "`n`n"
        
        ; Recent actions
        report .= "Recent Actions:`n"
        recentCount := Min(10, this.actionLog.Length)
        for i in Range(this.actionLog.Length - recentCount + 1, this.actionLog.Length) {
            action := this.actionLog[i]
            status := action.success ? "✓" : "✗"
            report .= status . " " . action.type . " on " . action.control . " (attempts: " . action.attempts . ")`n"
        }
        
        return report
    }
    
    static EnableLogging() {
        this.logAllActions := true
    }
    
    static DisableLogging() {
        this.logAllActions := false
    }
    
    static SetClickDelay(milliseconds) {
        this.clickDelay := milliseconds
    }
    
    static EnableVerification() {
        this.verificationEnabled := true
    }
    
    static DisableVerification() {
        this.verificationEnabled := false
    }
}

; Application testing framework
class AppTester {
    static testSuites := Map()
    static testResults := []
    static currentSuite := ""
    static testTimeout := 30000
    
    static Initialize() {
        this.testSuites.Clear()
        this.testResults := []
        GUIAutomator.Initialize()
    }
    
    static CreateTestSuite(name, windowSpec) {
        ; Create new test suite for specific application
        this.testSuites[name] := {
            name: name,
            windowSpec: windowSpec,
            tests: [],
            setup: "",
            teardown: "",
            beforeEach: "",
            afterEach: ""
        }
    }
    
    static AddTest(suiteName, testName, testFunction) {
        ; Add test to suite
        if (!this.testSuites.Has(suiteName)) {
            throw ValueError("Test suite '" . suiteName . "' does not exist")
        }
        
        suite := this.testSuites[suiteName]
        suite.tests.Push({
            name: testName,
            function: testFunction,
            enabled: true
        })
    }
    
    static RunTestSuite(suiteName) {
        ; Run all tests in a suite
        if (!this.testSuites.Has(suiteName)) {
            throw ValueError("Test suite '" . suiteName . "' does not exist")
        }
        
        suite := this.testSuites[suiteName]
        this.currentSuite := suiteName
        
        suiteResults := {
            suite: suiteName,
            startTime: A_TickCount,
            endTime: 0,
            tests: [],
            passed: 0,
            failed: 0,
            skipped: 0
        }
        
        try {
            ; Run suite setup
            if (suite.setup && IsFunc(suite.setup)) {
                suite.setup.Call()
            }
            
            ; Run each test
            for test in suite.tests {
                if (!test.enabled) {
                    suiteResults.skipped++
                    continue
                }
                
                testResult := this.RunTest(suite, test)
                suiteResults.tests.Push(testResult)
                
                if (testResult.passed) {
                    suiteResults.passed++
                } else {
                    suiteResults.failed++
                }
            }
            
            ; Run suite teardown
            if (suite.teardown && IsFunc(suite.teardown)) {
                suite.teardown.Call()
            }
            
        } catch Error as err {
            suiteResults.error := err.Message
        } finally {
            suiteResults.endTime := A_TickCount
            this.testResults.Push(suiteResults)
        }
        
        return suiteResults
    }
    
    static RunTest(suite, test) {
        ; Run individual test
        testResult := {
            name: test.name,
            startTime: A_TickCount,
            endTime: 0,
            passed: false,
            error: "",
            actions: []
        }
        
        try {
            ; Run beforeEach
            if (suite.beforeEach && IsFunc(suite.beforeEach)) {
                suite.beforeEach.Call()
            }
            
            ; Run the test
            test.function.Call(suite.windowSpec)
            
            ; Run afterEach
            if (suite.afterEach && IsFunc(suite.afterEach)) {
                suite.afterEach.Call()
            }
            
            testResult.passed := true
            
        } catch Error as err {
            testResult.error := err.Message
        } finally {
            testResult.endTime := A_TickCount
        }
        
        return testResult
    }
    
    static Assert(condition, message := "Assertion failed") {
        ; Test assertion
        if (!condition) {
            throw Error("Assertion failed: " . message)
        }
    }
    
    static AssertControlExists(controlSpec, windowSpec, message := "Control does not exist") {
        ; Assert that control exists
        if (!ControlGetHwnd(controlSpec, windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")) {
            throw Error("Assertion failed: " . message)
        }
    }
    
    static AssertControlText(controlSpec, windowSpec, expectedText, message := "Control text mismatch") {
        ; Assert control text matches expected value
        actualText := ControlGetText(controlSpec, windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")
        if (actualText != expectedText) {
            throw Error("Assertion failed: " . message . " (expected: '" . expectedText . "', actual: '" . actualText . "')")
        }
    }
    
    static GenerateTestReport() {
        ; Generate comprehensive test report
        report := "Application Test Report`n"
        report .= "======================`n`n"
        
        totalTests := 0
        totalPassed := 0
        totalFailed := 0
        totalSkipped := 0
        
        for suiteResult in this.testResults {
            totalTests += suiteResult.tests.Length
            totalPassed += suiteResult.passed
            totalFailed += suiteResult.failed
            totalSkipped += suiteResult.skipped
        }
        
        report .= "Overall Summary:`n"
        report .= "Total Tests: " . totalTests . "`n"
        report .= "Passed: " . totalPassed . "`n"
        report .= "Failed: " . totalFailed . "`n"
        report .= "Skipped: " . totalSkipped . "`n"
        
        if (totalTests > 0) {
            passRate := Round((totalPassed / totalTests) * 100, 1)
            report .= "Pass Rate: " . passRate . "%`n"
        }
        
        report .= "`n"
        
        ; Suite details
        for suiteResult in this.testResults {
            report .= "Suite: " . suiteResult.suite . "`n"
            report .= "Duration: " . Round((suiteResult.endTime - suiteResult.startTime) / 1000, 1) . "s`n"
            report .= "Tests: " . suiteResult.tests.Length . " (P:" . suiteResult.passed . " F:" . suiteResult.failed . " S:" . suiteResult.skipped . ")`n"
            
            ; Failed tests
            for testResult in suiteResult.tests {
                if (!testResult.passed) {
                    report .= "  FAILED: " . testResult.name . " - " . testResult.error . "`n"
                }
            }
            
            report .= "`n"
        }
        
        return report
    }
}

; Legacy application integration system
class LegacyAppIntegrator {
    static appSpecs := Map()
    static currentApp := ""
    static connectionTimeout := 10000
    static operationTimeout := 5000
    
    static RegisterApplication(name, spec) {
        ; Register legacy application specification
        this.appSpecs[name] := {
            name: name,
            executable: spec.executable,
            windowTitle: spec.windowTitle,
            windowClass: spec.HasProp("windowClass") ? spec.windowClass : "",
            startupTime: spec.HasProp("startupTime") ? spec.startupTime : 5000,
            controls: spec.HasProp("controls") ? spec.controls : Map(),
            workflows: spec.HasProp("workflows") ? spec.workflows : Map()
        }
    }
    
    static ConnectToApplication(appName) {
        ; Connect to or launch legacy application
        if (!this.appSpecs.Has(appName)) {
            throw ValueError("Application '" . appName . "' not registered")
        }
        
        appSpec := this.appSpecs[appName]
        
        ; Check if already running
        if (WinExist(appSpec.windowTitle)) {
            WinActivate(appSpec.windowTitle)
            this.currentApp := appName
            return true
        }
        
        ; Launch application
        try {
            Run(appSpec.executable)
            
            ; Wait for application window
            if (!WinWait(appSpec.windowTitle, , appSpec.startupTime / 1000)) {
                throw TimeoutError("Application did not start within timeout")
            }
            
            WinActivate(appSpec.windowTitle)
            this.currentApp := appName
            return true
            
        } catch Error as err {
            throw Error("Failed to connect to application: " . err.Message)
        }
    }
    
    static ExecuteWorkflow(appName, workflowName, parameters := {}) {
        ; Execute predefined workflow in legacy application
        if (!this.appSpecs.Has(appName)) {
            throw ValueError("Application '" . appName . "' not registered")
        }
        
        appSpec := this.appSpecs[appName]
        
        if (!appSpec.workflows.Has(workflowName)) {
            throw ValueError("Workflow '" . workflowName . "' not found")
        }
        
        workflow := appSpec.workflows[workflowName]
        
        ; Connect to application
        this.ConnectToApplication(appName)
        
        ; Execute workflow steps
        for step in workflow {
            this.ExecuteWorkflowStep(appSpec, step, parameters)
        }
    }
    
    static ExecuteWorkflowStep(appSpec, step, parameters) {
        ; Execute individual workflow step
        windowSpec := {title: appSpec.windowTitle}
        
        switch step.action {
            case "click":
                controlSpec := this.ResolveControlSpec(appSpec, step.control)
                GUIAutomator.ClickControl(controlSpec, windowSpec, step.HasProp("options") ? step.options : {})
                
            case "type":
                controlSpec := this.ResolveControlSpec(appSpec, step.control)
                text := step.HasProp("text") ? step.text : ""
                
                ; Replace parameters in text
                for paramName, paramValue in parameters {
                    text := StrReplace(text, "{" . paramName . "}", paramValue)
                }
                
                ControlSetText(text, controlSpec, windowSpec.title)
                
            case "wait":
                waitTime := step.HasProp("time") ? step.time : 1000
                Sleep(waitTime)
                
            case "verify":
                controlSpec := this.ResolveControlSpec(appSpec, step.control)
                expectedValue := step.HasProp("value") ? step.value : ""
                actualValue := ControlGetText(controlSpec, windowSpec.title)
                
                if (actualValue != expectedValue) {
                    throw Error("Verification failed: expected '" . expectedValue . "', got '" . actualValue . "'")
                }
        }
    }
    
    static ResolveControlSpec(appSpec, controlName) {
        ; Resolve control name to actual control specification
        if (appSpec.controls.Has(controlName)) {
            return appSpec.controls[controlName]
        } else {
            return controlName  ; Use as-is if not in control map
        }
    }
}

; Example usage and demonstrations
; Initialize GUI automation system
GUIAutomator.Initialize()

; Basic control clicking examples
; Click OK button in any dialog
try {
    GUIAutomator.ClickControl("OK", {title: "Save As"})
} catch Error as err {
    MsgBox("Click failed: " . err.Message)
}

; Click with specific options
GUIAutomator.ClickControl("Button1", {title: "Calculator"}, {
    button: "Right",
    clickCount: 2,
    waitBefore: 500,
    verify: true,
    verifyMethod: "focus"
})

; Advanced GUI testing example
AppTester.Initialize()

; Create test suite for Calculator
AppTester.CreateTestSuite("Calculator", {title: "Calculator"})

; Add tests
AppTester.AddTest("Calculator", "Basic Addition", (windowSpec) => {
    ; Test basic addition: 2 + 3 = 5
    GUIAutomator.ClickControl("Button2", windowSpec)  ; Click 2
    GUIAutomator.ClickControl("Button_Plus", windowSpec)  ; Click +
    GUIAutomator.ClickControl("Button3", windowSpec)  ; Click 3
    GUIAutomator.ClickControl("Button_Equals", windowSpec)  ; Click =
    
    ; Verify result
    result := ControlGetText("Edit1", windowSpec.title)
    AppTester.Assert(result = "5", "Addition result should be 5")
})

AppTester.AddTest("Calculator", "Clear Function", (windowSpec) => {
    ; Test clear function
    GUIAutomator.ClickControl("Button1", windowSpec)  ; Click 1
    GUIAutomator.ClickControl("Button_Clear", windowSpec)  ; Click Clear
    
    ; Verify display is cleared
    result := ControlGetText("Edit1", windowSpec.title)
    AppTester.Assert(result = "0" || result = "", "Display should be cleared")
})

; Run calculator tests
try {
    ; Ensure Calculator is running
    if (!WinExist("Calculator")) {
        Run("calc.exe")
        WinWait("Calculator", , 10)
    }
    
    results := AppTester.RunTestSuite("Calculator")
    testReport := AppTester.GenerateTestReport()
    
    MsgBox("Tests completed. Passed: " . results.passed . ", Failed: " . results.failed)
    
} catch Error as err {
    MsgBox("Test execution failed: " . err.Message)
}

; Click sequence example
clickSequence := [
    {control: "Edit1", options: {waitBefore: 100}},
    {control: "Button1", options: {verify: true}},
    {control: "Button_Plus", options: {waitAfter: 50}},
    {control: "Button2", options: {verify: true}},
    {control: "Button_Equals", options: {waitAfter: 200}}
]

try {
    GUIAutomator.ClickSequence(clickSequence, {title: "Calculator"}, {
        sequenceDelay: 100,
        stopOnError: true
    })
} catch Error as err {
    MsgBox("Click sequence failed: " . err.Message)
}

; Find and click by text
try {
    GUIAutomator.FindAndClickByText("OK", {title: "Confirm"}, {
        searchMethod: "exact",
        controlTypes: ["Button", "Static"]
    })
} catch Error as err {
    MsgBox("Text-based click failed: " . err.Message)
}

; Wait and click example
try {
    GUIAutomator.WaitAndClick("Button_Submit", {title: "Form"}, 15000, {
        checkInterval: 500,
        waitAfter: 1000
    })
} catch TimeoutError {
    MsgBox("Button did not become available within timeout")
}

; Legacy application integration example
LegacyAppIntegrator.RegisterApplication("OldApp", {
    executable: "C:\\LegacyApps\\OldApp.exe",
    windowTitle: "Legacy Application v1.0",
    startupTime: 8000,
    controls: Map(
        "Username", "Edit1",
        "Password", "Edit2", 
        "LoginButton", "Button1",
        "StatusBar", "Static1"
    ),
    workflows: Map(
        "Login", [
            {action: "click", control: "Username"},
            {action: "type", control: "Username", text: "{username}"},
            {action: "click", control: "Password"},
            {action: "type", control: "Password", text: "{password}"},
            {action: "click", control: "LoginButton"},
            {action: "wait", time: 2000},
            {action: "verify", control: "StatusBar", value: "Logged in successfully"}
        ]
    )
})

; Execute legacy application workflow
try {
    LegacyAppIntegrator.ExecuteWorkflow("OldApp", "Login", {
        username: "admin",
        password: "secret123"
    })
    
    MsgBox("Legacy application login completed successfully")
    
} catch Error as err {
    MsgBox("Legacy application workflow failed: " . err.Message)
}

; Advanced verification examples
GUIAutomator.ClickControl("CheckBox1", {title: "Settings"}, {
    verify: true,
    verifyMethod: "custom",
    verifyFunction: (control, window) => {
        ; Custom verification: check if checkbox is checked
        return ControlGetChecked(control, window.title)
    }
})

; Performance monitoring
stats := GUIAutomator.GetActionStatistics()
MsgBox("GUI Automation Statistics:`n" .
       "Total Actions: " . stats.total . "`n" .
       "Success Rate: " . stats.successRate . "%`n" .
       "Errors: " . stats.errorCount)

; Generate automation report
automationReport := GUIAutomator.GenerateAutomationReport()
; MsgBox(automationReport, "Automation Report")

; Configuration adjustments
GUIAutomator.SetClickDelay(100)  ; Increase delay between clicks
GUIAutomator.EnableLogging()     ; Enable detailed logging
GUIAutomator.EnableVerification() ; Enable click verification

; Error handling example with retry
RetryClickWithBackoff(controlSpec, windowSpec, maxAttempts := 5) {
    baseDelay := 1000
    
    for attempt in Range(1, maxAttempts) {
        try {
            GUIAutomator.ClickControl(controlSpec, windowSpec, {
                retries: 1,  ; Single attempt per call
                waitBefore: attempt > 1 ? baseDelay * (2 ** (attempt - 2)) : 0
            })
            return true
            
        } catch Error as err {
            if (attempt = maxAttempts) {
                throw err
            }
            
            OutputDebug("Click attempt " . attempt . " failed, retrying with backoff...")
        }
    }
}

; Use retry with exponential backoff
try {
    RetryClickWithBackoff("Submit", {title: "Busy Application"}, 3)
} catch Error as err {
    MsgBox("All retry attempts failed: " . err.Message)
}
```

## Implementation Notes

**Control Identification Reliability:**
- Control identifiers can change between application versions or when UI is modified
- Use multiple identification methods (ClassNN, text, HWND) for robust automation
- Test control identification across different system configurations and application states
- Implement fallback identification strategies when primary methods fail

**Timing and Synchronization:**
- GUI controls may not be immediately available after window creation or state changes
- Implement appropriate delays and wait conditions for reliable control interaction
- Consider application performance variations when setting timeouts and retry intervals
- Use verification methods to ensure clicks had expected effects rather than assuming success

**Application State Management:**
- Control availability and behavior can depend on application state and context
- Verify prerequisites before attempting control interactions in complex workflows
- Handle modal dialogs and popup windows that can interrupt automation sequences
- Implement state recovery mechanisms for automation that gets interrupted

**Error Handling and Recovery:**
- Control clicks can fail due to timing, application state, or system resource issues
- Implement comprehensive retry mechanisms with appropriate backoff strategies
- Provide detailed error information for debugging failed automation sequences
- Consider graceful degradation when automation encounters unexpected conditions

**Cross-Platform and Accessibility:**
- Control behavior may vary across different Windows versions and accessibility settings
- Test automation with various system accessibility configurations and user settings
- Consider high-DPI displays and display scaling factors that can affect control positioning
- Ensure automation works with screen readers and other accessibility tools when appropriate

## Related AHK Concepts

- [ControlGetText](./controlgettext.md) - Retrieving text from GUI controls
- [ControlSetText](../../10_Language_Core/01-Functions/Built_In_Functions/controlsettext.md) - Setting control text values
- [WinActivate](../../40_Advanced_Features/05-System_Integration/winactivate.md) - Activating windows before control interaction
- [Click](../../40_Advanced_Features/00-Hotkeys_and_Input/click.md) - Alternative coordinate-based clicking
- [Send](../../40_Advanced_Features/00-Hotkeys_and_Input/send.md) - Sending keystrokes to controls

## Tags

#AutoHotkey #ControlClick #GUIAutomation #ControlInteraction #AutomatedTesting #LegacyApplications #ApplicationTesting #UserInterfaceAutomation #AccessibilityAutomation #GUITesting
