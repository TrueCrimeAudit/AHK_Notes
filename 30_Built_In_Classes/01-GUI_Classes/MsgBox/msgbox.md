# Function: MsgBox

## Category

Built-in Function

## Overview

MsgBox displays a message dialog box to the user with customizable text, buttons, and icons. It's the most fundamental function for user communication in AutoHotkey, providing immediate feedback and user interaction capabilities essential for debugging, notifications, and simple user interfaces.

## Syntax

```cpp
Result := MsgBox([Text, Title, Options])
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Text | String | No | The message to display. If omitted, displays an empty message box |
| Title | String | No | The title bar text. If omitted, defaults to script name |
| Options | String/Integer | No | Combination of button types, icon types, and other options |

## Return Value

| Type | Description |
|------|-------------|
| String | The button pressed by the user: "OK", "Cancel", "Yes", "No", "Abort", "Retry", "Ignore", "Continue", "TryAgain" |

## Options Parameter

### Button Types
| Option | Value | Description |
|--------|-------|-------------|
| OK | 0 | OK button only (default) |
| OKCancel | 1 | OK and Cancel buttons |
| AbortRetryIgnore | 2 | Abort, Retry, and Ignore buttons |
| YesNoCancel | 3 | Yes, No, and Cancel buttons |
| YesNo | 4 | Yes and No buttons |
| RetryCancel | 5 | Retry and Cancel buttons |
| CancelTryAgainContinue | 6 | Cancel, Try Again, and Continue buttons |

### Icon Types
| Option | Value | Description |
|--------|-------|-------------|
| None | 0 | No icon (default) |
| Error | 16 | Error/Stop icon |
| Question | 32 | Question mark icon |
| Warning | 48 | Warning/Exclamation icon |
| Information | 64 | Information icon |

### Additional Options
| Option | Value | Description |
|--------|-------|-------------|
| Default2 | 256 | Second button is default |
| Default3 | 512 | Third button is default |
| SystemModal | 4096 | System modal (appears above all windows) |
| TaskModal | 8192 | Task modal |
| AlwaysOnTop | 262144 | Always on top |

## Exceptions

| Exception Type | When Thrown |
|----------------|-------------|
| ValueError | When Options parameter contains invalid combination |
| OSError | When system cannot display message box (rare) |

## Basic Examples

```cpp
; Simple message
MsgBox("Hello, World!")

; Message with title
MsgBox("Operation completed successfully", "Success")

; Yes/No question
result := MsgBox("Do you want to continue?", "Confirmation", "YesNo")
if (result == "Yes") {
    ; User clicked Yes
    MsgBox("Continuing operation...")
} else {
    ; User clicked No
    MsgBox("Operation cancelled")
}

; Warning message with icon
MsgBox("This action cannot be undone!", "Warning", "OKCancel + Warning")
```

## Advanced Examples

```cpp
; Advanced message box function with error handling
ShowUserMessage(message, title := "", type := "Information") {
    local options := ""
    local icon := ""
    
    ; Configure options based on message type
    switch type {
        case "Error":
            options := "OK + Error"
            icon := "Error"
        case "Warning":
            options := "YesNo + Warning"
            icon := "Warning"
        case "Question":
            options := "YesNoCancel + Question"
            icon := "Question"
        case "Information":
            options := "OK + Information"
            icon := "Information"
        default:
            options := "OK"
    }
    
    ; Use script name as default title
    if (title == "") {
        title := A_ScriptName
    }
    
    try {
        result := MsgBox(message, title, options)
        return result
    } catch Error as err {
        ; Fallback if MsgBox fails
        OutputDebug("MsgBox failed: " . err.Message)
        return "Error"
    }
}

; Usage examples
ShowUserMessage("File saved successfully!", "File Operation", "Information")
response := ShowUserMessage("Delete all files?", "Dangerous Operation", "Warning")
if (response == "Yes") {
    ; Proceed with deletion
}
```

```cpp
; Message box utility class for consistent application messaging
class MessageManager {
    static appName := A_ScriptName
    static defaultIcon := "Information"
    
    static Info(message, title := "") {
        return this.ShowMessage(message, title, "OK + Information")
    }
    
    static Warning(message, title := "") {
        return this.ShowMessage(message, title, "OK + Warning")
    }
    
    static Error(message, title := "") {
        return this.ShowMessage(message, title, "OK + Error")
    }
    
    static Question(message, title := "") {
        return this.ShowMessage(message, title, "YesNo + Question")
    }
    
    static Confirm(message, title := "") {
        return this.ShowMessage(message, title, "OKCancel + Question")
    }
    
    static ShowMessage(message, title, options) {
        if (title == "") {
            title := this.appName
        }
        
        ; Add timestamp for debugging
        local timestamp := FormatTime(A_Now, "yyyy-MM-dd HH:mm:ss")
        OutputDebug("[" . timestamp . "] MsgBox: " . message)
        
        return MsgBox(message, title, options)
    }
}

; Usage examples
MessageManager.Info("Application started successfully")
if (MessageManager.Question("Save changes before closing?") == "Yes") {
    SaveDocument()
}
if (MessageManager.Confirm("Delete selected items?") == "OK") {
    DeleteSelectedItems()
}
```

## Real-World Example

```cpp
; File processing application with comprehensive user feedback
class FileProcessor {
    static ProcessFiles(fileList) {
        local processed := 0
        local errors := Array()
        local startTime := A_TickCount
        
        ; Confirm operation
        local confirmMsg := "Process " . fileList.Length . " files?\n\n"
        confirmMsg .= "This operation may take several minutes."
        
        if (MsgBox(confirmMsg, "File Processing", "YesNo + Question") != "Yes") {
            return false
        }
        
        ; Process each file
        for index, filePath in fileList {
            try {
                ; Update progress
                local progressMsg := "Processing file " . index . " of " . fileList.Length
                progressMsg .= "\n\nCurrent: " . FileNameFromPath(filePath)
                
                ; Show progress (non-blocking would be better, but this is for example)
                if (Mod(index, 5) == 0) {  ; Every 5th file
                    ToolTip(progressMsg)
                }
                
                ; Simulate file processing
                this.ProcessSingleFile(filePath)
                processed++
                
            } catch Error as err {
                errors.Push({file: filePath, error: err.Message})
                
                ; Ask user how to handle error
                local errorMsg := "Error processing file:\n" . filePath
                errorMsg .= "\n\nError: " . err.Message
                errorMsg .= "\n\nContinue with remaining files?"
                
                local response := MsgBox(errorMsg, "Processing Error", "YesNoCancel + Error")
                
                if (response == "Cancel") {
                    break
                } else if (response == "No") {
                    ; Skip remaining files
                    break
                }
                ; "Yes" continues processing
            }
        }
        
        ToolTip()  ; Clear progress tooltip
        
        ; Show completion summary
        local duration := (A_TickCount - startTime) / 1000
        local summaryMsg := "Processing Complete\n\n"
        summaryMsg .= "Files processed: " . processed . " of " . fileList.Length
        summaryMsg .= "\nTime taken: " . Round(duration, 1) . " seconds"
        
        if (errors.Length > 0) {
            summaryMsg .= "\nErrors encountered: " . errors.Length
            summaryMsg .= "\n\nView error details?"
            
            if (MsgBox(summaryMsg, "Processing Summary", "YesNo + Information") == "Yes") {
                this.ShowErrorDetails(errors)
            }
        } else {
            summaryMsg .= "\nNo errors encountered"
            MsgBox(summaryMsg, "Processing Summary", "OK + Information")
        }
        
        return {processed: processed, errors: errors, duration: duration}
    }
    
    static ShowErrorDetails(errors) {
        local errorDetails := "Error Details:\n\n"
        
        for index, errorInfo in errors {
            errorDetails .= index . ". " . errorInfo.file . "\n"
            errorDetails .= "   " . errorInfo.error . "\n\n"
            
            ; Limit display to prevent overly long messages
            if (index >= 10) {
                errorDetails .= "... and " . (errors.Length - 10) . " more errors"
                break
            }
        }
        
        MsgBox(errorDetails, "Error Details", "OK + Error")
    }
}
```

## Common Use Cases

- **User Notifications**: Informing users of operation completion or status
- **Error Reporting**: Displaying error messages with appropriate icons
- **Confirmation Dialogs**: Getting user approval before destructive operations
- **Debugging**: Quick output during script development and testing
- **Simple User Input**: Basic yes/no or ok/cancel decisions
- **Progress Communication**: Updating users on long-running operations

## Performance Notes

- MsgBox is synchronous - execution stops until user responds
- Minimal performance impact for display, but blocking behavior affects script flow
- For non-blocking notifications, consider ToolTip or custom GUI solutions
- System modal messages have higher performance cost but ensure visibility

## Common Pitfalls

### Pitfall 1: Blocking Script Execution
**Problem**: MsgBox stops script execution until user responds, which can interfere with automation
**Solution**: Use conditional MsgBox calls or alternative notification methods for automated scripts
```cpp
; Problematic in automation
Loop 100 {
    ; This stops automation every iteration
    MsgBox("Processing item " . A_Index)
    ProcessItem()
}

; Better approach
Loop 100 {
    ; Only show every 10th item, or use ToolTip
    if (Mod(A_Index, 10) == 0) {
        ToolTip("Processing item " . A_Index)
    }
    ProcessItem()
}
```

### Pitfall 2: Incorrect Option Combinations
**Problem**: Using invalid combinations of button and icon options
**Solution**: Use proper option syntax and test combinations
```cpp
; Incorrect - invalid option combination
; MsgBox("Message", "Title", "OK + YesNo")  ; Can't have both OK and YesNo

; Correct - proper option combinations
MsgBox("Info message", "Title", "OK + Information")
MsgBox("Question", "Title", "YesNo + Question")
```

### Pitfall 3: Not Handling Return Values
**Problem**: Ignoring the return value when multiple buttons are available
**Solution**: Always check return value for multi-button message boxes
```cpp
; Problematic - ignoring return value
MsgBox("Delete files?", "Confirm", "YesNo")
DeleteFiles()  ; Always executes regardless of choice

; Correct - handling return value
if (MsgBox("Delete files?", "Confirm", "YesNo") == "Yes") {
    DeleteFiles()
}
```

## Version History

- **v2.0**: Updated syntax with named parameters and improved Unicode support
- **v2.0-a**: Early alpha versions had different parameter order
- **v2.1**: Enhanced error handling and additional modal options

## Related Functions

- [InputBox](../InputBox/inputbox.md) - Get text input from user
- [ToolTip](../ToolTip/tooltip.md) - Non-blocking text display
- [TrayTip](../TrayTip/traytip.md) - System tray notifications
- [Gui](../Gui/gui.md) - Custom dialog creation for complex interactions

## Related Concepts

- [User Interface Design](../../../50_Ecosystem/01-Best_Practices/ui-design.md) - Best practices for user communication
- [Error Handling](../../../00_Fundamentals/05-Error_Basics/error-handling.md) - Using MsgBox for error reporting
- [Debugging Techniques](../../../50_Ecosystem/03-Debugging_and_Testing/debugging-basics.md) - MsgBox for debugging output

## See Also

- [GUI Development Guide](../../../Index/learning-paths.md#gui-developer-path) - Building complete user interfaces
- [User Experience Best Practices](../../../50_Ecosystem/01-Best_Practices/user-experience.md) - Effective user communication
- [Script Flow Control](../../../10_Language_Core/00-Control_Flow/script-flow.md) - Managing script execution with user input

## Tags

#AutoHotkey #Function #GUI #MessageBox #UserInterface #Dialog #Communication #Debugging #BuiltIn #Essential