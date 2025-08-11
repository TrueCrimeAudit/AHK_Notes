# Topic: Run Function - System Process Execution

## Category

Function

## Overview

The Run function executes external programs, opens files, or launches system commands from within AutoHotkey scripts. It provides comprehensive process control including working directory specification, window state management, and optional process ID retrieval for advanced process monitoring and control.

## Key Points

- Launches any executable, file, or system command with full parameter support
- Provides control over initial window state (minimized, maximized, hidden)
- Returns process ID (PID) for advanced process management and monitoring
- Supports working directory specification for proper file context
- Handles both blocking and non-blocking execution patterns

## Syntax and Parameters

```cpp
Run(Target [, WorkingDir, Options, &OutputVarPID])

; Parameters:
; Target      - Program/file path, URL, or system command to execute
; WorkingDir  - Working directory for the launched process (optional)
; Options     - Window state and execution options (optional)
; OutputVarPID - Variable to receive the Process ID (optional, by reference)

; Options values:
; "Max"       - Maximized window
; "Min"       - Minimized window  
; "Hide"      - Hidden window (no taskbar entry)
; "UseErrorLevel" - Suppress error dialogs, check ErrorLevel instead
```

## Code Examples

```cpp
; Basic program execution
Run("notepad.exe")

; Open file with default program
Run("C:\MyFile.txt")

; Launch with specific working directory
Run("cmd.exe", "C:\MyProject")

; Control window state
Run("calc.exe", , "Max")        ; Maximized calculator
Run("cmd.exe", , "Min")         ; Minimized command prompt
Run("program.exe", , "Hide")    ; Hidden execution

; Capture Process ID for monitoring
Run("notepad.exe", , , &ProcessID)
MsgBox("Notepad PID: " . ProcessID)

; Error handling with UseErrorLevel
Run("nonexistent.exe", , "UseErrorLevel")
if (ErrorLevel) {
    MsgBox("Failed to launch program. Error: " . ErrorLevel)
}

; Complex example: Launch with full control
Target := "C:\Program Files\MyApp\app.exe"
Args := ' --config "C:\Config\settings.ini" --verbose'
WorkDir := "C:\Program Files\MyApp"
Run(Target . Args, WorkDir, "Max", &AppPID)

; System commands and URLs
Run("shutdown /s /t 60")        ; System shutdown in 60 seconds
Run("https://www.example.com")  ; Open URL in default browser
Run("mailto:user@example.com")  ; Open email client

; File operations
Run("explorer.exe C:\Windows")  ; Open Windows Explorer
Run('rundll32.exe shell32.dll,Control_RunDLL desk.cpl') ; Display settings
```

## Implementation Notes

**Process Management:**
- Process ID can be used with ProcessWait(), ProcessClose(), and WinWait()
- Hidden processes continue running without user interface visibility
- Working directory affects relative file path resolution

**Security Considerations:**
- Always validate user input when constructing Target parameter
- Be cautious with elevated privileges and system commands
- Consider using quotes around paths containing spaces

**Error Handling:**
- UseErrorLevel option prevents error dialogs and sets ErrorLevel variable
- ErrorLevel values: 0 = success, "ERROR" = failure, specific codes for different failures
- Without UseErrorLevel, failed launches show system error dialogs

**Performance Notes:**
- Run() returns immediately (non-blocking) unless waiting for process
- Large applications may take time to fully initialize after Run() returns
- Use ProcessWait() or WinWait() to synchronize with launched applications

**Platform Compatibility:**
- File associations respect user's default program settings
- System commands may vary between Windows versions
- Some programs require administrator privileges to launch properly

## Related AHK Concepts

- [RunWait](../runwait.md) - Blocking version that waits for process completion
- [ProcessWait](../../10_Language_Core/01-Functions/Built_In_Functions/processwait.md) - Wait for process startup
- [ProcessClose](../../10_Language_Core/01-Functions/Built_In_Functions/processclose.md) - Terminate processes
- [WinWait](../../30_Built_In_Classes/01-GUI_Classes/winwait.md) - Wait for window appearance
- [ProcessExist](../../10_Language_Core/01-Functions/Built_In_Functions/processexist.md) - Check process existence
- [DllCall](../../40_Advanced_Features/02-DLL_Integration/dllcall.md) - Advanced system integration

## Tags

#AutoHotkey #Process #SystemIntegration #Execution #PID #ErrorHandling #WorkingDirectory