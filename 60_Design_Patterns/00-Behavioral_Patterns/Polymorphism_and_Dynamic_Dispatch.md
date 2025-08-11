# Topic: Polymorphism and Dynamic Dispatch in AutoHotkey v2

## Category

Pattern

## Overview

AutoHotkey v2 implements ad-hoc polymorphism primarily through duck typing and dynamic dispatch mechanisms. Unlike traditional OOP languages with formal interfaces, AHK v2 allows calling methods on objects without compile-time type checking, enabling flexible polymorphic behavior through Map-based handlers and dynamic method resolution.

## Key Points

- AHK v2 supports only ad-hoc polymorphism (duck typing), not parametric or subtype polymorphism
- Map-based function handlers often provide cleaner solutions than class hierarchies for simple dispatch
- Dynamic method calling can be achieved through Maps, avoiding switch-case statements
- Static properties in classes have accessibility limitations from instance methods
- Object lifecycle management is crucial when using constructors repeatedly

## Syntax and Parameters

```cpp
; Map-based polymorphic dispatch
WindowClassHandlers := Map(
    "dopus.lister", DopusHandler,
    "CabinetWClass", ExplorerHandler,
    "ProgMan", DesktopHandler
)

; Calling handlers dynamically
WindowClassHandlers[WindowClass](hwnd)

; Alternative syntax for clarity
(WindowClassHandlers[WindowClass])(hwnd)

; Class-based static method reference
HandlerMap := Map(
    "dopus", FileManager.Dopus.GetPaths,
    "explorer", FileManager.Explorer.GetPaths
)
```

## Code Examples

```cpp
; Example 1: Map-based polymorphic file manager handlers
class FileManagerDispatcher {
    static Handlers := Map(
        "dopus.lister", (*) => this.HandleDopus(WinGetList("dopus.lister")[1]),
        "CabinetWClass", (*) => this.HandleExplorer(WinGetList("CabinetWClass")[1]),
        "ProgMan", (*) => this.HandleDesktop()
    )
    
    static ProcessWindows() {
        for windowClass in this.Handlers {
            if WinExist("ahk_class " windowClass) {
                this.Handlers[windowClass]()
            }
        }
    }
    
    static HandleDopus(hwnd) {
        ; Specific Dopus path extraction logic
        return ["C:\Projects", "D:\Documents"]
    }
    
    static HandleExplorer(hwnd) {
        ; Specific Explorer path extraction logic
        return ["C:\Users\Current"]
    }
    
    static HandleDesktop() {
        ; Desktop path logic
        return ["C:\Users\Public\Desktop"]
    }
}

; Example 2: Class-based polymorphism with static methods
class FileManager {
    class Dopus {
        static GetPaths(hwnd) {
            ; Implementation for Dopus
            return ["C:\Dopus\Path1", "C:\Dopus\Path2"]
        }
        
        static GetSelection(hwnd) {
            ; Get selected files in Dopus
        }
    }
    
    class Explorer {
        static GetPaths(hwnd) {
            ; Implementation for Windows Explorer
            return ["C:\Explorer\Path1"]
        }
        
        static GetSelection(hwnd) {
            ; Get selected files in Explorer
        }
    }
}

; Usage with nested classes
HandlerMap := Map(
    "dopus.lister", FileManager.Dopus.GetPaths,
    "CabinetWClass", FileManager.Explorer.GetPaths
)

; Dynamic dispatch
for windowClass, handler in HandlerMap {
    if WinExist("ahk_class " windowClass) {
        paths := handler(WinGetID("ahk_class " windowClass))
        ; Process paths...
    }
}

; Example 3: Handling static property access issues
class ConfigManager {
    static DefaultPath := "C:\Config"
    static Instance := ""
    
    ; Static constructor workaround
    static __New() {
        ; Initialize static properties that need complex setup
        this.Prototype.ConfigPath := this.DefaultPath
    }
    
    __New() {
        ; Access static property through prototype
        this.LocalPath := this.ConfigPath  ; Works via prototype chain
        ; Alternative: use class name directly
        ; this.LocalPath := ConfigManager.DefaultPath
    }
    
    GetConfig() {
        ; Method that needs access to static data
        return this.ConfigPath  ; Accessible via prototype
    }
}

; Example 4: Function object polymorphism
CreateHandler(type) {
    switch type {
        case "file":
            return (path) => FileUtils.Process(path)
        case "registry":
            return (key) => RegUtils.Process(key)
        case "network":
            return (url) => NetworkUtils.Process(url)
        default:
            return (*) => throw Error("Unknown handler type")
    }
}

; Usage
handlers := Map()
handlers["config"] := CreateHandler("file")
handlers["settings"] := CreateHandler("registry")

; Polymorphic calls
for name, handler in handlers {
    result := handler(configData[name])
}

; Example 5: Duck typing demonstration
class AudioPlayer {
    Play() => "Playing audio..."
    Stop() => "Stopping audio..."
}

class VideoPlayer {
    Play() => "Playing video..."
    Stop() => "Stopping video..."
}

class MediaController {
    static PlayMedia(player) {
        ; Duck typing - we don't care about the specific type
        ; as long as it has Play() method
        return player.Play()
    }
}

; Usage
audio := AudioPlayer()
video := VideoPlayer()

; Both work polymorphically
MediaController.PlayMedia(audio)  ; "Playing audio..."
MediaController.PlayMedia(video)  ; "Playing video..."
```

## Implementation Notes

**Map vs Class Decision:** Use Maps when handlers are simple functions without state. Use classes when you need to group related methods, maintain state, or implement inheritance hierarchies.

**Static Property Access:** AHK v2 doesn't allow `this.staticProperty` access from instance methods. Workarounds include:
- Using the class name directly: `ClassName.staticProperty`
- Storing a class reference: `this.classRef := %this.__class%`
- Setting up prototype properties in static constructors

**Dynamic Method Resolution:** The `()` operator can be applied to any expression that evaluates to a function reference, enabling flexible polymorphic dispatch patterns.

**Performance Considerations:** Map lookups are generally faster than switch-case statements for large numbers of cases. However, direct method calls are fastest when the type is known at compile time.

**Object Lifecycle:** When using `ClassName().Method()` patterns, remember that `__New()` is called each time, creating temporary objects that are immediately garbage collected.

**Duck Typing Best Practices:**
- Design consistent method signatures across different implementations
- Document expected interface contracts in comments
- Use meaningful method names that clearly indicate behavior
- Consider error handling for missing methods

## Related AHK Concepts

- Duck Typing and Interface Contracts
- Map Data Structure and Function References  
- Static vs Instance Members
- Method Resolution and Prototype Chain
- Function Objects and Bound Functions
- Class Inheritance and Method Overriding

## Tags

#AutoHotkey #OOP #Polymorphism #DuckTyping #DynamicDispatch #StaticMethods #Maps #FunctionObjects #BehavioralPattern