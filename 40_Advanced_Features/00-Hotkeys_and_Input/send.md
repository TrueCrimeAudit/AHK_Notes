# Function: Send

## Category

Built-in Function

## Overview

Send simulates keystrokes and key combinations, enabling automation of keyboard input to any application. It's the cornerstone function for AutoHotkey automation, allowing scripts to interact with programs by sending text, special keys, and complex key combinations with precise timing and formatting.

## Syntax

```cpp
Send(Keys)
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Keys | String | Yes | The sequence of keys to send, including text and special key sequences |

## Return Value

| Type | Description |
|------|-------------|
| None | Send does not return a value; it performs the key sending operation |

## Key Sequences

### Text Characters
```cpp
Send("Hello World")  ; Sends literal text
Send("user@email.com")  ; Sends email address
```

### Special Keys
| Key Sequence | Description | Example |
|--------------|-------------|---------|
| `{Enter}` | Enter key | `Send("{Enter}")` |
| `{Tab}` | Tab key | `Send("{Tab}")` |
| `{Space}` | Space key | `Send("{Space}")` |
| `{Backspace}` | Backspace key | `Send("{Backspace}")` |
| `{Delete}` | Delete key | `Send("{Delete}")` |
| `{Escape}` | Escape key | `Send("{Escape}")` |
| `{Home}` | Home key | `Send("{Home}")` |
| `{End}` | End key | `Send("{End}")` |
| `{Up}`, `{Down}`, `{Left}`, `{Right}` | Arrow keys | `Send("{Down 3}")` |
| `{F1}` through `{F24}` | Function keys | `Send("{F5}")` |

### Modifier Keys
| Modifier | Symbol | Description | Example |
|----------|--------|-------------|---------|
| Shift | `+` | Hold Shift | `Send("+{Tab}")` |
| Ctrl | `^` | Hold Ctrl | `Send("^c")` |
| Alt | `!` | Hold Alt | `Send("!{F4}")` |
| Win | `#` | Hold Windows key | `Send("#r")` |

### Advanced Sequences
```cpp
; Multiple key repetition
Send("{Tab 5}")           ; Press Tab 5 times
Send("{Backspace 10}")    ; Press Backspace 10 times

; Key combinations
Send("^a")                ; Ctrl+A (Select All)
Send("^c")                ; Ctrl+C (Copy)
Send("^v")                ; Ctrl+V (Paste)
Send("!{Tab}")           ; Alt+Tab (Switch windows)

; Complex combinations
Send("^+{Home}")         ; Ctrl+Shift+Home
Send("#+{Left}")         ; Win+Shift+Left Arrow
```

## Exceptions

| Exception Type | When Thrown |
|----------------|-------------|
| ValueError | When key sequence contains invalid syntax |
| OSError | When system cannot send keys (rare hardware/driver issues) |

## Basic Examples

```cpp
; Send simple text
Send("Hello, this is automated text!")

; Navigate through a form
Send("{Tab}")              ; Move to next field
Send("John Doe")           ; Enter name
Send("{Tab}")              ; Move to next field
Send("john@email.com")     ; Enter email
Send("{Enter}")            ; Submit form

; File operations
Send("^o")                 ; Ctrl+O (Open dialog)
Sleep(500)                 ; Wait for dialog
Send("document.txt")       ; Type filename
Send("{Enter}")            ; Open file

; Text editing
Send("^a")                 ; Select all text
Send("This replaces all content")  ; Type new content
Send("^s")                 ; Save file
```

## Advanced Examples

```cpp
; Advanced text input with formatting and validation
SendFormattedText(text, validateTarget := true) {
    ; Validate active window if requested
    if (validateTarget) {
        local activeWindow := WinGetTitle("A")
        if (InStr(activeWindow, "Notepad") == 0 && InStr(activeWindow, "Word") == 0) {
            MsgBox("Warning: Sending text to " . activeWindow . ". Continue?", "Target Validation", "YesNo")
            if (result != "Yes") {
                return false
            }
        }
    }
    
    ; Clear existing content safely
    Send("^a")  ; Select all
    Sleep(50)   ; Brief pause for selection
    
    ; Send text with proper escaping
    local escapedText := EscapeForSend(text)
    Send(escapedText)
    
    return true
}

EscapeForSend(text) {
    ; Escape special characters that have meaning in Send
    text := StrReplace(text, "{", "{{}")
    text := StrReplace(text, "}", "{}}")
    text := StrReplace(text, "+", "{+}")
    text := StrReplace(text, "^", "{^}")
    text := StrReplace(text, "!", "{!}")
    text := StrReplace(text, "#", "{#}")
    return text
}

; Usage
SendFormattedText("Special characters: {}+^!# are escaped properly")
```

```cpp
; Automated application interaction class
class AppAutomator {
    static SendKeysWithTiming(keySequence, baseDelay := 50) {
        ; Split complex key sequences and add timing
        local sequences := StrSplit(keySequence, "|")
        
        for index, sequence in sequences {
            sequence := Trim(sequence)
            
            ; Calculate delay based on sequence complexity
            local delay := baseDelay
            if (InStr(sequence, "^") || InStr(sequence, "!")) {
                delay *= 2  ; Longer delay for modifier combinations
            }
            
            Send(sequence)
            
            ; Don't delay after the last sequence
            if (index < sequences.Length) {
                Sleep(delay)
            }
        }
    }
    
    static FillForm(formData) {
        ; Automated form filling with error recovery
        for fieldName, fieldValue in formData {
            try {
                ; Navigate to field (assuming tab order)
                Send("{Tab}")
                Sleep(100)
                
                ; Clear field and enter value
                Send("^a")
                Sleep(50)
                Send(fieldValue)
                Sleep(100)
                
                ; Verify field was filled (basic check)
                Send("^a")
                Send("^c")
                Sleep(100)
                
                ; Could add clipboard verification here
                
            } catch Error as err {
                MsgBox("Error filling field '" . fieldName . "': " . err.Message)
                return false
            }
        }
        
        return true
    }
    
    static NavigateApplication(navigationPath) {
        ; Navigate through application menus
        for index, step in navigationPath {
            switch step.type {
                case "menu":
                    Send("!" . step.key)  ; Alt + menu key
                case "key":
                    Send(step.key)
                case "wait":
                    Sleep(step.duration)
                case "text":
                    Send(step.content)
            }
            
            ; Default pause between actions
            Sleep(step.hasOwnProp("delay") ? step.delay : 200)
        }
    }
}

; Usage examples
AppAutomator.SendKeysWithTiming("^o|document.txt|{Enter}")

formData := Map("name", "John Doe", "email", "john@example.com", "phone", "555-1234")
AppAutomator.FillForm(formData)

navigation := [
    {type: "menu", key: "f"},           ; Alt+F (File menu)
    {type: "key", key: "o"},            ; O (Open)
    {type: "wait", duration: 500},      ; Wait for dialog
    {type: "text", content: "data.txt"}, ; Type filename
    {type: "key", key: "{Enter}"}       ; Confirm
]
AppAutomator.NavigateApplication(navigation)
```

## Real-World Example

```cpp
; Complete email composition automation
class EmailAutomator {
    static ComposeEmail(recipient, subject, body, attachments := []) {
        ; Open email client (assuming Outlook/similar)
        Send("^n")  ; New email shortcut
        Sleep(1000) ; Wait for compose window
        
        ; Verify compose window opened
        if (!WinExist("- Message")) {
            MsgBox("Email compose window not detected", "Error", "OK + Error")
            return false
        }
        
        try {
            ; Fill recipient
            Send(recipient)
            Send("{Tab}")  ; Move to subject
            
            ; Fill subject
            Send(subject)
            Send("{Tab}")  ; Move to body
            
            ; Handle body content (could be multiline)
            local bodyLines := StrSplit(body, "`n")
            for index, line in bodyLines {
                Send(line)
                if (index < bodyLines.Length) {
                    Send("{Enter}")
                }
            }
            
            ; Add attachments if provided
            if (attachments.Length > 0) {
                this.AddAttachments(attachments)
            }
            
            ; Ask for confirmation before sending
            local confirmMsg := "Email ready to send to: " . recipient
            confirmMsg .= "`nSubject: " . subject
            confirmMsg .= "`n`nSend now?"
            
            if (MsgBox(confirmMsg, "Confirm Send", "YesNo + Question") == "Yes") {
                Send("^{Enter}")  ; Send email (common shortcut)
                return true
            } else {
                MsgBox("Email composed but not sent", "Info")
                return false
            }
            
        } catch Error as err {
            MsgBox("Error composing email: " . err.Message, "Error", "OK + Error")
            return false
        }
    }
    
    static AddAttachments(attachments) {
        ; Open attachment dialog
        Send("^{Shift}a")  ; Common attachment shortcut
        Sleep(500)
        
        for index, filePath in attachments {
            if (FileExist(filePath)) {
                Send(filePath)
                Send("{Enter}")
                Sleep(300)
            } else {
                MsgBox("Attachment not found: " . filePath, "Warning", "OK + Warning")
            }
        }
        
        ; Close attachment dialog
        Send("{Escape}")
    }
    
    static SendBulkEmails(emailList) {
        local sent := 0
        local failed := Array()
        
        for index, emailData in emailList {
            MsgBox("Sending email " . index . " of " . emailList.Length, "Progress", "OK")
            
            if (this.ComposeEmail(emailData.recipient, emailData.subject, emailData.body, emailData.attachments)) {
                sent++
                Sleep(2000)  ; Pause between emails
            } else {
                failed.Push(emailData.recipient)
            }
        }
        
        ; Report results
        local report := "Bulk email complete:`n"
        report .= "Sent: " . sent . "`n"
        report .= "Failed: " . failed.Length
        
        if (failed.Length > 0) {
            report .= "`n`nFailed recipients: " . StrJoin(failed, ", ")
        }
        
        MsgBox(report, "Bulk Email Results", "OK + Information")
    }
}

; Usage
EmailAutomator.ComposeEmail(
    "colleague@company.com",
    "Weekly Report - " . FormatTime(A_Now, "yyyy-MM-dd"),
    "Hi Team,`n`nPlease find attached this week's report.`n`nBest regards,`nAutomated System",
    ["C:\Reports\weekly_report.pdf"]
)
```

## Common Use Cases

- **Form Automation**: Filling web forms and application dialogs
- **Text Entry**: Inserting boilerplate text and templates
- **Application Control**: Navigating menus and interfaces via keyboard
- **Data Entry**: Automating repetitive typing tasks
- **Hotkey Responses**: Sending complex key sequences from simple hotkeys
- **Window Management**: Using keyboard shortcuts for window operations

## Performance Notes

- Send rate is limited by target application's input processing speed
- Complex key sequences may require timing adjustments with Sleep()
- Modifier key combinations have slight additional overhead
- Very rapid sending may overwhelm some applications - add delays as needed

## Common Pitfalls

### Pitfall 1: Special Character Escaping
**Problem**: Special characters in text aren't properly escaped for Send()
**Solution**: Escape special characters that have meaning in Send syntax
```cpp
; Problematic - special characters interfere
Send("Email: user@domain.com {urgent}")  ; {urgent} treated as key sequence

; Correct - escape special characters
Send("Email: user@domain.com {{urgent}}")  ; Literal {urgent}

; Better - use SendText for literal text
SendText("Email: user@domain.com {urgent}")  ; SendText handles escaping
```

### Pitfall 2: Timing Issues
**Problem**: Sending keys too fast for target application to process
**Solution**: Add appropriate delays between key sequences
```cpp
; Problematic - too fast for some applications
Send("^o")
Send("filename.txt")
Send("{Enter}")

; Better - add timing delays
Send("^o")
Sleep(500)        ; Wait for dialog
Send("filename.txt")
Sleep(100)
Send("{Enter}")
```

### Pitfall 3: Target Window Focus
**Problem**: Sending keys to wrong window because focus changed
**Solution**: Verify target window before sending keys
```cpp
; Problematic - assumes correct window has focus
Send("important data")

; Better - verify target window
if (WinActive("Notepad")) {
    Send("important data")
} else {
    MsgBox("Target window not active!")
}

; Best - activate target window first
WinActivate("Notepad")
WinWaitActive("Notepad", , 2)  ; Wait up to 2 seconds
if (WinActive("Notepad")) {
    Send("important data")
}
```

## Version History

- **v2.0**: Improved Unicode support and better application compatibility
- **v2.0-a**: Enhanced key sequence parsing and timing control
- **v2.1**: Additional special key support and performance optimizations

## Related Functions

- [SendText](../sendtext.md) - Send literal text without key sequence interpretation
- [SendInput](../sendinput.md) - Send input using more reliable input method
- [Click](../click.md) - Simulate mouse clicks
- [ControlSend](../controlsend.md) - Send keys directly to specific controls

## Related Concepts

- [Hotkey Programming](../../../40_Advanced_Features/00-Hotkeys_and_Input/hotkey-basics.md) - Using Send in hotkey responses
- [Application Automation](../../../50_Ecosystem/00-Design_Patterns/automation-patterns.md) - Patterns for automating applications
- [Input Simulation](../../../40_Advanced_Features/00-Hotkeys_and_Input/input-simulation.md) - Advanced input automation techniques

## See Also

- [Input Automation Guide](../../../Index/learning-paths.md#system-automator-path) - Complete automation development
- [Timing and Synchronization](../../../50_Ecosystem/01-Best_Practices/timing-control.md) - Managing automation timing
- [Application Integration](../../../50_Ecosystem/02-Performance_Optimization/app-integration.md) - Optimizing application automation

## Tags

#AutoHotkey #Function #Input #Automation #Keyboard #KeySend #SendKeys #Essential #BuiltIn #ApplicationControl