# AutoHotkey v2 Timers Deep Dive Research Report

## Basic SetTimer Syntax and Parameters

SetTimer causes a function to be called automatically and repeatedly at a specified time interval. The basic syntax in AutoHotkey v2 is:

```autohotkey
SetTimer(Function, Period, Priority)
```

**Parameters:**
- **Function**: A function object or reference to call. The function object is kept in the script's list of timers and is not released unless the timer is deleted
- **Period**: Integer specifying milliseconds between executions. The absolute value of Period must be no larger than 4294967295 ms (49.7 days)
- **Priority**: Optional integer between -2147483648 and 2147483647 for thread priority

**Basic Example:**
```autohotkey
; Simple timer calling a function every 1000ms
SetTimer(MyFunction, 1000)

MyFunction() {
    ToolTip("Timer fired at " . A_Now)
}
```

**Timer Precision:** Due to the granularity of the OS's time-keeping system, Period is typically rounded up to the nearest multiple of 10 or 15.6 milliseconds. For faster execution, alternative methods like Loop+Sleep with DllCall+timeBeginPeriod may be needed.

## One-time vs Recurring Timers

The sign of the Period parameter determines timer behavior:

**Recurring Timers (Positive Period):**
```autohotkey
; Repeats every 500ms until explicitly stopped
SetTimer(RepeatFunction, 500)

RepeatFunction() {
    static count := 0
    ToolTip("Execution #" . ++count)
}

; Stop the timer
SetTimer(RepeatFunction, 0)
```

**One-time Timers (Negative Period):**
If Period is less than 0, the timer will run only once. For example, specifying -100 would call Function 100 ms from now then delete the timer

```autohotkey
; Executes once after 2000ms, then self-deletes
SetTimer(OneTimeFunction, -2000)

OneTimeFunction() {
    MsgBox("This executes only once!")
    ; Timer automatically deletes itself after execution
}
```

**Use Cases:**
- **Recurring timers**: Status monitoring, auto-save, periodic cleanup, animation loops
- **One-time timers**: Delayed execution, timeouts, debouncing user input, cleanup after delays

## Using .Bind() with Class Methods

When using methods as timer functions, you need to create a function which encapsulates "this" and the method to call. AutoHotkey v2 provides two approaches:

**Method 1: Using .Bind()**
```autohotkey
class TimerExample {
    __New() {
        this.count := 0
        this.interval := 1000
        ; Bind method to this instance
        this.timerFunc := this.Tick.Bind(this)
    }
    
    Start() {
        SetTimer(this.timerFunc, this.interval)
        ToolTip("Timer started")
    }
    
    Stop() {
        ; Must pass the same bound function object
        SetTimer(this.timerFunc, 0)
        ToolTip("Timer stopped at count: " . this.count)
    }
    
    Tick() {
        this.count++
        ToolTip("Count: " . this.count)
    }
}

; Usage
counter := TimerExample()
counter.Start()
; ... later ...
counter.Stop()
```

**Method 2: Using ObjBindMethod() (Alternative)**
```autohotkey
class AlternativeTimer {
    __New() {
        this.count := 0
        ; Alternative binding method
        this.timer := ObjBindMethod(this, "Tick")
    }
    
    Start() {
        SetTimer(this.timer, 1000)
    }
    
    Stop() {
        SetTimer(this.timer, 0)
    }
    
    Tick() {
        ToolTip(++this.count)
    }
}
```

We can also use this.timer := this.Tick.Bind(this). When this.timer is called, it will effectively invoke tick_function.Call(this).

## Timer Cleanup Best Practices

Proper timer cleanup is crucial to prevent memory leaks and resource waste:

**1. Always Store Timer References**
```autohotkey
class SafeTimer {
    __New() {
        this.timerRef := this.Execute.Bind(this)
        this.isRunning := false
    }
    
    Start() {
        if (!this.isRunning) {
            SetTimer(this.timerRef, 1000)
            this.isRunning := true
        }
    }
    
    Stop() {
        if (this.isRunning) {
            SetTimer(this.timerRef, 0)
            this.isRunning := false
        }
    }
    
    Execute() {
        ; Timer logic here
    }
    
    ; Clean shutdown
    __Delete() {
        this.Stop()
    }
}
```

**2. Use Self-Deletion Pattern**
If Function is omitted, SetTimer will operate on the timer which launched the current thread. For example, SetTimer , 0 can be used inside a timer function to mark the timer for deletion

```autohotkey
CountdownTimer() {
    static count := 5
    
    if (count > 0) {
        ToolTip("Countdown: " . count--)
    } else {
        ToolTip("Done!")
        ; Self-delete the timer
        SetTimer(, 0)
    }
}

SetTimer(CountdownTimer, 1000)
```

**3. Prevent Multiple Timer Instances**
```autohotkey
class SingletonTimer {
    static instance := ""
    
    __New() {
        if (SingletonTimer.instance) {
            throw Error("Timer already running")
        }
        SingletonTimer.instance := this
        this.timerFunc := this.Execute.Bind(this)
    }
    
    Start() {
        SetTimer(this.timerFunc, 1000)
    }
    
    Stop() {
        SetTimer(this.timerFunc, 0)
        SingletonTimer.instance := ""
    }
    
    Execute() {
        ; Timer logic
    }
}
```

## Advanced Timer Patterns

**Conditional Timers**
```autohotkey
class ConditionalTimer {
    __New() {
        this.enabled := true
        this.timerFunc := this.Execute.Bind(this)
        SetTimer(this.timerFunc, 100)
    }
    
    Execute() {
        if (!this.enabled) {
            return  ; Skip execution but keep timer running
        }
        
        ; Conditional logic
        if (GetKeyState("Ctrl", "P")) {
            ; Do something when Ctrl is held
            ToolTip("Ctrl detected")
        }
    }
    
    Toggle() {
        this.enabled := !this.enabled
    }
}
```

**Timer Chaining**
```autohotkey
class ChainedTimer {
    __New() {
        this.stage := 1
        this.timerFunc := this.Execute.Bind(this)
    }
    
    Start() {
        this.stage := 1
        SetTimer(this.timerFunc, 1000)
    }
    
    Execute() {
        switch this.stage {
            case 1:
                ToolTip("Stage 1: Initialize")
                this.stage := 2
                SetTimer(this.timerFunc, 2000)  ; Change interval
                
            case 2:
                ToolTip("Stage 2: Process")
                this.stage := 3
                SetTimer(this.timerFunc, 500)   ; Change interval again
                
            case 3:
                ToolTip("Stage 3: Finalize")
                SetTimer(this.timerFunc, 0)     ; Stop timer
        }
    }
}
```

**Timer State Management**
```autohotkey
class StatefulTimer {
    __New() {
        this.state := "stopped"
        this.pausedAt := 0
        this.totalTime := 0
        this.timerFunc := this.Tick.Bind(this)
    }
    
    Start() {
        if (this.state == "paused") {
            this.Resume()
        } else {
            this.state := "running"
            this.startTime := A_TickCount
            SetTimer(this.timerFunc, 100)
        }
    }
    
    Pause() {
        if (this.state == "running") {
            this.state := "paused"
            this.pausedAt := A_TickCount
            SetTimer(this.timerFunc, 0)
        }
    }
    
    Resume() {
        if (this.state == "paused") {
            this.state := "running"
            this.totalTime += this.pausedAt - this.startTime
            this.startTime := A_TickCount
            SetTimer(this.timerFunc, 100)
        }
    }
    
    Stop() {
        this.state := "stopped"
        SetTimer(this.timerFunc, 0)
        this.totalTime := 0
    }
    
    Tick() {
        if (this.state == "running") {
            elapsed := A_TickCount - this.startTime + this.totalTime
            ToolTip("Elapsed: " . elapsed . "ms")
        }
    }
}
```

## Debugging and Common Pitfalls

**Common Issues:**

1. **Memory Leaks with Bound Functions**
If I am using SetTimer with a bound function, do I need to worry about deleting the bound function to prevent memory leaks? - Store bound functions in variables and clear them when done.

2. **Timer Precision Expectations**
A timer might not be able to run at the expected time under conditions like: Other applications putting heavy load on the CPU, the timer's function still running when the timer period expires again, too many competing timers

3. **Syntax Errors in v2**
SetTimer requires a function object, not a string label like in v1

**Debugging Techniques:**
```autohotkey
class DebugTimer {
    __New() {
        this.execCount := 0
        this.lastExecTime := 0
        this.timerFunc := this.DebugExecute.Bind(this)
    }
    
    DebugExecute() {
        currentTime := A_TickCount
        timeSinceLastExec := currentTime - this.lastExecTime
        this.execCount++
        
        ; Log timing information
        OutputDebug("Timer #" . this.execCount . 
                   " - Interval: " . timeSinceLastExec . "ms")
        
        this.lastExecTime := currentTime
        
        ; Your actual timer logic here
        this.ActualWork()
    }
    
    ActualWork() {
        ; Placeholder for actual timer functionality
        ToolTip("Timer executed " . this.execCount . " times")
    }
}
```

**Performance Monitoring:**
```autohotkey
; Monitor timer performance
class TimerProfiler {
    static timers := Map()
    
    static StartProfile(timerName) {
        TimerProfiler.timers[timerName] := {
            startTime: A_TickCount,
            execCount: 0,
            totalTime: 0
        }
    }
    
    static ProfileExecution(timerName, func) {
        if (!TimerProfiler.timers.Has(timerName))
            return
            
        startTime := A_TickCount
        func.Call()
        execTime := A_TickCount - startTime
        
        timer := TimerProfiler.timers[timerName]
        timer.execCount++
        timer.totalTime += execTime
        
        ; Log if execution is taking too long
        if (execTime > 50) {
            OutputDebug("Warning: " . timerName . " took " . execTime . "ms")
        }
    }
    
    static GetStats(timerName) {
        if (!TimerProfiler.timers.Has(timerName))
            return "Timer not found"
            
        timer := TimerProfiler.timers[timerName]
        avgTime := timer.totalTime / timer.execCount
        return "Executions: " . timer.execCount . 
               ", Avg time: " . Round(avgTime, 2) . "ms"
    }
}
```

## Summary

AutoHotkey v2 timers provide powerful asynchronous execution capabilities with several key improvements over v1. Critical points to remember:

- Always use function objects, not string labels
- Store timer references for proper cleanup
- Use negative periods for one-time execution
- Bind class methods properly to maintain context  
- Consider timer precision limitations (10-15ms granularity)
- Implement proper cleanup in class destructors
- Monitor performance for frequently executing timers

The .Bind() method is essential for class-based timer implementations, ensuring that 'this' context is preserved when the timer executes. Proper resource management through systematic cleanup prevents memory leaks and ensures reliable timer operation in long-running applications.