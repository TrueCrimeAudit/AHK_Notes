# Topic: FileCopy Function - Advanced File Duplication and Management

## Category

Function

## Overview

The FileCopy function copies files from source to destination locations with comprehensive options for overwrite handling, preservation of attributes, and error management. It's essential for backup operations, deployment scripts, file synchronization, data migration, and any application requiring reliable file duplication with advanced control over the copying process.

## Key Points

- Copies files with complete control over overwrite behavior and attribute preservation
- Supports both single file copying and pattern-based bulk operations with wildcard matching
- Provides comprehensive error handling for permissions, disk space, and network issues
- Enables sophisticated file management including backup strategies and synchronization operations
- Essential for deployment automation, backup systems, file organization, and data management workflows

## Syntax and Parameters

```cpp
FileCopy(SourcePattern, DestPattern [, Overwrite])

; Parameters:
; SourcePattern - Source file path or pattern (supports wildcards)
; DestPattern   - Destination file path or directory
; Overwrite     - Whether to overwrite existing files (1=yes, 0=no, default: 0)

; Source pattern formats:
; "C:\\file.txt"           - Single file path
; "C:\\folder\\*.txt"      - All .txt files in folder
; "C:\\folder\\*.*"        - All files in folder
; "C:\\**\\*.log"          - Recursive search (not standard, use custom implementation)

; Destination pattern formats:
; "D:\\newfile.txt"        - Specific destination file
; "D:\\folder\\"           - Copy to directory (preserve filename)
; "D:\\folder\\newname.*"  - Copy to directory with name pattern

; Overwrite behavior:
; 0 (default) - Do not overwrite existing files (throws error if file exists)
; 1           - Overwrite existing files without confirmation

; Return value:
; No return value on success
; Throws exception on error (file not found, permission denied, disk full, etc.)
```

## Code Examples

```cpp
; Basic file copying
FileCopy("C:\\source.txt", "C:\\destination.txt")

; Copy with overwrite
FileCopy("C:\\source.txt", "C:\\destination.txt", 1)

; Copy to directory
FileCopy("C:\\file.txt", "D:\\backup\\")  ; Preserves original filename

; Copy multiple files with wildcard
FileCopy("C:\\logs\\*.log", "D:\\backup\\logs\\")

; Advanced file copying and management system
class FileCopyManager {
    static operations := []
    static defaultChunkSize := 1048576  ; 1MB chunks for large files
    static preserveTimestamps := true
    static preserveAttributes := true
    static verifyAfterCopy := false
    static compressionEnabled := false
    static retryAttempts := 3
    static retryDelay := 1000
    
    static CopyFile(source, destination, options := {}) {
        ; Parse options
        overwrite := options.HasProp("overwrite") ? options.overwrite : false
        preserveAttribs := options.HasProp("preserveAttributes") ? options.preserveAttributes : this.preserveAttributes
        verify := options.HasProp("verify") ? options.verify : this.verifyAfterCopy
        retries := options.HasProp("retries") ? options.retries : this.retryAttempts
        createPath := options.HasProp("createPath") ? options.createPath : true
        
        ; Create operation record
        operation := {
            source: source,
            destination: destination,
            startTime: A_TickCount,
            endTime: 0,
            success: false,
            error: "",
            bytescopied: 0,
            sourceSize: 0,
            verified: false,
            attempts: 0
        }
        
        ; Validate source file
        if (!FileExist(source)) {
            operation.error := "Source file not found: " . source
            this.operations.Push(operation)
            throw FileNotFoundError(operation.error)
        }
        
        ; Get source file info
        operation.sourceSize := FileGetSize(source)
        
        ; Create destination directory if needed
        if (createPath) {
            destDir := this.GetDirectoryPath(destination)
            if (destDir && !DirExist(destDir)) {
                DirCreate(destDir)
            }
        }
        
        ; Attempt copy with retries
        for attempt in Range(1, retries + 1) {
            operation.attempts := attempt
            
            try {
                ; Check if destination exists
                if (!overwrite && FileExist(destination)) {
                    throw Error("Destination file already exists: " . destination)
                }
                
                ; Perform the copy
                if (operation.sourceSize > this.defaultChunkSize * 10) {
                    ; Large file - use chunked copy
                    this.CopyFileChunked(source, destination, operation)
                } else {
                    ; Small file - direct copy
                    FileCopy(source, destination, overwrite)
                    operation.bytescopied := operation.sourceSize
                }
                
                ; Preserve attributes if requested
                if (preserveAttribs) {
                    this.PreserveFileAttributes(source, destination)
                }
                
                ; Verify copy if requested
                if (verify) {
                    operation.verified := this.VerifyFileCopy(source, destination)
                    if (!operation.verified) {
                        throw Error("File verification failed after copy")
                    }
                }
                
                ; Success
                operation.success := true
                operation.endTime := A_TickCount
                break
                
            } catch Error as err {
                operation.error := err.Message
                
                if (attempt < retries) {
                    Sleep(this.retryDelay)
                    continue
                } else {
                    operation.endTime := A_TickCount
                    this.operations.Push(operation)
                    throw err
                }
            }
        }
        
        this.operations.Push(operation)
        return operation
    }
    
    static CopyFileChunked(source, destination, operation) {
        ; Copy large files in chunks to show progress and handle interruptions
        sourceFile := FileOpen(source, "r")
        if (!sourceFile) {
            throw Error("Cannot open source file: " . source)
        }
        
        destFile := FileOpen(destination, "w")
        if (!destFile) {
            sourceFile.Close()
            throw Error("Cannot create destination file: " . destination)
        }
        
        try {
            bytesRemaining := operation.sourceSize
            
            while (bytesRemaining > 0) {
                chunkSize := Min(this.defaultChunkSize, bytesRemaining)
                
                ; Read chunk
                data := sourceFile.RawRead(chunkSize)
                if (!data) {
                    throw Error("Failed to read from source file")
                }
                
                ; Write chunk
                bytesWritten := destFile.RawWrite(data)
                if (bytesWritten != chunkSize) {
                    throw Error("Failed to write complete chunk to destination")
                }
                
                operation.bytescopied += bytesWritten
                bytesRemaining -= bytesWritten
                
                ; Allow for progress callbacks or interruption
                if (Mod(operation.bytescopied, this.defaultChunkSize * 5) = 0) {
                    ; Progress checkpoint every 5MB
                    this.NotifyProgress(operation)
                }
            }
            
        } finally {
            sourceFile.Close()
            destFile.Close()
        }
    }
    
    static CopyMultipleFiles(sourcePattern, destinationDir, options := {}) {
        ; Copy multiple files matching a pattern
        recursive := options.HasProp("recursive") ? options.recursive : false
        overwrite := options.HasProp("overwrite") ? options.overwrite : false
        preserveStructure := options.HasProp("preserveStructure") ? options.preserveStructure : true
        
        results := {
            totalFiles: 0,
            copiedFiles: 0,
            failedFiles: 0,
            totalBytes: 0,
            copiedBytes: 0,
            operations: [],
            errors: []
        }
        
        ; Find matching files
        files := this.FindMatchingFiles(sourcePattern, recursive)
        results.totalFiles := files.Length
        
        for file in files {
            try {
                ; Calculate destination path
                if (preserveStructure) {
                    relativePath := this.GetRelativePath(file, this.GetDirectoryPath(sourcePattern))
                    destPath := destinationDir . "\\" . relativePath
                } else {
                    destPath := destinationDir . "\\" . this.GetFileName(file)
                }
                
                ; Copy file
                operation := this.CopyFile(file, destPath, {
                    overwrite: overwrite,
                    createPath: true
                })
                
                results.operations.Push(operation)
                results.copiedFiles++
                results.copiedBytes += operation.bytescopied
                results.totalBytes += operation.sourceSize
                
            } catch Error as err {
                results.failedFiles++
                results.errors.Push({
                    file: file,
                    error: err.Message
                })
            }
        }
        
        return results
    }
    
    static SynchronizeDirectories(sourceDir, destDir, options := {}) {
        ; Synchronize two directories
        mode := options.HasProp("mode") ? options.mode : "mirror"  ; mirror, update, merge
        deleteExtra := options.HasProp("deleteExtra") ? options.deleteExtra : (mode = "mirror")
        compareBy := options.HasProp("compareBy") ? options.compareBy : "datetime"  ; datetime, size, checksum
        
        results := {
            sourceFiles: Map(),
            destFiles: Map(),
            actions: [],
            copiedFiles: 0,
            updatedFiles: 0,
            deletedFiles: 0,
            errors: []
        }
        
        ; Scan source directory
        this.ScanDirectory(sourceDir, results.sourceFiles, true)
        
        ; Scan destination directory
        this.ScanDirectory(destDir, results.destFiles, true)
        
        ; Compare and determine actions
        for relativePath, sourceFile in results.sourceFiles {
            destPath := destDir . "\\" . relativePath
            
            if (results.destFiles.Has(relativePath)) {
                ; File exists in both locations
                destFile := results.destFiles[relativePath]
                
                if (this.ShouldUpdateFile(sourceFile, destFile, compareBy)) {
                    results.actions.Push({
                        action: "update",
                        source: sourceFile.fullPath,
                        dest: destPath,
                        reason: "newer or different"
                    })
                }
            } else {
                ; File only exists in source
                results.actions.Push({
                    action: "copy",
                    source: sourceFile.fullPath,
                    dest: destPath,
                    reason: "missing in destination"
                })
            }
        }
        
        ; Check for files only in destination
        if (deleteExtra) {
            for relativePath, destFile in results.destFiles {
                if (!results.sourceFiles.Has(relativePath)) {
                    results.actions.Push({
                        action: "delete",
                        dest: destFile.fullPath,
                        reason: "not in source"
                    })
                }
            }
        }
        
        ; Execute actions
        for action in results.actions {
            try {
                switch action.action {
                    case "copy", "update":
                        this.CopyFile(action.source, action.dest, {
                            overwrite: true,
                            createPath: true
                        })
                        
                        if (action.action = "copy") {
                            results.copiedFiles++
                        } else {
                            results.updatedFiles++
                        }
                        
                    case "delete":
                        FileDelete(action.dest)
                        results.deletedFiles++
                }
                
            } catch Error as err {
                results.errors.Push({
                    action: action,
                    error: err.Message
                })
            }
        }
        
        return results
    }
    
    static CreateBackup(sourcePath, backupDir, options := {}) {
        ; Create timestamped backup
        includeTimestamp := options.HasProp("includeTimestamp") ? options.includeTimestamp : true
        compression := options.HasProp("compression") ? options.compression : false
        retention := options.HasProp("retention") ? options.retention : 0  ; 0 = keep all
        
        ; Generate backup name
        timestamp := includeTimestamp ? "_" . FormatTime(, "yyyyMMdd_HHmmss") : ""
        
        if (DirExist(sourcePath)) {
            ; Directory backup
            sourceName := this.GetDirectoryName(sourcePath)
            backupPath := backupDir . "\\" . sourceName . timestamp
            
            result := this.CopyMultipleFiles(sourcePath . "\\*.*", backupPath, {
                recursive: true,
                preserveStructure: true,
                overwrite: false
            })
            
        } else if (FileExist(sourcePath)) {
            ; File backup
            fileName := this.GetFileNameWithoutExtension(sourcePath)
            fileExt := this.GetFileExtension(sourcePath)
            backupPath := backupDir . "\\" . fileName . timestamp . fileExt
            
            result := this.CopyFile(sourcePath, backupPath, {
                overwrite: false,
                verify: true
            })
        } else {
            throw FileNotFoundError("Source path not found: " . sourcePath)
        }
        
        ; Apply retention policy
        if (retention > 0) {
            this.ApplyRetentionPolicy(backupDir, retention)
        }
        
        return result
    }
    
    static VerifyFileCopy(source, destination) {
        ; Verify that files are identical
        if (!FileExist(destination)) {
            return false
        }
        
        ; Compare file sizes
        sourceSize := FileGetSize(source)
        destSize := FileGetSize(destination)
        
        if (sourceSize != destSize) {
            return false
        }
        
        ; For small files, compare checksums
        if (sourceSize < 1048576) {  ; 1MB threshold
            return this.CalculateChecksum(source) = this.CalculateChecksum(destination)
        }
        
        ; For large files, assume size match is sufficient
        return true
    }
    
    static CalculateChecksum(filePath) {
        ; Simple checksum calculation (CRC32 would be better)
        content := FileRead(filePath, "RAW")
        checksum := 0
        
        for i in Range(1, content.Size) {
            checksum := checksum ^ NumGet(content.Ptr + i - 1, "UChar")
        }
        
        return checksum
    }
    
    static PreserveFileAttributes(source, destination) {
        ; Copy file attributes and timestamps
        try {
            sourceAttribs := FileGetAttrib(source)
            FileSetAttrib(sourceAttribs, destination)
            
            sourceTime := FileGetTime(source)
            FileSetTime(sourceTime, destination)
            
        } catch Error {
            ; Non-critical error, continue
        }
    }
    
    static FindMatchingFiles(pattern, recursive := false) {
        files := []
        
        if (recursive) {
            ; Use Loop Files for recursive search
            Loop Files, pattern, "R" {
                files.Push(A_LoopFileFullPath)
            }
        } else {
            Loop Files, pattern {
                files.Push(A_LoopFileFullPath)
            }
        }
        
        return files
    }
    
    static ScanDirectory(dirPath, fileMap, recursive := true) {
        basePath := RTrim(dirPath, "\\")
        
        Loop Files, dirPath . "\\*.*", recursive ? "RF" : "F" {
            relativePath := SubStr(A_LoopFileFullPath, StrLen(basePath) + 2)
            
            fileMap[relativePath] := {
                fullPath: A_LoopFileFullPath,
                size: A_LoopFileSize,
                modified: A_LoopFileTimeModified,
                attributes: A_LoopFileAttrib
            }
        }
    }
    
    static ShouldUpdateFile(sourceFile, destFile, compareBy) {
        switch compareBy {
            case "datetime":
                return sourceFile.modified > destFile.modified
            case "size":
                return sourceFile.size != destFile.size
            case "checksum":
                return this.CalculateChecksum(sourceFile.fullPath) != this.CalculateChecksum(destFile.fullPath)
            default:
                return sourceFile.modified > destFile.modified
        }
    }
    
    static GetDirectoryPath(filePath) {
        return RTrim(filePath, "\\" . this.GetFileName(filePath))
    }
    
    static GetFileName(filePath) {
        return SubStr(filePath, InStr(filePath, "\\", , -1) + 1)
    }
    
    static GetFileNameWithoutExtension(filePath) {
        fileName := this.GetFileName(filePath)
        lastDot := InStr(fileName, ".", , -1)
        return lastDot > 0 ? SubStr(fileName, 1, lastDot - 1) : fileName
    }
    
    static GetFileExtension(filePath) {
        lastDot := InStr(filePath, ".", , -1)
        return lastDot > 0 ? SubStr(filePath, lastDot) : ""
    }
    
    static GetDirectoryName(dirPath) {
        trimmed := RTrim(dirPath, "\\")
        return SubStr(trimmed, InStr(trimmed, "\\", , -1) + 1)
    }
    
    static GetRelativePath(fullPath, basePath) {
        baseLen := StrLen(RTrim(basePath, "\\"))
        return SubStr(fullPath, baseLen + 2)
    }
    
    static ApplyRetentionPolicy(backupDir, keepCount) {
        ; Keep only the most recent N backups
        backups := []
        
        Loop Files, backupDir . "\\*.*", "D" {
            backups.Push({
                path: A_LoopFileFullPath,
                time: A_LoopFileTimeModified
            })
        }
        
        ; Sort by modification time (newest first)
        backups.Sort((a, b) => b.time - a.time)
        
        ; Delete old backups
        for i in Range(keepCount + 1, backups.Length) {
            try {
                DirDelete(backups[i].path, true)
            } catch Error {
                ; Continue with other deletions
            }
        }
    }
    
    static NotifyProgress(operation) {
        ; Override this method to provide progress feedback
        percentage := Round((operation.bytescopied / operation.sourceSize) * 100, 1)
        ; Could show in tooltip, write to log, etc.
    }
    
    static GetCopyStatistics() {
        totalOps := this.operations.Length
        successfulOps := 0
        totalBytes := 0
        totalTime := 0
        
        for operation in this.operations {
            if (operation.success) {
                successfulOps++
                totalBytes += operation.bytescopied
                totalTime += operation.endTime - operation.startTime
            }
        }
        
        return {
            totalOperations: totalOps,
            successfulOperations: successfulOps,
            failureRate: totalOps > 0 ? Round((totalOps - successfulOps) / totalOps * 100, 1) : 0,
            totalBytesCopied: totalBytes,
            averageSpeed: totalTime > 0 ? Round(totalBytes / (totalTime / 1000), 0) : 0,  ; bytes per second
            averageTime: successfulOps > 0 ? Round(totalTime / successfulOps, 0) : 0
        }
    }
    
    static ClearHistory() {
        this.operations := []
    }
}

; Deployment and distribution system
class DeploymentManager {
    static deploymentProfiles := Map()
    static currentDeployment := ""
    
    static CreateProfile(name, config) {
        this.deploymentProfiles[name] := {
            name: name,
            source: config.source,
            destinations: config.destinations,
            fileFilters: config.fileFilters,
            excludePatterns: config.excludePatterns,
            preserveStructure: config.preserveStructure,
            overwritePolicy: config.overwritePolicy,
            verificationEnabled: config.verificationEnabled,
            backupEnabled: config.backupEnabled,
            compressionEnabled: config.compressionEnabled
        }
    }
    
    static ExecuteDeployment(profileName) {
        if (!this.deploymentProfiles.Has(profileName)) {
            throw ValueError("Deployment profile not found: " . profileName)
        }
        
        profile := this.deploymentProfiles[profileName]
        this.currentDeployment := profileName
        
        results := {
            profile: profileName,
            startTime: A_TickCount,
            destinations: [],
            totalFiles: 0,
            totalBytes: 0,
            errors: [],
            success: false
        }
        
        try {
            ; Deploy to each destination
            for dest in profile.destinations {
                destResult := this.DeployToDestination(profile, dest)
                results.destinations.Push(destResult)
                results.totalFiles += destResult.filesCopied
                results.totalBytes += destResult.bytesCopied
                
                if (destResult.errors.Length > 0) {
                    results.errors.Push(...destResult.errors)
                }
            }
            
            results.success := results.errors.Length = 0
            results.endTime := A_TickCount
            
        } catch Error as err {
            results.errors.Push({
                stage: "deployment",
                error: err.Message
            })
            results.endTime := A_TickCount
        }
        
        return results
    }
    
    static DeployToDestination(profile, destination) {
        destResult := {
            destination: destination,
            filesCopied: 0,
            bytesCopied: 0,
            errors: []
        }
        
        ; Create backup if enabled
        if (profile.backupEnabled && DirExist(destination)) {
            try {
                FileCopyManager.CreateBackup(destination, destination . "_backup", {
                    includeTimestamp: true,
                    retention: 5
                })
            } catch Error as err {
                destResult.errors.Push({
                    stage: "backup",
                    error: err.Message
                })
            }
        }
        
        ; Copy files based on filters
        for filter in profile.fileFilters {
            sourcePattern := profile.source . "\\" . filter
            
            try {
                copyResult := FileCopyManager.CopyMultipleFiles(sourcePattern, destination, {
                    recursive: true,
                    preserveStructure: profile.preserveStructure,
                    overwrite: profile.overwritePolicy = "overwrite"
                })
                
                destResult.filesCopied += copyResult.copiedFiles
                destResult.bytesCopied += copyResult.copiedBytes
                
                if (copyResult.errors.Length > 0) {
                    destResult.errors.Push(...copyResult.errors)
                }
                
            } catch Error as err {
                destResult.errors.Push({
                    stage: "copy",
                    filter: filter,
                    error: err.Message
                })
            }
        }
        
        return destResult
    }
}

; Example usage and demonstrations
; Initialize file copy manager
FileCopyManager.preserveTimestamps := true
FileCopyManager.verifyAfterCopy := true

; Basic file copying
try {
    FileCopy("C:\\source.txt", "C:\\backup\\destination.txt", 1)
    MsgBox("File copied successfully!")
} catch Error as err {
    MsgBox("Copy failed: " . err.Message)
}

; Advanced file copying with options
operation := FileCopyManager.CopyFile("C:\\important.doc", "D:\\backup\\important.doc", {
    overwrite: true,
    verify: true,
    preserveAttributes: true,
    retries: 3
})

if (operation.success) {
    MsgBox("File copied and verified successfully!")
}

; Copy multiple files
result := FileCopyManager.CopyMultipleFiles("C:\\logs\\*.log", "D:\\backup\\logs\\", {
    recursive: true,
    overwrite: false,
    preserveStructure: true
})

MsgBox("Copied " . result.copiedFiles . " files (" . result.copiedBytes . " bytes)")

; Directory synchronization
syncResult := FileCopyManager.SynchronizeDirectories("C:\\project", "D:\\backup\\project", {
    mode: "mirror",
    compareBy: "datetime",
    deleteExtra: true
})

MsgBox("Sync complete: " . syncResult.copiedFiles . " copied, " . syncResult.updatedFiles . " updated, " . syncResult.deletedFiles . " deleted")

; Create timestamped backup
backupResult := FileCopyManager.CreateBackup("C:\\important_folder", "D:\\backups", {
    includeTimestamp: true,
    retention: 10
})

; Deployment system usage
DeploymentManager.CreateProfile("WebApp", {
    source: "C:\\dev\\webapp",
    destinations: ["D:\\test\\webapp", "E:\\production\\webapp"],
    fileFilters: ["*.html", "*.css", "*.js", "*.php"],
    excludePatterns: ["*.tmp", "*.log"],
    preserveStructure: true,
    overwritePolicy: "overwrite",
    verificationEnabled: true,
    backupEnabled: true
})

deployResult := DeploymentManager.ExecuteDeployment("WebApp")

if (deployResult.success) {
    MsgBox("Deployment completed successfully!")
} else {
    MsgBox("Deployment had " . deployResult.errors.Length . " errors")
}

; Bulk file operations with wildcard patterns
FileCopy("C:\\temp\\*.tmp", "C:\\archive\\temp\\", 0)  ; Copy all .tmp files
FileCopy("C:\\docs\\*.doc", "D:\\backup\\docs\\")     ; Copy Word documents

; Copy with error handling for permissions or disk space
try {
    FileCopy("C:\\system\\important.dll", "C:\\backup\\", 1)
} catch Error as err {
    if (InStr(err.Message, "Access denied")) {
        MsgBox("Permission denied - run as administrator")
    } else if (InStr(err.Message, "disk full")) {
        MsgBox("Insufficient disk space")
    } else {
        MsgBox("Copy error: " . err.Message)
    }
}

; Get copy statistics
stats := FileCopyManager.GetCopyStatistics()
MsgBox("Copy Statistics:`n" .
       "Operations: " . stats.totalOperations . "`n" .
       "Success Rate: " . (100 - stats.failureRate) . "%`n" .
       "Average Speed: " . stats.averageSpeed . " bytes/sec")
```

## Implementation Notes

**Performance and Large Files:**
- Large file copying can be memory-intensive and time-consuming
- Consider implementing progress feedback for user experience during long operations
- Network copies are significantly slower than local copies and more prone to interruption
- Chunked copying enables better progress tracking and recovery from interruptions

**Error Handling and Recovery:**
- Permission errors are common when copying system files or across network shares
- Disk space exhaustion can occur during large copy operations, implement space checking
- Network interruptions require retry mechanisms with exponential backoff
- Lock files and concurrent access can cause intermittent copy failures

**File System Considerations:**
- NTFS alternate data streams are not copied by standard file operations
- File compression attributes may not be preserved across different file systems
- Long path names (>260 characters) require special handling on Windows
- Case sensitivity differences between file systems can cause issues in cross-platform scenarios

**Security and Permissions:**
- Copied files inherit destination directory permissions, not source permissions
- Administrative privileges may be required for system file copying
- Network shares require appropriate authentication and access rights
- Antivirus software may scan copied files, causing temporary access delays

**Backup and Synchronization Strategies:**
- Implement verification mechanisms for critical file operations
- Consider atomic operations for mission-critical file copying scenarios
- Timestamp preservation is important for backup integrity and synchronization logic
- Retention policies prevent unlimited backup accumulation and disk space exhaustion

## Related AHK Concepts

- [FileMove](./filemove.md) - Moving files instead of copying
- [DirCreate](../../30_Built_In_Classes/02-File_IO_Classes/Dir/dircreate.md) - Creating destination directories
- [FileExist](./fileexist.md) - Checking file existence before copying
- [FileGetSize](./filegetsize.md) - Getting file sizes for copy operations
- [Loop Files](../../10_Language_Core/00-Control_Flow/Loop_Constructs/loop-files.md) - Iterating through files for bulk operations

## Tags

#AutoHotkey #FileCopy #FileOperations #BackupSystems #FileManagement #Deployment #Synchronization #BulkOperations #FileIO #DataManagement
