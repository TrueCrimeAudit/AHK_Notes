# Topic: FileSelectFile Function - Interactive File Selection Dialogs

## Category

Function

## Overview

The FileSelectFile function displays interactive file selection dialogs that allow users to browse, filter, and select files through the standard Windows file picker interface. It provides essential user interaction capabilities for file-based operations, enabling professional applications to implement intuitive file management, document processing, and data import/export workflows with proper user experience standards.

## Key Points

- Displays native Windows file selection dialogs with filtering, multi-selection, and navigation capabilities
- Supports comprehensive file type filtering, default directories, and customizable dialog titles and behavior
- Enables both single and multiple file selection with proper path handling and validation
- Essential for user-friendly applications requiring file input, document processing, and data import/export operations
- Integrates seamlessly with Windows shell extensions and provides familiar user interface patterns

## Syntax and Parameters

```cpp
SelectedFile := FileSelectFile([Options, RootDir, Title, Filter])

; Parameters:
; Options      - Dialog behavior options (optional)
; RootDir      - Initial directory path (optional)
; Title        - Dialog window title (optional)
; Filter       - File type filters (optional)

; Options parameter:
; Blank        - Standard file open dialog
; "S"          - Save dialog instead of open dialog  
; "M"          - Multi-select mode (allows multiple files)
; "D"          - Select directories (folders) instead of files
; Combination  - Multiple options can be combined (e.g., "MS")

; RootDir parameter:
; ""           - Use current directory or last used directory
; "C:\\Docs"   - Start in specific directory
; "*C:\\Docs"  - Start in directory, create if doesn't exist

; Title parameter:
; ""           - Use default dialog title
; "Select Document" - Custom dialog title

; Filter parameter:
; ""                    - All files (*.*)
; "Text Files (*.txt)"  - Single filter
; "Text (*.txt)|All (*.*)" - Multiple filters separated by |
; "Images (*.jpg;*.png)" - Multiple extensions in one filter

; Return values:
; Single file:  Full path to selected file, or empty string if cancelled
; Multi-select: Array of file paths, or empty array if cancelled
; Save dialog:  Full path including filename, even if file doesn't exist

; File path format:
; Returns full absolute paths
; Paths use backslash separators on Windows
; Handles long file names and paths properly
; Unicode file names supported
```

## Code Examples

```cpp
; Basic file selection
selectedFile := FileSelectFile()
if (selectedFile) {
    MsgBox("Selected file: " . selectedFile)
    ; Process the selected file
} else {
    MsgBox("No file selected")
}

; Advanced file dialog and management system
class FileDialogManager {
    static dialogHistory := []
    static defaultSettings := Map()
    static fileTypeFilters := Map()
    static recentDirectories := []
    static maxRecentDirs := 10
    static dialogTemplates := Map()
    static validationRules := Map()
    static securitySettings := {allowedExtensions: [], blockedExtensions: [], maxFileSize: 0}
    static operationHistory := []
    static customActions := Map()
    static dialogMetrics := {opens: 0, saves: 0, cancellations: 0, errors: 0}
    
    static Initialize() {
        this.dialogHistory := []
        this.recentDirectories := []
        this.SetupDefaultSettings()
        this.SetupDefaultFilters()
        this.SetupDefaultTemplates()
        this.ResetMetrics()
    }
    
    static SelectFile(options := {}) {
        ; Enhanced file selection with validation and history
        startTime := A_TickCount
        
        ; Build dialog parameters
        dialogOptions := this.BuildDialogOptions(options)
        rootDir := this.DetermineRootDirectory(options)
        title := options.HasProp("title") ? options.title : this.GetDefaultTitle(options)
        filter := this.BuildFileFilter(options)
        
        ; Create dialog record
        dialogRecord := {
            timestamp: A_TickCount,
            formattedTime: FormatTime(),
            dialogType: "select",
            options: dialogOptions,
            rootDir: rootDir,
            title: title,
            filter: filter,
            result: "",
            cancelled: false,
            dialogTime: 0,
            validated: false,
            error: ""
        }
        
        try {
            ; Show file dialog
            selectedFile := FileSelectFile(dialogOptions, rootDir, title, filter)
            
            dialogRecord.result := selectedFile
            dialogRecord.cancelled := !selectedFile
            dialogRecord.dialogTime := A_TickCount - startTime
            
            ; Validate selection if file was chosen
            if (selectedFile) {
                if (this.ValidateFileSelection(selectedFile, options)) {
                    dialogRecord.validated := true
                    
                    ; Update recent directories
                    this.UpdateRecentDirectories(SplitPath(selectedFile,, &dir))
                    
                    ; Process custom actions
                    this.ProcessCustomActions(selectedFile, options)
                    
                    ; Update metrics
                    this.dialogMetrics.opens++
                    
                } else {
                    throw ValidationError("File selection failed validation")
                }
            } else {
                this.dialogMetrics.cancellations++
            }
            
        } catch Error as err {
            dialogRecord.error := err.Message
            this.dialogMetrics.errors++
            throw err
        } finally {
            ; Log dialog operation
            this.LogDialogOperation(dialogRecord)
        }
        
        return dialogRecord.result
    }
    
    static SelectMultipleFiles(options := {}) {
        ; Select multiple files with enhanced processing
        multiOptions := options.Clone()
        multiOptions.multiSelect := true
        
        ; Ensure multi-select option is set
        if (!multiOptions.HasProp("options")) {
            multiOptions.options := "M"
        } else if (!InStr(multiOptions.options, "M")) {
            multiOptions.options .= "M"
        }
        
        result := this.SelectFile(multiOptions)
        
        ; Process multi-select result
        if (result) {
            ; FileSelectFile returns newline-separated paths for multi-select
            files := StrSplit(result, "`n")
            
            ; Validate each file if validation is enabled
            if (multiOptions.HasProp("validateFiles") && multiOptions.validateFiles) {
                validatedFiles := []
                for file in files {
                    if (this.ValidateFileSelection(file, multiOptions)) {
                        validatedFiles.Push(file)
                    }
                }
                return validatedFiles
            }
            
            return files
        }
        
        return []
    }
    
    static SaveFileDialog(options := {}) {
        ; Enhanced save file dialog
        saveOptions := options.Clone()
        
        ; Ensure save option is set
        if (!saveOptions.HasProp("options")) {
            saveOptions.options := "S"
        } else if (!InStr(saveOptions.options, "S")) {
            saveOptions.options .= "S"
        }
        
        ; Provide default filename if specified
        if (saveOptions.HasProp("defaultFileName")) {
            if (!saveOptions.HasProp("rootDir")) {
                saveOptions.rootDir := ""
            }
            saveOptions.rootDir .= "\" . saveOptions.defaultFileName
        }
        
        result := this.SelectFile(saveOptions)
        
        ; Handle save-specific processing
        if (result) {
            this.dialogMetrics.saves++
            
            ; Check for file overwrite
            if (FileExist(result) && (!saveOptions.HasProp("confirmOverwrite") || saveOptions.confirmOverwrite)) {
                overwrite := MsgBox("File already exists. Overwrite?", "Confirm Overwrite", "YN")
                if (overwrite = "No") {
                    return ""
                }
            }
            
            ; Create directory if it doesn't exist
            SplitPath(result,, &dir)
            if (!DirExist(dir) && (saveOptions.HasProp("createDirectory") && saveOptions.createDirectory)) {
                DirCreate(dir)
            }
        }
        
        return result
    }
    
    static SelectWithTemplate(templateName, options := {}) {
        ; Select file using predefined template
        if (!this.dialogTemplates.Has(templateName)) {
            throw ValueError("Dialog template '" . templateName . "' not found")
        }
        
        template := this.dialogTemplates[templateName]
        
        ; Merge template with options
        mergedOptions := template.Clone()
        for prop, value in options.OwnProps() {
            mergedOptions.%prop% := value
        }
        
        return this.SelectFile(mergedOptions)
    }
    
    static CreateFileDialog(dialogSpec) {
        ; Create custom file dialog with advanced options
        dialog := {
            id: this.GenerateDialogId(),
            spec: dialogSpec,
            created: A_TickCount,
            useCount: 0,
            lastUsed: 0
        }
        
        ; Validate dialog specification
        this.ValidateDialogSpec(dialogSpec)
        
        return dialog
    }
    
    static BatchFileOperation(files, operation, options := {}) {
        ; Perform batch operations on selected files
        results := []
        errors := []
        
        for file in files {
            try {
                result := this.ProcessFileOperation(file, operation, options)
                results.Push({
                    file: file,
                    operation: operation,
                    result: result,
                    success: true
                })
            } catch Error as err {
                errors.Push({
                    file: file,
                    operation: operation,
                    error: err.Message,
                    success: false
                })
            }
        }
        
        return {
            results: results,
            errors: errors,
            successCount: results.Length,
            errorCount: errors.Length,
            totalFiles: files.Length
        }
    }
    
    static ProcessFileOperation(file, operation, options) {
        ; Process individual file operation
        switch operation {
            case "copy":
                if (!options.HasProp("destination")) {
                    throw ValueError("Copy operation requires destination")
                }
                FileCopy(file, options.destination)
                return options.destination
                
            case "move":
                if (!options.HasProp("destination")) {
                    throw ValueError("Move operation requires destination")
                }
                FileMove(file, options.destination)
                return options.destination
                
            case "delete":
                FileDelete(file)
                return "deleted"
                
            case "info":
                return {
                    size: FileGetSize(file),
                    time: FileGetTime(file),
                    attrib: FileGetAttrib(file)
                }
                
            case "validate":
                return this.ValidateFileSelection(file, options)
                
            default:
                throw ValueError("Unknown file operation: " . operation)
        }
    }
    
    static CreateFileFilter(filterSpec) {
        ; Create file filter from specification
        if (IsString(filterSpec)) {
            return filterSpec  ; Already a filter string
        }
        
        filterParts := []
        
        for name, extensions in filterSpec {
            if (IsArray(extensions)) {
                extString := ""
                for ext in extensions {
                    if (extString) {
                        extString .= ";"
                    }
                    extString .= "*." . ext
                }
                filterParts.Push(name . " (" . extString . ")")
            } else {
                filterParts.Push(name . " (*." . extensions . ")")
            }
        }
        
        return filterParts.Join("|")
    }
    
    static BuildDialogOptions(options) {
        ; Build dialog options string
        optionString := ""
        
        if (options.HasProp("save") && options.save) {
            optionString .= "S"
        }
        
        if (options.HasProp("multiSelect") && options.multiSelect) {
            optionString .= "M"
        }
        
        if (options.HasProp("selectDirectories") && options.selectDirectories) {
            optionString .= "D"
        }
        
        if (options.HasProp("options")) {
            optionString := options.options  ; Override with explicit options
        }
        
        return optionString
    }
    
    static DetermineRootDirectory(options) {
        ; Determine starting directory for dialog
        if (options.HasProp("rootDir")) {
            return options.rootDir
        }
        
        if (options.HasProp("useRecentDir") && options.useRecentDir && this.recentDirectories.Length > 0) {
            return this.recentDirectories[1]
        }
        
        if (this.defaultSettings.Has("defaultDirectory")) {
            return this.defaultSettings["defaultDirectory"]
        }
        
        return ""  ; Use system default
    }
    
    static BuildFileFilter(options) {
        ; Build file filter string
        if (options.HasProp("filter")) {
            if (IsString(options.filter)) {
                return options.filter
            } else {
                return this.CreateFileFilter(options.filter)
            }
        }
        
        if (options.HasProp("fileTypes")) {
            return this.GetFileTypeFilter(options.fileTypes)
        }
        
        return ""  ; No filter (all files)
    }
    
    static GetFileTypeFilter(fileTypes) {
        ; Get predefined file type filters
        if (IsString(fileTypes)) {
            fileTypes := [fileTypes]
        }
        
        filterParts := []
        
        for fileType in fileTypes {
            if (this.fileTypeFilters.Has(fileType)) {
                filterParts.Push(this.fileTypeFilters[fileType])
            }
        }
        
        if (filterParts.Length > 0) {
            return filterParts.Join("|")
        }
        
        return ""
    }
    
    static GetDefaultTitle(options) {
        ; Get default dialog title based on operation
        if (options.HasProp("save") && options.save) {
            return "Save File"
        }
        
        if (options.HasProp("multiSelect") && options.multiSelect) {
            return "Select Files"
        }
        
        if (options.HasProp("selectDirectories") && options.selectDirectories) {
            return "Select Folder"
        }
        
        return "Select File"
    }
    
    static ValidateFileSelection(filePath, options) {
        ; Validate selected file against rules
        try {
            ; Check if file exists (for open operations)
            if (options.HasProp("mustExist") && options.mustExist && !FileExist(filePath)) {
                throw FileNotFoundError("Selected file does not exist")
            }
            
            ; Check file extension
            if (this.securitySettings.allowedExtensions.Length > 0) {
                SplitPath(filePath,,, &ext)
                if (!this.securitySettings.allowedExtensions.Find(StrLower(ext))) {
                    throw SecurityError("File extension not allowed: " . ext)
                }
            }
            
            ; Check blocked extensions
            if (this.securitySettings.blockedExtensions.Length > 0) {
                SplitPath(filePath,,, &ext)
                if (this.securitySettings.blockedExtensions.Find(StrLower(ext))) {
                    throw SecurityError("File extension blocked: " . ext)
                }
            }
            
            ; Check file size
            if (this.securitySettings.maxFileSize > 0 && FileExist(filePath)) {
                fileSize := FileGetSize(filePath)
                if (fileSize > this.securitySettings.maxFileSize) {
                    throw SecurityError("File size exceeds limit: " . Round(fileSize / 1024 / 1024, 1) . " MB")
                }
            }
            
            ; Apply custom validation rules
            if (options.HasProp("validationRule") && this.validationRules.Has(options.validationRule)) {
                rule := this.validationRules[options.validationRule]
                if (!this.ApplyValidationRule(filePath, rule)) {
                    throw ValidationError("File failed custom validation: " . options.validationRule)
                }
            }
            
            return true
            
        } catch Error as err {
            if (options.HasProp("throwOnValidationError") && options.throwOnValidationError) {
                throw err
            }
            return false
        }
    }
    
    static ApplyValidationRule(filePath, rule) {
        ; Apply custom validation rule
        switch rule.type {
            case "size":
                if (!FileExist(filePath)) return true
                fileSize := FileGetSize(filePath)
                if (rule.HasProp("minSize") && fileSize < rule.minSize) return false
                if (rule.HasProp("maxSize") && fileSize > rule.maxSize) return false
                return true
                
            case "extension":
                SplitPath(filePath,,, &ext)
                return rule.HasProp("extensions") && rule.extensions.Find(StrLower(ext))
                
            case "path":
                if (rule.HasProp("allowedPaths")) {
                    for allowedPath in rule.allowedPaths {
                        if (InStr(filePath, allowedPath) = 1) {
                            return true
                        }
                    }
                    return false
                }
                return true
                
            case "custom":
                if (rule.HasProp("validator") && IsFunc(rule.validator)) {
                    return rule.validator.Call(filePath)
                }
                return true
                
            default:
                return true
        }
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
    
    static ProcessCustomActions(filePath, options) {
        ; Process custom actions on selected file
        if (options.HasProp("customActions")) {
            for actionName in options.customActions {
                if (this.customActions.Has(actionName)) {
                    action := this.customActions[actionName]
                    try {
                        if (IsFunc(action.handler)) {
                            action.handler.Call(filePath, options)
                        }
                    } catch Error as err {
                        OutputDebug("Custom action error: " . err.Message)
                    }
                }
            }
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
    
    static SetSecuritySettings(settings) {
        ; Update security settings
        if (settings.HasProp("allowedExtensions")) {
            this.securitySettings.allowedExtensions := settings.allowedExtensions
        }
        if (settings.HasProp("blockedExtensions")) {
            this.securitySettings.blockedExtensions := settings.blockedExtensions
        }
        if (settings.HasProp("maxFileSize")) {
            this.securitySettings.maxFileSize := settings.maxFileSize
        }
    }
    
    static SetupDefaultSettings() {
        ; Set up default dialog settings
        this.defaultSettings := Map(
            "defaultDirectory", A_MyDocuments,
            "confirmOverwrite", true,
            "createDirectory", false,
            "validateFiles", true
        )
    }
    
    static SetupDefaultFilters() {
        ; Set up common file type filters
        this.fileTypeFilters := Map(
            "text", "Text Files (*.txt)",
            "image", "Image Files (*.jpg;*.png;*.gif;*.bmp)",
            "document", "Documents (*.doc;*.docx;*.pdf;*.rtf)",
            "spreadsheet", "Spreadsheets (*.xls;*.xlsx;*.csv)",
            "audio", "Audio Files (*.mp3;*.wav;*.flac;*.aac)",
            "video", "Video Files (*.mp4;*.avi;*.mkv;*.mov)",
            "archive", "Archive Files (*.zip;*.rar;*.7z;*.tar)",
            "executable", "Executable Files (*.exe;*.msi;*.bat)",
            "script", "Script Files (*.ahk;*.ps1;*.vbs;*.js)",
            "all", "All Files (*.*)"
        )
    }
    
    static SetupDefaultTemplates() {
        ; Set up dialog templates
        this.dialogTemplates := Map()
        
        this.dialogTemplates["image_import"] := {
            title: "Import Images",
            fileTypes: ["image"],
            multiSelect: true,
            validateFiles: true
        }
        
        this.dialogTemplates["document_export"] := {
            title: "Export Document",
            save: true,
            fileTypes: ["document"],
            confirmOverwrite: true,
            createDirectory: true
        }
        
        this.dialogTemplates["backup_restore"] := {
            title: "Select Backup File",
            fileTypes: ["archive"],
            mustExist: true,
            validationRule: "backup_file"
        }
    }
    
    static LogDialogOperation(dialogRecord) {
        ; Log dialog operation
        this.dialogHistory.Push(dialogRecord)
        
        ; Limit history size
        if (this.dialogHistory.Length > 100) {
            this.dialogHistory.RemoveAt(1)
        }
        
        this.operationHistory.Push({
            type: "dialog",
            timestamp: dialogRecord.timestamp,
            operation: dialogRecord.dialogType,
            success: !dialogRecord.cancelled && !dialogRecord.error,
            duration: dialogRecord.dialogTime
        })
    }
    
    static GenerateDialogId() {
        ; Generate unique dialog ID
        return "dialog_" . A_TickCount . "_" . Random(1000, 9999)
    }
    
    static ValidateDialogSpec(spec) {
        ; Validate dialog specification
        requiredProps := ["title"]
        
        for prop in requiredProps {
            if (!spec.HasProp(prop)) {
                throw ValueError("Dialog specification missing required property: " . prop)
            }
        }
    }
    
    static ResetMetrics() {
        ; Reset dialog metrics
        this.dialogMetrics := {opens: 0, saves: 0, cancellations: 0, errors: 0}
    }
    
    static GetDialogStatistics() {
        ; Get dialog usage statistics
        stats := this.dialogMetrics.Clone()
        
        stats.total := stats.opens + stats.saves
        stats.successRate := stats.total > 0 ? Round(((stats.total - stats.errors) / stats.total) * 100, 1) : 0
        stats.cancellationRate := (stats.total + stats.cancellations) > 0 ? Round((stats.cancellations / (stats.total + stats.cancellations)) * 100, 1) : 0
        
        return stats
    }
    
    static GetRecentDirectories() {
        ; Get list of recent directories
        return this.recentDirectories.Clone()
    }
    
    static ClearRecentDirectories() {
        ; Clear recent directories list
        this.recentDirectories := []
    }
    
    static GetDialogHistory(maxEntries := 20) {
        ; Get recent dialog history
        historyCount := Min(maxEntries, this.dialogHistory.Length)
        recentHistory := []
        
        for i in Range(this.dialogHistory.Length - historyCount + 1, this.dialogHistory.Length) {
            recentHistory.Push(this.dialogHistory[i])
        }
        
        return recentHistory
    }
    
    static GenerateUsageReport() {
        ; Generate comprehensive usage report
        stats := this.GetDialogStatistics()
        
        report := "File Dialog Usage Report`n"
        report .= "==========================`n`n"
        
        report .= "Dialog Statistics:`n"
        report .= "Total Operations: " . stats.total . "`n"
        report .= "Open Dialogs: " . stats.opens . "`n"
        report .= "Save Dialogs: " . stats.saves . "`n"
        report .= "Cancellations: " . stats.cancellations . "`n"
        report .= "Errors: " . stats.errors . "`n"
        report .= "Success Rate: " . stats.successRate . "%`n"
        report .= "Cancellation Rate: " . stats.cancellationRate . "%`n`n"
        
        report .= "Recent Directories (" . this.recentDirectories.Length . "):`n"
        for i, dir in this.recentDirectories {
            report .= i . ". " . dir . "`n"
            if (i >= 5) break  ; Limit display
        }
        
        report .= "`nRegistered Templates: " . this.dialogTemplates.Count . "`n"
        report .= "Custom Actions: " . this.customActions.Count . "`n"
        report .= "Validation Rules: " . this.validationRules.Count . "`n"
        
        return report
    }
}

; Example usage and demonstrations
; Initialize file dialog manager
FileDialogManager.Initialize()

; Basic file selection
selectedFile := FileSelectFile()
if (selectedFile) {
    MsgBox("Selected file: " . selectedFile)
    ; Process the file...
} else {
    MsgBox("No file selected")
}

; Enhanced file selection with validation
imageFile := FileDialogManager.SelectFile({
    title: "Select Image File",
    fileTypes: ["image"],
    mustExist: true,
    useRecentDir: true
})

if (imageFile) {
    MsgBox("Image selected: " . imageFile)
    ; Process image file...
}

; Multiple file selection
selectedFiles := FileDialogManager.SelectMultipleFiles({
    title: "Select Multiple Documents",
    filter: "Documents (*.doc;*.docx;*.pdf)|All Files (*.*)",
    validateFiles: true
})

if (selectedFiles.Length > 0) {
    MsgBox("Selected " . selectedFiles.Length . " files")
    
    ; Process each file
    for file in selectedFiles {
        OutputDebug("Processing: " . file)
    }
} else {
    MsgBox("No files selected")
}

; Save file dialog with validation
saveFile := FileDialogManager.SaveFileDialog({
    title: "Save Report",
    filter: "Excel Files (*.xlsx)|PDF Files (*.pdf)|All Files (*.*)",
    defaultFileName: "report_" . FormatTime(, "yyyy-MM-dd") . ".xlsx",
    confirmOverwrite: true,
    createDirectory: true
})

if (saveFile) {
    MsgBox("Will save to: " . saveFile)
    ; Perform save operation...
}

; Using predefined templates
documentFile := FileDialogManager.SelectWithTemplate("document_export", {
    defaultFileName: "my_document.pdf"
})

if (documentFile) {
    MsgBox("Document export location: " . documentFile)
}

; Custom file filter creation
customFilter := FileDialogManager.CreateFileFilter(Map(
    "AutoHotkey Scripts", ["ahk", "ah2"],
    "Batch Files", "bat",
    "PowerShell Scripts", ["ps1", "psm1"],
    "All Scripts", ["ahk", "ah2", "bat", "ps1", "vbs", "js"]
))

scriptFile := FileSelectFile(, , "Select Script File", customFilter)
if (scriptFile) {
    MsgBox("Script selected: " . scriptFile)
}

; Register custom validation rule
FileDialogManager.RegisterValidationRule("image_size", {
    type: "size",
    maxSize: 10 * 1024 * 1024,  ; 10MB limit
    description: "Image files must be under 10MB"
})

; Register custom action
FileDialogManager.RegisterCustomAction("backup_copy", (filePath, options) => {
    backupPath := filePath . ".backup"
    FileCopy(filePath, backupPath)
    OutputDebug("Created backup: " . backupPath)
}, "Create backup copy of selected file")

; Use custom validation and actions
validatedFile := FileDialogManager.SelectFile({
    title: "Select Image with Validation",
    fileTypes: ["image"],
    validationRule: "image_size",
    customActions: ["backup_copy"],
    throwOnValidationError: true
})

; Batch file operations
filesToProcess := FileDialogManager.SelectMultipleFiles({
    title: "Select Files to Copy",
    validateFiles: true
})

if (filesToProcess.Length > 0) {
    ; Select destination directory
    destDir := FileSelectFile("D", , "Select Destination Folder")
    
    if (destDir) {
        ; Perform batch copy operation
        batchResult := FileDialogManager.BatchFileOperation(filesToProcess, "copy", {
            destination: destDir
        })
        
        MsgBox("Batch operation complete:`n" .
               "Successful: " . batchResult.successCount . "`n" .
               "Errors: " . batchResult.errorCount . "`n" .
               "Total: " . batchResult.totalFiles)
        
        ; Display any errors
        if (batchResult.errors.Length > 0) {
            errorText := "Errors occurred:`n"
            for error in batchResult.errors {
                errorText .= "• " . error.file . ": " . error.error . "`n"
            }
            MsgBox(errorText, "Batch Operation Errors")
        }
    }
}

; Configure security settings
FileDialogManager.SetSecuritySettings({
    allowedExtensions: ["txt", "doc", "docx", "pdf", "jpg", "png"],
    blockedExtensions: ["exe", "bat", "com", "scr"],
    maxFileSize: 50 * 1024 * 1024  ; 50MB limit
})

; Test security restrictions
try {
    secureFile := FileDialogManager.SelectFile({
        title: "Select Secure File",
        mustExist: true,
        throwOnValidationError: true
    })
    
    if (secureFile) {
        MsgBox("Secure file selected: " . secureFile)
    }
    
} catch SecurityError as err {
    MsgBox("Security validation failed: " . err.Message)
} catch ValidationError as err {
    MsgBox("File validation failed: " . err.Message)
}

; Advanced dialog with all options
advancedFile := FileDialogManager.SelectFile({
    title: "Advanced File Selection",
    options: "M",  ; Multi-select
    rootDir: A_MyDocuments,
    filter: FileDialogManager.CreateFileFilter(Map(
        "Office Documents", ["doc", "docx", "xls", "xlsx", "ppt", "pptx"],
        "Text Files", ["txt", "rtf", "md"],
        "All Files", "*"
    )),
    mustExist: true,
    useRecentDir: false,
    validateFiles: true,
    customActions: ["backup_copy"],
    validationRule: "image_size"
})

; Dialog usage statistics and reporting
stats := FileDialogManager.GetDialogStatistics()
MsgBox("Dialog Usage Statistics:`n" .
       "Total Operations: " . stats.total . "`n" .
       "Success Rate: " . stats.successRate . "%`n" .
       "Cancellation Rate: " . stats.cancellationRate . "%")

; Get recent directories
recentDirs := FileDialogManager.GetRecentDirectories()
if (recentDirs.Length > 0) {
    dirList := "Recent Directories:`n"
    for i, dir in recentDirs {
        dirList .= i . ". " . dir . "`n"
        if (i >= 5) break
    }
    ; MsgBox(dirList)
}

; Generate comprehensive usage report
usageReport := FileDialogManager.GenerateUsageReport()
; MsgBox(usageReport, "File Dialog Usage Report")

; Dialog history analysis
dialogHistory := FileDialogManager.GetDialogHistory(10)
if (dialogHistory.Length > 0) {
    historyText := "Recent Dialog Operations:`n"
    for record in dialogHistory {
        status := record.cancelled ? "Cancelled" : (record.error ? "Error" : "Success")
        historyText .= "• " . record.formattedTime . " - " . record.title . " (" . status . ")`n"
    }
    ; MsgBox(historyText, "Dialog History")
}

; Cleanup demonstration
MsgBox("File dialog demonstration complete.`n" .
       "Operations performed: " . FileDialogManager.GetDialogStatistics().total . "`n" .
       "Recent directories tracked: " . FileDialogManager.GetRecentDirectories().Length)

; Clear recent directories if desired
; FileDialogManager.ClearRecentDirectories()
```

## Implementation Notes

**User Experience and Interface Consistency:**
- FileSelectFile displays the standard Windows file dialog, ensuring familiar user experience and accessibility compliance
- Dialog behavior should match user expectations from other Windows applications including keyboard shortcuts and navigation patterns
- Consider providing appropriate default directories and recently used locations to improve workflow efficiency
- Implement proper error handling and user feedback for invalid selections or system errors

**File Path and Unicode Handling:**
- File paths returned by FileSelectFile are always full absolute paths using Windows path format with backslashes
- Handle Unicode file names and paths properly, especially in international environments or with special characters
- Be aware of maximum path length limitations (260 characters in traditional Windows, longer with extended paths)
- Validate and sanitize file paths when using them in subsequent operations to prevent security issues

**Multi-Selection and Result Processing:**
- Multi-select mode returns a newline-separated string of file paths that must be parsed appropriately
- Handle edge cases where selected files may become unavailable between selection and processing
- Implement proper validation for each file in multi-select scenarios, including size and accessibility checks
- Consider memory usage when processing large numbers of selected files simultaneously

**Dialog Customization and Filtering:**
- File filters should be clear and specific to guide users toward appropriate file types
- Support both simple wildcard patterns and complex multi-extension filters for flexible file type specification
- Test filter specifications across different Windows versions to ensure consistent behavior
- Provide meaningful filter descriptions that clearly indicate supported file types

**Security and Validation Considerations:**
- Implement appropriate file type validation to prevent processing of unexpected or potentially dangerous files
- Consider file size limitations and accessibility checks before attempting to process selected files
- Validate that selected file paths are within expected directories and don't contain path traversal attempts
- Handle cases where users might select files they don't have permission to read or modify

## Related AHK Concepts

- [DirSelect](./dirselect.md) - Directory selection dialogs for folder operations
- [FileExist](../../30_Built_In_Classes/02-File_IO_Classes/File/fileexist.md) - Validating selected file availability
- [SplitPath](../../10_Language_Core/01-Functions/Built_In_Functions/splitpath.md) - Parsing file paths and extracting components
- [FileRead](../../30_Built_In_Classes/02-File_IO_Classes/File/fileread.md) - Reading selected file contents
- [FileCopy](../../30_Built_In_Classes/02-File_IO_Classes/File/filecopy.md) - Processing selected files

## Tags

#AutoHotkey #FileSelectFile #FileDialog #UserInterface #FileSelection #DialogManagement #FileOperations #UserExperience #FileValidation #FileManagement
