# Topic: FileWrite Function - File System Output Operations

## Category

Function

## Overview

The FileWrite function writes text or binary data to files with comprehensive options for encoding, file creation, and append modes. It provides robust file output capabilities essential for logging, data export, configuration management, and any application requiring persistent data storage.

## Key Points

- Writes text or binary data to files with automatic encoding handling
- Supports file creation, overwriting, and append modes for flexible output control
- Handles various text encodings (UTF-8, UTF-16, ANSI) for international compatibility
- Provides atomic write operations and proper error handling for data integrity
- Essential for logging, configuration management, data export, and backup operations

## Syntax and Parameters

```cpp
FileWrite(Text, Filename [, Options])

; Parameters:
; Text     - Text or binary data to write to file
; Filename - Path to target file (relative or absolute)
; Options  - String containing write options (optional)

; Options format: "Encoding AppendMode CreatePath"
; Encoding:
;   UTF-8, UTF-16, UTF-8-Raw, UTF-16-Raw, CP<number>
;   Default: UTF-8 (with BOM for new files)
; AppendMode:
;   "`n" - Append with newline
;   "`r`n" - Append with Windows line ending
;   "" - Overwrite file (default)
; CreatePath:
;   Creates parent directories if they don't exist
```

## Code Examples

```cpp
; Basic file writing
text := "Hello, World!"
FileWrite(text, "output.txt")

; Append to existing file
logEntry := FormatTime() . " - Application started`n"
FileWrite(logEntry, "app.log", "`n")

; Write with specific encoding
unicodeText := "Unicode: αβγδε"
FileWrite(unicodeText, "unicode.txt", "UTF-16")

; Write binary data
binaryData := Buffer(256)
; Fill buffer with data...
FileWrite(binaryData, "data.bin", "UTF-8-Raw")

; Create directories and write file
configData := "setting1=value1`nsetting2=value2"
FileWrite(configData, "Config\User\settings.ini", "UTF-8 CreatePath")

; Safe file writing with error handling
function SafeFileWrite(content, filename, options := "") {
    try {
        ; Backup existing file if it exists
        if (FileExist(filename)) {
            backupName := filename . ".backup"
            FileCopy(filename, backupName, true)
        }
        
        ; Write new content
        FileWrite(content, filename, options)
        
        ; Verify write was successful
        if (FileExist(filename)) {
            written := FileRead(filename)
            if (written = content) {
                ; Clean up backup on success
                if (FileExist(filename . ".backup")) {
                    FileDelete(filename . ".backup")
                }
                return true
            }
        }
        
        throw Error("File write verification failed")
        
    } catch Error as err {
        ; Restore backup if write failed
        backupFile := filename . ".backup"
        if (FileExist(backupFile)) {
            FileMove(backupFile, filename, true)
        }
        
        throw Error("Failed to write file: " . err.Message)
    }
}

; Logging system with rotation
class Logger {
    static logFile := "application.log"
    static maxFileSize := 1048576  ; 1MB
    static maxBackups := 5
    
    static Log(level, message) {
        timestamp := FormatTime(, "yyyy-MM-dd HH:mm:ss")
        logEntry := Format("[{1}] {2}: {3}`n", timestamp, level, message)
        
        ; Check if log rotation is needed
        this.CheckRotation()
        
        ; Append to log file
        try {
            FileWrite(logEntry, this.logFile, "`n")
        } catch Error as err {
            ; Fallback to system event log or stdout
            OutputDebug("Logging failed: " . err.Message)
        }
    }
    
    static CheckRotation() {
        if (!FileExist(this.logFile)) {
            return
        }
        
        ; Get file size
        file := FileOpen(this.logFile, "r")
        fileSize := file.Length
        file.Close()
        
        if (fileSize > this.maxFileSize) {
            this.RotateLog()
        }
    }
    
    static RotateLog() {
        ; Rotate existing backup files
        loop this.maxBackups {
            index := this.maxBackups - A_Index + 1
            oldFile := this.logFile . "." . index
            newFile := this.logFile . "." . (index + 1)
            
            if (FileExist(oldFile)) {
                if (index = this.maxBackups) {
                    FileDelete(oldFile)  ; Delete oldest
                } else {
                    FileMove(oldFile, newFile, true)
                }
            }
        }
        
        ; Move current log to .1
        if (FileExist(this.logFile)) {
            FileMove(this.logFile, this.logFile . ".1", true)
        }
    }
    
    static Info(message) => this.Log("INFO", message)
    static Warning(message) => this.Log("WARN", message)
    static Error(message) => this.Log("ERROR", message)
    static Debug(message) => this.Log("DEBUG", message)
}

; Configuration file management
class ConfigManager {
    configFile := ""
    settings := Map()
    
    __New(filename) {
        this.configFile := filename
        this.LoadConfig()
    }
    
    LoadConfig() {
        try {
            if (FileExist(this.configFile)) {
                content := FileRead(this.configFile)
                this.ParseConfig(content)
            }
        } catch Error as err {
            Logger.Error("Failed to load config: " . err.Message)
        }
    }
    
    ParseConfig(content) {
        this.settings.Clear()
        
        loop parse, content, "`n", "`r" {
            line := Trim(A_LoopField)
            
            ; Skip comments and empty lines
            if (!line || SubStr(line, 1, 1) = "#") {
                continue
            }
            
            ; Parse key=value pairs
            if (pos := InStr(line, "=")) {
                key := Trim(SubStr(line, 1, pos - 1))
                value := Trim(SubStr(line, pos + 1))
                this.settings[key] := value
            }
        }
    }
    
    SaveConfig() {
        try {
            ; Build configuration content
            content := "# Configuration file generated on " . FormatTime() . "`n"
            content .= "# Do not edit manually`n`n"
            
            for key, value in this.settings {
                content .= key . "=" . value . "`n"
            }
            
            ; Ensure directory exists
            SplitPath(this.configFile, , &dir)
            if (dir && !DirExist(dir)) {
                DirCreate(dir)
            }
            
            ; Write with atomic operation
            tempFile := this.configFile . ".tmp"
            FileWrite(content, tempFile, "UTF-8")
            
            ; Replace original file
            if (FileExist(this.configFile)) {
                FileDelete(this.configFile)
            }
            FileMove(tempFile, this.configFile)
            
            Logger.Info("Configuration saved successfully")
            return true
            
        } catch Error as err {
            Logger.Error("Failed to save config: " . err.Message)
            return false
        }
    }
    
    Set(key, value) {
        this.settings[key] := value
    }
    
    Get(key, defaultValue := "") {
        return this.settings.Has(key) ? this.settings[key] : defaultValue
    }
}

; CSV export functionality
function ExportToCSV(data, filename, headers := "") {
    try {
        content := ""
        
        ; Add headers if provided
        if (headers) {
            content .= headers . "`n"
        }
        
        ; Add data rows
        for index, row in data {
            if (Type(row) = "Array") {
                ; Array of values
                line := ""
                for i, value in row {
                    if (i > 1) line .= ","
                    
                    ; Escape CSV special characters
                    valueStr := String(value)
                    if (InStr(valueStr, ",") || InStr(valueStr, "`"") || InStr(valueStr, "`n")) {
                        valueStr := "`"" . StrReplace(valueStr, "`"", "`"`"") . "`""
                    }
                    line .= valueStr
                }
                content .= line . "`n"
            } else {
                ; Single value
                content .= String(row) . "`n"
            }
        }
        
        ; Write to file
        FileWrite(content, filename, "UTF-8")
        return true
        
    } catch Error as err {
        MsgBox("CSV export failed: " . err.Message)
        return false
    }
}

; Usage example
data := [
    ["Name", "Age", "City"],
    ["John Doe", 30, "New York"],
    ["Jane Smith", 25, "Los Angeles"],
    ["Bob Johnson", 35, "Chicago"]
]

ExportToCSV(data, "export.csv")
```

## Implementation Notes

**File Encoding Handling:**
- UTF-8 is default and recommended for cross-platform compatibility
- BOM (Byte Order Mark) added automatically for UTF-8 and UTF-16
- Raw encodings (UTF-8-Raw, UTF-16-Raw) write without BOM
- Legacy codepages supported via CP<number> format

**Write Modes and Behavior:**
- Default mode overwrites existing files completely
- Append modes preserve existing content and add new data
- File creation is atomic - either succeeds completely or fails safely
- Directory creation optional via CreatePath option

**Error Handling Best Practices:**
- Always wrap FileWrite in try-catch blocks
- Verify write success by reading back critical data
- Implement backup strategies for important files
- Handle disk space and permission errors gracefully

**Performance Considerations:**
- Large files written in single operation are more efficient than multiple small writes
- Buffer writes when possible for better performance
- Consider using FileAppend for log files instead of repeated FileWrite calls
- File locking may occur with concurrent access - handle appropriately

**Security and Data Integrity:**
- Validate input data before writing to prevent corruption
- Use temporary files and atomic operations for critical data
- Set appropriate file permissions for sensitive data
- Consider encryption for confidential information

## Related AHK Concepts

- [FileRead](./fileread.md) - Reading files and input operations
- [FileAppend](./fileappend.md) - Specialized function for appending data
- [FileOpen](./fileopen.md) - Low-level file handle operations
- [DirCreate](../Dir/dircreate.md) - Directory creation for file paths
- [FileCopy](./filecopy.md) - File copying and backup operations
- [Buffer](../../../10_Language_Core/02-Data_Types/buffer.md) - Binary data handling

## Tags

#AutoHotkey #FileWrite #FileIO #Output #Encoding #Logging #Configuration #Export #TextProcessing