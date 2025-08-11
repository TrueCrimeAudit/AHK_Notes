# Topic: FileAppend Function - File Content Appending Operations

## Category

Function

## Overview

The FileAppend function appends text or binary data to the end of existing files or creates new files if they don't exist. It's essential for logging systems, data accumulation, incremental file building, and any application requiring efficient sequential data addition without overwriting existing content.

## Key Points

- Appends content to existing files or creates new files with automatic encoding detection
- Provides efficient incremental file building without reading existing content into memory
- Supports all text encodings with automatic line ending handling for cross-platform compatibility
- Enables high-performance logging and data accumulation operations
- Essential for log files, data collection, incremental backups, and sequential file operations

## Syntax and Parameters

```cpp
FileAppend(Text, Filename [, Options])

; Parameters:
; Text     - Text or binary data to append to file
; Filename - Path to target file (created if doesn't exist)
; Options  - String containing append options (optional)

; Options format: "Encoding LineEnding CreatePath"
; Encoding:
;   UTF-8, UTF-16, UTF-8-Raw, UTF-16-Raw, CP<number>
;   Default: UTF-8 (matches existing file encoding when possible)
; LineEnding:
;   "`n"     - Unix line endings (LF)
;   "`r`n"   - Windows line endings (CRLF) - Default
;   "`r"     - Mac line endings (CR)
; CreatePath:
;   Creates parent directories if they don't exist
```

## Code Examples

```cpp
; Basic file appending
FileAppend("New line of text`n", "log.txt")

; Append with automatic line ending
logEntry := FormatTime() . " - Application event occurred"
FileAppend(logEntry . "`n", "application.log")

; Create file if it doesn't exist
FileAppend("First line`n", "newfile.txt")

; Append with specific encoding
unicodeData := "Unicode content: αβγδε`n"
FileAppend(unicodeData, "unicode.log", "UTF-16")

; Professional logging system
class Logger {
    static logFile := "application.log"
    static maxFileSize := 10485760  ; 10MB
    static logLevel := "INFO"
    static dateFormat := "yyyy-MM-dd HH:mm:ss"
    static enabled := true
    
    static SetLogFile(filename) {
        this.logFile := filename
        this.EnsureLogDirectory()
    }
    
    static SetLevel(level) {
        validLevels := ["DEBUG", "INFO", "WARN", "ERROR", "FATAL"]
        if (validLevels.IndexOf(level) > 0) {
            this.logLevel := level
        }
    }
    
    static Log(level, message, category := "GENERAL") {
        if (!this.enabled) return
        
        ; Check if level should be logged
        levelPriority := Map("DEBUG", 1, "INFO", 2, "WARN", 3, "ERROR", 4, "FATAL", 5)
        currentPriority := levelPriority.Get(this.logLevel, 2)
        messagePriority := levelPriority.Get(level, 2)
        
        if (messagePriority < currentPriority) return
        
        ; Check file size and rotate if necessary
        this.CheckRotation()
        
        ; Format log entry
        timestamp := FormatTime(, this.dateFormat)
        processId := ProcessExist()
        threadId := A_ScriptHwnd
        
        logEntry := Format("[{1}] [{2}] [{3}] [{4}:{5}] {6}`n",
                          timestamp, level, category, processId, threadId, message)
        
        ; Append to log file with error handling
        try {
            FileAppend(logEntry, this.logFile, "UTF-8")
        } catch Error as err {
            ; Fallback to system event log or console output
            OutputDebug("Logging failed: " . err.Message . " - Original message: " . message)
        }
    }
    
    static CheckRotation() {
        if (!FileExist(this.logFile)) return
        
        try {
            ; Get current file size
            file := FileOpen(this.logFile, "r")
            size := file.Length
            file.Close()
            
            if (size > this.maxFileSize) {
                this.RotateLog()
            }
        } catch Error as err {
            OutputDebug("Log rotation check failed: " . err.Message)
        }
    }
    
    static RotateLog() {
        ; Create timestamped backup
        timestamp := FormatTime(, "yyyyMMdd_HHmmss")
        backupFile := StrReplace(this.logFile, ".log", "_" . timestamp . ".log")
        
        try {
            ; Move current log to backup
            FileMove(this.logFile, backupFile)
            
            ; Start fresh log with rotation notice
            rotationMsg := Format("Log rotated at {1}, previous log: {2}`n",
                                FormatTime(, this.dateFormat), backupFile)
            FileAppend(rotationMsg, this.logFile, "UTF-8")
            
        } catch Error as err {
            OutputDebug("Log rotation failed: " . err.Message)
        }
    }
    
    static EnsureLogDirectory() {
        SplitPath(this.logFile, , &dir)
        if (dir && !DirExist(dir)) {
            try {
                DirCreate(dir)
            } catch Error as err {
                OutputDebug("Failed to create log directory: " . err.Message)
            }
        }
    }
    
    ; Convenience methods for different log levels
    static Debug(message, category := "GENERAL") => this.Log("DEBUG", message, category)
    static Info(message, category := "GENERAL") => this.Log("INFO", message, category)
    static Warn(message, category := "GENERAL") => this.Log("WARN", message, category)
    static Error(message, category := "GENERAL") => this.Log("ERROR", message, category)
    static Fatal(message, category := "GENERAL") => this.Log("FATAL", message, category)
    
    static Enable() => this.enabled := true
    static Disable() => this.enabled := false
    
    static Flush() {
        ; Force any pending writes (Windows handles this automatically for FileAppend)
        ; This method exists for API compatibility
    }
}

; Data collection and CSV building
class DataCollector {
    static dataFile := ""
    static headers := []
    static initialized := false
    static delimiter := ","
    static quoteChar := '"'
    
    static Initialize(filename, headerArray) {
        this.dataFile := filename
        this.headers := headerArray.Clone()
        
        ; Write headers if file doesn't exist
        if (!FileExist(filename)) {
            headerLine := this.FormatCSVLine(headerArray)
            FileAppend(headerLine . "`n", filename, "UTF-8")
        }
        
        this.initialized := true
    }
    
    static AddRecord(dataArray) {
        if (!this.initialized) {
            throw ValueError("DataCollector not initialized. Call Initialize() first.")
        }
        
        if (dataArray.Length != this.headers.Length) {
            throw ValueError("Data array length (" . dataArray.Length . 
                           ") doesn't match headers (" . this.headers.Length . ")")
        }
        
        try {
            csvLine := this.FormatCSVLine(dataArray)
            FileAppend(csvLine . "`n", this.dataFile, "UTF-8")
        } catch Error as err {
            throw OSError("Failed to append data: " . err.Message)
        }
    }
    
    static AddRecordObject(dataObject) {
        if (!this.initialized) {
            throw ValueError("DataCollector not initialized")
        }
        
        ; Convert object to array based on header order
        dataArray := []
        for header in this.headers {
            value := dataObject.HasProp(header) ? dataObject.%header% : ""
            dataArray.Push(value)
        }
        
        this.AddRecord(dataArray)
    }
    
    static FormatCSVLine(dataArray) {
        formattedValues := []
        
        for value in dataArray {
            valueStr := String(value)
            
            ; Quote if contains delimiter, quote char, or line breaks
            if (InStr(valueStr, this.delimiter) || 
                InStr(valueStr, this.quoteChar) || 
                InStr(valueStr, "`n") || 
                InStr(valueStr, "`r")) {
                
                ; Escape quotes by doubling them
                valueStr := StrReplace(valueStr, this.quoteChar, this.quoteChar . this.quoteChar)
                valueStr := this.quoteChar . valueStr . this.quoteChar
            }
            
            formattedValues.Push(valueStr)
        }
        
        return formattedValues.Join(this.delimiter)
    }
    
    static GetRecordCount() {
        if (!FileExist(this.dataFile)) return 0
        
        try {
            content := FileRead(this.dataFile)
            lines := StrSplit(content, "`n", "`r")
            return lines.Length - 1  ; Subtract header row
        } catch {
            return 0
        }
    }
    
    static Backup(backupSuffix := "") {
        if (!FileExist(this.dataFile)) return false
        
        if (!backupSuffix) {
            backupSuffix := "_backup_" . FormatTime(, "yyyyMMdd_HHmmss")
        }
        
        SplitPath(this.dataFile, &name, &dir, &ext)
        backupFile := dir . "\" . name . backupSuffix . "." . ext
        
        try {
            FileCopy(this.dataFile, backupFile)
            return backupFile
        } catch Error as err {
            throw OSError("Backup failed: " . err.Message)
        }
    }
}

; Configuration file builder
class ConfigBuilder {
    static configFile := ""
    static sections := Map()
    static comments := []
    
    static SetFile(filename) {
        this.configFile := filename
        this.sections.Clear()
        this.comments := []
    }
    
    static AddComment(comment) {
        this.comments.Push("# " . comment)
    }
    
    static AddSection(sectionName) {
        if (!this.sections.Has(sectionName)) {
            this.sections[sectionName] := Map()
        }
    }
    
    static SetValue(section, key, value) {
        this.AddSection(section)
        this.sections[section][key] := value
    }
    
    static Build() {
        if (!this.configFile) {
            throw ValueError("Config file not set. Call SetFile() first.")
        }
        
        try {
            ; Start with comments
            for comment in this.comments {
                FileAppend(comment . "`n", this.configFile, "UTF-8")
            }
            
            if (this.comments.Length > 0) {
                FileAppend("`n", this.configFile, "UTF-8")
            }
            
            ; Add sections
            for sectionName, sectionData in this.sections {
                FileAppend("[" . sectionName . "]`n", this.configFile, "UTF-8")
                
                for key, value in sectionData {
                    line := key . "=" . String(value) . "`n"
                    FileAppend(line, this.configFile, "UTF-8")
                }
                
                FileAppend("`n", this.configFile, "UTF-8")
            }
            
        } catch Error as err {
            throw OSError("Failed to build config file: " . err.Message)
        }
    }
    
    static AppendToSection(section, key, value) {
        ; Add to existing config file without rebuilding
        if (!FileExist(this.configFile)) {
            this.Build()
            return
        }
        
        ; Check if section exists in file
        content := FileRead(this.configFile)
        sectionPattern := "\[" . RegExReplace(section, "[\[\]\\^$.*+?{}|()]", "\\$0") . "\]"
        
        if (RegExMatch(content, sectionPattern)) {
            ; Section exists, append to it
            line := key . "=" . String(value) . "`n"
            FileAppend(line, this.configFile, "UTF-8")
        } else {
            ; Section doesn't exist, create it
            FileAppend("`n[" . section . "]`n", this.configFile, "UTF-8")
            FileAppend(key . "=" . String(value) . "`n", this.configFile, "UTF-8")
        }
    }
}

; Performance monitoring with file output
class PerformanceLogger {
    static logFile := "performance.log"
    static startTimes := Map()
    static enabled := true
    
    static StartTimer(operation) {
        if (!this.enabled) return
        
        this.startTimes[operation] := A_TickCount
    }
    
    static EndTimer(operation, details := "") {
        if (!this.enabled) return
        
        if (!this.startTimes.Has(operation)) {
            Logger.Warn("EndTimer called for operation '" . operation . "' without StartTimer")
            return
        }
        
        duration := A_TickCount - this.startTimes[operation]
        this.startTimes.Delete(operation)
        
        ; Format performance entry
        timestamp := FormatTime(, "yyyy-MM-dd HH:mm:ss")
        entry := Format("{1} | {2} | {3}ms", timestamp, operation, duration)
        
        if (details) {
            entry .= " | " . details
        }
        
        entry .= "`n"
        
        try {
            FileAppend(entry, this.logFile, "UTF-8")
        } catch Error as err {
            OutputDebug("Performance logging failed: " . err.Message)
        }
    }
    
    static LogMetric(metricName, value, unit := "") {
        if (!this.enabled) return
        
        timestamp := FormatTime(, "yyyy-MM-dd HH:mm:ss")
        entry := Format("{1} | METRIC | {2} | {3}", timestamp, metricName, value)
        
        if (unit) {
            entry .= " " . unit
        }
        
        entry .= "`n"
        
        try {
            FileAppend(entry, this.logFile, "UTF-8")
        } catch Error as err {
            OutputDebug("Metric logging failed: " . err.Message)
        }
    }
    
    static GenerateReport() {
        if (!FileExist(this.logFile)) {
            return "No performance data available"
        }
        
        try {
            content := FileRead(this.logFile)
            lines := StrSplit(content, "`n", "`r")
            
            ; Analyze performance data
            operations := Map()
            metrics := Map()
            
            for line in lines {
                if (!line) continue
                
                parts := StrSplit(line, " | ")
                if (parts.Length < 3) continue
                
                if (parts[2] = "METRIC") {
                    metricName := parts[3]
                    if (!metrics.Has(metricName)) {
                        metrics[metricName] := []
                    }
                    metrics[metricName].Push(parts[4])
                } else {
                    operation := parts[2]
                    duration := Integer(StrReplace(parts[3], "ms", ""))
                    
                    if (!operations.Has(operation)) {
                        operations[operation] := []
                    }
                    operations[operation].Push(duration)
                }
            }
            
            ; Generate summary report
            report := "Performance Analysis Report`n"
            report .= "Generated: " . FormatTime() . "`n`n"
            
            if (operations.Count > 0) {
                report .= "Operation Performance:`n"
                for operation, durations in operations {
                    sum := 0
                    min := 999999
                    max := 0
                    
                    for duration in durations {
                        sum += duration
                        if (duration < min) min := duration
                        if (duration > max) max := duration
                    }
                    
                    avg := Round(sum / durations.Length, 2)
                    report .= Format("  {1}: {2} calls, avg {3}ms, min {4}ms, max {5}ms`n",
                                   operation, durations.Length, avg, min, max)
                }
            }
            
            return report
            
        } catch Error as err {
            return "Error generating report: " . err.Message
        }
    }
}

; Example usage
; Basic logging
Logger.SetLogFile("app.log")
Logger.Info("Application started")
Logger.Warn("Configuration file not found, using defaults")
Logger.Error("Database connection failed")

; Data collection
DataCollector.Initialize("sales_data.csv", ["Date", "Product", "Amount", "Customer"])
DataCollector.AddRecord([FormatTime(, "yyyy-MM-dd"), "Widget A", "29.99", "John Doe"])
DataCollector.AddRecordObject({Date: FormatTime(, "yyyy-MM-dd"), Product: "Widget B", Amount: "39.99", Customer: "Jane Smith"})

; Configuration building
ConfigBuilder.SetFile("settings.ini")
ConfigBuilder.AddComment("Application Configuration")
ConfigBuilder.SetValue("Database", "Host", "localhost")
ConfigBuilder.SetValue("Database", "Port", "5432")
ConfigBuilder.SetValue("UI", "Theme", "dark")
ConfigBuilder.Build()

; Performance monitoring
PerformanceLogger.StartTimer("DataProcessing")
; ... some operation ...
PerformanceLogger.EndTimer("DataProcessing", "Processed 1000 records")
```

## Implementation Notes

**File Encoding Behavior:**
- FileAppend attempts to match existing file encoding automatically
- BOM (Byte Order Mark) preserved when appending to files that have one
- UTF-8 default for new files provides best cross-platform compatibility
- Raw encodings bypass BOM handling for binary data operations

**Performance Characteristics:**
- More efficient than FileRead + FileWrite for append operations
- No memory overhead for existing file content
- Optimized for sequential writes and log file operations
- File handles managed automatically for optimal performance

**Atomic Operations:**
- Each FileAppend call is atomic (either succeeds completely or fails)
- No partial writes under normal circumstances
- File locking prevents corruption during concurrent access
- Consider using file locking mechanisms for multi-process scenarios

**Line Ending Handling:**
- Windows default (`r`n) works on all platforms
- Unix line endings (`n) more efficient and widely supported
- Consistent line ending choice important for text processing
- Binary data should use Raw encodings to avoid line ending conversion

**Error Handling Strategies:**
- Check directory existence and create if necessary
- Handle permission errors gracefully with fallback options
- Monitor disk space for large append operations
- Implement retry logic for temporary file access issues

## Related AHK Concepts

- [FileWrite](./filewrite.md) - Complete file writing and overwriting
- [FileRead](./fileread.md) - Reading files for processing
- [FileOpen](./fileopen.md) - Low-level file handle operations
- [DirCreate](../Dir/dircreate.md) - Directory creation for file paths
- [FormatTime](../../../10_Language_Core/01-Functions/Built_In_Functions/formattime.md) - Timestamp formatting for logs

## Tags

#AutoHotkey #FileAppend #Logging #DataCollection #FileIO #Incremental #CSV #Configuration #Performance