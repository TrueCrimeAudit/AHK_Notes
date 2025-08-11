# Topic: DirCreate Function - Directory Creation and Management

## Category

Function

## Overview

The DirCreate function creates directories (folders) including any necessary parent directories in the path. It's essential for file system organization, project structure setup, automated directory management, and ensuring proper file storage locations exist before performing file operations.

## Key Points

- Creates directories and all necessary parent directories in a single operation
- Handles complex nested directory structures with automatic path resolution
- Provides error handling for permission issues, invalid paths, and existing directories
- Supports both absolute and relative path specifications for flexible usage
- Essential for project setup, file organization, backup systems, and automated file management

## Syntax and Parameters

```cpp
DirCreate(DirName)

; Parameters:
; DirName - Full or relative path of directory to create

; Return Value:
; No return value (function succeeds silently or throws exception)

; Behavior:
; - Creates all parent directories if they don't exist
; - Succeeds silently if directory already exists
; - Throws OSError if creation fails (permissions, invalid path, etc.)
; - Supports long paths (> 260 characters) on compatible systems
```

## Code Examples

```cpp
; Basic directory creation
DirCreate("C:\MyProject")

; Create nested directory structure
DirCreate("C:\Projects\WebApp\src\components")

; Relative path creation
DirCreate("logs\archive\2024")

; Create directory with error handling
try {
    DirCreate("C:\TempData\Processing")
    MsgBox("Directory created successfully")
} catch OSError as err {
    MsgBox("Failed to create directory: " . err.Message)
}

; Project structure creator
class ProjectStructure {
    static basePath := ""
    static structure := Map()
    
    static Initialize(projectPath, projectType := "general") {
        this.basePath := projectPath
        this.structure.Clear()
        
        ; Define project structures for different types
        switch projectType {
            case "web":
                this.DefineWebStructure()
            case "software":
                this.DefineSoftwareStructure()
            case "data":
                this.DefineDataStructure()
            default:
                this.DefineGeneralStructure()
        }
    }
    
    static DefineWebStructure() {
        this.structure := Map(
            "src", Map(
                "components", Map(),
                "styles", Map(),
                "scripts", Map(),
                "assets", Map(
                    "images", Map(),
                    "fonts", Map(),
                    "icons", Map()
                )
            ),
            "dist", Map(),
            "tests", Map(
                "unit", Map(),
                "integration", Map(),
                "e2e", Map()
            ),
            "docs", Map(),
            "config", Map()
        )
    }
    
    static DefineSoftwareStructure() {
        this.structure := Map(
            "src", Map(
                "core", Map(),
                "modules", Map(),
                "interfaces", Map(),
                "utils", Map()
            ),
            "bin", Map(),
            "lib", Map(),
            "tests", Map(),
            "docs", Map(
                "api", Map(),
                "user", Map(),
                "dev", Map()
            ),
            "resources", Map(),
            "build", Map()
        )
    }
    
    static DefineDataStructure() {
        this.structure := Map(
            "raw", Map(),
            "processed", Map(),
            "analysis", Map(),
            "reports", Map(),
            "exports", Map(),
            "archive", Map(),
            "scripts", Map(
                "processing", Map(),
                "analysis", Map(),
                "utilities", Map()
            ),
            "docs", Map()
        )
    }
    
    static DefineGeneralStructure() {
        this.structure := Map(
            "documents", Map(),
            "resources", Map(),
            "output", Map(),
            "temp", Map(),
            "archive", Map()
        )
    }
    
    static CreateStructure() {
        if (!this.basePath) {
            throw ValueError("Project path not set. Call Initialize() first.")
        }
        
        ; Create base directory
        try {
            DirCreate(this.basePath)
        } catch OSError as err {
            throw OSError("Failed to create project base directory: " . err.Message)
        }
        
        ; Create nested structure
        this.CreateNestedDirs(this.structure, this.basePath)
        
        return this.basePath
    }
    
    static CreateNestedDirs(dirMap, currentPath) {
        for dirName, subDirs in dirMap {
            dirPath := currentPath . "\" . dirName
            
            try {
                DirCreate(dirPath)
                
                ; Recursively create subdirectories
                if (Type(subDirs) = "Map" && subDirs.Count > 0) {
                    this.CreateNestedDirs(subDirs, dirPath)
                }
                
            } catch OSError as err {
                throw OSError("Failed to create directory '" . dirPath . "': " . err.Message)
            }
        }
    }
    
    static AddCustomDirectory(path) {
        fullPath := this.basePath . "\" . path
        try {
            DirCreate(fullPath)
            return fullPath
        } catch OSError as err {
            throw OSError("Failed to create custom directory: " . err.Message)
        }
    }
    
    static GetProjectInfo() {
        if (!this.basePath || !DirExist(this.basePath)) {
            return {exists: false, path: this.basePath}
        }
        
        info := {
            exists: true,
            path: this.basePath,
            directories: [],
            totalSize: 0
        }
        
        ; Enumerate all directories
        this.EnumerateDirectories(this.basePath, info.directories)
        
        return info
    }
    
    static EnumerateDirectories(path, dirList) {
        loop Files, path . "\*", "D" {
            dirList.Push(A_LoopFileFullPath)
            this.EnumerateDirectories(A_LoopFileFullPath, dirList)
        }
    }
}

; Backup directory manager
class BackupManager {
    static backupRoot := ""
    static retention := Map()
    
    static Initialize(rootPath) {
        this.backupRoot := rootPath
        
        ; Set default retention policies
        this.retention := Map(
            "daily", 30,     ; Keep 30 daily backups
            "weekly", 12,    ; Keep 12 weekly backups
            "monthly", 24    ; Keep 24 monthly backups
        )
        
        ; Create backup structure
        this.CreateBackupStructure()
    }
    
    static CreateBackupStructure() {
        try {
            DirCreate(this.backupRoot)
            DirCreate(this.backupRoot . "\daily")
            DirCreate(this.backupRoot . "\weekly")
            DirCreate(this.backupRoot . "\monthly")
            DirCreate(this.backupRoot . "\archive")
            DirCreate(this.backupRoot . "\temp")
        } catch OSError as err {
            throw OSError("Failed to create backup structure: " . err.Message)
        }
    }
    
    static CreateBackupDirectory(type, timestamp := "") {
        if (!timestamp) {
            timestamp := FormatTime(, "yyyyMMdd_HHmmss")
        }
        
        validTypes := ["daily", "weekly", "monthly", "archive"]
        if (validTypes.IndexOf(type) = 0) {
            throw ValueError("Invalid backup type: " . type)
        }
        
        backupDir := this.backupRoot . "\" . type . "\" . timestamp
        
        try {
            DirCreate(backupDir)
            return backupDir
        } catch OSError as err {
            throw OSError("Failed to create backup directory: " . err.Message)
        }
    }
    
    static CleanupOldBackups() {
        for backupType, maxCount in this.retention {
            this.CleanupBackupType(backupType, maxCount)
        }
    }
    
    static CleanupBackupType(type, maxCount) {
        typePath := this.backupRoot . "\" . type
        if (!DirExist(typePath)) return
        
        ; Get all backup directories for this type
        backups := []
        loop Files, typePath . "\*", "D" {
            backups.Push({
                path: A_LoopFileFullPath,
                name: A_LoopFileName,
                time: A_LoopFileTimeModified
            })
        }
        
        ; Sort by modification time (newest first)
        backups.Sort((a, b) => b.time - a.time)
        
        ; Remove excess backups
        if (backups.Length > maxCount) {
            excessCount := backups.Length - maxCount
            for i in Range(maxCount + 1, backups.Length) {
                try {
                    DirDelete(backups[i].path, true)
                } catch OSError as err {
                    OutputDebug("Failed to delete old backup: " . err.Message)
                }
            }
        }
    }
}

; Log directory organizer
class LogDirectoryManager {
    static baseLogDir := "logs"
    static categories := Map()
    static rotationSchedule := Map()
    
    static Initialize(baseDir := "logs") {
        this.baseLogDir := baseDir
        this.SetupDefaultCategories()
        this.CreateLogStructure()
    }
    
    static SetupDefaultCategories() {
        this.categories := Map(
            "application", Map(
                "current", this.baseLogDir . "\application\current",
                "archive", this.baseLogDir . "\application\archive",
                "rotation", "daily"
            ),
            "error", Map(
                "current", this.baseLogDir . "\error\current",
                "archive", this.baseLogDir . "\error\archive",
                "rotation", "daily"
            ),
            "access", Map(
                "current", this.baseLogDir . "\access\current",
                "archive", this.baseLogDir . "\access\archive",
                "rotation", "weekly"
            ),
            "debug", Map(
                "current", this.baseLogDir . "\debug\current",
                "archive", this.baseLogDir . "\debug\archive",
                "rotation", "weekly"
            ),
            "performance", Map(
                "current", this.baseLogDir . "\performance\current",
                "archive", this.baseLogDir . "\performance\archive",
                "rotation", "monthly"
            )
        )
    }
    
    static CreateLogStructure() {
        try {
            ; Create base log directory
            DirCreate(this.baseLogDir)
            
            ; Create category directories
            for category, paths in this.categories {
                DirCreate(paths["current"])
                DirCreate(paths["archive"])
                
                ; Create rotation subdirectories
                rotation := paths["rotation"]
                archivePath := paths["archive"]
                
                DirCreate(archivePath . "\" . rotation)
                
                ; Create date-based subdirectories for archives
                currentYear := FormatTime(, "yyyy")
                DirCreate(archivePath . "\" . rotation . "\" . currentYear)
            }
            
        } catch OSError as err {
            throw OSError("Failed to create log structure: " . err.Message)
        }
    }
    
    static GetLogPath(category, archived := false) {
        if (!this.categories.Has(category)) {
            throw ValueError("Unknown log category: " . category)
        }
        
        return archived ? this.categories[category]["archive"] : this.categories[category]["current"]
    }
    
    static ArchiveLogs(category) {
        if (!this.categories.Has(category)) {
            throw ValueError("Unknown log category: " . category)
        }
        
        currentPath := this.categories[category]["current"]
        archivePath := this.categories[category]["archive"]
        rotation := this.categories[category]["rotation"]
        
        timestamp := FormatTime(, "yyyyMMdd_HHmmss")
        
        ; Create archive directory for this timestamp
        archiveDir := archivePath . "\" . rotation . "\" . FormatTime(, "yyyy") . "\" . timestamp
        
        try {
            DirCreate(archiveDir)
            
            ; Move all files from current to archive
            loop Files, currentPath . "\*.*" {
                FileMove(A_LoopFileFullPath, archiveDir . "\" . A_LoopFileName)
            }
            
            return archiveDir
            
        } catch OSError as err {
            throw OSError("Failed to archive logs: " . err.Message)
        }
    }
    
    static AddCategory(name, rotationType := "daily") {
        categoryPath := Map(
            "current", this.baseLogDir . "\" . name . "\current",
            "archive", this.baseLogDir . "\" . name . "\archive",
            "rotation", rotationType
        )
        
        this.categories[name] := categoryPath
        
        ; Create directories for new category
        try {
            DirCreate(categoryPath["current"])
            DirCreate(categoryPath["archive"])
            DirCreate(categoryPath["archive"] . "\" . rotationType)
        } catch OSError as err {
            throw OSError("Failed to create category directories: " . err.Message)
        }
    }
}

; Safe directory operations
class SafeDirectoryOps {
    static CreateWithPermissions(dirPath, checkWritable := true) {
        try {
            DirCreate(dirPath)
            
            if (checkWritable) {
                ; Test write permissions
                testFile := dirPath . "\~write_test_" . A_TickCount . ".tmp"
                FileWrite("test", testFile)
                FileDelete(testFile)
            }
            
            return true
            
        } catch OSError as err {
            throw OSError("Directory creation failed: " . err.Message)
        }
    }
    
    static EnsureDirectoryExists(dirPath) {
        if (DirExist(dirPath)) {
            return true
        }
        
        try {
            DirCreate(dirPath)
            return true
        } catch OSError {
            return false
        }
    }
    
    static CreateTempDirectory(prefix := "temp_") {
        tempBase := A_Temp
        timestamp := FormatTime(, "yyyyMMdd_HHmmss")
        random := Random(1000, 9999)
        
        tempDir := tempBase . "\" . prefix . timestamp . "_" . random
        
        try {
            DirCreate(tempDir)
            return tempDir
        } catch OSError as err {
            throw OSError("Failed to create temp directory: " . err.Message)
        }
    }
    
    static ValidatePath(dirPath) {
        ; Check for invalid characters
        invalidChars := ['<', '>', '"', '|', '?', '*']
        for char in invalidChars {
            if (InStr(dirPath, char)) {
                return false
            }
        }
        
        ; Check path length
        if (StrLen(dirPath) > 260) {
            return false  ; May need long path support
        }
        
        ; Check for reserved names
        reservedNames := ["CON", "PRN", "AUX", "NUL", "COM1", "COM2", "COM3", "COM4", 
                         "COM5", "COM6", "COM7", "COM8", "COM9", "LPT1", "LPT2", 
                         "LPT3", "LPT4", "LPT5", "LPT6", "LPT7", "LPT8", "LPT9"]
        
        for part in StrSplit(dirPath, "\") {
            if (reservedNames.IndexOf(StrUpper(part)) > 0) {
                return false
            }
        }
        
        return true
    }
}

; Example usage
; Create project structure
ProjectStructure.Initialize("C:\MyWebProject", "web")
projectPath := ProjectStructure.CreateStructure()
MsgBox("Web project structure created at: " . projectPath)

; Setup backup system
BackupManager.Initialize("C:\Backups\MyApp")
dailyBackup := BackupManager.CreateBackupDirectory("daily")
MsgBox("Daily backup directory created: " . dailyBackup)

; Organize logging
LogDirectoryManager.Initialize("C:\AppLogs")
errorLogPath := LogDirectoryManager.GetLogPath("error")
MsgBox("Error logs will be stored in: " . errorLogPath)

; Safe directory creation
try {
    if (SafeDirectoryOps.ValidatePath("C:\Safe\Project\Path")) {
        SafeDirectoryOps.CreateWithPermissions("C:\Safe\Project\Path")
        MsgBox("Directory created successfully with permission verification")
    }
} catch OSError as err {
    MsgBox("Directory creation failed: " . err.Message)
}
```

## Implementation Notes

**Path Resolution:**
- Supports both absolute paths (C:\...) and relative paths (subfolder\...)
- Automatically resolves parent directory references (..\)
- Handles forward slashes (/) and backslashes (\) in paths
- Long path support depends on Windows version and system configuration

**Permission Handling:**
- Requires write permissions for the parent directory
- Administrative privileges may be needed for system directories
- Network paths require appropriate network permissions
- Some paths may be protected by system security policies

**Error Conditions:**
- Invalid characters in path names cause OSError
- Insufficient permissions result in access denied errors
- Disk space limitations can prevent directory creation
- Path length restrictions vary by file system and OS version

**Performance Characteristics:**
- Very fast for single directory creation
- Nested structure creation may be slower on network drives
- SSD vs HDD performance differences minimal for directory operations
- File system type (NTFS, FAT32) affects feature availability

**Cross-Platform Considerations:**
- Windows path separators (\) vs Unix (/)
- Case sensitivity differences between file systems
- Maximum path length varies by platform
- Reserved names and character restrictions differ by OS

## Related AHK Concepts

- [DirDelete](./dirdelete.md) - Directory removal and cleanup
- [DirExist](./direxist.md) - Directory existence checking
- [FileExist](../File/fileexist.md) - File and directory validation
- [FileWrite](../File/filewrite.md) - File creation requiring directory structure
- [A_WorkingDir](../../../40_Advanced_Features/05-System_Integration/a-workingdir.md) - Current directory management

## Tags

#AutoHotkey #DirCreate #DirectoryManagement #FileSystem #ProjectStructure #Backup #Organization #PathCreation