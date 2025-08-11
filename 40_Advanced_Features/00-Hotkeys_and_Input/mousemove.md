# Topic: MouseMove Function - Precise Mouse Cursor Control

## Category

Function

## Overview

The MouseMove function moves the mouse cursor to specified screen coordinates with precise control over speed and movement patterns. It's essential for automation scripts, user interface testing, accessibility applications, and any scenario requiring programmatic mouse cursor positioning.

## Key Points

- Moves mouse cursor to precise screen coordinates with customizable speed and acceleration
- Supports absolute positioning, relative movement, and multiple coordinate systems
- Provides natural movement simulation with speed control for human-like automation
- Enables complex mouse movement patterns and multi-monitor support
- Essential for GUI automation, accessibility tools, gaming applications, and user interface testing

## Syntax and Parameters

```cpp
MouseMove(X, Y [, Speed, Relative])

; Parameters:
; X        - Target X coordinate (pixels)
; Y        - Target Y coordinate (pixels)
; Speed    - Movement speed 0-100 (0=instant, 100=slowest, default: uses SetDefaultMouseSpeed)
; Relative - "R" or "Rel" for relative movement (default: absolute positioning)

; Coordinate systems (affected by CoordMode):
; Screen  - Coordinates relative to entire screen (default)
; Window  - Coordinates relative to active window's client area
; Client  - Coordinates relative to active window's client area

; Speed behavior:
; 0      - Instant movement (no animation)
; 1-100  - Animated movement (higher = slower)
; -1     - Use default speed from SetDefaultMouseSpeed
```

## Code Examples

```cpp
; Basic mouse movement
MouseMove(500, 300)  ; Move to screen coordinates (500, 300)

; Movement with speed control
MouseMove(800, 600, 25)  ; Move slowly to (800, 600)

; Relative movement
MouseMove(50, -30, 10, "Rel")  ; Move 50 pixels right, 30 pixels up

; Instant movement
MouseMove(0, 0, 0)  ; Move instantly to top-left corner

; Complex mouse automation framework
class MouseController {
    static defaultSpeed := 25
    static currentPos := {x: 0, y: 0}
    static isRecording := false
    static recordedMoves := []
    static smoothingEnabled := true
    
    static Initialize() {
        ; Set default mouse speed
        SetDefaultMouseSpeed(this.defaultSpeed)
        
        ; Update current position
        this.UpdateCurrentPosition()
    }
    
    static UpdateCurrentPosition() {
        MouseGetPos(&x, &y)
        this.currentPos := {x: x, y: y}
    }
    
    static MoveTo(x, y, speed := -1, coordinateMode := "Screen") {
        ; Set coordinate mode
        oldMode := CoordMode("Mouse")
        CoordMode("Mouse", coordinateMode)
        
        try {
            ; Apply smoothing if enabled
            if (this.smoothingEnabled && speed > 0) {
                this.SmoothMoveTo(x, y, speed)
            } else {
                MouseMove(x, y, speed)
            }
            
            ; Update tracked position
            this.UpdateCurrentPosition()
            
            ; Record movement if recording
            if (this.isRecording) {
                this.recordedMoves.Push({
                    type: "move",
                    x: x,
                    y: y,
                    speed: speed,
                    timestamp: A_TickCount
                })
            }
            
        } finally {
            ; Restore coordinate mode
            CoordMode("Mouse", oldMode)
        }
    }
    
    static SmoothMoveTo(targetX, targetY, speed) {
        this.UpdateCurrentPosition()
        startX := this.currentPos.x
        startY := this.currentPos.y
        
        ; Calculate distance and steps
        deltaX := targetX - startX
        deltaY := targetY - startY
        distance := Sqrt(deltaX**2 + deltaY**2)
        
        if (distance < 5) {
            ; Too close for smooth movement
            MouseMove(targetX, targetY, speed)
            return
        }
        
        ; Number of steps based on distance and speed
        steps := Max(5, Min(50, distance / (speed / 10)))
        stepDelay := Max(1, speed * 2)
        
        ; Perform smooth movement
        for i in Range(1, steps) {
            progress := i / steps
            
            ; Use easing function for natural movement
            easedProgress := this.EaseInOutCubic(progress)
            
            currentX := Round(startX + deltaX * easedProgress)
            currentY := Round(startY + deltaY * easedProgress)
            
            MouseMove(currentX, currentY, 0)
            Sleep(stepDelay)
        }
        
        ; Ensure we reach exact target
        MouseMove(targetX, targetY, 0)
    }
    
    static EaseInOutCubic(t) {
        ; Smooth easing function for natural movement
        return t < 0.5 ? 4 * t**3 : 1 - (-2 * t + 2)**3 / 2
    }
    
    static MoveRelative(deltaX, deltaY, speed := -1) {
        this.UpdateCurrentPosition()
        newX := this.currentPos.x + deltaX
        newY := this.currentPos.y + deltaY
        this.MoveTo(newX, newY, speed)
    }
    
    static CircularMove(centerX, centerY, radius, steps := 36, speed := 20) {
        ; Move mouse in a circle
        for i in Range(0, steps - 1) {
            angle := (i / steps) * 2 * 3.14159  ; Full circle in radians
            x := Round(centerX + radius * Cos(angle))
            y := Round(centerY + radius * Sin(angle))
            
            MouseMove(x, y, speed)
            Sleep(50)
        }
    }
    
    static SpiralMove(centerX, centerY, maxRadius, revolutions := 3, speed := 15) {
        ; Move mouse in expanding spiral
        steps := revolutions * 36
        
        for i in Range(0, steps - 1) {
            progress := i / steps
            angle := progress * revolutions * 2 * 3.14159
            radius := maxRadius * progress
            
            x := Round(centerX + radius * Cos(angle))
            y := Round(centerY + radius * Sin(angle))
            
            MouseMove(x, y, speed)
            Sleep(25)
        }
    }
    
    static StartRecording() {
        this.isRecording := true
        this.recordedMoves := []
    }
    
    static StopRecording() {
        this.isRecording := false
        return this.recordedMoves.Clone()
    }
    
    static PlaybackRecording(moves, speedMultiplier := 1.0) {
        if (!moves || moves.Length = 0) return
        
        startTime := A_TickCount
        
        for move in moves {
            if (move.type = "move") {
                ; Calculate adjusted timing
                targetTime := move.timestamp - moves[1].timestamp
                adjustedSpeed := Max(0, move.speed * speedMultiplier)
                
                ; Wait for correct timing
                elapsed := A_TickCount - startTime
                if (elapsed < targetTime) {
                    Sleep(targetTime - elapsed)
                }
                
                ; Execute movement
                MouseMove(move.x, move.y, adjustedSpeed)
            }
        }
    }
    
    static EnableSmoothing() => this.smoothingEnabled := true
    static DisableSmoothing() => this.smoothingEnabled := false
    static SetDefaultSpeed(speed) => this.defaultSpeed := speed
}

; Screen region navigator for multi-monitor setups
class ScreenNavigator {
    static monitors := []
    static currentMonitor := 1
    
    static Initialize() {
        this.DetectMonitors()
    }
    
    static DetectMonitors() {
        this.monitors := []
        
        ; Get primary monitor info
        primaryWidth := SysGet(0)   ; SM_CXSCREEN
        primaryHeight := SysGet(1)  ; SM_CYSCREEN
        
        this.monitors.Push({
            left: 0,
            top: 0,
            right: primaryWidth,
            bottom: primaryHeight,
            width: primaryWidth,
            height: primaryHeight,
            isPrimary: true
        })
        
        ; Note: For complete multi-monitor support, would need additional API calls
        ; This provides basic primary monitor support
    }
    
    static MoveToMonitor(monitorIndex, x := 0.5, y := 0.5) {
        if (monitorIndex < 1 || monitorIndex > this.monitors.Length) {
            throw ValueError("Invalid monitor index: " . monitorIndex)
        }
        
        monitor := this.monitors[monitorIndex]
        
        ; Calculate position as percentage of monitor size
        targetX := monitor.left + (x * monitor.width)
        targetY := monitor.top + (y * monitor.height)
        
        MouseController.MoveTo(targetX, targetY)
        this.currentMonitor := monitorIndex
    }
    
    static MoveToCorner(corner, monitorIndex := 0) {
        if (!monitorIndex) {
            monitorIndex := this.currentMonitor
        }
        
        monitor := this.monitors[monitorIndex]
        
        switch corner {
            case "TopLeft":
                MouseController.MoveTo(monitor.left, monitor.top)
            case "TopRight":
                MouseController.MoveTo(monitor.right - 1, monitor.top)
            case "BottomLeft":
                MouseController.MoveTo(monitor.left, monitor.bottom - 1)
            case "BottomRight":
                MouseController.MoveTo(monitor.right - 1, monitor.bottom - 1)
            case "Center":
                MouseController.MoveTo(monitor.left + monitor.width / 2, monitor.top + monitor.height / 2)
            default:
                throw ValueError("Invalid corner: " . corner)
        }
    }
    
    static GetCurrentMonitorInfo() {
        MouseGetPos(&x, &y)
        
        for index, monitor in this.monitors {
            if (x >= monitor.left && x <= monitor.right && 
                y >= monitor.top && y <= monitor.bottom) {
                return {
                    index: index,
                    monitor: monitor,
                    relativeX: (x - monitor.left) / monitor.width,
                    relativeY: (y - monitor.top) / monitor.height
                }
            }
        }
        
        return {index: 1, monitor: this.monitors[1], relativeX: 0, relativeY: 0}
    }
}

; GUI element targeting and navigation
class ElementNavigator {
    static elements := Map()
    static activeElement := ""
    
    static RegisterElement(name, x, y, width := 0, height := 0, description := "") {
        this.elements[name] := {
            x: x,
            y: y,
            width: width,
            height: height,
            description: description,
            lastVisited: 0,
            visitCount: 0
        }
    }
    
    static NavigateToElement(name, position := "center", speed := 25) {
        if (!this.elements.Has(name)) {
            throw ValueError("Element '" . name . "' not registered")
        }
        
        element := this.elements[name]
        
        ; Calculate target coordinates based on position
        switch position {
            case "center":
                targetX := element.x + (element.width / 2)
                targetY := element.y + (element.height / 2)
            case "topleft":
                targetX := element.x
                targetY := element.y
            case "topright":
                targetX := element.x + element.width
                targetY := element.y
            case "bottomleft":
                targetX := element.x
                targetY := element.y + element.height
            case "bottomright":
                targetX := element.x + element.width
                targetY := element.y + element.height
            default:
                targetX := element.x + (element.width / 2)
                targetY := element.y + (element.height / 2)
        }
        
        ; Move to element
        MouseController.MoveTo(targetX, targetY, speed)
        
        ; Update statistics
        element.lastVisited := A_TickCount
        element.visitCount++
        this.activeElement := name
        
        return {x: targetX, y: targetY}
    }
    
    static GetElementAt(x, y) {
        for name, element in this.elements {
            if (x >= element.x && x <= element.x + element.width &&
                y >= element.y && y <= element.y + element.height) {
                return name
            }
        }
        return ""
    }
    
    static CreateElementMap(windowTitle) {
        ; Basic element detection (would need more sophisticated implementation)
        this.elements.Clear()
        
        if (!WinExist(windowTitle)) {
            throw ValueError("Window not found: " . windowTitle)
        }
        
        WinGetPos(&winX, &winY, &winWidth, &winHeight, windowTitle)
        
        ; Register common UI element positions (simplified)
        this.RegisterElement("WindowCenter", winX + winWidth/2, winY + winHeight/2, 0, 0, "Window center")
        this.RegisterElement("TitleBar", winX, winY, winWidth, 30, "Window title bar")
        this.RegisterElement("CloseButton", winX + winWidth - 45, winY + 5, 40, 20, "Close button")
        
        return this.elements.Count
    }
    
    static GetNavigationStatistics() {
        stats := Map()
        totalVisits := 0
        
        for name, element in this.elements {
            stats[name] := {
                visits: element.visitCount,
                lastVisited: element.lastVisited,
                description: element.description
            }
            totalVisits += element.visitCount
        }
        
        return {
            elementStats: stats,
            totalElements: this.elements.Count,
            totalVisits: totalVisits,
            activeElement: this.activeElement
        }
    }
}

; Accessibility mouse control for assistive applications
class AccessibilityMouse {
    static dwellTime := 1000
    static dwellEnabled := false
    static dwellCallback := ""
    static lastMoveTime := 0
    static currentHoverElement := ""
    
    static EnableDwellClick(dwellTimeMs := 1000, callback := "") {
        this.dwellTime := dwellTimeMs
        this.dwellEnabled := true
        this.dwellCallback := callback
        
        ; Start monitoring timer
        SetTimer(this.CheckDwellStatus.Bind(this), 100)
    }
    
    static DisableDwellClick() {
        this.dwellEnabled := false
        SetTimer(this.CheckDwellStatus.Bind(this), 0)
    }
    
    static CheckDwellStatus() {
        if (!this.dwellEnabled) return
        
        MouseGetPos(&currentX, &currentY)
        static lastX := -1, lastY := -1
        
        ; Check if mouse has moved
        if (currentX != lastX || currentY != lastY) {
            this.lastMoveTime := A_TickCount
            lastX := currentX
            lastY := currentY
            return
        }
        
        ; Check if dwell time has elapsed
        if (A_TickCount - this.lastMoveTime >= this.dwellTime) {
            ; Trigger dwell click
            this.PerformDwellClick(currentX, currentY)
            this.lastMoveTime := A_TickCount  ; Reset timer
        }
    }
    
    static PerformDwellClick(x, y) {
        ; Execute dwell click action
        Click(x, y)
        
        ; Call custom callback if provided
        if (this.dwellCallback && IsFunc(this.dwellCallback)) {
            this.dwellCallback.Call(x, y)
        }
        
        ; Visual feedback
        this.ShowDwellFeedback(x, y)
    }
    
    static ShowDwellFeedback(x, y) {
        ; Show visual feedback for dwell click
        ToolTip("Dwell Click", x + 10, y + 10)
        SetTimer(() => ToolTip(), -1000)  ; Hide after 1 second
    }
    
    static MoveInSteps(targetX, targetY, stepSize := 10) {
        ; Move mouse in discrete steps for easier control
        MouseGetPos(&currentX, &currentY)
        
        deltaX := targetX - currentX
        deltaY := targetY - currentY
        distance := Sqrt(deltaX**2 + deltaY**2)
        
        if (distance <= stepSize) {
            MouseMove(targetX, targetY, 20)
            return
        }
        
        steps := Ceil(distance / stepSize)
        stepX := deltaX / steps
        stepY := deltaY / steps
        
        for i in Range(1, steps) {
            newX := Round(currentX + stepX * i)
            newY := Round(currentY + stepY * i)
            MouseMove(newX, newY, 15)
            Sleep(100)  ; Pause between steps
        }
        
        ; Final position
        MouseMove(targetX, targetY, 15)
    }
    
    static CreateMovementGrid(gridSize := 50) {
        ; Create a grid of movement targets for keyboard navigation
        SysGet(&screenWidth, 0)
        SysGet(&screenHeight, 1)
        
        grid := []
        
        for x in Range(gridSize, screenWidth - gridSize, gridSize) {
            for y in Range(gridSize, screenHeight - gridSize, gridSize) {
                grid.Push({x: x, y: y})
            }
        }
        
        return grid
    }
    
    static SetDwellTime(milliseconds) {
        this.dwellTime := milliseconds
    }
}

; Performance optimization for mouse movements
class MouseOptimizer {
    static movementCache := Map()
    static cacheTimeout := 5000  ; 5 seconds
    
    static OptimizedMoveTo(x, y, speed := 25) {
        ; Check if this is a recent movement to same location
        cacheKey := x . "," . y
        currentTime := A_TickCount
        
        if (this.movementCache.Has(cacheKey)) {
            lastMove := this.movementCache[cacheKey]
            if (currentTime - lastMove.timestamp < this.cacheTimeout) {
                ; Skip movement if too recent and close enough
                MouseGetPos(&currentX, &currentY)
                distance := Sqrt((x - currentX)**2 + (y - currentY)**2)
                
                if (distance < 5) {
                    return  ; Skip redundant movement
                }
            }
        }
        
        ; Perform movement
        MouseMove(x, y, speed)
        
        ; Cache the movement
        this.movementCache[cacheKey] := {
            timestamp: currentTime,
            speed: speed
        }
        
        ; Clean old cache entries
        if (this.movementCache.Count > 100) {
            this.CleanCache()
        }
    }
    
    static CleanCache() {
        currentTime := A_TickCount
        
        for key, data in this.movementCache {
            if (currentTime - data.timestamp > this.cacheTimeout) {
                this.movementCache.Delete(key)
            }
        }
    }
    
    static BatchMovements(movements, optimizeOrder := true) {
        if (optimizeOrder) {
            movements := this.OptimizeMovementOrder(movements)
        }
        
        for movement in movements {
            this.OptimizedMoveTo(movement.x, movement.y, movement.speed)
            
            if (movement.HasProp("action")) {
                movement.action.Call()
            }
            
            if (movement.HasProp("delay")) {
                Sleep(movement.delay)
            }
        }
    }
    
    static OptimizeMovementOrder(movements) {
        ; Simple nearest-neighbor optimization
        optimized := []
        remaining := movements.Clone()
        
        ; Start from current position
        MouseGetPos(&currentX, &currentY)
        
        while (remaining.Length > 0) {
            closestIndex := 1
            minDistance := 999999
            
            for i, movement in remaining {
                distance := Sqrt((movement.x - currentX)**2 + (movement.y - currentY)**2)
                if (distance < minDistance) {
                    minDistance := distance
                    closestIndex := i
                }
            }
            
            ; Add closest movement to optimized path
            closest := remaining.RemoveAt(closestIndex)
            optimized.Push(closest)
            
            currentX := closest.x
            currentY := closest.y
        }
        
        return optimized
    }
}

; Example usage
; Initialize mouse control systems
MouseController.Initialize()
ScreenNavigator.Initialize()

; Basic movement
MouseController.MoveTo(500, 300, 25)

; Complex movement patterns
MouseController.CircularMove(400, 300, 100, 36, 15)
MouseController.SpiralMove(600, 400, 150, 2, 20)

; Record and playback movements
MouseController.StartRecording()
MouseController.MoveTo(100, 100, 20)
MouseController.MoveTo(200, 200, 20)
MouseController.MoveTo(300, 100, 20)
recording := MouseController.StopRecording()
MouseController.PlaybackRecording(recording, 1.5)  ; Play 1.5x speed

; Screen navigation
ScreenNavigator.MoveToCorner("Center")
ScreenNavigator.MoveToCorner("TopRight")

; Element navigation
ElementNavigator.RegisterElement("LoginButton", 100, 200, 80, 30, "Login button")
ElementNavigator.NavigateToElement("LoginButton", "center", 20)

; Accessibility features
AccessibilityMouse.EnableDwellClick(1500)  ; 1.5 second dwell time
AccessibilityMouse.MoveInSteps(400, 300, 15)  ; Move in 15-pixel steps

; Optimized batch movements
movements := [
    {x: 100, y: 100, speed: 20},
    {x: 300, y: 200, speed: 25},
    {x: 500, y: 150, speed: 15}
]
MouseOptimizer.BatchMovements(movements, true)  ; Optimize order
```

## Implementation Notes

**Movement Speed and Smoothness:**
- Speed 0 provides instant movement without animation
- Higher speed values create slower, more visible movement
- Natural movement requires coordination between speed and distance
- System performance affects movement smoothness at high speeds

**Coordinate System Handling:**
- Default Screen coordinates work across entire desktop
- Window coordinates require active window for proper positioning
- Multi-monitor setups may require special coordinate calculations
- DPI scaling can affect coordinate accuracy on high-resolution displays

**Performance Considerations:**
- Rapid mouse movements can impact system performance
- Large coordinate changes may appear jerky at slow speeds
- Cache frequently used positions to reduce redundant movements
- Consider user experience when automating mouse movements

**User Experience Guidelines:**
- Avoid excessively fast movements that are difficult to follow
- Provide visual feedback for accessibility applications
- Respect user control and provide override mechanisms
- Test movement patterns on different screen sizes and resolutions

**System Integration:**
- Mouse movement respects system acceleration settings
- Some security software may block programmatic mouse control
- Gaming applications may capture mouse input exclusively
- Remote desktop scenarios may have different coordinate behavior

## Related AHK Concepts

- [Click](./click.md) - Mouse clicking at specific coordinates
- [MouseGetPos](./mousegetpos.md) - Retrieving current mouse position
- [CoordMode](../../10_Language_Core/01-Functions/Built_In_Functions/coordmode.md) - Coordinate system configuration
- [SetDefaultMouseSpeed](../../10_Language_Core/01-Functions/Built_In_Functions/setdefaultmousespeed.md) - Global speed settings
- [WinGetPos](../../40_Advanced_Features/05-System_Integration/wingetpos.md) - Window position for coordinate calculations

## Tags

#AutoHotkey #MouseMove #MouseControl #Automation #Coordinates #Accessibility #GUIAutomation #CursorControl #Movement