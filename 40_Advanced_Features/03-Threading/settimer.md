# Topic: SetTimer Function - Advanced Timing and Scheduling Control

## Category

Function

## Overview

The SetTimer function creates, modifies, and manages timed events and scheduled operations, providing precise control over automation timing, periodic tasks, and background processing. It's essential for scheduling systems, monitoring applications, periodic data collection, performance optimization, and any automation requiring sophisticated time-based event management.

## Key Points

- Creates periodic and one-shot timer events with precise timing control and priority management
- Supports advanced timer management including modification, deletion, and execution state control
- Provides foundation for automation scheduling, background processing, and time-based triggers
- Enables sophisticated timing frameworks including cron-like scheduling and complex timing patterns
- Essential for monitoring systems, data collection, performance optimization, and enterprise automation workflows

## Syntax and Parameters

```cpp
SetTimer([Function, Period, Priority])

; Parameters:
; Function  - Function object, method, or label to execute (optional)
; Period    - Timer interval in milliseconds (optional)
; Priority  - Timer thread priority (optional)

; Function parameter formats:
; SetTimer(MyFunction, 1000)     - Call MyFunction every 1000ms
; SetTimer(MyFunction, -5000)    - Call MyFunction once after 5000ms (negative = one-shot)
; SetTimer(MyFunction, 0)        - Disable/delete the timer
; SetTimer()                     - Delete the timer that called this function

; Period values:
; Positive number - Periodic timer (repeats every N milliseconds)
; Negative number - One-shot timer (runs once after |N| milliseconds)
; 0               - Disable/delete the timer
; Omitted         - Keep existing period when modifying timer

; Priority values:
; Integer from -2147483648 to 2147483647
; Higher numbers = higher priority
; Default priority is 0
; Negative priorities run after other threads

; Timer behavior:
; Minimum reliable interval: ~10ms (system dependent)
; Timers run in separate threads
; Function execution can delay subsequent timer calls
; System performance affects timer accuracy
```

## Code Examples

```cpp
; Basic timer usage
MyTimer() {
    ToolTip("Timer fired: " . A_TickCount)
}

; Start periodic timer (every 2 seconds)
SetTimer(MyTimer, 2000)

; Start one-shot timer (5 seconds from now)
SetTimer(MyTimer, -5000)

; Stop the timer
SetTimer(MyTimer, 0)

; Advanced timer management and scheduling system
class TimerManager {
    static timers := Map()
    static timerStats := Map()
    static globalEnabled := true
    static debugMode := false
    static maxConcurrentTimers := 100
    static defaultPriority := 0
    
    static Initialize() {
        this.timers.Clear()
        this.timerStats.Clear()
        this.StartSystemMonitoring()
    }
    
    static CreateTimer(name, callback, interval, options := {}) {
        ; Advanced timer creation with comprehensive options
        if (this.timers.Has(name)) {
            throw ValueError("Timer '" . name . "' already exists")
        }
        
        if (this.timers.Count >= this.maxConcurrentTimers) {
            throw Error("Maximum concurrent timers exceeded: " . this.maxConcurrentTimers)
        }
        
        ; Parse options
        priority := options.HasProp("priority") ? options.priority : this.defaultPriority
        oneShot := options.HasProp("oneShot") ? options.oneShot : false
        autoStart := options.HasProp("autoStart") ? options.autoStart : true
        enabled := options.HasProp("enabled") ? options.enabled : true
        maxExecutions := options.HasProp("maxExecutions") ? options.maxExecutions : 0
        timeout := options.HasProp("timeout") ? options.timeout : 0
        retryOnError := options.HasProp("retryOnError") ? options.retryOnError : false
        maxRetries := options.HasProp("maxRetries") ? options.maxRetries : 3
        
        ; Create timer info object
        timerInfo := {
            name: name,
            callback: callback,
            interval: interval,
            priority: priority,
            oneShot: oneShot,
            enabled: enabled,
            running: false,
            created: A_TickCount,
            lastExecuted: 0,
            executionCount: 0,
            maxExecutions: maxExecutions,
            timeout: timeout,
            retryOnError: retryOnError,
            maxRetries: maxRetries,
            currentRetries: 0,
            totalErrors: 0,
            averageExecutionTime: 0,
            maxExecutionTime: 0,
            minExecutionTime: 999999999
        }
        
        ; Create wrapper function for execution tracking
        wrapperFunction := this.CreateTimerWrapper(name, timerInfo)
        timerInfo.wrapperFunction := wrapperFunction
        
        ; Store timer
        this.timers[name] := timerInfo
        this.timerStats[name] := {
            totalExecutions: 0,
            totalTime: 0,
            errors: [],
            performanceHistory: []
        }
        
        ; Start timer if auto-start enabled
        if (autoStart && enabled) {
            this.StartTimer(name)
        }
        
        return timerInfo
    }
    
    static CreateTimerWrapper(name, timerInfo) {
        ; Create execution wrapper with error handling and statistics
        return () => {
            if (!this.globalEnabled || !timerInfo.enabled) return
            
            ; Check maximum executions
            if (timerInfo.maxExecutions > 0 && timerInfo.executionCount >= timerInfo.maxExecutions) {
                this.StopTimer(name)
                return
            }
            
            ; Check timeout
            if (timerInfo.timeout > 0 && A_TickCount - timerInfo.created > timerInfo.timeout) {
                this.StopTimer(name)
                return
            }
            
            startTime := A_TickCount
            
            try {
                ; Execute the actual callback
                if (IsFunc(timerInfo.callback)) {
                    result := timerInfo.callback.Call()
                } else {
                    throw Error("Invalid callback function")
                }
                
                ; Update execution statistics
                executionTime := A_TickCount - startTime
                timerInfo.lastExecuted := startTime
                timerInfo.executionCount++
                timerInfo.currentRetries := 0  ; Reset retry count on success
                
                ; Update performance metrics
                this.UpdatePerformanceMetrics(name, executionTime)
                
                ; Stop if one-shot timer
                if (timerInfo.oneShot) {
                    this.StopTimer(name)
                }
                
            } catch Error as err {
                timerInfo.totalErrors++
                
                ; Log error
                if (this.debugMode) {
                    this.LogError(name, err.Message)
                }
                
                ; Handle retry logic
                if (timerInfo.retryOnError && timerInfo.currentRetries < timerInfo.maxRetries) {
                    timerInfo.currentRetries++
                    if (this.debugMode) {
                        OutputDebug("Timer '" . name . "' retry " . timerInfo.currentRetries . "/" . timerInfo.maxRetries)
                    }
                } else {
                    ; Max retries exceeded or no retry - stop timer
                    this.StopTimer(name)
                    this.NotifyTimerError(name, err)
                }
            }
        }
    }
    
    static StartTimer(name) {
        if (!this.timers.Has(name)) {
            throw ValueError("Timer '" . name . "' not found")
        }
        
        timerInfo := this.timers[name]
        
        if (timerInfo.running) {
            return false  ; Already running
        }
        
        ; Calculate actual interval (negative for one-shot)
        actualInterval := timerInfo.oneShot ? -timerInfo.interval : timerInfo.interval
        
        ; Start the system timer
        SetTimer(timerInfo.wrapperFunction, actualInterval, timerInfo.priority)
        
        timerInfo.running := true
        timerInfo.lastExecuted := 0
        
        if (this.debugMode) {
            OutputDebug("Timer '" . name . "' started with interval " . actualInterval . "ms")
        }
        
        return true
    }
    
    static StopTimer(name) {
        if (!this.timers.Has(name)) {
            return false
        }
        
        timerInfo := this.timers[name]
        
        if (!timerInfo.running) {
            return false  ; Already stopped
        }
        
        ; Stop the system timer
        SetTimer(timerInfo.wrapperFunction, 0)
        
        timerInfo.running := false
        
        if (this.debugMode) {
            OutputDebug("Timer '" . name . "' stopped")
        }
        
        return true
    }
    
    static DeleteTimer(name) {
        if (!this.timers.Has(name)) {
            return false
        }
        
        ; Stop timer first
        this.StopTimer(name)
        
        ; Remove from collections
        this.timers.Delete(name)
        this.timerStats.Delete(name)
        
        if (this.debugMode) {
            OutputDebug("Timer '" . name . "' deleted")
        }
        
        return true
    }
    
    static ModifyTimer(name, newInterval := "", newPriority := "") {
        if (!this.timers.Has(name)) {
            throw ValueError("Timer '" . name . "' not found")
        }
        
        timerInfo := this.timers[name]
        wasRunning := timerInfo.running
        
        ; Stop timer for modification
        if (wasRunning) {
            this.StopTimer(name)
        }
        
        ; Update properties
        if (newInterval != "") {
            timerInfo.interval := Abs(newInterval)
            timerInfo.oneShot := newInterval < 0
        }
        
        if (newPriority != "") {
            timerInfo.priority := newPriority
        }
        
        ; Restart if it was running
        if (wasRunning) {
            this.StartTimer(name)
        }
        
        return true
    }
    
    static EnableTimer(name) {
        if (!this.timers.Has(name)) {
            return false
        }
        
        this.timers[name].enabled := true
        return true
    }
    
    static DisableTimer(name) {
        if (!this.timers.Has(name)) {
            return false
        }
        
        timerInfo := this.timers[name]
        timerInfo.enabled := false
        
        ; Stop if currently running
        if (timerInfo.running) {
            this.StopTimer(name)
        }
        
        return true
    }
    
    static UpdatePerformanceMetrics(name, executionTime) {
        if (!this.timers.Has(name) || !this.timerStats.Has(name)) {
            return
        }
        
        timerInfo := this.timers[name]
        stats := this.timerStats[name]
        
        ; Update timer info metrics
        timerInfo.averageExecutionTime := (timerInfo.averageExecutionTime * (timerInfo.executionCount - 1) + executionTime) / timerInfo.executionCount
        timerInfo.maxExecutionTime := Max(timerInfo.maxExecutionTime, executionTime)
        timerInfo.minExecutionTime := Min(timerInfo.minExecutionTime, executionTime)
        
        ; Update stats
        stats.totalExecutions++
        stats.totalTime += executionTime
        
        ; Add to performance history (limited to last 100 executions)
        stats.performanceHistory.Push({
            timestamp: A_TickCount,
            executionTime: executionTime
        })
        
        if (stats.performanceHistory.Length > 100) {
            stats.performanceHistory.RemoveAt(1)
        }
    }
    
    static GetTimerInfo(name) {
        return this.timers.Has(name) ? this.timers[name].Clone() : {}
    }
    
    static GetTimerStatistics(name) {
        return this.timerStats.Has(name) ? this.timerStats[name].Clone() : {}
    }
    
    static GetAllTimers() {
        timerList := []
        
        for name, timerInfo in this.timers {
            timerList.Push({
                name: name,
                running: timerInfo.running,
                enabled: timerInfo.enabled,
                interval: timerInfo.interval,
                executions: timerInfo.executionCount,
                errors: timerInfo.totalErrors,
                avgTime: Round(timerInfo.averageExecutionTime, 2)
            })
        }
        
        return timerList
    }
    
    static StartSystemMonitoring() {
        ; Monitor system timer performance
        SetTimer(() => this.MonitorTimerHealth(), 5000)  ; Check every 5 seconds
    }
    
    static MonitorTimerHealth() {
        ; Check for problematic timers
        for name, timerInfo in this.timers {
            ; Check for timers with high error rates
            if (timerInfo.executionCount > 10 && timerInfo.totalErrors / timerInfo.executionCount > 0.5) {
                if (this.debugMode) {
                    OutputDebug("Warning: Timer '" . name . "' has high error rate")
                }
            }
            
            ; Check for timers with long execution times
            if (timerInfo.maxExecutionTime > timerInfo.interval * 0.8) {
                if (this.debugMode) {
                    OutputDebug("Warning: Timer '" . name . "' execution time approaching interval")
                }
            }
        }
    }
    
    static LogError(timerName, errorMessage) {
        if (this.timerStats.Has(timerName)) {
            this.timerStats[timerName].errors.Push({
                timestamp: A_TickCount,
                message: errorMessage
            })
        }
    }
    
    static NotifyTimerError(timerName, error) {
        ; Override this method to implement custom error notification
        if (this.debugMode) {
            OutputDebug("Timer '" . timerName . "' stopped due to error: " . error.Message)
        }
    }
    
    static EnableGlobal() {
        this.globalEnabled := true
    }
    
    static DisableGlobal() {
        this.globalEnabled := false
    }
    
    static EnableDebugMode() {
        this.debugMode := true
    }
    
    static DisableDebugMode() {
        this.debugMode := false
    }
    
    static StopAllTimers() {
        for name in this.timers {
            this.StopTimer(name)
        }
    }
    
    static DeleteAllTimers() {
        for name in this.timers {
            this.DeleteTimer(name)
        }
    }
    
    static GenerateReport() {
        report := "Timer Management Report`n"
        report .= "======================`n`n"
        
        report .= "Summary:`n"
        report .= "Total Timers: " . this.timers.Count . "`n"
        
        runningCount := 0
        enabledCount := 0
        
        for name, timerInfo in this.timers {
            if (timerInfo.running) runningCount++
            if (timerInfo.enabled) enabledCount++
        }
        
        report .= "Running: " . runningCount . "`n"
        report .= "Enabled: " . enabledCount . "`n"
        report .= "Global Enabled: " . (this.globalEnabled ? "Yes" : "No") . "`n`n"
        
        report .= "Timer Details:`n"
        report .= "Name`tStatus`tInterval`tExecutions`tErrors`tAvg Time`n"
        
        for name, timerInfo in this.timers {
            status := timerInfo.running ? "Running" : (timerInfo.enabled ? "Stopped" : "Disabled")
            report .= name . "`t" . status . "`t" . timerInfo.interval . "ms`t" . timerInfo.executionCount . "`t" . timerInfo.totalErrors . "`t" . Round(timerInfo.averageExecutionTime, 1) . "ms`n"
        }
        
        return report
    }
}

; Cron-like scheduling system
class CronScheduler {
    static schedules := Map()
    static running := false
    static checkInterval := 1000  ; Check every second
    
    static Initialize() {
        this.schedules.Clear()
        this.Start()
    }
    
    static AddSchedule(name, cronExpression, callback, options := {}) {
        ; Add cron-like scheduled task
        ; Format: "minute hour day month dayofweek"
        ; Examples: "0 9 * * 1-5" (9 AM, weekdays)
        ;          "30 14 * * *" (2:30 PM daily)
        ;          "0 0 1 * *" (midnight, first day of month)
        
        schedule := {
            name: name,
            expression: cronExpression,
            callback: callback,
            enabled: options.HasProp("enabled") ? options.enabled : true,
            timezone: options.HasProp("timezone") ? options.timezone : "Local",
            lastRun: 0,
            nextRun: 0,
            runCount: 0,
            maxRuns: options.HasProp("maxRuns") ? options.maxRuns : 0
        }
        
        ; Calculate next run time
        schedule.nextRun := this.CalculateNextRun(cronExpression)
        
        this.schedules[name] := schedule
        return schedule
    }
    
    static CalculateNextRun(cronExpression) {
        ; Simplified cron calculation (full implementation would be more complex)
        parts := StrSplit(cronExpression, " ")
        
        if (parts.Length != 5) {
            throw ValueError("Invalid cron expression format")
        }
        
        ; For demonstration, calculate next run based on simple patterns
        currentTime := A_Now
        
        ; Add basic logic for common patterns
        if (cronExpression = "0 9 * * 1-5") {
            ; 9 AM on weekdays
            return this.NextWeekdayTime(9, 0)
        } else if (cronExpression = "30 14 * * *") {
            ; 2:30 PM daily
            return this.NextDailyTime(14, 30)
        } else {
            ; Default: next hour
            return DateAdd(currentTime, 1, "Hours")
        }
    }
    
    static NextWeekdayTime(hour, minute) {
        currentTime := A_Now
        targetTime := FormatTime(currentTime, "yyyyMMdd") . Format("{:02d}{:02d}00", hour, minute)
        
        ; If target time has passed today, move to tomorrow
        if (targetTime <= currentTime) {
            targetTime := DateAdd(targetTime, 1, "Days")
        }
        
        ; Skip weekends
        dayOfWeek := FormatTime(targetTime, "WDay")
        while (dayOfWeek = 1 || dayOfWeek = 7) {  ; Sunday = 1, Saturday = 7
            targetTime := DateAdd(targetTime, 1, "Days")
            dayOfWeek := FormatTime(targetTime, "WDay")
        }
        
        return targetTime
    }
    
    static NextDailyTime(hour, minute) {
        currentTime := A_Now
        targetTime := FormatTime(currentTime, "yyyyMMdd") . Format("{:02d}{:02d}00", hour, minute)
        
        ; If target time has passed today, move to tomorrow
        if (targetTime <= currentTime) {
            targetTime := DateAdd(targetTime, 1, "Days")
        }
        
        return targetTime
    }
    
    static Start() {
        if (this.running) return
        
        this.running := true
        TimerManager.CreateTimer("CronChecker", this.CheckSchedules.Bind(this), this.checkInterval)
    }
    
    static Stop() {
        this.running := false
        TimerManager.DeleteTimer("CronChecker")
    }
    
    static CheckSchedules() {
        if (!this.running) return
        
        currentTime := A_Now
        
        for name, schedule in this.schedules {
            if (!schedule.enabled) continue
            
            ; Check if it's time to run
            if (currentTime >= schedule.nextRun) {
                this.ExecuteSchedule(name, schedule)
            }
        }
    }
    
    static ExecuteSchedule(name, schedule) {
        try {
            ; Execute the callback
            schedule.callback.Call()
            
            ; Update statistics
            schedule.lastRun := A_Now
            schedule.runCount++
            
            ; Calculate next run
            schedule.nextRun := this.CalculateNextRun(schedule.expression)
            
            ; Check max runs
            if (schedule.maxRuns > 0 && schedule.runCount >= schedule.maxRuns) {
                schedule.enabled := false
            }
            
        } catch Error as err {
            ; Log error but continue with other schedules
            OutputDebug("Cron schedule '" . name . "' failed: " . err.Message)
        }
    }
    
    static GetScheduleInfo(name) {
        return this.schedules.Has(name) ? this.schedules[name].Clone() : {}
    }
    
    static EnableSchedule(name) {
        if (this.schedules.Has(name)) {
            this.schedules[name].enabled := true
            return true
        }
        return false
    }
    
    static DisableSchedule(name) {
        if (this.schedules.Has(name)) {
            this.schedules[name].enabled := false
            return true
        }
        return false
    }
    
    static DeleteSchedule(name) {
        return this.schedules.Delete(name)
    }
}

; Performance monitoring and optimization
class TimerPerformanceMonitor {
    static performanceData := Map()
    static monitoring := false
    static alertThresholds := {
        maxExecutionTime: 1000,  ; ms
        maxErrorRate: 0.1,       ; 10%
        maxMemoryUsage: 100000000 ; 100MB
    }
    
    static StartMonitoring() {
        if (this.monitoring) return
        
        this.monitoring := true
        
        ; Monitor timer performance every 10 seconds
        TimerManager.CreateTimer("PerformanceMonitor", this.CollectPerformanceData.Bind(this), 10000)
    }
    
    static StopMonitoring() {
        this.monitoring := false
        TimerManager.DeleteTimer("PerformanceMonitor")
    }
    
    static CollectPerformanceData() {
        currentTime := A_TickCount
        
        ; Collect system metrics
        systemMetrics := {
            timestamp: currentTime,
            memoryUsage: this.GetMemoryUsage(),
            cpuUsage: this.GetCpuUsage(),
            activeTimers: TimerManager.timers.Count
        }
        
        ; Store metrics
        if (!this.performanceData.Has("system")) {
            this.performanceData["system"] := []
        }
        
        this.performanceData["system"].Push(systemMetrics)
        
        ; Limit history
        if (this.performanceData["system"].Length > 100) {
            this.performanceData["system"].RemoveAt(1)
        }
        
        ; Check for performance issues
        this.CheckPerformanceAlerts()
    }
    
    static CheckPerformanceAlerts() {
        ; Check individual timers
        for name, timerInfo in TimerManager.timers {
            ; Check execution time
            if (timerInfo.maxExecutionTime > this.alertThresholds.maxExecutionTime) {
                this.TriggerAlert("EXECUTION_TIME", name, timerInfo.maxExecutionTime)
            }
            
            ; Check error rate
            if (timerInfo.executionCount > 10) {
                errorRate := timerInfo.totalErrors / timerInfo.executionCount
                if (errorRate > this.alertThresholds.maxErrorRate) {
                    this.TriggerAlert("ERROR_RATE", name, errorRate)
                }
            }
        }
        
        ; Check system memory
        memoryUsage := this.GetMemoryUsage()
        if (memoryUsage > this.alertThresholds.maxMemoryUsage) {
            this.TriggerAlert("MEMORY_USAGE", "system", memoryUsage)
        }
    }
    
    static TriggerAlert(alertType, source, value) {
        ; Override this method to implement custom alerting
        alertMessage := "Performance Alert: " . alertType . " for " . source . " = " . value
        OutputDebug(alertMessage)
    }
    
    static GetMemoryUsage() {
        ; Simplified memory usage (real implementation would use WMI)
        return Random(50000000, 150000000)  ; Placeholder
    }
    
    static GetCpuUsage() {
        ; Simplified CPU usage (real implementation would use performance counters)
        return Random(5, 25)  ; Placeholder
    }
    
    static GetPerformanceReport() {
        report := "Timer Performance Report`n"
        report .= "========================`n`n"
        
        ; System overview
        if (this.performanceData.Has("system") && this.performanceData["system"].Length > 0) {
            latest := this.performanceData["system"][-1]
            report .= "Current System Status:`n"
            report .= "Memory Usage: " . Round(latest.memoryUsage / 1024 / 1024, 1) . " MB`n"
            report .= "CPU Usage: " . latest.cpuUsage . "%`n"
            report .= "Active Timers: " . latest.activeTimers . "`n`n"
        }
        
        ; Timer performance details
        report .= "Timer Performance:`n"
        report .= "Name`tAvg Time`tMax Time`tError Rate`tStatus`n"
        
        for name, timerInfo in TimerManager.timers {
            errorRate := timerInfo.executionCount > 0 ? Round(timerInfo.totalErrors / timerInfo.executionCount * 100, 1) : 0
            status := timerInfo.running ? "Running" : "Stopped"
            
            report .= name . "`t" . Round(timerInfo.averageExecutionTime, 1) . "ms`t" . timerInfo.maxExecutionTime . "ms`t" . errorRate . "%`t" . status . "`n"
        }
        
        return report
    }
}

; Example usage and comprehensive demonstrations
; Initialize timer management system
TimerManager.Initialize()
TimerManager.EnableDebugMode()

; Basic SetTimer usage
BasicTimer() {
    ToolTip("Basic timer executed at " . A_TickCount)
    SetTimer(() => ToolTip(), -2000)  ; Hide tooltip after 2 seconds
}

; Start basic timer every 3 seconds
SetTimer(BasicTimer, 3000)

; Advanced timer management examples
; Create a data collection timer
DataCollector() {
    ; Simulate data collection
    data := "Data point: " . Random(1, 100)
    FileAppend(FormatTime() . " - " . data . "`n", "data_log.txt")
}

TimerManager.CreateTimer("DataCollector", DataCollector, 5000, {
    priority: 1,
    maxExecutions: 100,
    retryOnError: true
})

; Create a system health monitor
HealthMonitor() {
    ; Check system health
    memUsage := ProcessGetMemoryUsage()
    if (memUsage > 500000000) {  ; 500MB threshold
        MsgBox("High memory usage detected: " . Round(memUsage/1024/1024) . " MB")
    }
}

TimerManager.CreateTimer("HealthMonitor", HealthMonitor, 30000, {
    priority: 2,
    autoStart: true
})

; Create one-shot notification timer
WelcomeMessage() {
    MsgBox("Welcome! This application has been running for 30 seconds.")
}

TimerManager.CreateTimer("WelcomeMessage", WelcomeMessage, 30000, {
    oneShot: true,
    priority: 0
})

; Cron-like scheduling examples
CronScheduler.Initialize()

; Daily backup at 2:30 PM
DailyBackup() {
    ; Perform backup operations
    MsgBox("Daily backup started at " . FormatTime())
}

CronScheduler.AddSchedule("DailyBackup", "30 14 * * *", DailyBackup)

; Weekday report at 9 AM
WeekdayReport() {
    ; Generate morning report
    report := TimerManager.GenerateReport()
    FileWrite(report, "morning_report_" . FormatTime(, "yyyyMMdd") . ".txt")
}

CronScheduler.AddSchedule("WeekdayReport", "0 9 * * 1-5", WeekdayReport)

; Performance monitoring
TimerPerformanceMonitor.StartMonitoring()

; Complex timer coordination example
class ApplicationScheduler {
    static Initialize() {
        ; Create coordinated timers for application lifecycle
        
        ; High-frequency monitoring (every 100ms)
        TimerManager.CreateTimer("HighFreqMonitor", this.HighFrequencyCheck.Bind(this), 100, {
            priority: 3
        })
        
        ; Medium-frequency tasks (every 5 seconds)
        TimerManager.CreateTimer("MediumFreqTasks", this.MediumFrequencyTasks.Bind(this), 5000, {
            priority: 1
        })
        
        ; Low-frequency maintenance (every 5 minutes)
        TimerManager.CreateTimer("MaintenanceTasks", this.MaintenanceTasks.Bind(this), 300000, {
            priority: -1
        })
        
        ; Status update (every 10 seconds)
        TimerManager.CreateTimer("StatusUpdate", this.UpdateStatus.Bind(this), 10000)
    }
    
    static HighFrequencyCheck() {
        ; Critical real-time monitoring
        ; Keep this function very lightweight
        static lastCheck := 0
        currentTime := A_TickCount
        
        if (currentTime - lastCheck > 1000) {  ; Log every second
            OutputDebug("High frequency check: " . currentTime)
            lastCheck := currentTime
        }
    }
    
    static MediumFrequencyTasks() {
        ; Regular application tasks
        ; Update configurations, check connections, etc.
        
        ; Example: Check if critical processes are running
        criticalProcesses := ["notepad.exe", "explorer.exe"]
        for processName in criticalProcesses {
            if (!ProcessExist(processName)) {
                OutputDebug("Warning: Critical process not found: " . processName)
            }
        }
    }
    
    static MaintenanceTasks() {
        ; Periodic maintenance operations
        ; Cleanup, optimization, garbage collection, etc.
        
        ; Example: Clean up old log files
        Loop Files, "*.log" {
            if (A_LoopFileTimeModified < DateAdd(A_Now, -7, "Days")) {
                try {
                    FileDelete(A_LoopFileFullPath)
                    OutputDebug("Deleted old log file: " . A_LoopFileFullPath)
                } catch Error {
                    ; Continue with other files
                }
            }
        }
    }
    
    static UpdateStatus() {
        ; Update application status
        status := "Application Status:`n"
        status .= "Active Timers: " . TimerManager.timers.Count . "`n"
        status .= "System Time: " . FormatTime() . "`n"
        status .= "Uptime: " . Round(A_TickCount / 1000 / 60, 1) . " minutes"
        
        ToolTip(status, 10, 10, 20)
        SetTimer(() => ToolTip("", , , 20), -5000)
    }
}

; Initialize application scheduler
ApplicationScheduler.Initialize()

; Timer modification examples
; Modify existing timer interval
TimerManager.ModifyTimer("DataCollector", 10000)  ; Change to 10 seconds

; Temporarily disable timer
TimerManager.DisableTimer("HealthMonitor")

; Re-enable timer
TimerManager.EnableTimer("HealthMonitor")

; Stop all timers for maintenance
; TimerManager.StopAllTimers()

; Generate comprehensive reports
timerReport := TimerManager.GenerateReport()
performanceReport := TimerPerformanceMonitor.GetPerformanceReport()

; Display reports (uncomment to show)
; MsgBox(timerReport, "Timer Report")
; MsgBox(performanceReport, "Performance Report")

; Cleanup example (commented out to keep timers running)
; Clean up specific timer
; TimerManager.DeleteTimer("WelcomeMessage")

; Clean up all timers and systems
; TimerManager.DeleteAllTimers()
; CronScheduler.Stop()
; TimerPerformanceMonitor.StopMonitoring()

; Advanced error handling example
ErrorProneTimer() {
    ; Simulate random errors for demonstration
    if (Random(1, 10) > 8) {
        throw Error("Simulated error for testing")
    }
    
    OutputDebug("Error-prone timer executed successfully")
}

TimerManager.CreateTimer("ErrorProneTimer", ErrorProneTimer, 2000, {
    retryOnError: true,
    maxRetries: 3,
    maxExecutions: 50
})

; Precision timing example for performance-critical applications
PrecisionTimer() {
    static lastTime := A_TickCount
    currentTime := A_TickCount
    drift := currentTime - lastTime - 100  ; Expected 100ms interval
    
    if (Abs(drift) > 5) {  ; Alert if drift > 5ms
        OutputDebug("Timer drift detected: " . drift . "ms")
    }
    
    lastTime := currentTime
}

TimerManager.CreateTimer("PrecisionTimer", PrecisionTimer, 100, {
    priority: 10  ; Highest priority
})
```

## Implementation Notes

**Timer Precision and Accuracy:**
- Windows timer resolution is typically 15.6ms, affecting timer accuracy for intervals shorter than ~16ms
- System load and other high-priority processes can cause timer drift and delayed execution
- Use high-priority threads for time-critical operations, but avoid overusing high priorities
- Consider using multimedia timers or QueryPerformanceCounter for microsecond precision requirements

**Memory and Resource Management:**
- Each timer consumes system resources and thread handles, monitor timer count in long-running applications
- Timer callbacks should execute quickly to prevent blocking other timers and system responsiveness
- Implement proper cleanup and timer deletion to prevent resource leaks in complex applications
- Monitor timer execution times and optimize or split long-running operations into smaller chunks

**Threading and Concurrency:**
- Timer callbacks execute in separate threads, requiring thread-safe code for shared data access
- Avoid GUI operations directly in timer callbacks; use PostMessage or similar mechanisms instead
- Multiple timers can execute simultaneously, requiring synchronization for shared resources
- Consider timer dependencies and execution order when coordinating multiple timed operations

**Error Handling and Reliability:**
- Timer callbacks should include comprehensive error handling to prevent timer termination
- Implement retry mechanisms for transient errors and logging for persistent issues
- Consider graceful degradation strategies when critical timers fail or encounter errors
- Monitor timer health and performance to detect and resolve issues proactively

**Performance Optimization:**
- Group multiple short-interval timers into fewer longer-interval timers when possible
- Use appropriate priorities to ensure critical timers execute reliably under system load
- Implement timer pooling and reuse for applications with dynamic timer creation and deletion
- Consider using timer coalescence and batching techniques for improved system efficiency

## Related AHK Concepts

- [Sleep](../../10_Language_Core/01-Functions/Built_In_Functions/sleep.md) - Simple delays and timing
- [Critical](../../10_Language_Core/01-Functions/Built_In_Functions/critical.md) - Thread synchronization for timer callbacks
- [OnExit](../../10_Language_Core/01-Functions/Built_In_Functions/onexit.md) - Cleanup handling for timer resources
- [A_TickCount](../../00_Fundamentals/00-Variables_and_Expressions/builtin-variables.md) - High-precision timing measurements
- [ProcessSetPriority](../../40_Advanced_Features/05-System_Integration/processsetpriority.md) - Process priority for timer performance

## Tags

#AutoHotkey #SetTimer #Timing #Scheduling #Threading #BackgroundProcessing #Automation #PerformanceMonitoring #TaskScheduling #SystemIntegration
