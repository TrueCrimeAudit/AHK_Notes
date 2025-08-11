# Topic: FileExist Function - File System Validation and Existence Checking

## Category

Function

## Overview

The FileExist function checks for the existence of files and directories while providing detailed attribute information. It's essential for file system validation, conditional file operations, path verification, and building robust scripts that handle file system states gracefully before attempting operations.

## Key Points

- Checks existence of files, directories, and special file system objects with comprehensive attribute detection
- Returns attribute string indicating file type, permissions, and special characteristics
- Provides foundation for safe file operations by validating paths before processing
- Supports wildcard patterns for flexible file system queries and batch operations
- Essential for error prevention, conditional logic, and robust file system interaction

## Syntax and Parameters

```cpp
Attributes := FileExist(FilePattern)

; Parameters:
; FilePattern - File path, directory path, or wildcard pattern to check

; Return Values:
; "" (empty string) - File/directory does not exist
; Attribute string  - File exists, string contains attribute characters:
;   D - Directory
;   A - Archive (file has been modified since last backup)
;   H - Hidden
;   R - Read-only
;   S - System file
;   L - Symbolic link or junction point
;   C - Compressed
;   O - Offline (file is not immediately available)
;   T - Temporary file
;   N - Normal (no special attributes)

; Wildcard support:
; * - Matches any number of characters
; ? - Matches exactly one character
; Only applies to filename, not directory path
```

## Code Examples

```cpp
; Basic file existence check
if (FileExist("document.txt")) {
    MsgBox("File exists")
} else {
    MsgBox("File not found")
}

; Directory existence check
if (FileExist("C:\MyFolder")) {
    MsgBox("Directory exists")
}

; Check specific file attributes
attributes := FileExist("system32.dll")
if (attributes) {
    if (InStr(attributes, "S")) {
        MsgBox("This is a system file")
    }
    if (InStr(attributes, "R")) {
        MsgBox("File is read-only")
    }
    if (InStr(attributes, "H")) {
        MsgBox("File is hidden")
    }
}

; Comprehensive file system checker
class FileSystemChecker {
    static CheckPath(path) {
        attributes := FileExist(path)
        
        if (!attributes) {
            return {
                exists: false,
                type: "none",
                attributes: "",
                readable: false,
                writable: false,
                executable: false
            }
        }
        
        ; Determine file type
        type := InStr(attributes, "D") ? "directory" : "file"
        
        ; Parse attributes
        result := {
            exists: true,
            type: type,
            attributes: attributes,
            isDirectory: InStr(attributes, "D") > 0,
            isArchive: InStr(attributes, "A") > 0,
            isHidden: InStr(attributes, "H") > 0,
            isReadOnly: InStr(attributes, "R") > 0,
            isSystem: InStr(attributes, "S") > 0,
            isSymbolicLink: InStr(attributes, "L") > 0,
            isCompressed: InStr(attributes, "C") > 0,
            isOffline: InStr(attributes, "O") > 0,
            isTemporary: InStr(attributes, "T") > 0,
            isNormal: InStr(attributes, "N") > 0
        }
        
        ; Determine access permissions
        result.readable := this.TestReadAccess(path, type)
        result.writable := this.TestWriteAccess(path, type)
        result.executable := this.TestExecuteAccess(path, type)
        
        return result
    }
    
    static TestReadAccess(path, type) {
        try {
            if (type = "directory") {
                ; Try to list directory contents
                DirExist(path)
                return true
            } else {
                ; Try to open file for reading
                file := FileOpen(path, "r")
                if (file) {
                    file.Close()
                    return true
                }
            }
        } catch {
            return false
        }
        return false
    }
    
    static TestWriteAccess(path, type) {
        try {
            if (type = "directory") {
                ; Try to create a temporary file in directory
                testFile := path . "\~test_write_" . A_TickCount . ".tmp"
                FileWrite("test", testFile)
                FileDelete(testFile)
                return true
            } else {
                ; Try to open file for append
                file := FileOpen(path, "a")
                if (file) {
                    file.Close()
                    return true
                }
            }
        } catch {
            return false
        }
        return false
    }
    
    static TestExecuteAccess(path, type) {
        if (type = "directory") {
            ; For directories, execute access means ability to traverse
            return this.TestReadAccess(path, type)
        }
        
        ; For files, check if it's an executable type
        SplitPath(path, , , &extension)
        executableExtensions := ["exe", "com", "bat", "cmd", "scr", "msi", "ps1"]
        
        return executableExtensions.IndexOf(StrLower(extension)) > 0
    }
}

; Safe file operations with validation
class SafeFileOperations {
    static SafeRead(filePath) {
        ; Validate before reading
        info := FileSystemChecker.CheckPath(filePath)
        
        if (!info.exists) {
            throw FileNotFoundError("File does not exist: " . filePath)
        }
        
        if (info.isDirectory) {
            throw ValueError("Path is a directory, not a file: " . filePath)
        }
        
        if (!info.readable) {
            throw OSError("File is not readable: " . filePath)
        }
        
        try {
            return FileRead(filePath)
        } catch Error as err {
            throw OSError("Failed to read file: " . err.Message)
        }
    }
    
    static SafeWrite(filePath, content, backup := true) {
        ; Check if file exists and is writable
        info := FileSystemChecker.CheckPath(filePath)
        
        if (info.exists) {
            if (info.isDirectory) {
                throw ValueError("Path is a directory, not a file: " . filePath)
            }
            
            if (!info.writable) {
                throw OSError("File is not writable: " . filePath)
            }
            
            ; Create backup if requested
            if (backup) {
                backupPath := filePath . ".backup." . FormatTime(, "yyyyMMdd_HHmmss")
                try {
                    FileCopy(filePath, backupPath)
                } catch Error as err {
                    throw OSError("Failed to create backup: " . err.Message)
                }
            }
        }
        
        ; Ensure directory exists
        SplitPath(filePath, , &dir)
        if (dir && !FileExist(dir)) {
            try {
                DirCreate(dir)
            } catch Error as err {
                throw OSError("Failed to create directory: " . err.Message)
            }
        }
        
        try {
            FileWrite(content, filePath)
        } catch Error as err {
            throw OSError("Failed to write file: " . err.Message)
        }
    }
    
    static SafeDelete(filePath, confirm := true) {
        info := FileSystemChecker.CheckPath(filePath)
        
        if (!info.exists) {
            return false ; Already doesn't exist
        }
        
        if (!info.writable) {
            throw OSError("File cannot be deleted (not writable): " . filePath)
        }
        
        if (confirm) {
            response := MsgBox("Delete " . (info.isDirectory ? "directory" : "file") . ":`n" . filePath, "Confirm Delete", "YesNo")
            if (response != "Yes") {
                return false
            }
        }
        
        try {
            if (info.isDirectory) {
                DirDelete(filePath, true) ; Recursive delete
            } else {
                FileDelete(filePath)
            }
            return true
        } catch Error as err {
            throw OSError("Failed to delete: " . err.Message)
        }
    }
}

; Batch file processor with validation
class BatchFileProcessor {
    static ProcessFiles(pattern, operation, options := {}) {
        ; Default options
        defaults := {
            recursive: false,
            includeHidden: false,
            includeSystem: false,
            continueOnError: true,
            logErrors: true
        }
        
        ; Merge options with defaults
        for key, value in defaults {
            if (!options.HasProp(key)) {
                options.%key% := value
            }
        }
        
        files := this.FindFiles(pattern, options)
        results := []
        errors := []
        
        for filePath in files {
            try {
                result := operation.Call(filePath)
                results.Push({path: filePath, success: true, result: result})
            } catch Error as err {
                error := {path: filePath, success: false, error: err.Message}
                results.Push(error)
                errors.Push(error)
                
                if (options.logErrors) {
                    OutputDebug("Error processing " . filePath . ": " . err.Message)
                }
                
                if (!options.continueOnError) {
                    break
                }
            }
        }
        
        return {
            results: results,
            errors: errors,
            totalFiles: files.Length,
            successCount: results.Length - errors.Length,
            errorCount: errors.Length
        }
    }
    
    static FindFiles(pattern, options) {
        files := []
        
        ; Split pattern into directory and filename parts
        SplitPath(pattern, &fileName, &dir)
        
        if (!dir) {
            dir := A_WorkingDir
        }
        
        ; Check if base directory exists
        if (!FileExist(dir)) {
            return files
        }
        
        ; Find matching files in directory
        this.ScanDirectory(dir, fileName, files, options, 0)
        
        return files
    }
    
    static ScanDirectory(dir, pattern, files, options, depth) {
        ; Get all entries in directory
        loop Files, dir . "\*", (options.recursive ? "R" : "") . "DF" {
            filePath := A_LoopFileFullPath
            fileName := A_LoopFileName
            attributes := FileExist(filePath)
            
            ; Skip based on attribute filters
            if (!options.includeHidden && InStr(attributes, "H")) {
                continue
            }
            if (!options.includeSystem && InStr(attributes, "S")) {
                continue
            }
            
            ; Check if filename matches pattern
            if (this.MatchesPattern(fileName, pattern)) {
                files.Push(filePath)
            }
        }
    }
    
    static MatchesPattern(fileName, pattern) {
        ; Simple wildcard matching
        ; Convert pattern to regex
        regexPattern := StrReplace(pattern, "*", ".*")
        regexPattern := StrReplace(regexPattern, "?", ".")
        regexPattern := "^" . regexPattern . "$"
        
        return RegExMatch(fileName, "i)" . regexPattern) > 0
    }
}

; File system monitoring with change detection
class FileSystemMonitor {
    static watchedPaths := Map()
    static monitorTimer := 0
    static lastStates := Map()
    
    static AddPath(path, callback, options := {}) {
        ; Validate path exists
        if (!FileExist(path)) {
            throw FileNotFoundError("Path does not exist: " . path)
        }
        
        ; Default monitoring options
        defaults := {
            checkInterval: 1000,  ; 1 second
            trackModification: true,
            trackSize: true,
            trackAttributes: false,
            recursive: false
        }
        
        ; Merge options
        for key, value in defaults {
            if (!options.HasProp(key)) {
                options.%key% := value
            }
        }
        
        this.watchedPaths[path] := {
            callback: callback,
            options: options
        }
        
        ; Store initial state
        this.lastStates[path] := this.GetPathState(path, options)
        
        ; Start monitoring if not already running
        if (!this.monitorTimer) {
            this.StartMonitoring()
        }
    }
    
    static RemovePath(path) {
        this.watchedPaths.Delete(path)
        this.lastStates.Delete(path)
        
        ; Stop monitoring if no paths to watch
        if (this.watchedPaths.Count = 0 && this.monitorTimer) {
            SetTimer(this.monitorTimer, 0)
            this.monitorTimer := 0
        }
    }
    
    static StartMonitoring() {
        if (this.monitorTimer) return
        
        ; Use shortest interval from all watched paths
        minInterval := 1000
        for path, data in this.watchedPaths {
            if (data.options.checkInterval < minInterval) {
                minInterval := data.options.checkInterval
            }
        }
        
        this.monitorTimer := SetTimer(this.CheckChanges.Bind(this), minInterval)
    }
    
    static CheckChanges() {
        for path, data in this.watchedPaths {
            try {
                currentState := this.GetPathState(path, data.options)
                lastState := this.lastStates.Get(path, {})
                
                changes := this.DetectChanges(lastState, currentState, data.options)
                
                if (changes.Length > 0) {
                    ; Update stored state
                    this.lastStates[path] := currentState
                    
                    ; Notify callback
                    try {
                        data.callback.Call({
                            path: path,
                            changes: changes,
                            currentState: currentState,
                            previousState: lastState
                        })
                    } catch Error as err {
                        OutputDebug("Monitor callback error for " . path . ": " . err.Message)
                    }
                }
                
            } catch Error as err {
                OutputDebug("Monitor error for " . path . ": " . err.Message)
            }
        }
    }
    
    static GetPathState(path, options) {
        attributes := FileExist(path)
        if (!attributes) {
            return {exists: false}
        }
        
        state := {
            exists: true,
            attributes: attributes,
            isDirectory: InStr(attributes, "D") > 0
        }
        
        if (options.trackModification || options.trackSize) {
            try {
                if (state.isDirectory) {
                    ; For directories, track file count
                    count := 0
                    loop Files, path . "\*", options.recursive ? "RF" : "F" {
                        count++
                    }
                    state.fileCount := count
                } else {
                    ; For files, get modification time and size
                    state.modTime := FileGetTime(path, "M")
                    state.size := FileGetSize(path)
                }
            } catch {
                ; Ignore errors getting detailed info
            }
        }
        
        return state
    }
    
    static DetectChanges(oldState, newState, options) {
        changes := []
        
        ; Check existence change
        if (oldState.exists != newState.exists) {
            changes.Push(newState.exists ? "created" : "deleted")
        }
        
        if (!oldState.exists || !newState.exists) {
            return changes ; No point checking other changes
        }
        
        ; Check modification
        if (options.trackModification && oldState.modTime != newState.modTime) {
            changes.Push("modified")
        }
        
        ; Check size
        if (options.trackSize && oldState.size != newState.size) {
            changes.Push("size_changed")
        }
        
        ; Check attributes
        if (options.trackAttributes && oldState.attributes != newState.attributes) {
            changes.Push("attributes_changed")
        }
        
        ; Check file count for directories
        if (newState.isDirectory && oldState.fileCount != newState.fileCount) {
            changes.Push("content_changed")
        }
        
        return changes
    }
}

; Example usage
; Check file system information
info := FileSystemChecker.CheckPath("C:\Windows\System32")
if (info.exists) {
    MsgBox("System32 exists and is " . (info.isDirectory ? "a directory" : "a file"))
}

; Safe file operations
try {
    content := SafeFileOperations.SafeRead("config.txt")
    SafeFileOperations.SafeWrite("backup_config.txt", content)
} catch Error as err {
    MsgBox("File operation failed: " . err.Message)
}

; Batch process all text files
result := BatchFileProcessor.ProcessFiles("*.txt", (filePath) => {
    content := FileRead(filePath)
    return StrLen(content) ; Return character count
})

MsgBox("Processed " . result.successCount . " files with " . result.errorCount . " errors")

; Monitor a directory for changes
FileSystemMonitor.AddPath("C:\MyProject", (event) => {
    MsgBox("Changes detected in " . event.path . ": " . StrReplace(event.changes.Join(", "), "_", " "))
})
```

## Implementation Notes

**Attribute String Interpretation:**
- Empty string indicates non-existence; any non-empty string indicates existence
- Multiple attributes can be present simultaneously (e.g., "AH" = Archive + Hidden)
- Attribute order is not guaranteed; always use InStr() to check specific attributes
- Some attributes may not be supported on all file systems (NTFS vs FAT32)

**Wildcard Pattern Matching:**
- Wildcards only apply to the filename portion, not directory paths
- Use Loop Files for complex directory traversal with patterns
- Pattern matching is case-insensitive on Windows file systems
- Consider RegEx for more sophisticated pattern matching needs

**Performance Considerations:**
- FileExist is relatively fast but avoid excessive calls in tight loops
- Cache results when checking the same path repeatedly within short timeframes
- Network paths may have significant latency; consider timeouts for network operations
- Very large directories may slow down wildcard pattern matching

**File System Limitations:**
- Some file systems don't support all attribute types
- Network shares may have different permission models
- Symbolic links and junctions require special handling
- Long path names (> 260 characters) may need special API calls

**Error Handling Strategies:**
- FileExist doesn't throw exceptions but returns empty string for non-existence
- Always validate file existence before performing operations
- Consider permission issues separate from existence issues
- Handle network connectivity problems for remote paths

## Related AHK Concepts

- [FileRead](./fileread.md) - Reading files after existence validation
- [FileWrite](./filewrite.md) - Writing files with path validation
- [DirExist](../Dir/direxist.md) - Directory-specific existence checking
- [Loop Files](../../../10_Language_Core/00-Control_Flow/Loop_Constructs/loop-files.md) - File system enumeration
- [FileGetSize](./filegetsize.md) - Additional file information retrieval
- [FileGetTime](./filegettime.md) - File timestamp information

## Tags

#AutoHotkey #FileExist #FileSystem #Validation #Attributes #SafeOperations #PathChecking #FileManagement #Existence