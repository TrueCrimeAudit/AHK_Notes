# Topic: InputBox Function - GUI User Input Collection

## Category

Function

## Overview

The InputBox function displays a dialog box prompting the user to enter text input. It provides a simple way to collect user data interactively, supporting input validation, default values, and password masking for secure data entry in AutoHotkey applications.

## Key Points

- Creates modal dialog for collecting text input from users
- Supports input validation, default values, and character limits
- Provides password masking for secure data entry
- Returns both user input and dialog result status for proper flow control
- Essential for interactive scripts requiring user configuration or data entry

## Syntax and Parameters

```cpp
result := InputBox([Prompt, Title, Options, Default])

; Parameters:
; Prompt  - Text displayed to prompt user for input (optional)
; Title   - Dialog box title (optional, defaults to script name)
; Options - String containing formatting options (optional)
; Default - Default text in input field (optional)

; Options format: "W<width> H<height> T<timeout> Password"
; W<width>   - Dialog width in pixels
; H<height>  - Dialog height in pixels  
; T<timeout> - Timeout in seconds (0 = no timeout)
; Password   - Mask input with asterisks

; Return Value:
; Returns object with properties:
; .Result - "OK", "Cancel", or "Timeout"
; .Value  - User's input text (empty if cancelled/timeout)
```

## Code Examples

```cpp
; Basic input collection
userInput := InputBox("Please enter your name:", "User Registration")
if (userInput.Result = "OK") {
    MsgBox("Hello, " . userInput.Value . "!")
} else {
    MsgBox("Input cancelled.")
}

; Input with default value and validation
function GetValidEmail() {
    loop {
        email := InputBox("Enter your email address:", "Email Setup", , "user@example.com")
        
        if (email.Result != "OK") {
            return ""  ; User cancelled
        }
        
        ; Simple email validation
        if (RegExMatch(email.Value, "^[^@]+@[^@]+\.[^@]+$")) {
            return email.Value
        }
        
        MsgBox("Invalid email format. Please try again.")
    }
}

; Password input with masking
function GetSecurePassword() {
    password := InputBox("Enter password:", "Security", "Password", "")
    
    if (password.Result = "OK" && StrLen(password.Value) >= 8) {
        return password.Value
    } else if (password.Result = "OK") {
        MsgBox("Password must be at least 8 characters.")
        return GetSecurePassword()  ; Recursive retry
    }
    
    return ""  ; Cancelled or failed
}

; Custom dialog dimensions and timeout
function GetTimedInput() {
    input := InputBox("Quick response needed:", "Urgent Input", "W400 H150 T10")
    
    switch input.Result {
        case "OK":
            return input.Value
        case "Cancel":
            MsgBox("Input cancelled by user.")
        case "Timeout":
            MsgBox("Input timed out after 10 seconds.")
    }
    
    return ""
}

; Multi-step configuration wizard
class ConfigurationWizard {
    settings := Map()
    
    RunWizard() {
        ; Step 1: Basic information
        name := InputBox("Enter application name:", "Configuration - Step 1/3")
        if (name.Result != "OK") return false
        this.settings["AppName"] := name.Value
        
        ; Step 2: File location
        path := InputBox("Enter data directory path:", "Configuration - Step 2/3", "W500", "C:\MyApp\Data")
        if (path.Result != "OK") return false
        this.settings["DataPath"] := path.Value
        
        ; Step 3: Advanced settings
        timeout := InputBox("Enter timeout (seconds):", "Configuration - Step 3/3", , "30")
        if (timeout.Result != "OK") return false
        
        ; Validate numeric input
        if (!IsNumber(timeout.Value)) {
            MsgBox("Invalid timeout value. Using default of 30 seconds.")
            this.settings["Timeout"] := 30
        } else {
            this.settings["Timeout"] := Integer(timeout.Value)
        }
        
        return this.FinalizeConfiguration()
    }
    
    FinalizeConfiguration() {
        ; Show summary for confirmation
        summary := "Configuration Summary:`n"
        for key, value in this.settings {
            summary .= key . ": " . value . "`n"
        }
        
        confirm := MsgBox(summary . "`nSave this configuration?", "Confirm Settings", "YesNo")
        
        if (confirm = "Yes") {
            this.SaveConfiguration()
            MsgBox("Configuration saved successfully!")
            return true
        }
        
        return false
    }
    
    SaveConfiguration() {
        ; Save to file or registry
        configFile := "config.ini"
        for key, value in this.settings {
            IniWrite(value, configFile, "Settings", key)
        }
    }
}

; Advanced input validation with retry logic
function GetValidatedInput(prompt, title, validator, errorMsg) {
    loop {
        input := InputBox(prompt, title)
        
        if (input.Result != "OK") {
            return ""  ; User cancelled
        }
        
        ; Call validation function
        if (validator.Call(input.Value)) {
            return input.Value
        }
        
        ; Show error and retry
        MsgBox(errorMsg, "Invalid Input")
    }
}

; Usage examples for validated input
function ValidateNumber(value) {
    return IsNumber(value) && value > 0
}

function ValidateFilePath(value) {
    return FileExist(value) != ""
}

; Get validated numeric input
number := GetValidatedInput("Enter a positive number:", "Numeric Input", 
                           ValidateNumber, "Please enter a valid positive number.")

; Get validated file path
filePath := GetValidatedInput("Enter file path:", "File Selection",
                             ValidateFilePath, "File does not exist. Please try again.")
```

## Implementation Notes

**Dialog Behavior:**
- InputBox creates a modal dialog that blocks script execution until dismissed
- Dialog appears centered on screen with standard Windows styling
- Supports standard Windows keyboard shortcuts (Tab, Enter, Escape)

**Input Validation Strategies:**
- Client-side validation using RegEx patterns for format checking
- Recursive functions for retry logic on invalid input
- Validation functions can be passed as parameters for reusability

**Return Value Handling:**
- Always check `.Result` property before using `.Value`
- Handle all three possible results: "OK", "Cancel", "Timeout"
- Empty string is returned for `.Value` when cancelled or timed out

**Security Considerations:**
- Password option masks input but doesn't encrypt storage
- Sensitive data should be cleared from memory after use
- Consider using secure string handling for highly sensitive inputs

**Performance Notes:**
- InputBox is a blocking operation - script pauses until user responds
- Timeout option prevents indefinite blocking in automated scenarios
- Multiple InputBox calls should consider user experience flow

**Cross-Platform Compatibility:**
- Dialog styling follows system theme and DPI settings
- Text rendering respects system font scaling
- Keyboard shortcuts work consistently across Windows versions

## Related AHK Concepts

- [MsgBox](../MsgBox/msgbox.md) - Related dialog for displaying information
- [Gui](../Gui/gui-constructor.md) - Custom GUI creation for complex input forms
- [FileSelect](../../02-File_IO_Classes/FileSelect/fileselect.md) - File selection dialogs
- [RegExMatch](../../../10_Language_Core/01-Functions/Built_In_Functions/regexmatch.md) - Pattern matching for input validation
- [IniWrite](../../02-File_IO_Classes/File/iniwrite.md) - Saving configuration data

## Tags

#AutoHotkey #InputBox #GUI #UserInput #Dialog #Validation #Interactive #Modal