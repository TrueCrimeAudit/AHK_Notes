# Function: FileRead

## Category

Built-in Function

## Overview

FileRead reads the entire contents of a file into a string variable, providing essential file input capabilities for AutoHotkey scripts. It handles various text encodings and is fundamental for configuration loading, data processing, and text file manipulation in automation tasks.

## Syntax

```cpp
Text := FileRead(Filename [, Options])
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Filename | String | Yes | Path to the file to read. Can be absolute or relative to working directory |
| Options | String | No | Encoding and other options (see Options table below) |

## Options Parameter

| Option | Description | Example |
|--------|-------------|---------|
| `UTF-8` | Read file as UTF-8 encoded text | `FileRead("file.txt", "UTF-8")` |
| `UTF-16` | Read file as UTF-16 encoded text | `FileRead("file.txt", "UTF-16")` |
| `CP1252` | Read file as Windows-1252 encoded | `FileRead("file.txt", "CP1252")` |
| `RAW` | Read file as raw binary data | `FileRead("data.bin", "RAW")` |
| `*m1000` | Limit read to first 1000 bytes | `FileRead("file.txt", "*m1000")` |

## Return Value

| Type | Description |
|------|-------------|
| String | The complete contents of the file as a string |

## Exceptions

| Exception Type | When Thrown |
|----------------|-------------|
| OSError | When file cannot be opened (doesn't exist, no permissions, etc.) |
| MemoryError | When file is too large to fit in available memory |
| ValueError | When invalid options are specified |

## Basic Examples

```cpp
; Read a simple text file
content := FileRead("config.txt")
MsgBox("File contents: " . content)

; Read with specific encoding
unicodeContent := FileRead("unicode_file.txt", "UTF-8")

; Read with error handling
try {
    data := FileRead("data.txt")
    MsgBox("Successfully read " . StrLen(data) . " characters")
} catch OSError as err {
    MsgBox("Failed to read file: " . err.Message)
}

; Read first 500 characters only
preview := FileRead("large_file.txt", "*m500")
```

## Advanced Examples

```cpp
; Robust file reading with comprehensive error handling
SafeFileRead(filePath, encoding := "", maxSize := 0) {
    local options := ""
    
    ; Build options string
    if (encoding != "") {
        options := encoding
    }
    if (maxSize > 0) {
        options .= (options != "" ? " " : "") . "*m" . maxSize
    }
    
    try {
        ; Check if file exists first
        if (!FileExist(filePath)) {
            throw OSError("File does not exist: " . filePath)
        }
        
        ; Get file size for validation
        local fileSize := FileGetSize(filePath)
        if (fileSize > 100000000) {  ; 100MB limit
            throw MemoryError("File too large: " . Round(fileSize/1024/1024, 1) . "MB")
        }
        
        ; Read the file
        local startTime := A_TickCount
        local content := FileRead(filePath, options)
        local readTime := A_TickCount - startTime
        
        ; Log successful read
        OutputDebug("FileRead: " . filePath . " (" . StrLen(content) . " chars, " . readTime . "ms)")
        
        return {
            content: content,
            success: true,
            size: StrLen(content),
            readTime: readTime,
            encoding: encoding != "" ? encoding : "default"
        }
        
    } catch Error as err {
        ; Log error
        OutputDebug("FileRead failed: " . filePath . " - " . err.Message)
        
        return {
            content: "",
            success: false,
            error: err.Message,
            errorType: Type(err)
        }
    }
}

; Usage examples
result := SafeFileRead("config.ini", "UTF-8")
if (result.success) {
    ProcessConfigData(result.content)
} else {
    MsgBox("Config read failed: " . result.error)
}
```

```cpp
; Configuration file manager using FileRead
class ConfigManager {
    static configPath := ""
    static configData := Map()
    static lastModified := ""
    
    static LoadConfig(filePath) {
        this.configPath := filePath
        
        try {
            ; Check if config file exists
            if (!FileExist(filePath)) {
                this.CreateDefaultConfig(filePath)
            }
            
            ; Read configuration file
            local configText := FileRead(filePath, "UTF-8")
            local fileTime := FileGetTime(filePath, "M")
            
            ; Parse configuration
            this.configData := this.ParseConfigText(configText)
            this.lastModified := fileTime
            
            return {
                success: true,
                itemsLoaded: this.configData.Count,
                lastModified: fileTime
            }
            
        } catch Error as err {
            return {
                success: false,
                error: err.Message,
                errorType: Type(err)
            }
        }
    }
    
    static ParseConfigText(configText) {
        local config := Map()
        local currentSection := "default"
        local lines := StrSplit(configText, "`n")
        
        for lineNum, line in lines {
            line := Trim(line)
            
            ; Skip empty lines and comments
            if (line == "" || SubStr(line, 1, 1) == ";") {
                continue
            }
            
            ; Handle sections [SectionName]
            if (SubStr(line, 1, 1) == "[" && SubStr(line, -1) == "]") {
                currentSection := SubStr(line, 2, -1)
                if (!config.Has(currentSection)) {
                    config[currentSection] := Map()
                }
                continue
            }
            
            ; Handle key=value pairs
            local equalPos := InStr(line, "=")
            if (equalPos > 0) {
                local key := Trim(SubStr(line, 1, equalPos - 1))
                local value := Trim(SubStr(line, equalPos + 1))
                
                ; Ensure section exists
                if (!config.Has(currentSection)) {
                    config[currentSection] := Map()
                }
                
                config[currentSection][key] := value
            }
        }
        
        return config
    }
    
    static GetValue(section, key, defaultValue := "") {
        if (this.configData.Has(section) && this.configData[section].Has(key)) {
            return this.configData[section][key]
        }
        return defaultValue
    }
    
    static ReloadIfModified() {
        if (this.configPath == "") {
            return false
        }
        
        try {
            local currentTime := FileGetTime(this.configPath, "M")
            if (currentTime != this.lastModified) {
                return this.LoadConfig(this.configPath).success
            }
        } catch {
            ; File might have been deleted
            return false
        }
        
        return true
    }
    
    static CreateDefaultConfig(filePath) {
        local defaultConfig := "; AutoHotkey Configuration File`n"
        defaultConfig .= "; Generated: " . FormatTime(A_Now) . "`n`n"
        defaultConfig .= "[General]`n"
        defaultConfig .= "AppName=AutoHotkey Application`n"
        defaultConfig .= "Version=1.0`n"
        defaultConfig .= "Debug=false`n`n"
        defaultConfig .= "[Paths]`n"
        defaultConfig .= "DataDir=Data`n"
        defaultConfig .= "LogDir=Logs`n"
        
        FileAppend(defaultConfig, filePath, "UTF-8")
    }
}

; Usage
ConfigManager.LoadConfig("app_config.ini")
appName := ConfigManager.GetValue("General", "AppName", "Default App")
debugMode := ConfigManager.GetValue("General", "Debug", "false") == "true"
```

## Real-World Example

```cpp
; Log file analyzer using FileRead
class LogAnalyzer {
    static AnalyzeLogFile(logPath, options := "") {
        local analysis := {
            file: logPath,
            totalLines: 0,
            errors: 0,
            warnings: 0,
            info: 0,
            timeRange: {start: "", end: ""},
            topErrors: Map(),
            summary: ""
        }
        
        try {
            ; Read log file with encoding detection
            local encoding := this.DetectEncoding(logPath)
            local logContent := FileRead(logPath, encoding)
            
            ; Basic statistics
            analysis.fileSize := StrLen(logContent)
            analysis.encoding := encoding
            
            ; Parse log content
            local lines := StrSplit(logContent, "`n")
            analysis.totalLines := lines.Length
            
            ; Analyze each line
            for lineNum, line in lines {
                line := Trim(line)
                if (line == "") continue
                
                ; Extract timestamp (assuming standard format)
                local timestamp := this.ExtractTimestamp(line)
                if (timestamp != "") {
                    if (analysis.timeRange.start == "" || timestamp < analysis.timeRange.start) {
                        analysis.timeRange.start := timestamp
                    }
                    if (analysis.timeRange.end == "" || timestamp > analysis.timeRange.end) {
                        analysis.timeRange.end := timestamp
                    }
                }
                
                ; Categorize log level
                if (InStr(line, "ERROR") > 0) {
                    analysis.errors++
                    this.TrackError(line, analysis.topErrors)
                } else if (InStr(line, "WARN") > 0) {
                    analysis.warnings++
                } else if (InStr(line, "INFO") > 0) {
                    analysis.info++
                }
            }
            
            ; Generate summary
            analysis.summary := this.GenerateSummary(analysis)
            
            return analysis
            
        } catch Error as err {
            return {
                file: logPath,
                error: err.Message,
                success: false
            }
        }
    }
    
    static DetectEncoding(filePath) {
        ; Simple encoding detection by reading first few bytes
        try {
            local sample := FileRead(filePath, "*m100")
            
            ; Check for BOM
            if (Ord(SubStr(sample, 1, 1)) == 0xEF) {
                return "UTF-8"  ; UTF-8 BOM
            }
            if (Ord(SubStr(sample, 1, 1)) == 0xFF || Ord(SubStr(sample, 1, 1)) == 0xFE) {
                return "UTF-16"  ; UTF-16 BOM
            }
            
            ; Default to UTF-8 for text files
            return "UTF-8"
            
        } catch {
            return ""  ; Use default encoding
        }
    }
    
    static ExtractTimestamp(line) {
        ; Extract timestamp from log line (basic implementation)
        ; Assumes format like "2024-01-15 10:30:45"
        local timePattern := "\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}"
        
        if (RegExMatch(line, timePattern, &match)) {
            return match[0]
        }
        
        return ""
    }
    
    static TrackError(errorLine, topErrors) {
        ; Extract error type/message for tracking
        local errorMsg := ""
        
        ; Try to extract meaningful error from line
        if (RegExMatch(errorLine, "ERROR[:\s]+([^,\n\r]+)", &match)) {
            errorMsg := Trim(match[1])
        } else {
            ; Fallback to first 50 characters after ERROR
            local errorPos := InStr(errorLine, "ERROR")
            if (errorPos > 0) {
                errorMsg := SubStr(errorLine, errorPos + 5, 50)
                errorMsg := Trim(errorMsg)
            }
        }
        
        if (errorMsg != "") {
            if (topErrors.Has(errorMsg)) {
                topErrors[errorMsg]++
            } else {
                topErrors[errorMsg] := 1
            }
        }
    }
    
    static GenerateSummary(analysis) {
        local summary := "Log Analysis Summary:`n"
        summary .= "File: " . analysis.file . "`n"
        summary .= "Size: " . analysis.fileSize . " characters`n"
        summary .= "Lines: " . analysis.totalLines . "`n"
        summary .= "Encoding: " . analysis.encoding . "`n`n"
        
        summary .= "Log Levels:`n"
        summary .= "  Errors: " . analysis.errors . "`n"
        summary .= "  Warnings: " . analysis.warnings . "`n"
        summary .= "  Info: " . analysis.info . "`n`n"
        
        if (analysis.timeRange.start != "") {
            summary .= "Time Range:`n"
            summary .= "  Start: " . analysis.timeRange.start . "`n"
            summary .= "  End: " . analysis.timeRange.end . "`n`n"
        }
        
        if (analysis.topErrors.Count > 0) {
            summary .= "Top Errors:`n"
            local count := 0
            for error, frequency in analysis.topErrors {
                if (++count > 5) break  ; Top 5 errors
                summary .= "  " . frequency . "x: " . error . "`n"
            }
        }
        
        return summary
    }
    
    static ProcessMultipleLogs(logDirectory) {
        local results := Array()
        
        ; Find all log files
        local logFiles := []
        
        ; Use Dir() to enumerate files (assuming *.log pattern)
        for file in DirEnum(logDirectory, "*.log") {
            logFiles.Push(file.Name)
        }
        
        ; Analyze each log file
        for index, logFile in logFiles {
            local fullPath := logDirectory . "\" . logFile
            local analysis := this.AnalyzeLogFile(fullPath)
            results.Push(analysis)
            
            ; Progress indication
            MsgBox("Analyzed " . index . " of " . logFiles.Length . " files", "Progress", "OK T1")
        }
        
        return results
    }
}

; Usage examples
analysis := LogAnalyzer.AnalyzeLogFile("application.log")
if (analysis.hasOwnProp("error")) {
    MsgBox("Analysis failed: " . analysis.error)
} else {
    MsgBox(analysis.summary)
}

; Batch processing
batchResults := LogAnalyzer.ProcessMultipleLogs("C:\Logs")
```

## Common Use Cases

- **Configuration Loading**: Reading application settings from INI, JSON, or custom config files
- **Data Processing**: Loading CSV, TSV, or other structured text files
- **Template Processing**: Reading template files for content generation
- **Log Analysis**: Processing log files for monitoring and debugging
- **Script Configuration**: Loading external script parameters and settings
- **Content Management**: Reading text content for processing and transformation

## Performance Notes

- FileRead loads entire file into memory - not suitable for very large files (>100MB)
- Performance scales linearly with file size
- UTF-8 encoding detection adds minimal overhead
- Consider streaming approaches (File object) for very large files
- SSD vs HDD significantly affects read performance for large files

## Common Pitfalls

### Pitfall 1: Large File Memory Issues
**Problem**: Attempting to read very large files can cause memory issues
**Solution**: Check file size before reading and use streaming for large files
```cpp
; Problematic - reading large file without size check
largeContent := FileRead("huge_file.txt")  ; May cause memory error

; Better - check size first
fileSize := FileGetSize("huge_file.txt")
if (fileSize > 50000000) {  ; 50MB limit
    MsgBox("File too large for FileRead")
} else {
    content := FileRead("huge_file.txt")
}

; Best - use File object for large files
file := FileOpen("huge_file.txt", "r")
while (!file.AtEOF) {
    chunk := file.Read(8192)  ; Read in chunks
    ProcessChunk(chunk)
}
file.Close()
```

### Pitfall 2: Encoding Issues
**Problem**: Not specifying correct encoding leads to garbled text
**Solution**: Detect or specify appropriate encoding for the file
```cpp
; Problematic - assuming default encoding
content := FileRead("unicode_file.txt")  ; May show garbled text

; Better - specify encoding explicitly
content := FileRead("unicode_file.txt", "UTF-8")

; Best - detect encoding or try multiple encodings
encodings := ["UTF-8", "UTF-16", "CP1252"]
for encoding in encodings {
    try {
        content := FileRead("file.txt", encoding)
        ; Validate content makes sense
        if (IsValidText(content)) {
            break
        }
    } catch {
        continue
    }
}
```

### Pitfall 3: File Path Issues
**Problem**: Using incorrect or non-existent file paths
**Solution**: Always validate file existence and use proper path handling
```cpp
; Problematic - assuming file exists
content := FileRead("config.txt")  ; Throws error if file doesn't exist

; Better - check existence first
if (FileExist("config.txt")) {
    content := FileRead("config.txt")
} else {
    MsgBox("Config file not found")
}

; Best - use full path and error handling
configPath := A_ScriptDir . "\config.txt"
try {
    content := FileRead(configPath)
} catch OSError as err {
    MsgBox("Cannot read config: " . err.Message)
    ; Create default config or handle gracefully
}
```

## Version History

- **v2.0**: Improved Unicode support and better encoding detection
- **v2.0-a**: Enhanced error handling and memory management
- **v2.1**: Performance optimizations for large files and better encoding options

## Related Functions

- [FileAppend](../fileappend.md) - Write/append content to files
- [FileOpen](../fileopen.md) - Open files for streaming read/write operations
- [FileExist](../fileexist.md) - Check file existence before reading
- [FileGetSize](../filegetsize.md) - Get file size for memory planning

## Related Concepts

- [File System Operations](../../../40_Advanced_Features/05-System_Integration/file-operations.md) - Complete file manipulation guide
- [Data Processing Patterns](../../../50_Ecosystem/00-Design_Patterns/data-processing.md) - Patterns for processing file data
- [Error Handling](../../../00_Fundamentals/05-Error_Basics/error-handling.md) - Managing file operation errors

## See Also

- [File Processing Guide](../../../Index/learning-paths.md#file-processing) - Complete file manipulation techniques
- [Configuration Management](../../../50_Ecosystem/01-Best_Practices/config-management.md) - Best practices for config files
- [Data Validation](../../../50_Ecosystem/01-Best_Practices/data-validation.md) - Validating file content after reading

## Tags

#AutoHotkey #Function #File #FileIO #Read #DataProcessing #Configuration #TextProcessing #BuiltIn #Essential #FileSystem