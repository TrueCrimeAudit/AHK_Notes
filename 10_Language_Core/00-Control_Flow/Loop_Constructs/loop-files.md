# Topic: Loop Files - File System Iteration and Processing

## Category

Loop Construct

## Overview

The Loop Files construct provides powerful file system iteration capabilities, enabling scripts to process files and directories with sophisticated filtering, recursive traversal, and pattern matching. It's essential for batch file operations, automated maintenance tasks, backup systems, and any application requiring systematic file processing.

## Key Points

- Iterates through files and directories with comprehensive filtering and pattern matching
- Supports recursive directory traversal for deep file system operations
- Provides built-in variables for file attributes, paths, and metadata during iteration
- Enables sophisticated file processing workflows with minimal code complexity
- Essential for automation tasks, backup systems, and file management applications

## Syntax and Parameters

```cpp
Loop Files, FilePattern [, Mode]

; Parameters:
; FilePattern - File path pattern with wildcards (* and ?)
; Mode        - Optional mode flags for iteration behavior

; Mode flags:
; D - Include directories in iteration
; F - Include files in iteration (default if no mode specified)
; R - Recursive traversal of subdirectories

; Mode combinations:
; "F"   - Files only (default)
; "D"   - Directories only  
; "FD"  - Both files and directories
; "R"   - Recursive files only
; "DR"  - Recursive directories only
; "FDR" - Recursive files and directories

; Built-in variables available during iteration:
; A_LoopFileName      - Current file/directory name (without path)
; A_LoopFileFullPath  - Complete path to current file/directory
; A_LoopFileShortPath - 8.3 format path (legacy compatibility)
; A_LoopFileDir       - Directory containing current file
; A_LoopFileExt       - File extension (without dot)
; A_LoopFileTimeModified   - Last modification timestamp
; A_LoopFileTimeCreated    - Creation timestamp  
; A_LoopFileTimeAccessed   - Last access timestamp
; A_LoopFileAttrib    - File attributes string
; A_LoopFileSize      - File size in bytes
; A_LoopFileSizeKB    - File size in kilobytes
; A_LoopFileSizeMB    - File size in megabytes
```

## Code Examples

```cpp
; Basic file iteration
Loop Files, "C:\MyDocuments\*.txt" {
    MsgBox("Found text file: " . A_LoopFileName)
}

; Recursive directory traversal
Loop Files, "C:\Projects\*.*", "R" {
    if (A_LoopFileExt = "ahk") {
        MsgBox("AutoHotkey script: " . A_LoopFileFullPath)
    }
}

; Include directories in iteration
Loop Files, "C:\MyFolder\*", "FD" {
    if (InStr(A_LoopFileAttrib, "D")) {
        MsgBox("Directory: " . A_LoopFileName)
    } else {
        MsgBox("File: " . A_LoopFileName)
    }
}

; Comprehensive file processor
class FileProcessor {
    static statistics := Map()
    static processedCount := 0
    static errors := []
    
    static ProcessDirectory(rootPath, pattern := "*.*", recursive := true) {
        ; Reset statistics
        this.statistics.Clear()
        this.processedCount := 0
        this.errors := []
        
        ; Set up loop mode
        mode := recursive ? "FR" : "F"
        searchPattern := rootPath . "\" . pattern
        
        ; Process files
        try {
            Loop Files, searchPattern, mode {
                this.ProcessFile()
            }
        } catch Error as err {
            this.errors.Push({
                type: "loop_error",
                message: err.Message,
                path: searchPattern
            })
        }
        
        return this.GetResults()
    }
    
    static ProcessFile() {
        try {
            ; Gather file information
            fileInfo := {
                name: A_LoopFileName,
                fullPath: A_LoopFileFullPath,
                extension: A_LoopFileExt,
                size: A_LoopFileSize,
                modified: A_LoopFileTimeModified,
                attributes: A_LoopFileAttrib
            }
            
            ; Update statistics
            this.UpdateStatistics(fileInfo)
            
            ; Perform processing based on file type
            this.ProcessByExtension(fileInfo)
            
            this.processedCount++
            
        } catch Error as err {
            this.errors.Push({
                type: "file_error",
                message: err.Message,
                path: A_LoopFileFullPath
            })
        }
    }
    
    static UpdateStatistics(fileInfo) {
        ext := StrLower(fileInfo.extension)
        if (!ext) ext := "no_extension"
        
        if (!this.statistics.Has(ext)) {
            this.statistics[ext] := {
                count: 0,
                totalSize: 0,
                files: []
            }
        }
        
        stats := this.statistics[ext]
        stats.count++
        stats.totalSize += fileInfo.size
        stats.files.Push(fileInfo)
    }
    
    static ProcessByExtension(fileInfo) {
        switch StrLower(fileInfo.extension) {
            case "txt", "log":
                this.ProcessTextFile(fileInfo)
            case "jpg", "png", "gif":
                this.ProcessImageFile(fileInfo)
            case "ahk":
                this.ProcessAutoHotkeyFile(fileInfo)
            case "pdf":
                this.ProcessPdfFile(fileInfo)
            default:
                this.ProcessGenericFile(fileInfo)
        }
    }
    
    static ProcessTextFile(fileInfo) {
        ; Example: Count lines in text files
        try {
            content := FileRead(fileInfo.fullPath)
            lineCount := StrSplit(content, "`n").Length
            fileInfo.lineCount := lineCount
        } catch {
            fileInfo.lineCount := "error"
        }
    }
    
    static ProcessImageFile(fileInfo) {
        ; Example: Analyze image files
        fileInfo.category := "image"
        ; Could add image dimension detection, etc.
    }
    
    static ProcessAutoHotkeyFile(fileInfo) {
        ; Example: Analyze AutoHotkey scripts
        try {
            content := FileRead(fileInfo.fullPath)
            fileInfo.hasClasses := InStr(content, "class ") > 0
            fileInfo.hasFunctions := InStr(content, "() {") > 0
            fileInfo.hasHotkeys := RegExMatch(content, "^[^;]*::", ) > 0
        } catch {
            fileInfo.analysis := "error"
        }
    }
    
    static ProcessPdfFile(fileInfo) {
        ; Example: PDF file handling
        fileInfo.category := "document"
    }
    
    static ProcessGenericFile(fileInfo) {
        ; Generic file processing
        fileInfo.category := "unknown"
    }
    
    static GetResults() {
        return {
            processedCount: this.processedCount,
            statistics: this.statistics,
            errors: this.errors,
            summary: this.GenerateSummary()
        }
    }
    
    static GenerateSummary() {
        totalFiles := 0
        totalSize := 0
        extensionCount := 0
        
        for ext, stats in this.statistics {
            totalFiles += stats.count
            totalSize += stats.totalSize
            extensionCount++
        }
        
        return {
            totalFiles: totalFiles,
            totalSize: totalSize,
            extensionTypes: extensionCount,
            averageSize: totalFiles ? Round(totalSize / totalFiles) : 0,
            errorCount: this.errors.Length
        }
    }
}

; Backup system using Loop Files
class BackupManager {
    static backupRoot := "C:\Backups"
    static excludePatterns := ["*.tmp", "*.log", "*.cache"]
    static maxBackupAge := 30 ; days
    
    static CreateBackup(sourceDir, backupName := "") {
        if (!backupName) {
            backupName := "Backup_" . FormatTime(, "yyyyMMdd_HHmmss")
        }
        
        backupPath := this.backupRoot . "\" . backupName
        
        ; Create backup directory
        try {
            DirCreate(backupPath)
        } catch Error as err {
            throw Error("Failed to create backup directory: " . err.Message)
        }
        
        ; Copy files recursively
        filesCopied := 0
        errors := []
        
        Loop Files, sourceDir . "\*.*", "FR" {
            try {
                ; Skip excluded patterns
                if (this.ShouldExclude(A_LoopFileName)) {
                    continue
                }
                
                ; Calculate relative path
                relativePath := StrReplace(A_LoopFileDir, sourceDir, "")
                if (relativePath && SubStr(relativePath, 1, 1) = "\") {
                    relativePath := SubStr(relativePath, 2)
                }
                
                ; Create destination directory if needed
                destDir := backupPath . "\" . relativePath
                if (relativePath && !DirExist(destDir)) {
                    DirCreate(destDir)
                }
                
                ; Copy file
                destFile := backupPath . "\" . relativePath . "\" . A_LoopFileName
                FileCopy(A_LoopFileFullPath, destFile, true)
                filesCopied++
                
            } catch Error as err {
                errors.Push({
                    file: A_LoopFileFullPath,
                    error: err.Message
                })
            }
        }
        
        ; Create backup manifest
        manifest := {
            backupName: backupName,
            sourceDir: sourceDir,
            backupPath: backupPath,
            timestamp: A_Now,
            filesCopied: filesCopied,
            errors: errors
        }
        
        manifestFile := backupPath . "\backup_manifest.json"
        FileWrite(JSON.stringify(manifest, , 2), manifestFile)
        
        return manifest
    }
    
    static ShouldExclude(fileName) {
        for pattern in this.excludePatterns {
            ; Simple wildcard matching
            regexPattern := StrReplace(pattern, "*", ".*")
            regexPattern := StrReplace(regexPattern, "?", ".")
            regexPattern := "^" . regexPattern . "$"
            
            if (RegExMatch(fileName, "i)" . regexPattern)) {
                return true
            }
        }
        return false
    }
    
    static CleanOldBackups() {
        cutoffTime := DateAdd(A_Now, -this.maxBackupAge, "Days")
        deletedBackups := []
        
        Loop Files, this.backupRoot . "\*", "D" {
            ; Check if this is a backup directory
            manifestFile := A_LoopFileFullPath . "\backup_manifest.json"
            
            if (FileExist(manifestFile)) {
                ; Read manifest to get backup timestamp
                try {
                    manifestContent := FileRead(manifestFile)
                    manifest := JSON.parse(manifestContent)
                    
                    if (manifest.timestamp < cutoffTime) {
                        ; Delete old backup
                        DirDelete(A_LoopFileFullPath, true)
                        deletedBackups.Push(A_LoopFileName)
                    }
                } catch {
                    ; Skip backups with invalid manifests
                    continue
                }
            }
        }
        
        return deletedBackups
    }
}

; File organization system
class FileOrganizer {
    static rules := Map()
    
    static AddRule(name, condition, action) {
        this.rules[name] := {
            condition: condition,
            action: action
        }
    }
    
    static OrganizeDirectory(dirPath) {
        results := {
            processed: 0,
            moved: 0,
            errors: []
        }
        
        Loop Files, dirPath . "\*.*", "F" {
            try {
                results.processed++
                
                ; Apply all rules to current file
                for ruleName, rule in this.rules {
                    if (rule.condition.Call(A_LoopFileFullPath, A_LoopFileName, A_LoopFileExt)) {
                        rule.action.Call(A_LoopFileFullPath, A_LoopFileName, A_LoopFileExt)
                        results.moved++
                        break ; Only apply first matching rule
                    }
                }
                
            } catch Error as err {
                results.errors.Push({
                    file: A_LoopFileFullPath,
                    error: err.Message
                })
            }
        }
        
        return results
    }
    
    static SetupDefaultRules(targetDir) {
        ; Rule: Move images to Images folder
        this.AddRule("images", 
            (path, name, ext) => ["jpg", "jpeg", "png", "gif", "bmp"].IndexOf(StrLower(ext)) > 0,
            (path, name, ext) => {
                destDir := targetDir . "\Images"
                if (!DirExist(destDir)) DirCreate(destDir)
                FileMove(path, destDir . "\" . name)
            })
        
        ; Rule: Move documents to Documents folder
        this.AddRule("documents",
            (path, name, ext) => ["pdf", "doc", "docx", "txt", "rtf"].IndexOf(StrLower(ext)) > 0,
            (path, name, ext) => {
                destDir := targetDir . "\Documents"
                if (!DirExist(destDir)) DirCreate(destDir)
                FileMove(path, destDir . "\" . name)
            })
        
        ; Rule: Move large files to Archive folder
        this.AddRule("large_files",
            (path, name, ext) => FileGetSize(path) > 100 * 1024 * 1024, ; > 100MB
            (path, name, ext) => {
                destDir := targetDir . "\Archive"
                if (!DirExist(destDir)) DirCreate(destDir)
                FileMove(path, destDir . "\" . name)
            })
    }
}

; Usage examples
; Process directory and get statistics
result := FileProcessor.ProcessDirectory("C:\MyProject", "*.ahk", true)
MsgBox("Processed " . result.processedCount . " files with " . result.errors.Length . " errors")

; Create backup
BackupManager.CreateBackup("C:\ImportantData", "Weekly_Backup")

; Clean old backups
deleted := BackupManager.CleanOldBackups()
MsgBox("Deleted " . deleted.Length . " old backups")

; Organize downloads folder
FileOrganizer.SetupDefaultRules("C:\Downloads")
organizeResult := FileOrganizer.OrganizeDirectory("C:\Downloads")
MsgBox("Organized " . organizeResult.moved . " out of " . organizeResult.processed . " files")

; Simple file search example
searchResults := []
Loop Files, "C:\*.*", "R" {
    if (InStr(A_LoopFileName, "config")) {
        searchResults.Push(A_LoopFileFullPath)
    }
}
MsgBox("Found " . searchResults.Length . " files containing 'config'")
```

## Implementation Notes

**Pattern Matching Behavior:**
- Wildcards (* and ?) only apply to filename, not directory path
- Case-insensitive matching on Windows file systems
- Empty pattern ("*") matches all files in directory
- Use specific patterns for performance optimization

**Recursive Traversal Considerations:**
- Recursive mode can be slow on large directory trees
- Monitor for infinite loops with symbolic links or junctions
- Consider implementing depth limits for very deep hierarchies
- Use early exit conditions to improve performance

**Built-in Variable Scope:**
- Loop variables are only valid within the loop body
- Store values in local variables if needed after loop completion
- Variables are reset for each iteration
- Nested loops require careful variable management

**Performance Optimization:**
- Filter early using specific patterns rather than checking all files
- Use file attributes to skip unnecessary operations
- Implement progress reporting for long-running operations
- Consider chunking large operations for responsiveness

**Error Handling Strategies:**
- Individual file errors don't stop loop execution
- Wrap file operations in try-catch blocks
- Log errors with sufficient context for debugging
- Implement retry logic for transient failures

**Memory Management:**
- Large directory iterations can consume significant memory
- Process files in batches for memory-intensive operations
- Clear large data structures after processing
- Monitor memory usage for recursive operations

## Related AHK Concepts

- [FileExist](../../30_Built_In_Classes/02-File_IO_Classes/File/fileexist.md) - File existence checking before processing
- [DirExist](../../30_Built_In_Classes/02-File_IO_Classes/Dir/direxist.md) - Directory validation
- [FileRead](../../30_Built_In_Classes/02-File_IO_Classes/File/fileread.md) - Reading files found during iteration
- [FileMove](../../30_Built_In_Classes/02-File_IO_Classes/File/filemove.md) - Moving files during processing
- [Loop Parse](./loop-parse.md) - Text parsing within file processing
- [RegExMatch](../../10_Language_Core/01-Functions/Built_In_Functions/regexmatch.md) - Pattern matching for file content

## Tags

#AutoHotkey #LoopFiles #FileSystem #Iteration #Automation #FileProcessing #DirectoryTraversal #BatchOperations