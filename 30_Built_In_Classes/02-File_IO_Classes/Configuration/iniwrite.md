# Topic: IniWrite Function - Configuration File Writing and Settings Management

## Category

Function

## Overview

The IniWrite function writes values to INI configuration files, providing essential configuration persistence capabilities for applications requiring durable settings storage, user preferences management, and configuration state preservation. It's crucial for application settings persistence, user customization storage, deployment configuration, and any system requiring reliable structured data writing and configuration management.

## Key Points

- Writes configuration values to INI files with automatic file creation and section management
- Supports hierarchical configuration writing with sections, keys, and nested configuration structures
- Provides foundation for settings persistence, user preference storage, and application state management
- Enables sophisticated configuration writing frameworks including atomic operations, backup management, and validation
- Essential for application configuration persistence, user customization, deployment automation, and multi-environment configuration management

## Syntax and Parameters

```cpp
IniWrite(Value, Filename, Section, Key)

; Parameters:
; Value    - Value to write (converted to string)
; Filename - Path to INI file (relative or absolute)
; Section  - Section name within the INI file
; Key      - Key name within the specified section

; Value handling:
; All values converted to strings for storage
; Boolean true/false converted to string representation
; Numbers converted to string format
; Empty values create empty key entries

; File behavior:
; Creates file automatically if it doesn't exist
; Creates sections automatically if they don't exist
; Creates intermediate directories if needed
; Overwrites existing key values
; Preserves other sections and keys

; INI structure created:
; [SectionName]
; KeyName=Value
; AnotherKey=AnotherValue
;
; [AnotherSection]
; Setting1=Value1
; Setting2=Value2

; Error handling:
; Throws exception on file access errors
; Throws exception on permission denied
; Throws exception on disk space issues
; Throws exception on invalid file paths

; Return value:
; No return value on success
; Throws exception on error
```

## Code Examples

```cpp
; Basic INI writing
IniWrite("JohnDoe", "config.ini", "User", "Username")
IniWrite(30, "app.ini", "Network", "Timeout")
IniWrite("Dark", A_ScriptDir . "\settings.ini", "UI", "Theme")

; Advanced configuration writing and management system
class ConfigWriter {
    static pendingWrites := Map()
    static writeQueue := []
    static batchMode := false
    static batchSize := 50
    static backupEnabled := true
    static validationEnabled := true
    static atomicWrites := true
    static writeStats := Map()
    static writeValidators := Map()
    
    static Initialize() {
        this.pendingWrites.Clear()
        this.writeQueue := []
        this.writeStats.Clear()
        this.SetupDefaultValidators()
    }
    
    static WriteValue(filename, section, key, value, options := {}) {
        ; Advanced configuration writing with comprehensive options
        validateWrite := options.HasProp("validate") ? options.validate : this.validationEnabled
        createBackup := options.HasProp("backup") ? options.backup : this.backupEnabled
        useAtomic := options.HasProp("atomic") ? options.atomic : this.atomicWrites
        immediate := options.HasProp("immediate") ? options.immediate : !this.batchMode
        
        ; Create write operation
        writeOp := {
            filename: filename,
            section: section,
            key: key,
            value: value,
            options: options,
            timestamp: A_TickCount,
            validated: false,
            written: false,
            error: ""
        }
        
        try {
            ; Validate write operation
            if (validateWrite) {
                this.ValidateWrite(writeOp)
                writeOp.validated := true
            }
            
            ; Execute immediately or queue for batch
            if (immediate) {
                this.ExecuteWrite(writeOp, createBackup, useAtomic)
            } else {
                this.QueueWrite(writeOp)
            }
            
            ; Update statistics
            this.UpdateWriteStatistics(filename, section, key, true)
            
            return writeOp
            
        } catch Error as err {
            writeOp.error := err.Message
            this.UpdateWriteStatistics(filename, section, key, false)
            throw err
        }
    }
    
    static ExecuteWrite(writeOp, createBackup := true, useAtomic := true) {
        ; Execute individual write operation
        filename := writeOp.filename
        
        try {
            ; Create backup if enabled
            if (createBackup && FileExist(filename)) {
                this.CreateBackup(filename)
            }
            
            ; Use atomic write if enabled
            if (useAtomic) {
                this.AtomicWrite(writeOp)
            } else {
                this.DirectWrite(writeOp)
            }
            
            writeOp.written := true
            
        } catch Error as err {
            writeOp.error := err.Message
            throw err
        }
    }
    
    static AtomicWrite(writeOp) {
        ; Atomic write operation using temporary file
        tempFile := writeOp.filename . ".tmp"
        
        try {
            ; If original file exists, copy to temp file first
            if (FileExist(writeOp.filename)) {
                FileCopy(writeOp.filename, tempFile, 1)
            }
            
            ; Write to temp file
            IniWrite(writeOp.value, tempFile, writeOp.section, writeOp.key)
            
            ; Verify temp file
            if (this.validationEnabled) {
                verifyValue := IniRead(tempFile, writeOp.section, writeOp.key, "##NOTFOUND##")
                if (verifyValue = "##NOTFOUND##" || verifyValue != String(writeOp.value)) {
                    throw Error("Write verification failed")
                }
            }
            
            ; Atomic move temp file to target
            FileMove(tempFile, writeOp.filename, 1)
            
        } catch Error as err {
            ; Cleanup temp file on error
            if (FileExist(tempFile)) {
                try {
                    FileDelete(tempFile)
                } catch {
                    ; Ignore cleanup errors
                }
            }
            throw err
        }
    }
    
    static DirectWrite(writeOp) {
        ; Direct write operation
        IniWrite(writeOp.value, writeOp.filename, writeOp.section, writeOp.key)
    }
    
    static QueueWrite(writeOp) {
        ; Add write operation to batch queue
        this.writeQueue.Push(writeOp)
        
        ; Store pending write
        writeKey := writeOp.filename . ":" . writeOp.section . ":" . writeOp.key
        this.pendingWrites[writeKey] := writeOp
        
        ; Process batch if queue is full
        if (this.writeQueue.Length >= this.batchSize) {
            this.ProcessBatch()
        }
    }
    
    static ProcessBatch() {
        ; Process queued write operations
        if (this.writeQueue.Length = 0) return
        
        ; Group writes by file
        fileGroups := Map()
        
        for writeOp in this.writeQueue {
            filename := writeOp.filename
            if (!fileGroups.Has(filename)) {
                fileGroups[filename] := []
            }
            fileGroups[filename].Push(writeOp)
        }
        
        ; Process each file group
        for filename, writes in fileGroups {
            try {
                this.ProcessFileGroup(filename, writes)
            } catch Error as err {
                ; Mark all writes in group as failed
                for writeOp in writes {
                    writeOp.error := err.Message
                }
            }
        }
        
        ; Clear processed operations
        this.writeQueue := []
        this.pendingWrites.Clear()
    }
    
    static ProcessFileGroup(filename, writes) {
        ; Process all writes for a single file
        
        ; Create backup if enabled
        if (this.backupEnabled && FileExist(filename)) {
            this.CreateBackup(filename)
        }
        
        ; Use atomic batch write if enabled
        if (this.atomicWrites) {
            this.AtomicBatchWrite(filename, writes)
        } else {
            this.DirectBatchWrite(filename, writes)
        }
        
        ; Mark all writes as completed
        for writeOp in writes {
            writeOp.written := true
        }
    }
    
    static AtomicBatchWrite(filename, writes) {
        ; Atomic batch write using temporary file
        tempFile := filename . ".tmp"
        
        try {
            ; Copy existing file to temp if it exists
            if (FileExist(filename)) {
                FileCopy(filename, tempFile, 1)
            }
            
            ; Apply all writes to temp file
            for writeOp in writes {
                IniWrite(writeOp.value, tempFile, writeOp.section, writeOp.key)
            }
            
            ; Verify all writes if validation enabled
            if (this.validationEnabled) {
                for writeOp in writes {
                    verifyValue := IniRead(tempFile, writeOp.section, writeOp.key, "##NOTFOUND##")
                    if (verifyValue = "##NOTFOUND##" || verifyValue != String(writeOp.value)) {
                        throw Error("Batch write verification failed for " . writeOp.key)
                    }
                }
            }
            
            ; Atomic move
            FileMove(tempFile, filename, 1)
            
        } catch Error as err {
            ; Cleanup temp file
            if (FileExist(tempFile)) {
                try {
                    FileDelete(tempFile)
                } catch {
                    ; Ignore cleanup errors
                }
            }
            throw err
        }
    }
    
    static DirectBatchWrite(filename, writes) {
        ; Direct batch write
        for writeOp in writes {
            IniWrite(writeOp.value, writeOp.filename, writeOp.section, writeOp.key)
        }
    }
    
    static WriteSection(filename, section, values, options := {}) {
        ; Write entire section with multiple key-value pairs
        replaceSection := options.HasProp("replace") ? options.replace : false
        
        ; Clear section if replace mode
        if (replaceSection) {
            this.ClearSection(filename, section)
        }
        
        ; Write all values
        writeOps := []
        for key, value in values {
            writeOp := this.WriteValue(filename, section, key, value, options)
            writeOps.Push(writeOp)
        }
        
        return writeOps
    }
    
    static ClearSection(filename, section) {
        ; Clear all keys in a section (simplified implementation)
        ; Real implementation would read section, then delete each key
        try {
            ; This is a placeholder - actual implementation would be more complex
            ; IniDelete(filename, section)  ; If such function existed
        } catch Error {
            ; Continue if section doesn't exist
        }
    }
    
    static CreateBackup(filename) {
        ; Create timestamped backup of configuration file
        if (!FileExist(filename)) return
        
        timestamp := FormatTime(, "yyyyMMdd_HHmmss")
        backupPath := filename . ".backup." . timestamp
        
        try {
            FileCopy(filename, backupPath, 0)  ; Don't overwrite existing backup
            
            ; Cleanup old backups (keep last 10)
            this.CleanupBackups(filename)
            
            return backupPath
            
        } catch Error as err {
            throw Error("Failed to create backup: " . err.Message)
        }
    }
    
    static CleanupBackups(filename) {
        ; Clean up old backup files
        baseDir := RegExReplace(filename, "[^\\]+$", "")
        baseName := RegExReplace(filename, "^.*\\", "")
        
        backups := []
        
        Loop Files, baseDir . baseName . ".backup.*" {
            backups.Push({
                path: A_LoopFileFullPath,
                time: A_LoopFileTimeModified
            })
        }
        
        ; Sort by time (newest first)
        backups.Sort((a, b) => b.time - a.time)
        
        ; Delete old backups (keep 10 most recent)
        for i in Range(11, backups.Length) {
            try {
                FileDelete(backups[i].path)
            } catch {
                ; Continue with other deletions
            }
        }
    }
    
    static ValidateWrite(writeOp) {
        ; Validate write operation before execution
        filename := writeOp.filename
        section := writeOp.section
        key := writeOp.key
        value := writeOp.value
        
        ; Check filename validity
        if (!filename || filename = "") {
            throw ValueError("Invalid filename")
        }
        
        ; Check section validity
        if (!section || section = "") {
            throw ValueError("Invalid section name")
        }
        
        ; Check key validity
        if (!key || key = "") {
            throw ValueError("Invalid key name")
        }
        
        ; Check for invalid characters
        invalidChars := ["[", "]", "=", "`n", "`r"]
        for char in invalidChars {
            if (InStr(section, char) || InStr(key, char)) {
                throw ValueError("Invalid characters in section or key")
            }
        }
        
        ; Apply custom validators
        validatorKey := section . "." . key
        if (this.writeValidators.Has(validatorKey)) {
            validator := this.writeValidators[validatorKey]
            if (!this.ApplyValidator(value, validator)) {
                throw ValueError("Value validation failed for " . validatorKey)
            }
        }
        
        ; Check file permissions (simplified)
        if (FileExist(filename)) {
            ; Check if file is writable
            testValue := IniRead(filename, section, key, "##TEST##")
            ; In a real implementation, would check file attributes
        }
    }
    
    static SetupDefaultValidators() {
        ; Set up default validation rules
        this.writeValidators := Map()
        
        ; Add common validators
        this.AddValidator("Network.Port", {type: "integer", min: 1, max: 65535})
        this.AddValidator("Network.Timeout", {type: "integer", min: 1, max: 3600})
        this.AddValidator("UI.Theme", {type: "enum", values: ["Light", "Dark", "Auto"]})
        this.AddValidator("Logging.Level", {type: "enum", values: ["DEBUG", "INFO", "WARN", "ERROR"]})
    }
    
    static AddValidator(path, validator) {
        ; Add custom validator for specific configuration path
        this.writeValidators[path] := validator
    }
    
    static ApplyValidator(value, validator) {
        ; Apply validation rule to value
        switch validator.type {
            case "integer":
                if (!IsInteger(value)) return false
                intValue := Integer(value)
                if (validator.HasProp("min") && intValue < validator.min) return false
                if (validator.HasProp("max") && intValue > validator.max) return false
                
            case "float":
                if (!IsFloat(value) && !IsInteger(value)) return false
                floatValue := Float(value)
                if (validator.HasProp("min") && floatValue < validator.min) return false
                if (validator.HasProp("max") && floatValue > validator.max) return false
                
            case "string":
                strValue := String(value)
                if (validator.HasProp("minLength") && StrLen(strValue) < validator.minLength) return false
                if (validator.HasProp("maxLength") && StrLen(strValue) > validator.maxLength) return false
                if (validator.HasProp("pattern") && !RegExMatch(strValue, validator.pattern)) return false
                
            case "enum":
                if (validator.HasProp("values") && !validator.values.Find(String(value))) return false
                
            case "boolean":
                if (value != "true" && value != "false" && value != true && value != false) return false
                
            default:
                return true
        }
        
        return true
    }
    
    static UpdateWriteStatistics(filename, section, key, success) {
        ; Update write operation statistics
        statsKey := filename . ":" . section . ":" . key
        
        if (!this.writeStats.Has(statsKey)) {
            this.writeStats[statsKey] := {
                totalWrites: 0,
                successfulWrites: 0,
                failedWrites: 0,
                lastWrite: 0,
                lastSuccess: 0,
                lastFailure: 0
            }
        }
        
        stats := this.writeStats[statsKey]
        stats.totalWrites++
        stats.lastWrite := A_TickCount
        
        if (success) {
            stats.successfulWrites++
            stats.lastSuccess := A_TickCount
        } else {
            stats.failedWrites++
            stats.lastFailure := A_TickCount
        }
    }
    
    static EnableBatchMode() {
        this.batchMode := true
    }
    
    static DisableBatchMode() {
        this.batchMode := false
        ; Process any pending writes
        this.ProcessBatch()
    }
    
    static FlushPendingWrites() {
        ; Force processing of all pending writes
        this.ProcessBatch()
    }
    
    static GetWriteStatistics() {
        return this.writeStats.Clone()
    }
    
    static GenerateWriteReport() {
        ; Generate comprehensive write statistics report
        report := "Configuration Write Report`n"
        report .= "=========================`n`n"
        
        totalOps := 0
        totalSuccess := 0
        totalFailures := 0
        
        for statsKey, stats in this.writeStats {
            totalOps += stats.totalWrites
            totalSuccess += stats.successfulWrites
            totalFailures += stats.failedWrites
        }
        
        report .= "Summary:`n"
        report .= "Total Operations: " . totalOps . "`n"
        report .= "Successful: " . totalSuccess . "`n"
        report .= "Failed: " . totalFailures . "`n"
        
        if (totalOps > 0) {
            successRate := Round((totalSuccess / totalOps) * 100, 1)
            report .= "Success Rate: " . successRate . "%`n"
        }
        
        report .= "`nBatch Mode: " . (this.batchMode ? "Enabled" : "Disabled") . "`n"
        report .= "Pending Writes: " . this.writeQueue.Length . "`n`n"
        
        ; Detailed statistics
        report .= "Detailed Statistics:`n"
        report .= "Key`tTotal`tSuccess`tFailed`tLast Write`n"
        
        for statsKey, stats in this.writeStats {
            lastWriteTime := stats.lastWrite > 0 ? FormatTime(DateAdd(A_Now, stats.lastWrite - A_TickCount, "Seconds")) : "Never"
            report .= statsKey . "`t" . stats.totalWrites . "`t" . stats.successfulWrites . "`t" . stats.failedWrites . "`t" . lastWriteTime . "`n"
        }
        
        return report
    }
}

; User settings persistence system
class UserSettings {
    static settingsFile := A_ScriptDir . "\user_settings.ini"
    static cache := Map()
    static autoSave := true
    static saveDelay := 2000
    static pendingSave := false
    
    static Initialize() {
        this.cache.Clear()
        this.LoadSettings()
        this.StartAutoSave()
    }
    
    static LoadSettings() {
        ; Load all settings into cache
        if (!FileExist(this.settingsFile)) {
            return
        }
        
        ; Load common setting categories
        categories := ["UI", "Behavior", "Performance", "Advanced"]
        
        for category in categories {
            this.cache[category] := Map()
            
            ; Load known settings for each category
            switch category {
                case "UI":
                    this.cache[category]["Theme"] := IniRead(this.settingsFile, category, "Theme", "Light")
                    this.cache[category]["Language"] := IniRead(this.settingsFile, category, "Language", "English")
                    this.cache[category]["WindowWidth"] := Integer(IniRead(this.settingsFile, category, "WindowWidth", "800"))
                    this.cache[category]["WindowHeight"] := Integer(IniRead(this.settingsFile, category, "WindowHeight", "600"))
                    
                case "Behavior":
                    this.cache[category]["AutoSave"] := IniRead(this.settingsFile, category, "AutoSave", "true") = "true"
                    this.cache[category]["ShowTips"] := IniRead(this.settingsFile, category, "ShowTips", "true") = "true"
                    this.cache[category]["ConfirmExit"] := IniRead(this.settingsFile, category, "ConfirmExit", "true") = "true"
                    
                case "Performance":
                    this.cache[category]["CacheSize"] := Integer(IniRead(this.settingsFile, category, "CacheSize", "100"))
                    this.cache[category]["MaxThreads"] := Integer(IniRead(this.settingsFile, category, "MaxThreads", "4"))
                    
                case "Advanced":
                    this.cache[category]["DebugMode"] := IniRead(this.settingsFile, category, "DebugMode", "false") = "true"
                    this.cache[category]["LogLevel"] := IniRead(this.settingsFile, category, "LogLevel", "INFO")
            }
        }
    }
    
    static GetSetting(category, key, defaultValue := "") {
        ; Get setting with caching
        if (!this.cache.Has(category)) {
            this.cache[category] := Map()
        }
        
        if (this.cache[category].Has(key)) {
            return this.cache[category][key]
        }
        
        ; Load from file if not cached
        value := IniRead(this.settingsFile, category, key, defaultValue)
        
        ; Convert to appropriate type
        if (value = "true" || value = "false") {
            value := value = "true"
        } else if (IsInteger(value)) {
            value := Integer(value)
        } else if (IsFloat(value)) {
            value := Float(value)
        }
        
        this.cache[category][key] := value
        return value
    }
    
    static SetSetting(category, key, value) {
        ; Set setting and mark for save
        if (!this.cache.Has(category)) {
            this.cache[category] := Map()
        }
        
        this.cache[category][key] := value
        this.MarkForSave()
    }
    
    static MarkForSave() {
        ; Mark settings for delayed save
        if (!this.autoSave) return
        
        this.pendingSave := true
        
        ; Set timer for delayed save
        SetTimer(() => this.SaveSettings(), -this.saveDelay)
    }
    
    static SaveSettings() {
        ; Save all cached settings to file
        if (!this.pendingSave) return
        
        try {
            ; Use ConfigWriter for atomic saves
            ConfigWriter.EnableBatchMode()
            
            for category, settings in this.cache {
                for key, value in settings {
                    ; Convert value to string
                    strValue := String(value)
                    ConfigWriter.WriteValue(this.settingsFile, category, key, strValue, {
                        immediate: false,
                        backup: true,
                        validate: true
                    })
                }
            }
            
            ; Flush all writes
            ConfigWriter.FlushPendingWrites()
            ConfigWriter.DisableBatchMode()
            
            this.pendingSave := false
            
        } catch Error as err {
            throw Error("Failed to save settings: " . err.Message)
        }
    }
    
    static ForceSave() {
        ; Force immediate save
        this.pendingSave := true
        this.SaveSettings()
    }
    
    static ResetCategory(category) {
        ; Reset category to defaults
        if (this.cache.Has(category)) {
            this.cache.Delete(category)
        }
        
        ; Reload defaults
        this.LoadSettings()
        this.MarkForSave()
    }
    
    static StartAutoSave() {
        ; Start auto-save monitoring
        if (this.autoSave) {
            SetTimer(() => {
                if (this.pendingSave) {
                    this.SaveSettings()
                }
            }, this.saveDelay)
        }
    }
}

; Application state management
class AppStateManager {
    static stateFile := A_ScriptDir . "\app_state.ini"
    static state := Map()
    static saveOnChange := true
    static stateVersion := "1.0"
    
    static Initialize() {
        this.LoadState()
    }
    
    static LoadState() {
        ; Load application state
        this.state.Clear()
        
        if (!FileExist(this.stateFile)) {
            this.InitializeDefaultState()
            return
        }
        
        try {
            ; Load state version
            version := IniRead(this.stateFile, "System", "Version", "1.0")
            
            ; Load window state
            this.state["WindowX"] := Integer(IniRead(this.stateFile, "Window", "X", "100"))
            this.state["WindowY"] := Integer(IniRead(this.stateFile, "Window", "Y", "100"))
            this.state["WindowWidth"] := Integer(IniRead(this.stateFile, "Window", "Width", "800"))
            this.state["WindowHeight"] := Integer(IniRead(this.stateFile, "Window", "Height", "600"))
            this.state["WindowMaximized"] := IniRead(this.stateFile, "Window", "Maximized", "false") = "true"
            
            ; Load session state
            this.state["LastSession"] := IniRead(this.stateFile, "Session", "LastSession", "")
            this.state["SessionCount"] := Integer(IniRead(this.stateFile, "Session", "Count", "0"))
            this.state["LastFile"] := IniRead(this.stateFile, "Session", "LastFile", "")
            
            ; Load recent files
            recentFiles := []
            Loop 10 {
                file := IniRead(this.stateFile, "RecentFiles", "File" . A_Index, "")
                if (file) {
                    recentFiles.Push(file)
                }
            }
            this.state["RecentFiles"] := recentFiles
            
        } catch Error {
            this.InitializeDefaultState()
        }
    }
    
    static InitializeDefaultState() {
        ; Initialize default application state
        this.state := Map(
            "WindowX", 100,
            "WindowY", 100,
            "WindowWidth", 800,
            "WindowHeight", 600,
            "WindowMaximized", false,
            "LastSession", "",
            "SessionCount", 0,
            "LastFile", "",
            "RecentFiles", []
        )
    }
    
    static GetState(key, defaultValue := "") {
        return this.state.Has(key) ? this.state[key] : defaultValue
    }
    
    static SetState(key, value) {
        this.state[key] := value
        
        if (this.saveOnChange) {
            this.SaveState()
        }
    }
    
    static SaveState() {
        ; Save application state
        try {
            ; Save system info
            ConfigWriter.WriteValue(this.stateFile, "System", "Version", this.stateVersion)
            ConfigWriter.WriteValue(this.stateFile, "System", "SaveTime", FormatTime())
            
            ; Save window state
            ConfigWriter.WriteValue(this.stateFile, "Window", "X", this.state["WindowX"])
            ConfigWriter.WriteValue(this.stateFile, "Window", "Y", this.state["WindowY"])
            ConfigWriter.WriteValue(this.stateFile, "Window", "Width", this.state["WindowWidth"])
            ConfigWriter.WriteValue(this.stateFile, "Window", "Height", this.state["WindowHeight"])
            ConfigWriter.WriteValue(this.stateFile, "Window", "Maximized", this.state["WindowMaximized"])
            
            ; Save session state
            ConfigWriter.WriteValue(this.stateFile, "Session", "LastSession", this.state["LastSession"])
            ConfigWriter.WriteValue(this.stateFile, "Session", "Count", this.state["SessionCount"])
            ConfigWriter.WriteValue(this.stateFile, "Session", "LastFile", this.state["LastFile"])
            
            ; Save recent files
            recentFiles := this.state["RecentFiles"]
            for i, file in recentFiles {
                if (i <= 10) {  ; Limit to 10 recent files
                    ConfigWriter.WriteValue(this.stateFile, "RecentFiles", "File" . i, file)
                }
            }
            
        } catch Error as err {
            throw Error("Failed to save application state: " . err.Message)
        }
    }
    
    static AddRecentFile(filePath) {
        ; Add file to recent files list
        recentFiles := this.state["RecentFiles"]
        
        ; Remove if already exists
        for i, file in recentFiles {
            if (file = filePath) {
                recentFiles.RemoveAt(i)
                break
            }
        }
        
        ; Add to beginning
        recentFiles.InsertAt(1, filePath)
        
        ; Limit to 10 files
        while (recentFiles.Length > 10) {
            recentFiles.Pop()
        }
        
        this.state["RecentFiles"] := recentFiles
        
        if (this.saveOnChange) {
            this.SaveState()
        }
    }
    
    static ClearRecentFiles() {
        ; Clear recent files list
        this.state["RecentFiles"] := []
        
        if (this.saveOnChange) {
            this.SaveState()
        }
    }
}

; Example usage and demonstrations
; Initialize configuration writing system
ConfigWriter.Initialize()

; Basic INI writing examples
IniWrite("Administrator", "config.ini", "User", "Role")
IniWrite(443, "network.ini", "SSL", "Port")
IniWrite("true", "app.ini", "Features", "AutoUpdate")

; Advanced configuration writing
; Enable batch mode for multiple writes
ConfigWriter.EnableBatchMode()

; Queue multiple writes
ConfigWriter.WriteValue("app_config.ini", "Database", "Host", "localhost")
ConfigWriter.WriteValue("app_config.ini", "Database", "Port", 5432)
ConfigWriter.WriteValue("app_config.ini", "Database", "Username", "app_user")
ConfigWriter.WriteValue("app_config.ini", "API", "BaseURL", "https://api.example.com")
ConfigWriter.WriteValue("app_config.ini", "API", "Timeout", 30)

; Process all queued writes
ConfigWriter.FlushPendingWrites()
ConfigWriter.DisableBatchMode()

; Write entire section at once
uiSettings := Map(
    "Theme", "Dark",
    "Language", "English",
    "WindowWidth", 1024,
    "WindowHeight", 768,
    "ShowToolbar", "true"
)

ConfigWriter.WriteSection("ui_config.ini", "Interface", uiSettings, {
    replace: true,
    backup: true,
    validate: true
})

; User settings management
UserSettings.Initialize()

; Set user preferences
UserSettings.SetSetting("UI", "Theme", "Dark")
UserSettings.SetSetting("UI", "WindowWidth", 1200)
UserSettings.SetSetting("Behavior", "AutoSave", true)
UserSettings.SetSetting("Performance", "CacheSize", 200)

; Force immediate save
UserSettings.ForceSave()

; Application state management
AppStateManager.Initialize()

; Update application state
AppStateManager.SetState("WindowX", 150)
AppStateManager.SetState("WindowY", 200)
AppStateManager.SetState("SessionCount", AppStateManager.GetState("SessionCount") + 1)

; Add recent file
AppStateManager.AddRecentFile("C:\\Documents\\important.txt")
AppStateManager.AddRecentFile("C:\\Projects\\myproject.ahk")

; Complex configuration example with validation
ConfigWriter.AddValidator("Database.Port", {type: "integer", min: 1, max: 65535})
ConfigWriter.AddValidator("API.Timeout", {type: "integer", min: 5, max: 300})
ConfigWriter.AddValidator("UI.Theme", {type: "enum", values: ["Light", "Dark", "Auto"]})

; Write validated configuration
try {
    ConfigWriter.WriteValue("validated_config.ini", "Database", "Port", 5432, {
        validate: true,
        backup: true,
        atomic: true
    })
    
    ConfigWriter.WriteValue("validated_config.ini", "UI", "Theme", "Dark", {
        validate: true
    })
    
    MsgBox("Configuration saved successfully with validation")
    
} catch Error as err {
    MsgBox("Configuration validation failed: " . err.Message)
}

; Atomic configuration update example
AtomicConfigUpdate() {
    ; Perform atomic update of multiple related settings
    ConfigWriter.EnableBatchMode()
    
    try {
        ; Database migration - update all related settings atomically
        ConfigWriter.WriteValue("app.ini", "Database", "Host", "new-db-server.example.com")
        ConfigWriter.WriteValue("app.ini", "Database", "Port", 5433)
        ConfigWriter.WriteValue("app.ini", "Database", "SSL", "true")
        ConfigWriter.WriteValue("app.ini", "Database", "Version", "2.0")
        ConfigWriter.WriteValue("app.ini", "System", "LastMigration", FormatTime())
        
        ; Commit all changes atomically
        ConfigWriter.FlushPendingWrites()
        
        MsgBox("Database configuration updated successfully")
        
    } catch Error as err {
        MsgBox("Failed to update database configuration: " . err.Message)
    } finally {
        ConfigWriter.DisableBatchMode()
    }
}

; Set up hotkey for atomic update
Hotkey("Ctrl+Alt+U", AtomicConfigUpdate)

; Configuration backup and recovery example
CreateConfigurationBackup() {
    configFiles := ["app.ini", "user_settings.ini", "app_state.ini"]
    
    for file in configFiles {
        if (FileExist(file)) {
            try {
                backup := ConfigWriter.CreateBackup(file)
                OutputDebug("Created backup: " . backup)
            } catch Error as err {
                OutputDebug("Failed to backup " . file . ": " . err.Message)
            }
        }
    }
    
    MsgBox("Configuration backup completed")
}

; Set up backup hotkey
Hotkey("Ctrl+Alt+B", CreateConfigurationBackup)

; Generate configuration reports
writeReport := ConfigWriter.GenerateWriteReport()
; MsgBox(writeReport, "Configuration Write Report")

; Configuration file watching for external changes
MonitorConfigurationFiles() {
    ; Check if configuration files have been modified externally
    configFiles := Map(
        "app.ini", 0,
        "user_settings.ini", 0,
        "app_state.ini", 0
    )
    
    ; Store last modification times
    for file in configFiles {
        if (FileExist(file)) {
            configFiles[file] := FileGetTime(file, "M")
        }
    }
    
    ; Check for changes every 5 seconds
    SetTimer(() => {
        for file, lastTime in configFiles {
            if (FileExist(file)) {
                currentTime := FileGetTime(file, "M")
                if (currentTime > lastTime) {
                    OutputDebug("Configuration file changed externally: " . file)
                    configFiles[file] := currentTime
                    
                    ; Reload configuration or notify user
                    ToolTip("Configuration file updated: " . file, 10, 10)
                    SetTimer(() => ToolTip(), -3000)
                }
            }
        }
    }, 5000)
}

; Start configuration monitoring
MonitorConfigurationFiles()

; Environment-specific configuration writing
WriteEnvironmentConfig(environment) {
    envFile := "config." . environment . ".ini"
    
    switch environment {
        case "development":
            ConfigWriter.WriteValue(envFile, "Database", "Host", "localhost")
            ConfigWriter.WriteValue(envFile, "Database", "Port", 5432)
            ConfigWriter.WriteValue(envFile, "Logging", "Level", "DEBUG")
            ConfigWriter.WriteValue(envFile, "Features", "DebugMode", "true")
            
        case "staging":
            ConfigWriter.WriteValue(envFile, "Database", "Host", "staging-db.example.com")
            ConfigWriter.WriteValue(envFile, "Database", "Port", 5432)
            ConfigWriter.WriteValue(envFile, "Logging", "Level", "INFO")
            ConfigWriter.WriteValue(envFile, "Features", "DebugMode", "false")
            
        case "production":
            ConfigWriter.WriteValue(envFile, "Database", "Host", "prod-db.example.com")
            ConfigWriter.WriteValue(envFile, "Database", "Port", 5432)
            ConfigWriter.WriteValue(envFile, "Logging", "Level", "WARN")
            ConfigWriter.WriteValue(envFile, "Features", "DebugMode", "false")
    }
    
    MsgBox("Environment configuration created: " . envFile)
}

; Create configurations for all environments
for env in ["development", "staging", "production"] {
    WriteEnvironmentConfig(env)
}
```

## Implementation Notes

**File System and Performance:**
- INI file writing is synchronous and can block for large operations or slow storage
- Use atomic write operations to prevent file corruption during power failures or interruptions
- Implement batch writing for multiple related configuration changes to improve performance
- Consider file locking mechanisms for configuration files accessed by multiple processes simultaneously

**Data Integrity and Reliability:**
- Always create backups before modifying existing configuration files in production environments
- Implement write verification to ensure data was written correctly, especially for critical settings
- Use atomic operations to prevent partial writes that could corrupt configuration state
- Provide rollback mechanisms for configuration changes that cause application failures

**Validation and Error Handling:**
- Validate configuration values before writing to prevent invalid data persistence
- Implement comprehensive error handling for disk space, permissions, and file access issues
- Provide meaningful error messages that guide users toward resolution of configuration problems
- Consider schema validation for complex configuration structures requiring type safety

**Security and Access Control:**
- Be aware that INI files store data in plain text format, visible to anyone with file access
- Implement appropriate file permissions for configuration files containing sensitive information
- Consider encryption for sensitive configuration values like passwords or API keys
- Validate configuration sources to prevent injection of malicious configuration data

**Backup and Recovery:**
- Implement automatic backup creation before significant configuration changes
- Provide backup rotation and cleanup to prevent unlimited disk space consumption
- Consider configuration versioning for complex applications requiring change tracking
- Implement recovery mechanisms for restoring from configuration backups when needed

## Related AHK Concepts

- [IniRead](./iniread.md) - Reading configuration values from INI files
- [FileWrite](../../30_Built_In_Classes/02-File_IO_Classes/File/filewrite.md) - Direct file writing for complex configurations
- [FileCopy](../../30_Built_In_Classes/02-File_IO_Classes/File/filecopy.md) - Creating configuration backups
- [FileExist](../../30_Built_In_Classes/02-File_IO_Classes/File/fileexist.md) - Checking configuration file existence
- [RegWrite](../../10_Language_Core/01-Functions/Built_In_Functions/regwrite.md) - Writing configuration to Windows registry

## Tags

#AutoHotkey #IniWrite #Configuration #Settings #ConfigurationManagement #SettingsPersistence #ApplicationState #UserPreferences #DataPersistence #ConfigurationWriting
