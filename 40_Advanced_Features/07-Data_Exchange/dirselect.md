# Topic: DirSelect Function - Interactive Directory Selection Dialogs

## Category

Function

## Overview

The DirSelect function displays interactive directory selection dialogs that allow users to browse, navigate, and select folders through the standard Windows folder picker interface. It provides essential directory management capabilities for automation scripts, backup operations, file organization tasks, and applications requiring folder-based operations with intuitive user experience and proper path handling.

## Key Points

- Displays native Windows folder selection dialogs with navigation, creation, and validation capabilities
- Supports custom dialog titles, starting directories, and folder creation options for flexible user workflows
- Enables single directory selection with proper path validation and accessibility checking
- Essential for backup operations, file organization, batch processing, and applications requiring directory-based operations
- Integrates seamlessly with Windows Explorer and provides familiar folder browsing experience

## Syntax and Parameters

```cpp
SelectedFolder := DirSelect([StartingFolder, Options, Title])

; Parameters:
; StartingFolder  - Initial directory path (optional)
; Options         - Dialog behavior options (optional)
; Title           - Dialog window title (optional)

; StartingFolder parameter:
; ""              - Use current directory or last used directory
; "C:\\Projects"  - Start in specific directory
; "*C:\\Projects" - Start in directory, create if doesn't exist
; "::{GUID}"      - Special shell folders (Desktop, Documents, etc.)

; Options parameter:
; Blank           - Standard folder selection dialog
; 1               - Allow creation of new folders in dialog
; 0               - Disable folder creation (read-only selection)

; Title parameter:
; ""              - Use default dialog title ("Select Folder")
; "Choose Backup" - Custom dialog title

; Special shell folder GUIDs:
; "::{20D04FE0-3AEA-1069-A2D8-08002B30309D}" - My Computer
; "::{450D8FBA-AD25-11D0-98A3-F1734F8C6872}" - My Documents
; "::{208D2C60-3AEA-1069-A2D7-08002B30309D}" - My Network Places

; Return values:
; Success: Full path to selected directory
; Cancel:  Empty string if user cancels dialog
; Error:   Throws exception for invalid parameters

; Path format:
; Returns full absolute directory paths
; Paths use backslash separators on Windows
; Trailing backslash is NOT included in returned path
; Unicode directory names fully supported
```

## Code Examples

```cpp
; Basic directory selection
selectedDir := DirSelect()
if (selectedDir) {
    MsgBox("Selected directory: " . selectedDir)
    ; Process the selected directory
} else {
    MsgBox("No directory selected")
}

; Advanced directory management and organization system
class DirectoryManager {
    static directoryHistory := []
    static favoriteDirectories := Map()
    static directoryTemplates := Map()
    static organizationRules := Map()
    static directoryMetrics := {selections: 0, creations: 0, validations: 0, operations: 0}
    static recentDirectories := []
    static maxRecentDirs := 15
    static defaultSettings := Map()
    static securitySettings := {allowedPaths: [], blockedPaths: [], requirePermissions: true}
    static operationHistory := []
    static directoryWatchers := Map()
    static customActions := Map()
    static validationRules := Map()
    
    static Initialize() {
        this.directoryHistory := []
        this.recentDirectories := []
        this.SetupDefaultSettings()
        this.SetupDefaultTemplates()
        this.SetupDefaultRules()
        this.ResetMetrics()
    }
    
    static SelectDirectory(options := {}) {
        ; Enhanced directory selection with validation and management
        startTime := A_TickCount
        
        ; Build dialog parameters
        startingFolder := this.DetermineStartingFolder(options)
        dialogOptions := this.BuildDialogOptions(options)
        title := options.HasProp("title") ? options.title : this.GetDefaultTitle(options)
        
        ; Create selection record
        selectionRecord := {
            timestamp: A_TickCount,
            formattedTime: FormatTime(),
            startingFolder: startingFolder,
            options: dialogOptions,
            title: title,
            result: "",
            cancelled: false,
            created: false,
            selectionTime: 0,
            validated: false,
            error: ""
        }
        
        try {
            ; Show directory dialog
            selectedDir := DirSelect(startingFolder, dialogOptions, title)
            
            selectionRecord.result := selectedDir
            selectionRecord.cancelled := !selectedDir
            selectionRecord.selectionTime := A_TickCount - startTime
            
            ; Validate selection if directory was chosen
            if (selectedDir) {
                if (this.ValidateDirectorySelection(selectedDir, options)) {
                    selectionRecord.validated := true
                    
                    ; Check if directory was created during selection
                    selectionRecord.created := this.CheckIfDirectoryWasCreated(selectedDir, options)
                    
                    ; Update recent directories
                    this.UpdateRecentDirectories(selectedDir)
                    
                    ; Process custom actions
                    this.ProcessCustomActions(selectedDir, options)
                    
                    ; Update metrics
                    this.directoryMetrics.selections++
                    if (selectionRecord.created) {
                        this.directoryMetrics.creations++
                    }
                    
                } else {
                    throw ValidationError("Directory selection failed validation")
                }
            }
            
        } catch Error as err {
            selectionRecord.error := err.Message
            throw err
        } finally {
            ; Log selection operation
            this.LogSelectionOperation(selectionRecord)
        }
        
        return selectionRecord.result
    }
    
    static SelectWithTemplate(templateName, options := {}) {
        ; Select directory using predefined template
        if (!this.directoryTemplates.Has(templateName)) {
            throw ValueError("Directory template '" . templateName . "' not found")
        }
        
        template := this.directoryTemplates[templateName]
        
        ; Merge template with options
        mergedOptions := template.Clone()
        for prop, value in options.OwnProps() {
            mergedOptions.%prop% := value
        }
        
        return this.SelectDirectory(mergedOptions)
    }
    
    static CreateDirectoryStructure(basePath, structure, options := {}) {
        ; Create directory structure from specification
        if (!DirExist(basePath)) {
            if (options.HasProp("createBase") && options.createBase) {
                DirCreate(basePath)
            } else {
                throw DirectoryError("Base directory does not exist: " . basePath)
            }
        }
        
        createdDirs := []
        errors := []
        
        for dirSpec in structure {
            try {
                if (IsString(dirSpec)) {
                    ; Simple directory name
                    dirPath := basePath . "\" . dirSpec
                    if (!DirExist(dirPath)) {
                        DirCreate(dirPath)
                        createdDirs.Push(dirPath)
                    }
                } else if (IsObject(dirSpec)) {
                    ; Directory with subdirectories
                    parentPath := basePath . "\" . dirSpec.name
                    if (!DirExist(parentPath)) {
                        DirCreate(parentPath)
                        createdDirs.Push(parentPath)
                    }
                    
                    ; Create subdirectories
                    if (dirSpec.HasProp("subdirs")) {
                        subResult := this.CreateDirectoryStructure(parentPath, dirSpec.subdirs, options)
                        createdDirs.Push(subResult.created*)
                        errors.Push(subResult.errors*)
                    }
                }
                
            } catch Error as err {
                errors.Push({
                    directory: IsString(dirSpec) ? dirSpec : dirSpec.name,
                    error: err.Message
                })
            }
        }
        
        return {
            created: createdDirs,
            errors: errors,
            successCount: createdDirs.Length,
            errorCount: errors.Length
        }
    }
    
    static OrganizeDirectory(directory, organizationRule, options := {}) {
        ; Organize directory contents according to rules
        if (!this.organizationRules.Has(organizationRule)) {
            throw ValueError("Organization rule '" . organizationRule . "' not found")
        }
        
        rule := this.organizationRules[organizationRule]
        
        if (!DirExist(directory)) {
            throw DirectoryError("Directory does not exist: " . directory)
        }
        
        organizationResults := {
            directory: directory,
            rule: organizationRule,
            processed: 0,
            moved: 0,
            created: 0,
            errors: []
        }
        
        ; Process files according to rule
        Loop Files, directory . "\*.*", "F" {
            try {
                organizationResults.processed++
                
                ; Apply organization rule
                result := this.ApplyOrganizationRule(A_LoopFileFullPath, rule, directory, options)
                
                if (result.moved) {
                    organizationResults.moved++
                }
                if (result.directoryCreated) {
                    organizationResults.created++
                }
                
            } catch Error as err {
                organizationResults.errors.Push({
                    file: A_LoopFileFullPath,
                    error: err.Message
                })
            }
        }
        
        return organizationResults
    }
    
    static ApplyOrganizationRule(filePath, rule, baseDirectory, options) {
        ; Apply organization rule to file
        result := {moved: false, directoryCreated: false, targetPath: ""}
        
        SplitPath(filePath, &fileName, , &fileExt, &fileNameNoExt)
        
        switch rule.type {
            case "by_extension":
                targetDir := baseDirectory . "\" . StrUpper(fileExt) . " Files"
                if (!DirExist(targetDir)) {
                    DirCreate(targetDir)
                    result.directoryCreated := true
                }
                targetPath := targetDir . "\" . fileName
                
            case "by_date":
                fileTime := FileGetTime(filePath, "M")
                targetDir := baseDirectory . "\" . FormatTime(fileTime, "yyyy-MM")
                if (!DirExist(targetDir)) {
                    DirCreate(targetDir)
                    result.directoryCreated := true
                }
                targetPath := targetDir . "\" . fileName
                
            case "by_size":
                fileSize := FileGetSize(filePath)
                if (fileSize < 1024 * 1024) {
                    sizeCategory := "Small"
                } else if (fileSize < 10 * 1024 * 1024) {
                    sizeCategory := "Medium"
                } else {
                    sizeCategory := "Large"
                }
                
                targetDir := baseDirectory . "\" . sizeCategory . " Files"
                if (!DirExist(targetDir)) {
                    DirCreate(targetDir)
                    result.directoryCreated := true
                }
                targetPath := targetDir . "\" . fileName
                
            case "by_pattern":
                if (rule.HasProp("patterns")) {
                    for pattern in rule.patterns {
                        if (RegExMatch(fileName, pattern.regex)) {
                            targetDir := baseDirectory . "\" . pattern.folder
                            if (!DirExist(targetDir)) {
                                DirCreate(targetDir)
                                result.directoryCreated := true
                            }
                            targetPath := targetDir . "\" . fileName
                            break
                        }
                    }
                }
                
                ; Default folder if no pattern matches
                if (!targetPath) {
                    targetDir := baseDirectory . "\Unsorted"
                    if (!DirExist(targetDir)) {
                        DirCreate(targetDir)
                        result.directoryCreated := true
                    }
                    targetPath := targetDir . "\" . fileName
                }
                
            default:
                throw ValueError("Unknown organization rule type: " . rule.type)
        }
        
        ; Move file if target path is different
        if (targetPath && targetPath != filePath) {
            ; Handle file name conflicts
            if (FileExist(targetPath)) {
                targetPath := this.GenerateUniqueFilePath(targetPath)
            }
            
            FileMove(filePath, targetPath)
            result.moved := true
            result.targetPath := targetPath
        }
        
        return result
    }
    
    static WatchDirectory(directory, callback, options := {}) {
        ; Monitor directory for changes
        if (!DirExist(directory)) {
            throw DirectoryError("Directory does not exist: " . directory)
        }
        
        watcherId := this.GenerateWatcherId()
        watchInterval := options.HasProp("interval") ? options.interval : 2000
        watchTimeout := options.HasProp("timeout") ? options.timeout : 0
        includeSubdirs := options.HasProp("includeSubdirs") ? options.includeSubdirs : false
        
        ; Get initial directory state
        initialState := this.GetDirectoryState(directory, includeSubdirs)
        
        watcherData := {
            watcherId: watcherId,
            directory: directory,
            callback: callback,
            options: options,
            initialState: initialState,
            lastState: initialState,
            startTime: A_TickCount,
            timeout: watchTimeout,
            changeCount: 0,
            checkCount: 0
        }
        
        ; Create monitoring timer
        timerFunction := () => {
            try {
                watcherData.checkCount++
                currentState := this.GetDirectoryState(directory, includeSubdirs)
                
                changes := this.CompareDirectoryStates(watcherData.lastState, currentState)
                
                if (changes.hasChanges) {
                    watcherData.changeCount++
                    
                    ; Call callback with change information
                    if (IsFunc(callback)) {
                        callbackData := {
                            watcherId: watcherId,
                            directory: directory,
                            changes: changes,
                            checkCount: watcherData.checkCount,
                            changeCount: watcherData.changeCount,
                            elapsedTime: A_TickCount - watcherData.startTime
                        }
                        
                        callback.Call(callbackData)
                    }
                    
                    watcherData.lastState := currentState
                }
                
                ; Check timeout
                if (watchTimeout > 0 && A_TickCount - watcherData.startTime > watchTimeout) {
                    this.StopDirectoryWatcher(watcherId)
                }
                
            } catch Error as err {
                OutputDebug("Directory watcher error: " . err.Message)
                this.StopDirectoryWatcher(watcherId)
            }
        }
        
        ; Store watcher and start monitoring
        this.directoryWatchers[watcherId] := {
            data: watcherData,
            timer: timerFunction
        }
        
        SetTimer(timerFunction, watchInterval)
        
        return {
            watcherId: watcherId,
            stop: () => this.StopDirectoryWatcher(watcherId),
            getStatus: () => this.GetWatcherStatus(watcherId)
        }
    }
    
    static StopDirectoryWatcher(watcherId) {
        ; Stop directory monitoring
        if (this.directoryWatchers.Has(watcherId)) {
            watcherData := this.directoryWatchers[watcherId]
            SetTimer(watcherData.timer, 0)
            this.directoryWatchers.Delete(watcherId)
            return true
        }
        return false
    }
    
    static GetDirectoryState(directory, includeSubdirs := false) {
        ; Get current state of directory
        state := {
            directory: directory,
            timestamp: A_TickCount,
            files: Map(),
            subdirs: Map(),
            totalFiles: 0,
            totalSize: 0
        }
        
        ; Scan files
        Loop Files, directory . "\*.*", includeSubdirs ? "FR" : "F" {
            fileInfo := {
                path: A_LoopFileFullPath,
                size: A_LoopFileSize,
                modified: A_LoopFileTimeModified,
                created: A_LoopFileTimeCreated
            }
            
            state.files[A_LoopFileFullPath] := fileInfo
            state.totalFiles++
            state.totalSize += A_LoopFileSize
        }
        
        ; Scan directories
        Loop Files, directory . "\*.*", includeSubdirs ? "DR" : "D" {
            dirInfo := {
                path: A_LoopFileFullPath,
                modified: A_LoopFileTimeModified,
                created: A_LoopFileTimeCreated
            }
            
            state.subdirs[A_LoopFileFullPath] := dirInfo
        }
        
        return state
    }
    
    static CompareDirectoryStates(oldState, newState) {
        ; Compare two directory states for changes
        changes := {
            hasChanges: false,
            newFiles: [],
            deletedFiles: [],
            modifiedFiles: [],
            newDirectories: [],
            deletedDirectories: []
        }
        
        ; Check for new and deleted files
        for filePath, fileInfo in newState.files {
            if (!oldState.files.Has(filePath)) {
                changes.newFiles.Push(filePath)
                changes.hasChanges := true
            }
        }
        
        for filePath, fileInfo in oldState.files {
            if (!newState.files.Has(filePath)) {
                changes.deletedFiles.Push(filePath)
                changes.hasChanges := true
            } else {
                ; Check for modifications
                newFileInfo := newState.files[filePath]
                if (fileInfo.modified != newFileInfo.modified || fileInfo.size != newFileInfo.size) {
                    changes.modifiedFiles.Push(filePath)
                    changes.hasChanges := true
                }
            }
        }
        
        ; Check for new and deleted directories
        for dirPath, dirInfo in newState.subdirs {
            if (!oldState.subdirs.Has(dirPath)) {
                changes.newDirectories.Push(dirPath)
                changes.hasChanges := true
            }
        }
        
        for dirPath, dirInfo in oldState.subdirs {
            if (!newState.subdirs.Has(dirPath)) {
                changes.deletedDirectories.Push(dirPath)
                changes.hasChanges := true
            }
        }
        
        return changes
    }
    
    static AddFavoriteDirectory(name, path, description := "") {
        ; Add directory to favorites
        if (!DirExist(path)) {
            throw DirectoryError("Directory does not exist: " . path)
        }
        
        this.favoriteDirectories[name] := {
            path: path,
            description: description,
            added: A_TickCount,
            addedTime: FormatTime(),
            accessCount: 0,
            lastAccess: 0
        }
    }
    
    static RemoveFavoriteDirectory(name) {
        ; Remove directory from favorites
        return this.favoriteDirectories.Delete(name)
    }
    
    static SelectFavoriteDirectory(favoriteName) {
        ; Select from favorite directories
        if (!this.favoriteDirectories.Has(favoriteName)) {
            throw ValueError("Favorite directory '" . favoriteName . "' not found")
        }
        
        favorite := this.favoriteDirectories[favoriteName]
        
        ; Update access statistics
        favorite.accessCount++
        favorite.lastAccess := A_TickCount
        
        return favorite.path
    }
    
    static GetFavoriteDirectories() {
        ; Get list of favorite directories
        favorites := []
        
        for name, favorite in this.favoriteDirectories {
            favorites.Push({
                name: name,
                path: favorite.path,
                description: favorite.description,
                addedTime: favorite.addedTime,
                accessCount: favorite.accessCount,
                lastAccess: favorite.lastAccess
            })
        }
        
        return favorites
    }
    
    static DetermineStartingFolder(options) {
        ; Determine starting folder for dialog
        if (options.HasProp("startingFolder")) {
            return options.startingFolder
        }
        
        if (options.HasProp("useRecentDir") && options.useRecentDir && this.recentDirectories.Length > 0) {
            return this.recentDirectories[1]
        }
        
        if (options.HasProp("useFavorite") && options.useFavorite && this.favoriteDirectories.Has(options.useFavorite)) {
            return this.favoriteDirectories[options.useFavorite].path
        }
        
        if (this.defaultSettings.Has("defaultDirectory")) {
            return this.defaultSettings["defaultDirectory"]
        }
        
        return ""  ; Use system default
    }
    
    static BuildDialogOptions(options) {
        ; Build dialog options
        if (options.HasProp("allowCreation")) {
            return options.allowCreation ? 1 : 0
        }
        
        return 1  ; Default: allow folder creation
    }
    
    static GetDefaultTitle(options) {
        ; Get default dialog title
        if (options.HasProp("operation")) {
            switch options.operation {
                case "backup":
                    return "Select Backup Directory"
                case "export":
                    return "Select Export Directory"
                case "import":
                    return "Select Import Directory"
                case "workspace":
                    return "Select Workspace Directory"
                default:
                    return "Select Directory"
            }
        }
        
        return "Select Directory"
    }
    
    static ValidateDirectorySelection(directory, options) {
        ; Validate selected directory
        try {
            ; Check if directory exists
            if (!DirExist(directory)) {
                throw DirectoryError("Selected directory does not exist")
            }
            
            ; Check permissions if required
            if (this.securitySettings.requirePermissions) {
                if (!this.CheckDirectoryPermissions(directory, options)) {
                    throw SecurityError("Insufficient permissions for directory")
                }
            }
            
            ; Check allowed paths
            if (this.securitySettings.allowedPaths.Length > 0) {
                allowed := false
                for allowedPath in this.securitySettings.allowedPaths {
                    if (InStr(directory, allowedPath) = 1) {
                        allowed := true
                        break
                    }
                }
                if (!allowed) {
                    throw SecurityError("Directory not in allowed paths")
                }
            }
            
            ; Check blocked paths
            for blockedPath in this.securitySettings.blockedPaths {
                if (InStr(directory, blockedPath) = 1) {
                    throw SecurityError("Directory in blocked paths")
                }
            }
            
            ; Apply custom validation
            if (options.HasProp("validationRule") && this.validationRules.Has(options.validationRule)) {
                rule := this.validationRules[options.validationRule]
                if (!this.ApplyValidationRule(directory, rule)) {
                    throw ValidationError("Directory failed custom validation")
                }
            }
            
            this.directoryMetrics.validations++
            return true
            
        } catch Error as err {
            if (options.HasProp("throwOnValidationError") && options.throwOnValidationError) {
                throw err
            }
            return false
        }
    }
    
    static CheckDirectoryPermissions(directory, options) {
        ; Check directory access permissions
        try {
            ; Test read permission
            Loop Files, directory . "\*.*", "F" {
                break  ; Just check if we can enumerate
            }
            
            ; Test write permission if required
            if (options.HasProp("requireWrite") && options.requireWrite) {
                testFile := directory . "\test_write_" . A_TickCount . ".tmp"
                FileAppend("test", testFile)
                FileDelete(testFile)
            }
            
            return true
            
        } catch Error {
            return false
        }
    }
    
    static ApplyValidationRule(directory, rule) {
        ; Apply custom validation rule
        switch rule.type {
            case "empty":
                ; Check if directory is empty
                isEmpty := true
                Loop Files, directory . "\*.*", "FD" {
                    isEmpty := false
                    break
                }
                return rule.requireEmpty ? isEmpty : !isEmpty
                
            case "contains":
                ; Check if directory contains specific files/folders
                if (rule.HasProp("requiredFiles")) {
                    for requiredFile in rule.requiredFiles {
                        if (!FileExist(directory . "\" . requiredFile)) {
                            return false
                        }
                    }
                }
                return true
                
            case "size":
                ; Check directory size constraints
                totalSize := this.CalculateDirectorySize(directory)
                if (rule.HasProp("maxSize") && totalSize > rule.maxSize) return false
                if (rule.HasProp("minSize") && totalSize < rule.minSize) return false
                return true
                
            case "custom":
                ; Custom validation function
                if (rule.HasProp("validator") && IsFunc(rule.validator)) {
                    return rule.validator.Call(directory)
                }
                return true
                
            default:
                return true
        }
    }
    
    static CalculateDirectorySize(directory) {
        ; Calculate total size of directory
        totalSize := 0
        
        Loop Files, directory . "\*.*", "FR" {
            totalSize += A_LoopFileSize
        }
        
        return totalSize
    }
    
    static CheckIfDirectoryWasCreated(directory, options) {
        ; Check if directory was newly created during selection
        ; This is a simplified check - in practice, you might track creation times
        return false  ; Placeholder implementation
    }
    
    static UpdateRecentDirectories(directory) {
        ; Update recent directories list
        if (!directory || !DirExist(directory)) {
            return
        }
        
        ; Remove if already exists
        for i, dir in this.recentDirectories {
            if (dir = directory) {
                this.recentDirectories.RemoveAt(i)
                break
            }
        }
        
        ; Add to front
        this.recentDirectories.InsertAt(1, directory)
        
        ; Limit size
        while (this.recentDirectories.Length > this.maxRecentDirs) {
            this.recentDirectories.Pop()
        }
    }
    
    static ProcessCustomActions(directory, options) {
        ; Process custom actions on selected directory
        if (options.HasProp("customActions")) {
            for actionName in options.customActions {
                if (this.customActions.Has(actionName)) {
                    action := this.customActions[actionName]
                    try {
                        if (IsFunc(action.handler)) {
                            action.handler.Call(directory, options)
                        }
                    } catch Error as err {
                        OutputDebug("Custom action error: " . err.Message)
                    }
                }
            }
        }
    }
    
    static GenerateUniqueFilePath(basePath) {
        ; Generate unique file path to avoid conflicts
        SplitPath(basePath, &fileName, &dir, &ext, &nameNoExt)
        
        counter := 1
        while (FileExist(basePath)) {
            newName := nameNoExt . "_" . counter . (ext ? "." . ext : "")
            basePath := dir . "\" . newName
            counter++
        }
        
        return basePath
    }
    
    static GenerateWatcherId() {
        ; Generate unique watcher ID
        return "watcher_" . A_TickCount . "_" . Random(1000, 9999)
    }
    
    static GetWatcherStatus(watcherId) {
        ; Get directory watcher status
        if (this.directoryWatchers.Has(watcherId)) {
            return this.directoryWatchers[watcherId].data
        }
        return false
    }
    
    static LogSelectionOperation(selectionRecord) {
        ; Log directory selection operation
        this.directoryHistory.Push(selectionRecord)
        
        ; Limit history size
        if (this.directoryHistory.Length > 100) {
            this.directoryHistory.RemoveAt(1)
        }
        
        this.operationHistory.Push({
            type: "selection",
            timestamp: selectionRecord.timestamp,
            directory: selectionRecord.result,
            success: !selectionRecord.cancelled && !selectionRecord.error,
            duration: selectionRecord.selectionTime
        })
    }
    
    static SetupDefaultSettings() {
        ; Set up default settings
        this.defaultSettings := Map(
            "defaultDirectory", A_MyDocuments,
            "allowCreation", true,
            "validateSelection", true,
            "requirePermissions", false
        )
    }
    
    static SetupDefaultTemplates() {
        ; Set up directory templates
        this.directoryTemplates := Map()
        
        this.directoryTemplates["backup"] := {
            title: "Select Backup Directory",
            operation: "backup",
            allowCreation: true,
            requireWrite: true
        }
        
        this.directoryTemplates["workspace"] := {
            title: "Select Workspace",
            operation: "workspace",
            allowCreation: true,
            validationRule: "workspace_structure"
        }
        
        this.directoryTemplates["export"] := {
            title: "Select Export Directory",
            operation: "export",
            allowCreation: true,
            requireWrite: true
        }
    }
    
    static SetupDefaultRules() {
        ; Set up organization and validation rules
        this.organizationRules := Map()
        
        this.organizationRules["by_type"] := {
            type: "by_extension",
            description: "Organize files by file type"
        }
        
        this.organizationRules["by_date"] := {
            type: "by_date",
            description: "Organize files by modification date"
        }
        
        this.organizationRules["by_project"] := {
            type: "by_pattern",
            patterns: [
                {regex: "^Project_(\w+)", folder: "Projects\\$1"},
                {regex: "^Backup_", folder: "Backups"},
                {regex: "^Temp_", folder: "Temporary"}
            ],
            description: "Organize files by project patterns"
        }
        
        ; Validation rules
        this.validationRules := Map()
        
        this.validationRules["workspace_structure"] := {
            type: "contains",
            requiredFiles: ["src", "docs", "tests"],
            description: "Valid workspace structure"
        }
        
        this.validationRules["empty_directory"] := {
            type: "empty",
            requireEmpty: true,
            description: "Directory must be empty"
        }
    }
    
    static ResetMetrics() {
        ; Reset directory metrics
        this.directoryMetrics := {
            selections: 0,
            creations: 0,
            validations: 0,
            operations: 0
        }
    }
    
    static RegisterCustomAction(actionName, handler, description := "") {
        ; Register custom action handler
        this.customActions[actionName] := {
            handler: handler,
            description: description,
            registered: A_TickCount
        }
    }
    
    static RegisterValidationRule(ruleName, rule) {
        ; Register custom validation rule
        this.validationRules[ruleName] := rule
    }
    
    static RegisterOrganizationRule(ruleName, rule) {
        ; Register organization rule
        this.organizationRules[ruleName] := rule
    }
    
    static SetSecuritySettings(settings) {
        ; Update security settings
        if (settings.HasProp("allowedPaths")) {
            this.securitySettings.allowedPaths := settings.allowedPaths
        }
        if (settings.HasProp("blockedPaths")) {
            this.securitySettings.blockedPaths := settings.blockedPaths
        }
        if (settings.HasProp("requirePermissions")) {
            this.securitySettings.requirePermissions := settings.requirePermissions
        }
    }
    
    static GetDirectoryStatistics() {
        ; Get directory management statistics
        stats := this.directoryMetrics.Clone()
        
        stats.favoriteCount := this.favoriteDirectories.Count
        stats.recentCount := this.recentDirectories.Length
        stats.templateCount := this.directoryTemplates.Count
        stats.watcherCount := this.directoryWatchers.Count
        
        return stats
    }
    
    static GenerateManagementReport() {
        ; Generate comprehensive directory management report
        stats := this.GetDirectoryStatistics()
        
        report := "Directory Management Report`n"
        report .= "=============================`n`n"
        
        report .= "Selection Statistics:`n"
        report .= "Total Selections: " . stats.selections . "`n"
        report .= "Directories Created: " . stats.creations . "`n"
        report .= "Validations Performed: " . stats.validations . "`n"
        report .= "Total Operations: " . stats.operations . "`n`n"
        
        report .= "Management Features:`n"
        report .= "Favorite Directories: " . stats.favoriteCount . "`n"
        report .= "Recent Directories: " . stats.recentCount . "`n"
        report .= "Directory Templates: " . stats.templateCount . "`n"
        report .= "Active Watchers: " . stats.watcherCount . "`n`n"
        
        ; Recent directories
        if (this.recentDirectories.Length > 0) {
            report .= "Recent Directories:`n"
            for i, dir in this.recentDirectories {
                report .= i . ". " . dir . "`n"
                if (i >= 5) break
            }
            report .= "`n"
        }
        
        ; Favorite directories
        if (this.favoriteDirectories.Count > 0) {
            report .= "Favorite Directories:`n"
            for name, favorite in this.favoriteDirectories {
                report .= "• " . name . ": " . favorite.path . " (" . favorite.accessCount . " accesses)`n"
            }
        }
        
        return report
    }
    
    static StopAllDirectoryWatchers() {
        ; Stop all directory watchers
        for watcherId in this.directoryWatchers {
            this.StopDirectoryWatcher(watcherId)
        }
    }
    
    static GetRecentDirectories() {
        return this.recentDirectories.Clone()
    }
    
    static ClearRecentDirectories() {
        this.recentDirectories := []
    }
}

; Example usage and demonstrations
; Initialize directory management system
DirectoryManager.Initialize()

; Basic directory selection
selectedDir := DirSelect()
if (selectedDir) {
    MsgBox("Selected directory: " . selectedDir)
    ; Process the selected directory
} else {
    MsgBox("No directory selected")
}

; Enhanced directory selection with validation
backupDir := DirectoryManager.SelectDirectory({
    title: "Select Backup Directory",
    operation: "backup",
    allowCreation: true,
    requireWrite: true,
    useRecentDir: true
})

if (backupDir) {
    MsgBox("Backup directory selected: " . backupDir)
    ; Perform backup operations...
}

; Using directory templates
workspaceDir := DirectoryManager.SelectWithTemplate("workspace", {
    startingFolder: "C:\\Projects"
})

if (workspaceDir) {
    MsgBox("Workspace directory: " . workspaceDir)
}

; Creating directory structure
projectStructure := [
    "src",
    "docs", 
    "tests",
    {name: "build", subdirs: ["debug", "release"]},
    {name: "resources", subdirs: ["images", "data", "config"]}
]

baseDir := DirectoryManager.SelectDirectory({
    title: "Select Project Root",
    allowCreation: true
})

if (baseDir) {
    try {
        structureResult := DirectoryManager.CreateDirectoryStructure(baseDir, projectStructure, {
            createBase: true
        })
        
        MsgBox("Project structure created:`n" .
               "Created: " . structureResult.successCount . " directories`n" .
               "Errors: " . structureResult.errorCount)
        
        ; Display created directories
        for dir in structureResult.created {
            OutputDebug("Created: " . dir)
        }
        
    } catch Error as err {
        MsgBox("Failed to create structure: " . err.Message)
    }
}

; Directory organization
messyDir := DirectoryManager.SelectDirectory({
    title: "Select Directory to Organize",
    validateSelection: true
})

if (messyDir) {
    try {
        orgResult := DirectoryManager.OrganizeDirectory(messyDir, "by_type")
        
        MsgBox("Directory organization complete:`n" .
               "Files processed: " . orgResult.processed . "`n" .
               "Files moved: " . orgResult.moved . "`n" .
               "Directories created: " . orgResult.created . "`n" .
               "Errors: " . orgResult.errors.Length)
        
    } catch Error as err {
        MsgBox("Organization failed: " . err.Message)
    }
}

; Directory monitoring
monitorDir := DirectoryManager.SelectDirectory({
    title: "Select Directory to Monitor"
})

if (monitorDir) {
    watcher := DirectoryManager.WatchDirectory(monitorDir, (callbackData) => {
        changes := callbackData.changes
        changeText := "Directory changes detected:`n"
        
        if (changes.newFiles.Length > 0) {
            changeText .= "New files: " . changes.newFiles.Length . "`n"
        }
        if (changes.deletedFiles.Length > 0) {
            changeText .= "Deleted files: " . changes.deletedFiles.Length . "`n"
        }
        if (changes.newDirectories.Length > 0) {
            changeText .= "New directories: " . changes.newDirectories.Length . "`n"
        }
        
        MsgBox(changeText . "Check count: " . callbackData.checkCount)
    }, {
        interval: 3000,
        timeout: 30000,
        includeSubdirs: false
    })
    
    MsgBox("Monitoring " . monitorDir . " for 30 seconds...")
    
    ; The watcher will automatically stop after timeout
    ; To stop manually: watcher.stop()
}

; Managing favorite directories
DirectoryManager.AddFavoriteDirectory("Projects", "C:\\MyProjects", "Main project directory")
DirectoryManager.AddFavoriteDirectory("Backup", "D:\\Backup", "Backup storage location")
DirectoryManager.AddFavoriteDirectory("Documents", A_MyDocuments, "Personal documents")

; Select from favorites
favorites := DirectoryManager.GetFavoriteDirectories()
if (favorites.Length > 0) {
    favoriteList := "Favorite Directories:`n"
    for favorite in favorites {
        favoriteList .= "• " . favorite.name . ": " . favorite.path . 
                       " (accessed " . favorite.accessCount . " times)`n"
    }
    ; MsgBox(favoriteList)
}

; Use favorite directory
try {
    projectsPath := DirectoryManager.SelectFavoriteDirectory("Projects")
    MsgBox("Projects directory: " . projectsPath)
} catch Error as err {
    MsgBox("Error accessing favorite: " . err.Message)
}

; Custom validation rules
DirectoryManager.RegisterValidationRule("project_directory", {
    type: "contains",
    requiredFiles: ["package.json", "src", "README.md"],
    description: "Valid Node.js project directory"
})

; Custom organization rule
DirectoryManager.RegisterOrganizationRule("by_project_type", {
    type: "by_pattern",
    patterns: [
        {regex: "\.js$", folder: "JavaScript"},
        {regex: "\.py$", folder: "Python"},
        {regex: "\.ahk$", folder: "AutoHotkey"},
        {regex: "\.(jpg|png|gif)$", folder: "Images"}
    ],
    description: "Organize by programming language and file type"
})

; Custom actions
DirectoryManager.RegisterCustomAction("create_readme", (directory, options) => {
    readmePath := directory . "\README.md"
    if (!FileExist(readmePath)) {
        FileAppend("# Project Directory`n`nCreated: " . FormatTime() . "`n", readmePath)
        OutputDebug("Created README.md in: " . directory)
    }
}, "Create README.md file if it doesn't exist")

; Use custom validation and actions
validatedDir := DirectoryManager.SelectDirectory({
    title: "Select Project Directory",
    validationRule: "project_directory",
    customActions: ["create_readme"],
    throwOnValidationError: false
})

; Security settings
DirectoryManager.SetSecuritySettings({
    allowedPaths: ["C:\\Users", "D:\\Projects", "E:\\Backup"],
    blockedPaths: ["C:\\Windows", "C:\\Program Files"],
    requirePermissions: true
})

; Test security restrictions
try {
    secureDir := DirectoryManager.SelectDirectory({
        title: "Select Secure Directory",
        requireWrite: true,
        throwOnValidationError: true
    })
    
    if (secureDir) {
        MsgBox("Secure directory selected: " . secureDir)
    }
    
} catch SecurityError as err {
    MsgBox("Security validation failed: " . err.Message)
} catch ValidationError as err {
    MsgBox("Directory validation failed: " . err.Message)
}

; Directory statistics and reporting
stats := DirectoryManager.GetDirectoryStatistics()
MsgBox("Directory Management Statistics:`n" .
       "Selections: " . stats.selections . "`n" .
       "Validations: " . stats.validations . "`n" .
       "Favorites: " . stats.favoriteCount . "`n" .
       "Recent: " . stats.recentCount . "`n" .
       "Active watchers: " . stats.watcherCount)

; Get recent directories
recentDirs := DirectoryManager.GetRecentDirectories()
if (recentDirs.Length > 0) {
    recentList := "Recent Directories:`n"
    for i, dir in recentDirs {
        recentList .= i . ". " . dir . "`n"
        if (i >= 5) break
    }
    ; MsgBox(recentList)
}

; Generate comprehensive report
managementReport := DirectoryManager.GenerateManagementReport()
; MsgBox(managementReport, "Directory Management Report")

; Cleanup demonstration
MsgBox("Directory management demonstration complete.`n" .
       "Total selections: " . DirectoryManager.GetDirectoryStatistics().selections . "`n" .
       "Favorite directories: " . DirectoryManager.GetFavoriteDirectories().Length . "`n" .
       "Recent directories: " . DirectoryManager.GetRecentDirectories().Length)

; Stop all watchers and cleanup
DirectoryManager.StopAllDirectoryWatchers()
DirectoryManager.ClearRecentDirectories()
```

## Implementation Notes

**Dialog Behavior and User Experience:**
- DirSelect displays the standard Windows folder browser dialog, ensuring consistent user experience across applications
- Dialog remembers the last used directory unless explicitly overridden, improving workflow efficiency
- Folder creation capability can be enabled or disabled based on application requirements and security policies
- Handle user cancellation gracefully and provide appropriate feedback for empty selections

**Path Handling and Validation:**
- Returned directory paths are always full absolute paths without trailing backslashes for consistency
- Unicode directory names and paths with special characters are fully supported in modern Windows versions
- Validate directory accessibility and permissions before using selected paths for file operations
- Consider network path support and handle cases where selected directories may become unavailable

**Security and Permission Considerations:**
- Verify that the application has appropriate access rights to selected directories before performing operations
- Implement path validation to prevent selection of system directories or locations that could compromise security
- Consider user account control (UAC) implications when selecting directories that require elevated permissions
- Handle cases where selected directories may be on removable media or network locations

**Integration with File Operations:**
- Combine DirSelect with file operations like FileCopy, FileMove, and DirCopy for complete directory management workflows
- Implement error handling for cases where selected directories become inaccessible between selection and use
- Consider directory monitoring and change detection for applications that need to track directory modifications
- Provide validation for directory structure requirements specific to application needs

**Performance and Resource Management:**
- Directory validation and permission checks can be time-consuming for large directory structures or network paths
- Implement appropriate timeouts and progress feedback for operations that scan directory contents extensively
- Consider caching directory information and validation results for frequently accessed locations
- Monitor memory usage when processing large directory structures or maintaining directory history

## Related AHK Concepts

- [FileSelectFile](./fileselectfile.md) - File selection dialogs for choosing individual files
- [DirCreate](../../30_Built_In_Classes/02-File_IO_Classes/Dir/dircreate.md) - Creating new directories programmatically
- [DirExist](../../10_Language_Core/01-Functions/Built_In_Functions/direxist.md) - Checking directory existence and accessibility
- [Loop Files](../../10_Language_Core/00-Control_Flow/Loop_Constructs/loop-files.md) - Iterating through directory contents
- [SplitPath](../../10_Language_Core/01-Functions/Built_In_Functions/splitpath.md) - Parsing directory paths and components

## Tags

#AutoHotkey #DirSelect #DirectorySelection #FolderDialog #DirectoryManagement #UserInterface #PathHandling #DirectoryOperations #FolderBrowser #DirectoryValidation
