# Topic: ToolTip Function - Dynamic Information Display and User Feedback

## Category

Function

## Overview

The ToolTip function displays temporary information windows at specified screen positions, providing immediate user feedback and contextual information without interrupting workflow. It's essential for user interface enhancement, debugging displays, progress indicators, help systems, and any application requiring non-intrusive information presentation.

## Key Points

- Creates temporary information windows with customizable positioning and automatic timeout
- Supports multiple simultaneous tooltips with individual management and styling control
- Provides immediate visual feedback without blocking user input or workflow interruption
- Enables rich information display for debugging, progress tracking, and user guidance
- Essential for accessibility features, status notifications, interactive help systems, and development tools

## Syntax and Parameters

```cpp
ToolTip([Text, X, Y, WhichToolTip])

; Parameters:
; Text         - Text to display (empty string or omitted hides tooltip)
; X            - X coordinate for tooltip position (optional, default: near cursor)
; Y            - Y coordinate for tooltip position (optional, default: near cursor)
; WhichToolTip - Tooltip number 1-20 for multiple tooltips (optional, default: 1)

; Position behavior:
; Omitted X,Y  - Tooltip follows mouse cursor
; Specified X,Y - Fixed position on screen
; Negative values - Relative to right/bottom edge of screen

; Text formatting:
; Plain text   - Standard tooltip display
; Empty string - Hides the specified tooltip
; Newlines (`n) - Multi-line tooltip support
; Long text    - Automatic wrapping and sizing

; Return value:
; No return value
; Function executes immediately and displays tooltip
```

## Code Examples

```cpp
; Basic tooltip display
ToolTip("Hello World!")

; Tooltip at specific position
ToolTip("Status: Processing...", 100, 50)

; Hide tooltip
ToolTip()  ; or ToolTip("")

; Multiple tooltips
ToolTip("Main Status", 10, 10, 1)
ToolTip("Secondary Info", 200, 10, 2)
ToolTip("Debug Data", 10, 100, 3)

; Advanced tooltip management system
class ToolTipManager {
    static tooltips := Map()
    static nextId := 1
    static maxTooltips := 20
    static defaultTimeout := 3000
    static autoHide := true
    static positioning := "smart"
    
    static Initialize() {
        ; Initialize tooltip tracking
        this.tooltips.Clear()
        this.nextId := 1
    }
    
    static Show(text, x := "", y := "", options := {}) {
        ; Parse options
        timeout := options.HasProp("timeout") ? options.timeout : this.defaultTimeout
        id := options.HasProp("id") ? options.id : this.GetNextId()
        autoHide := options.HasProp("autoHide") ? options.autoHide : this.autoHide
        
        ; Validate tooltip ID
        if (id < 1 || id > this.maxTooltips) {
            throw ValueError("Tooltip ID must be between 1 and " . this.maxTooltips)
        }
        
        ; Calculate position if needed
        if (x = "" || y = "") {
            pos := this.CalculatePosition(text, x, y)
            x := pos.x
            y := pos.y
        }
        
        ; Display tooltip
        ToolTip(text, x, y, id)
        
        ; Track tooltip info
        this.tooltips[id] := {
            text: text,
            x: x,
            y: y,
            showTime: A_TickCount,
            timeout: timeout,
            autoHide: autoHide,
            visible: true
        }
        
        ; Set auto-hide timer if enabled
        if (autoHide && timeout > 0) {
            SetTimer(() => this.Hide(id), -timeout)
        }
        
        return id
    }
    
    static Hide(id := 1) {
        if (this.tooltips.Has(id)) {
            ToolTip("", , , id)
            this.tooltips[id].visible := false
        }
    }
    
    static HideAll() {
        for id in this.tooltips {
            this.Hide(id)
        }
    }
    
    static Update(id, newText, newX := "", newY := "") {
        if (!this.tooltips.Has(id)) {
            throw ValueError("Tooltip ID " . id . " not found")
        }
        
        tooltipInfo := this.tooltips[id]
        
        ; Use existing position if not specified
        x := newX != "" ? newX : tooltipInfo.x
        y := newY != "" ? newY : tooltipInfo.y
        
        ; Update tooltip
        ToolTip(newText, x, y, id)
        
        ; Update tracking info
        tooltipInfo.text := newText
        tooltipInfo.x := x
        tooltipInfo.y := y
        tooltipInfo.visible := true
    }
    
    static GetNextId() {
        ; Find next available tooltip ID
        for i in Range(1, this.maxTooltips) {
            if (!this.tooltips.Has(i) || !this.tooltips[i].visible) {
                return i
            }
        }
        
        ; If no free IDs, reuse oldest
        oldestId := 1
        oldestTime := A_TickCount
        
        for id, info in this.tooltips {
            if (info.showTime < oldestTime) {
                oldestTime := info.showTime
                oldestId := id
            }
        }
        
        return oldestId
    }
    
    static CalculatePosition(text, x, y) {
        ; Get screen dimensions
        screenWidth := A_ScreenWidth
        screenHeight := A_ScreenHeight
        
        ; Estimate tooltip size (rough calculation)
        lines := StrSplit(text, "`n")
        maxLineLength := 0
        for line in lines {
            if (StrLen(line) > maxLineLength) {
                maxLineLength := StrLen(line)
            }
        }
        
        estimatedWidth := maxLineLength * 8  ; Rough character width
        estimatedHeight := lines.Length * 16  ; Rough line height
        
        ; Get mouse position if no coordinates specified
        if (x = "" && y = "") {
            MouseGetPos(&mouseX, &mouseY)
            x := mouseX + 15  ; Offset from cursor
            y := mouseY + 15
        } else {
            MouseGetPos(&mouseX, &mouseY)
            if (x = "") x := mouseX + 15
            if (y = "") y := mouseY + 15
        }
        
        ; Adjust if tooltip would go off screen
        if (x + estimatedWidth > screenWidth) {
            x := screenWidth - estimatedWidth - 10
        }
        if (y + estimatedHeight > screenHeight) {
            y := screenHeight - estimatedHeight - 10
        }
        
        ; Ensure minimum position
        x := Max(5, x)
        y := Max(5, y)
        
        return {x: x, y: y}
    }
    
    static GetTooltipInfo(id) {
        return this.tooltips.Has(id) ? this.tooltips[id] : {}
    }
    
    static GetAllTooltips() {
        return this.tooltips.Clone()
    }
    
    static GetVisibleCount() {
        count := 0
        for id, info in this.tooltips {
            if (info.visible) {
                count++
            }
        }
        return count
    }
    
    static SetDefaultTimeout(timeout) {
        this.defaultTimeout := timeout
    }
    
    static EnableAutoHide() {
        this.autoHide := true
    }
    
    static DisableAutoHide() {
        this.autoHide := false
    }
}

; Status and progress display system
class StatusDisplay {
    static currentStatus := ""
    static progressTooltip := 0
    static statusTooltip := 0
    static debugTooltip := 0
    static position := {x: 10, y: 10}
    
    static Initialize(x := 10, y := 10) {
        this.position := {x: x, y: y}
        this.statusTooltip := 1
        this.progressTooltip := 2
        this.debugTooltip := 3
    }
    
    static ShowStatus(message, timeout := 3000) {
        this.currentStatus := message
        ToolTip(message, this.position.x, this.position.y, this.statusTooltip)
        
        if (timeout > 0) {
            SetTimer(() => this.ClearStatus(), -timeout)
        }
    }
    
    static ClearStatus() {
        ToolTip("", , , this.statusTooltip)
        this.currentStatus := ""
    }
    
    static ShowProgress(current, total, operation := "Processing") {
        percentage := Round((current / total) * 100, 1)
        progressBar := this.CreateProgressBar(percentage)
        message := operation . " " . current . "/" . total . " (" . percentage . "%)`n" . progressBar
        
        ToolTip(message, this.position.x, this.position.y + 30, this.progressTooltip)
    }
    
    static CreateProgressBar(percentage, width := 20) {
        filled := Round((percentage / 100) * width)
        bar := "["
        
        Loop filled {
            bar .= "█"
        }
        
        Loop width - filled {
            bar .= "░"
        }
        
        bar .= "]"
        return bar
    }
    
    static ClearProgress() {
        ToolTip("", , , this.progressTooltip)
    }
    
    static ShowDebug(variable, value, label := "") {
        message := label ? label . ": " : ""
        message .= variable . " = " . value
        
        ToolTip(message, this.position.x, this.position.y + 60, this.debugTooltip)
        SetTimer(() => ToolTip("", , , this.debugTooltip), -2000)
    }
    
    static ShowTemporary(message, duration := 1500) {
        tempId := ToolTipManager.Show(message, this.position.x + 200, this.position.y, {
            timeout: duration,
            autoHide: true
        })
        return tempId
    }
}

; Interactive help system with tooltips
class HelpSystem {
    static helpData := Map()
    static currentHelp := 0
    static helpEnabled := true
    static hoverDelay := 1000
    static helpPosition := "cursor"
    
    static Initialize() {
        this.LoadHelpData()
        this.StartHoverDetection()
    }
    
    static LoadHelpData() {
        ; Define help information for UI elements
        this.helpData := Map(
            "Button1", "Click to save the current document",
            "Button2", "Click to open a new file",
            "EditBox", "Enter your text here - supports multi-line input",
            "ListView", "Double-click items to edit, right-click for context menu",
            "ComboBox", "Select from predefined options or type custom value"
        )
    }
    
    static RegisterHelp(elementName, helpText) {
        this.helpData[elementName] := helpText
    }
    
    static ShowElementHelp(elementName, x := "", y := "") {
        if (!this.helpEnabled || !this.helpData.Has(elementName)) {
            return
        }
        
        helpText := this.helpData[elementName]
        
        if (x = "" || y = "") {
            MouseGetPos(&mouseX, &mouseY)
            x := mouseX + 15
            y := mouseY + 15
        }
        
        this.currentHelp := ToolTipManager.Show(helpText, x, y, {
            timeout: 5000,
            autoHide: true,
            id: 10  ; Reserved for help system
        })
    }
    
    static HideHelp() {
        if (this.currentHelp) {
            ToolTipManager.Hide(10)
            this.currentHelp := 0
        }
    }
    
    static StartHoverDetection() {
        ; This would typically be integrated with GUI event handling
        ; For demonstration, showing the concept
        SetTimer(this.CheckHover.Bind(this), 250)
    }
    
    static CheckHover() {
        if (!this.helpEnabled) return
        
        ; Get window and control under cursor
        MouseGetPos(&x, &y, &windowId, &controlId)
        
        ; This is a simplified example - real implementation would
        ; integrate with GUI control detection
        static lastControl := ""
        static hoverStart := 0
        
        if (controlId != lastControl) {
            lastControl := controlId
            hoverStart := A_TickCount
            this.HideHelp()
        } else if (controlId && A_TickCount - hoverStart > this.hoverDelay) {
            ; Show help after hover delay
            this.ShowElementHelp(controlId, x, y)
        }
    }
    
    static EnableHelp() {
        this.helpEnabled := true
    }
    
    static DisableHelp() {
        this.helpEnabled := false
        this.HideHelp()
    }
    
    static SetHoverDelay(milliseconds) {
        this.hoverDelay := milliseconds
    }
}

; Development and debugging tooltip utilities
class DevTooltips {
    static debugLevel := 1
    static debugPosition := {x: A_ScreenWidth - 300, y: 10}
    static variableWatches := Map()
    static performanceMonitor := false
    static logTooltip := 15
    
    static Initialize() {
        this.StartPerformanceMonitoring()
    }
    
    static Debug(level, message, value := "") {
        if (level <= this.debugLevel) {
            debugText := "[DEBUG " . level . "] " . message
            if (value != "") {
                debugText .= ": " . value
            }
            
            ToolTip(debugText, this.debugPosition.x, this.debugPosition.y + (level * 20), this.logTooltip)
            SetTimer(() => ToolTip("", , , this.logTooltip), -3000)
        }
    }
    
    static WatchVariable(name, variableRef) {
        this.variableWatches[name] := variableRef
        this.UpdateWatches()
    }
    
    static UpdateWatches() {
        if (this.variableWatches.Count = 0) return
        
        watchText := "Variable Watches:`n"
        for name, varRef in this.variableWatches {
            watchText .= name . " = " . varRef[] . "`n"
        }
        
        ToolTip(watchText, this.debugPosition.x, this.debugPosition.y + 100, 16)
    }
    
    static StartPerformanceMonitoring() {
        if (this.performanceMonitor) return
        
        this.performanceMonitor := true
        SetTimer(this.ShowPerformanceInfo.Bind(this), 1000)
    }
    
    static ShowPerformanceInfo() {
        if (!this.performanceMonitor) return
        
        ; Get basic performance info
        memoryUsage := ProcessGetPhysicalMemoryUsage()
        
        perfText := "Performance Monitor:`n"
        perfText .= "Memory: " . Round(memoryUsage / 1024 / 1024, 1) . " MB`n"
        perfText .= "Tooltips: " . ToolTipManager.GetVisibleCount()
        
        ToolTip(perfText, this.debugPosition.x, this.debugPosition.y + 200, 17)
    }
    
    static StopPerformanceMonitoring() {
        this.performanceMonitor := false
        SetTimer(this.ShowPerformanceInfo.Bind(this), 0)
        ToolTip("", , , 17)
    }
    
    static SetDebugLevel(level) {
        this.debugLevel := level
    }
    
    static LogFunction(functionName, parameters := "") {
        logText := "CALL: " . functionName
        if (parameters) {
            logText .= "(" . parameters . ")"
        }
        
        this.Debug(2, logText)
    }
    
    static LogError(errorMessage, context := "") {
        errorText := "ERROR: " . errorMessage
        if (context) {
            errorText .= " in " . context
        }
        
        ToolTip(errorText, this.debugPosition.x, this.debugPosition.y - 30, 18)
        SetTimer(() => ToolTip("", , , 18), -5000)
    }
}

; Notification system using tooltips
class NotificationSystem {
    static notifications := []
    static maxNotifications := 5
    static notificationDelay := 300
    static basePosition := {x: A_ScreenWidth - 250, y: 50}
    
    static ShowNotification(title, message, type := "info", duration := 4000) {
        ; Create notification object
        notification := {
            title: title,
            message: message,
            type: type,
            duration: duration,
            showTime: A_TickCount,
            id: this.GetNextNotificationId()
        }
        
        ; Add to queue
        this.notifications.Push(notification)
        
        ; Remove old notifications if queue is full
        while (this.notifications.Length > this.maxNotifications) {
            this.HideNotification(this.notifications[1])
            this.notifications.RemoveAt(1)
        }
        
        ; Display notification
        this.DisplayNotification(notification)
        
        ; Auto-hide timer
        SetTimer(() => this.HideNotification(notification), -duration)
        
        return notification.id
    }
    
    static DisplayNotification(notification) {
        ; Find position in queue
        index := 0
        for i, notif in this.notifications {
            if (notif.id = notification.id) {
                index := i - 1
                break
            }
        }
        
        ; Calculate position
        x := this.basePosition.x
        y := this.basePosition.y + (index * 60)
        
        ; Format notification text
        icon := this.GetTypeIcon(notification.type)
        text := icon . " " . notification.title
        if (notification.message) {
            text .= "`n" . notification.message
        }
        
        ; Display tooltip
        ToolTip(text, x, y, notification.id)
    }
    
    static GetTypeIcon(type) {
        switch type {
            case "info":
                return "ℹ️"
            case "success":
                return "✅"
            case "warning":
                return "⚠️"
            case "error":
                return "❌"
            default:
                return "📢"
        }
    }
    
    static HideNotification(notification) {
        ToolTip("", , , notification.id)
        
        ; Remove from active notifications
        for i, notif in this.notifications {
            if (notif.id = notification.id) {
                this.notifications.RemoveAt(i)
                break
            }
        }
        
        ; Reposition remaining notifications
        this.RepositionNotifications()
    }
    
    static RepositionNotifications() {
        for i, notification in this.notifications {
            x := this.basePosition.x
            y := this.basePosition.y + ((i - 1) * 60)
            
            ; Update position without changing content
            ToolTip(this.GetNotificationText(notification), x, y, notification.id)
        }
    }
    
    static GetNotificationText(notification) {
        icon := this.GetTypeIcon(notification.type)
        text := icon . " " . notification.title
        if (notification.message) {
            text .= "`n" . notification.message
        }
        return text
    }
    
    static GetNextNotificationId() {
        ; Use tooltip IDs 11-20 for notifications
        static nextId := 11
        
        id := nextId
        nextId++
        if (nextId > 20) {
            nextId := 11  ; Wrap around
        }
        
        return id
    }
    
    static ClearAllNotifications() {
        for notification in this.notifications {
            ToolTip("", , , notification.id)
        }
        this.notifications := []
    }
    
    static ShowSuccess(title, message := "", duration := 3000) {
        return this.ShowNotification(title, message, "success", duration)
    }
    
    static ShowError(title, message := "", duration := 5000) {
        return this.ShowNotification(title, message, "error", duration)
    }
    
    static ShowWarning(title, message := "", duration := 4000) {
        return this.ShowNotification(title, message, "warning", duration)
    }
    
    static ShowInfo(title, message := "", duration := 3000) {
        return this.ShowNotification(title, message, "info", duration)
    }
}

; Example usage and demonstration
; Initialize tooltip systems
ToolTipManager.Initialize()
StatusDisplay.Initialize(10, 10)
HelpSystem.Initialize()
DevTooltips.Initialize()

; Basic tooltip examples
ToolTip("Simple tooltip")
Sleep(1000)
ToolTip("Positioned tooltip", 200, 100)
Sleep(1000)
ToolTip()  ; Hide tooltip

; Advanced tooltip management
statusId := ToolTipManager.Show("Processing data...", 10, 10, {timeout: 0})  ; No auto-hide
progressId := ToolTipManager.Show("0% Complete", 10, 40, {timeout: 0})

; Simulate progress updates
Loop 10 {
    ToolTipManager.Update(progressId, A_Index * 10 . "% Complete")
    Sleep(200)
}

ToolTipManager.Hide(statusId)
ToolTipManager.Hide(progressId)

; Status display system
StatusDisplay.ShowStatus("Application starting...", 2000)
Sleep(500)

; Progress display
Loop 20 {
    StatusDisplay.ShowProgress(A_Index, 20, "Loading")
    Sleep(100)
}
StatusDisplay.ClearProgress()

; Debug information
DevTooltips.Debug(1, "Variable check", "count = 5")
DevTooltips.LogFunction("ProcessData", "param1, param2")

; Notification examples
NotificationSystem.ShowSuccess("Task Complete", "File saved successfully")
NotificationSystem.ShowWarning("Low Memory", "Consider closing some applications")
NotificationSystem.ShowError("Connection Failed", "Unable to reach server")

; Help system demonstration
HelpSystem.RegisterHelp("SaveButton", "Save your current work to disk")
HelpSystem.ShowElementHelp("SaveButton")

; Multiple tooltips with different purposes
ToolTip("Main status: Ready", 10, 10, 1)
ToolTip("Memory: 45MB", 10, 30, 2)
ToolTip("Network: Connected", 10, 50, 3)
ToolTip("Debug: Variable X = 42", 200, 10, 4)

; Timed cleanup
SetTimer(() => {
    Loop 4 {
        ToolTip("", , , A_Index)
    }
}, -5000)

; Complex tooltip with formatted content
complexText := "System Status Report`n"
complexText .= "===================`n"
complexText .= "CPU Usage: 25%`n"
complexText .= "Memory: 2.1GB / 8GB`n"
complexText .= "Disk: 450GB / 1TB`n"
complexText .= "Network: 15.2 Mbps`n"
complexText .= "Uptime: 2 days, 14 hours"

ToolTip(complexText, A_ScreenWidth - 300, 100, 5)
SetTimer(() => ToolTip("", , , 5), -10000)
```

## Implementation Notes

**Performance Considerations:**
- Each tooltip consumes minimal system resources but scales with quantity and update frequency
- Frequent tooltip updates can impact visual smoothness, especially on slower systems
- Use timers efficiently to avoid excessive tooltip refreshing and CPU usage
- Consider tooltip lifecycle management to prevent memory leaks in long-running applications

**Position and Display Behavior:**
- Tooltips automatically adjust to stay within screen boundaries
- Multiple tooltips can overlap if positioned manually without collision detection
- System DPI scaling affects tooltip positioning accuracy on high-resolution displays
- Tooltip positioning near screen edges may require manual adjustment for optimal visibility

**Text Content and Formatting:**
- Long text content automatically wraps but may extend beyond visible screen area
- Newline characters enable multi-line tooltips with proper line spacing
- Unicode characters are supported for icons and special symbols in tooltip content
- Very long tooltips may be clipped by system limitations or screen boundaries

**Multi-Tooltip Management:**
- AutoHotkey supports up to 20 simultaneous tooltips with unique IDs
- Tooltip ID conflicts can cause unexpected hiding or overwriting of existing tooltips
- Proper tooltip lifecycle management prevents resource exhaustion and display conflicts
- Consider implementing tooltip pooling for applications with high tooltip turnover

**User Experience Guidelines:**
- Avoid excessive tooltip duration that may obstruct user workflow
- Provide clear visual distinction between different types of information tooltips
- Implement hover delays for help tooltips to prevent accidental triggering
- Ensure tooltip content is concise and immediately useful to the user

## Related AHK Concepts

- [TrayTip](../../30_Built_In_Classes/01-GUI_Classes/TrayTip/traytip.md) - System tray notifications
- [MsgBox](../../30_Built_In_Classes/01-GUI_Classes/MsgBox/msgbox.md) - Modal dialog boxes
- [Gui](../../30_Built_In_Classes/01-GUI_Classes/Gui/gui.md) - Custom user interface creation
- [SetTimer](../../40_Advanced_Features/03-Threading/settimer.md) - Automated tooltip management
- [MouseGetPos](../../40_Advanced_Features/00-Hotkeys_and_Input/mousegetpos.md) - Position-aware tooltip placement

## Tags

#AutoHotkey #ToolTip #UserInterface #InformationDisplay #StatusDisplay #Debugging #Notifications #UserFeedback #VisualDisplay #Development
