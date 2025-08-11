# Topic: ClipboardAll Function - Advanced Clipboard Data Management

## Category

Function

## Overview

The ClipboardAll function captures and manipulates the complete clipboard contents including all data formats (text, images, files, rich formatting), providing comprehensive clipboard management capabilities for data exchange, backup operations, and advanced automation scenarios. Unlike simple text clipboard access, ClipboardAll preserves all formats and metadata, making it essential for professional applications requiring full-fidelity data handling.

## Key Points

- Captures entire clipboard contents with all available data formats including text, images, files, and rich formatting
- Enables clipboard backup and restoration operations while preserving format integrity and metadata
- Supports clipboard monitoring, history management, and cross-application data exchange scenarios
- Essential for advanced automation, data backup, format conversion, and applications requiring comprehensive clipboard control
- Provides binary-safe storage and retrieval of clipboard data regardless of content type or encoding

## Syntax and Parameters

```cpp
ClipboardBackup := ClipboardAll()
ClipboardAll(ClipboardBackup)

; Capture current clipboard contents:
; ClipboardAll()         - Returns variable containing all clipboard data and formats
; ClipboardBackup        - Variable to store complete clipboard state

; Restore clipboard contents:
; ClipboardAll(Data)     - Restores clipboard from previously captured data
; Data                   - Variable containing clipboard data from previous capture

; Working with ClipboardAll:
; Save operation:
savedClip := ClipboardAll()         ; Capture current state
A_Clipboard := "New text content"   ; Modify clipboard
ClipboardAll(savedClip)             ; Restore original state

; Data characteristics:
; - Binary safe storage of all clipboard formats
; - Preserves format metadata and data relationships
; - Contains multiple format representations simultaneously
; - Maintains cross-application compatibility

; Important considerations:
; - ClipboardAll variables can be large (MB+ for images/files)
; - Data remains valid until variable is destroyed or reassigned
; - Restoration overwrites all current clipboard contents
; - Some applications may have format-specific requirements

; Memory and performance:
; - Large clipboard data consumes significant memory
; - Consider clearing variables when no longer needed
; - Restoration operation is atomic (all-or-nothing)
```

## Code Examples

```cpp
; Basic clipboard backup and restore
originalClip := ClipboardAll()  ; Save current clipboard
A_Clipboard := "Temporary text"  ; Use clipboard for automation
Sleep(1000)  ; Perform operations
ClipboardAll(originalClip)  ; Restore original contents

; Advanced clipboard management and monitoring system
class ClipboardManager {
    static clipboardHistory := []
    static maxHistorySize := 50
    static autoSaveEnabled := true
    static monitoringEnabled := false
    static monitoringInterval := 500
    static lastClipboardHash := ""
    static formatFilters := Map()
    static storageLocation := A_ScriptDir . "\clipboard_data"
    static encryptionEnabled := false
    static compressionEnabled := true
    static performanceMetrics := {saves: 0, restores: 0, totalSize: 0, compressionRatio: 0}
    static eventCallbacks := Map()
    static securitySettings := {maxSize: 50000000, allowedFormats: [], blockedFormats: []}
    
    static Initialize() {
        this.clipboardHistory := []
        this.ResetPerformanceMetrics()
        this.SetupDefaultFilters()
        this.CreateStorageDirectory()
        
        if (this.autoSaveEnabled) {
            this.StartAutoSave()
        }
    }
    
    static SaveClipboard(label := "", options := {}) {
        ; Save current clipboard with enhanced metadata
        try {
            startTime := A_TickCount
            
            ; Capture clipboard data
            clipboardData := ClipboardAll()
            
            if (!clipboardData) {
                throw Error("Clipboard is empty or inaccessible")
            }
            
            ; Generate metadata
            clipboardInfo := this.AnalyzeClipboardData(clipboardData)
            
            ; Check security restrictions
            if (!this.ValidateClipboardSecurity(clipboardInfo)) {
                throw SecurityError("Clipboard data violates security restrictions")
            }
            
            ; Create clipboard entry
            clipboardEntry := {
                id: this.GenerateClipboardId(),
                label: label ? label : "Auto-save " . FormatTime(),
                timestamp: A_TickCount,
                formattedTime: FormatTime(),
                data: clipboardData,
                info: clipboardInfo,
                size: this.CalculateDataSize(clipboardData),
                compressed: false,
                encrypted: false,
                saveTime: A_TickCount - startTime,
                accessCount: 0,
                lastAccess: 0
            }
            
            ; Apply compression if enabled
            if (this.compressionEnabled && clipboardEntry.size > 10000) {
                try {
                    compressedData := this.CompressClipboardData(clipboardData)
                    if (compressedData.size < clipboardEntry.size * 0.8) {  ; Only use if significant reduction
                        clipboardEntry.data := compressedData.data
                        clipboardEntry.compressed := true
                        clipboardEntry.compressionRatio := Round((clipboardEntry.size - compressedData.size) / clipboardEntry.size * 100, 1)
                        clipboardEntry.size := compressedData.size
                    }
                } catch Error {
                    ; Continue without compression
                }
            }
            
            ; Apply encryption if enabled
            if (this.encryptionEnabled) {
                try {
                    encryptedData := this.EncryptClipboardData(clipboardEntry.data)
                    clipboardEntry.data := encryptedData
                    clipboardEntry.encrypted := true
                } catch Error {
                    throw SecurityError("Failed to encrypt clipboard data")
                }
            }
            
            ; Add to history
            this.clipboardHistory.Push(clipboardEntry)
            
            ; Limit history size
            while (this.clipboardHistory.Length > this.maxHistorySize) {
                removedEntry := this.clipboardHistory.RemoveAt(1)
                this.CleanupClipboardEntry(removedEntry)
            }
            
            ; Update performance metrics
            this.performanceMetrics.saves++
            this.performanceMetrics.totalSize += clipboardEntry.size
            
            ; Trigger save event
            this.TriggerEvent("save", clipboardEntry)
            
            return clipboardEntry.id
            
        } catch Error as err {
            this.TriggerEvent("save_error", {error: err.Message})
            throw err
        }
    }
    
    static RestoreClipboard(entryId, options := {}) {
        ; Restore clipboard from history
        entry := this.FindClipboardEntry(entryId)
        
        if (!entry) {
            throw ValueError("Clipboard entry '" . entryId . "' not found")
        }
        
        try {
            startTime := A_TickCount
            
            ; Decrypt data if necessary
            clipboardData := entry.data
            if (entry.encrypted) {
                clipboardData := this.DecryptClipboardData(clipboardData)
            }
            
            ; Decompress data if necessary
            if (entry.compressed) {
                clipboardData := this.DecompressClipboardData(clipboardData)
            }
            
            ; Validate data integrity
            if (!this.ValidateClipboardData(clipboardData, entry.info)) {
                throw DataIntegrityError("Clipboard data integrity validation failed")
            }
            
            ; Restore to clipboard
            ClipboardAll(clipboardData)
            
            ; Update entry statistics
            entry.accessCount++
            entry.lastAccess := A_TickCount
            restoreTime := A_TickCount - startTime
            
            ; Update performance metrics
            this.performanceMetrics.restores++
            
            ; Trigger restore event
            this.TriggerEvent("restore", {entry: entry, restoreTime: restoreTime})
            
            return {
                success: true,
                entryId: entryId,
                label: entry.label,
                originalTimestamp: entry.timestamp,
                restoreTime: restoreTime,
                dataSize: entry.size
            }
            
        } catch Error as err {
            this.TriggerEvent("restore_error", {entryId: entryId, error: err.Message})
            throw err
        }
    }
    
    static GetClipboardHistory(options := {}) {
        ; Get clipboard history with filtering options
        maxEntries := options.HasProp("maxEntries") ? options.maxEntries : this.clipboardHistory.Length
        includeData := options.HasProp("includeData") ? options.includeData : false
        filterLabel := options.HasProp("filterLabel") ? options.filterLabel : ""
        sortBy := options.HasProp("sortBy") ? options.sortBy : "timestamp"
        sortDesc := options.HasProp("sortDesc") ? options.sortDesc : true
        
        ; Create filtered history
        filteredHistory := []
        
        for entry in this.clipboardHistory {
            ; Apply label filter
            if (filterLabel && InStr(entry.label, filterLabel) = 0) {
                continue
            }
            
            ; Create summary entry
            summaryEntry := {
                id: entry.id,
                label: entry.label,
                timestamp: entry.timestamp,
                formattedTime: entry.formattedTime,
                info: entry.info,
                size: entry.size,
                compressed: entry.compressed,
                encrypted: entry.encrypted,
                accessCount: entry.accessCount,
                lastAccess: entry.lastAccess
            }
            
            ; Include data if requested
            if (includeData) {
                summaryEntry.data := entry.data
            }
            
            filteredHistory.Push(summaryEntry)
        }
        
        ; Sort history
        if (sortBy = "timestamp") {
            filteredHistory.Sort((a, b) => sortDesc ? b.timestamp - a.timestamp : a.timestamp - b.timestamp)
        } else if (sortBy = "size") {
            filteredHistory.Sort((a, b) => sortDesc ? b.size - a.size : a.size - b.size)
        } else if (sortBy = "accessCount") {
            filteredHistory.Sort((a, b) => sortDesc ? b.accessCount - a.accessCount : a.accessCount - b.accessCount)
        }
        
        ; Limit results
        if (maxEntries < filteredHistory.Length) {
            filteredHistory := filteredHistory.Slice(1, maxEntries)
        }
        
        return {
            entries: filteredHistory,
            totalEntries: this.clipboardHistory.Length,
            filteredCount: filteredHistory.Length,
            totalSize: this.CalculateTotalHistorySize(),
            summary: this.GenerateHistorySummary()
        }
    }
    
    static MonitorClipboard(callback, options := {}) {
        ; Monitor clipboard changes
        this.monitoringEnabled := true
        interval := options.HasProp("interval") ? options.interval : this.monitoringInterval
        autoSave := options.HasProp("autoSave") ? options.autoSave : this.autoSaveEnabled
        
        ; Initialize monitoring
        this.lastClipboardHash := this.GetClipboardHash()
        
        monitorFunction := () => {
            if (!this.monitoringEnabled) {
                return
            }
            
            try {
                currentHash := this.GetClipboardHash()
                
                if (currentHash != this.lastClipboardHash) {
                    ; Clipboard changed
                    changeData := {
                        timestamp: A_TickCount,
                        formattedTime: FormatTime(),
                        previousHash: this.lastClipboardHash,
                        currentHash: currentHash,
                        info: this.AnalyzeClipboardData(ClipboardAll())
                    }
                    
                    ; Auto-save if enabled
                    if (autoSave) {
                        try {
                            entryId := this.SaveClipboard("Auto-monitor")
                            changeData.savedEntryId := entryId
                        } catch Error as err {
                            changeData.saveError := err.Message
                        }
                    }
                    
                    ; Call callback
                    if (IsFunc(callback)) {
                        callback.Call(changeData)
                    }
                    
                    this.lastClipboardHash := currentHash
                    this.TriggerEvent("clipboard_change", changeData)
                }
                
            } catch Error as err {
                this.TriggerEvent("monitor_error", {error: err.Message})
            }
        }
        
        ; Start monitoring timer
        this.monitorTimer := monitorFunction
        SetTimer(monitorFunction, interval)
        
        return {
            stop: () => this.StopMonitoring(),
            isActive: () => this.monitoringEnabled
        }
    }
    
    static StopMonitoring() {
        ; Stop clipboard monitoring
        this.monitoringEnabled := false
        if (this.HasProp("monitorTimer")) {
            SetTimer(this.monitorTimer, 0)
        }
    }
    
    static AnalyzeClipboardData(clipboardData) {
        ; Analyze clipboard data formats and content
        analysis := {
            formats: [],
            hasText: false,
            hasImage: false,
            hasFiles: false,
            hasRichText: false,
            hasHtml: false,
            estimatedSize: 0,
            contentHash: "",
            metadata: Map()
        }
        
        if (!clipboardData) {
            return analysis
        }
        
        ; Analyze available formats (simplified detection)
        try {
            ; Check for text content
            if (A_Clipboard) {
                analysis.hasText := true
                analysis.formats.Push("text")
                analysis.estimatedSize += StrLen(A_Clipboard)
            }
            
            ; Generate content hash for change detection
            analysis.contentHash := this.GenerateDataHash(clipboardData)
            
            ; Estimate total size
            analysis.estimatedSize := this.CalculateDataSize(clipboardData)
            
        } catch Error {
            ; Continue with limited analysis
        }
        
        return analysis
    }
    
    static FindClipboardEntry(entryId) {
        ; Find clipboard entry by ID
        for entry in this.clipboardHistory {
            if (entry.id = entryId) {
                return entry
            }
        }
        return false
    }
    
    static DeleteClipboardEntry(entryId) {
        ; Delete clipboard entry from history
        for i, entry in this.clipboardHistory {
            if (entry.id = entryId) {
                this.CleanupClipboardEntry(entry)
                this.clipboardHistory.RemoveAt(i)
                this.TriggerEvent("delete", {entryId: entryId})
                return true
            }
        }
        return false
    }
    
    static ClearHistory() {
        ; Clear all clipboard history
        for entry in this.clipboardHistory {
            this.CleanupClipboardEntry(entry)
        }
        
        this.clipboardHistory := []
        this.ResetPerformanceMetrics()
        this.TriggerEvent("clear_history", {})
    }
    
    static ExportHistory(filePath := "", format := "json") {
        ; Export clipboard history to file
        if (!filePath) {
            filePath := this.storageLocation . "\clipboard_export_" . FormatTime(, "yyyyMMdd_HHmmss") . "." . format
        }
        
        exportData := {
            exportTime: FormatTime(),
            version: "1.0",
            totalEntries: this.clipboardHistory.Length,
            entries: []
        }
        
        ; Prepare export entries (without large binary data)
        for entry in this.clipboardHistory {
            exportEntry := {
                id: entry.id,
                label: entry.label,
                timestamp: entry.timestamp,
                formattedTime: entry.formattedTime,
                info: entry.info,
                size: entry.size,
                compressed: entry.compressed,
                encrypted: entry.encrypted,
                accessCount: entry.accessCount
            }
            
            ; Include text data if available and reasonable size
            if (entry.info.hasText && entry.size < 10000) {
                try {
                    ; This is a simplified export - in real implementation,
                    ; you'd need to properly decode the clipboard data
                    exportEntry.textContent := "[Clipboard text content]"
                } catch Error {
                    exportEntry.textContent := "[Unable to export text]"
                }
            }
            
            exportData.entries.Push(exportEntry)
        }
        
        ; Write to file based on format
        switch StrLower(format) {
            case "json":
                ; In a real implementation, you'd use a proper JSON library
                content := this.SerializeToJson(exportData)
            case "csv":
                content := this.SerializeToCSV(exportData.entries)
            default:
                throw ValueError("Unsupported export format: " . format)
        }
        
        FileWrite(content, filePath)
        return filePath
    }
    
    static ImportHistory(filePath, options := {}) {
        ; Import clipboard history from file
        if (!FileExist(filePath)) {
            throw FileNotFoundError("Import file not found: " . filePath)
        }
        
        content := FileRead(filePath)
        format := options.HasProp("format") ? options.format : "json"
        
        switch StrLower(format) {
            case "json":
                importData := this.DeserializeFromJson(content)
            default:
                throw ValueError("Unsupported import format: " . format)
        }
        
        ; Import entries (metadata only, no clipboard data)
        importedCount := 0
        for entryData in importData.entries {
            try {
                ; Create placeholder entry
                placeholderEntry := {
                    id: this.GenerateClipboardId(),
                    label: entryData.label . " (imported)",
                    timestamp: entryData.timestamp,
                    formattedTime: entryData.formattedTime,
                    data: "",  ; No actual clipboard data
                    info: entryData.info,
                    size: 0,
                    compressed: false,
                    encrypted: false,
                    imported: true,
                    originalId: entryData.id
                }
                
                this.clipboardHistory.Push(placeholderEntry)
                importedCount++
                
            } catch Error {
                ; Skip problematic entries
                continue
            }
        }
        
        return {
            importedCount: importedCount,
            totalEntries: importData.entries.Length,
            filePath: filePath
        }
    }
    
    static CompressClipboardData(data) {
        ; Simplified compression placeholder
        ; In real implementation, would use actual compression algorithm
        compressedSize := Round(this.CalculateDataSize(data) * 0.7)  ; Simulate 30% compression
        return {
            data: data,  ; Would be actual compressed data
            size: compressedSize,
            algorithm: "deflate"
        }
    }
    
    static DecompressClipboardData(compressedData) {
        ; Simplified decompression placeholder
        return compressedData  ; Would decompress actual data
    }
    
    static EncryptClipboardData(data) {
        ; Simplified encryption placeholder
        return data  ; Would encrypt actual data
    }
    
    static DecryptClipboardData(encryptedData) {
        ; Simplified decryption placeholder
        return encryptedData  ; Would decrypt actual data
    }
    
    static ValidateClipboardData(data, expectedInfo) {
        ; Validate clipboard data integrity
        if (!data) {
            return false
        }
        
        ; In real implementation, would validate checksum, format integrity, etc.
        return true
    }
    
    static ValidateClipboardSecurity(clipboardInfo) {
        ; Validate clipboard data against security settings
        if (this.securitySettings.maxSize > 0 && clipboardInfo.estimatedSize > this.securitySettings.maxSize) {
            return false
        }
        
        ; Check blocked formats
        for blockedFormat in this.securitySettings.blockedFormats {
            if (clipboardInfo.formats.Find(blockedFormat)) {
                return false
            }
        }
        
        return true
    }
    
    static GetClipboardHash() {
        ; Generate hash of current clipboard content for change detection
        try {
            clipData := ClipboardAll()
            if (!clipData) {
                return "empty"
            }
            return this.GenerateDataHash(clipData)
        } catch Error {
            return "error"
        }
    }
    
    static GenerateDataHash(data) {
        ; Generate hash of data (simplified)
        if (!data) {
            return "0"
        }
        
        ; In real implementation, would use actual hash algorithm
        return String(Random(100000, 999999))  ; Placeholder hash
    }
    
    static CalculateDataSize(data) {
        ; Calculate approximate size of clipboard data
        if (!data) {
            return 0
        }
        
        ; Simplified size calculation
        return 1024  ; Placeholder size
    }
    
    static CalculateTotalHistorySize() {
        ; Calculate total size of all history entries
        totalSize := 0
        for entry in this.clipboardHistory {
            totalSize += entry.size
        }
        return totalSize
    }
    
    static GenerateClipboardId() {
        ; Generate unique ID for clipboard entry
        return "clip_" . A_TickCount . "_" . Random(1000, 9999)
    }
    
    static GenerateHistorySummary() {
        ; Generate summary statistics for history
        summary := {
            totalEntries: this.clipboardHistory.Length,
            totalSize: this.CalculateTotalHistorySize(),
            averageSize: 0,
            oldestEntry: "",
            newestEntry: "",
            mostAccessed: "",
            formatDistribution: Map()
        }
        
        if (this.clipboardHistory.Length > 0) {
            summary.averageSize := Round(summary.totalSize / this.clipboardHistory.Length, 0)
            
            ; Find oldest and newest
            oldestTime := 999999999999
            newestTime := 0
            mostAccessCount := 0
            
            for entry in this.clipboardHistory {
                if (entry.timestamp < oldestTime) {
                    oldestTime := entry.timestamp
                    summary.oldestEntry := entry.label
                }
                
                if (entry.timestamp > newestTime) {
                    newestTime := entry.timestamp
                    summary.newestEntry := entry.label
                }
                
                if (entry.accessCount > mostAccessCount) {
                    mostAccessCount := entry.accessCount
                    summary.mostAccessed := entry.label
                }
            }
        }
        
        return summary
    }
    
    static StartAutoSave() {
        ; Start automatic clipboard saving
        this.MonitorClipboard((changeData) => {
            ; Auto-save handled in monitor function
        }, {
            interval: 1000,
            autoSave: true
        })
    }
    
    static SetupDefaultFilters() {
        ; Set up default format filters
        this.formatFilters := Map()
    }
    
    static CreateStorageDirectory() {
        ; Create storage directory if it doesn't exist
        if (!DirExist(this.storageLocation)) {
            DirCreate(this.storageLocation)
        }
    }
    
    static CleanupClipboardEntry(entry) {
        ; Cleanup resources for clipboard entry
        entry.data := ""  ; Release memory
    }
    
    static ResetPerformanceMetrics() {
        ; Reset performance metrics
        this.performanceMetrics := {
            saves: 0,
            restores: 0,
            totalSize: 0,
            compressionRatio: 0
        }
    }
    
    static TriggerEvent(eventType, eventData) {
        ; Trigger event callbacks
        if (this.eventCallbacks.Has(eventType)) {
            for callback in this.eventCallbacks[eventType] {
                try {
                    if (IsFunc(callback)) {
                        callback.Call(eventData)
                    }
                } catch Error {
                    ; Continue with other callbacks
                }
            }
        }
    }
    
    static RegisterEventCallback(eventType, callback) {
        ; Register event callback
        if (!this.eventCallbacks.Has(eventType)) {
            this.eventCallbacks[eventType] := []
        }
        this.eventCallbacks[eventType].Push(callback)
    }
    
    static SerializeToJson(data) {
        ; Simplified JSON serialization (placeholder)
        return '{"message": "JSON export not implemented in example"}'
    }
    
    static SerializeToCSV(entries) {
        ; Simplified CSV serialization
        csv := "ID,Label,Timestamp,Size,Compressed,Encrypted,AccessCount`n"
        
        for entry in entries {
            csv .= entry.id . "," . entry.label . "," . entry.formattedTime . "," . 
                   entry.size . "," . entry.compressed . "," . entry.encrypted . "," . 
                   entry.accessCount . "`n"
        }
        
        return csv
    }
    
    static DeserializeFromJson(content) {
        ; Simplified JSON deserialization (placeholder)
        return {entries: []}
    }
    
    static GetPerformanceReport() {
        ; Get comprehensive performance report
        metrics := this.performanceMetrics
        
        report := "Clipboard Management Performance Report`n"
        report .= "========================================`n`n"
        
        report .= "Operations:`n"
        report .= "Total Saves: " . metrics.saves . "`n"
        report .= "Total Restores: " . metrics.restores . "`n"
        report .= "Total Data Size: " . Round(metrics.totalSize / 1024, 1) . " KB`n`n"
        
        report .= "History:`n"
        report .= "Current Entries: " . this.clipboardHistory.Length . "`n"
        report .= "Max History Size: " . this.maxHistorySize . "`n"
        report .= "Storage Location: " . this.storageLocation . "`n`n"
        
        report .= "Configuration:`n"
        report .= "Auto-save: " . (this.autoSaveEnabled ? "Enabled" : "Disabled") . "`n"
        report .= "Monitoring: " . (this.monitoringEnabled ? "Enabled" : "Disabled") . "`n"
        report .= "Compression: " . (this.compressionEnabled ? "Enabled" : "Disabled") . "`n"
        report .= "Encryption: " . (this.encryptionEnabled ? "Enabled" : "Disabled") . "`n"
        
        return report
    }
    
    static EnableAutoSave() {
        this.autoSaveEnabled := true
        if (!this.monitoringEnabled) {
            this.StartAutoSave()
        }
    }
    
    static DisableAutoSave() {
        this.autoSaveEnabled := false
    }
    
    static EnableCompression() {
        this.compressionEnabled := true
    }
    
    static DisableCompression() {
        this.compressionEnabled := false
    }
    
    static SetMaxHistorySize(size) {
        if (size < 1) {
            throw ValueError("History size must be at least 1")
        }
        this.maxHistorySize := size
        
        ; Trim history if necessary
        while (this.clipboardHistory.Length > this.maxHistorySize) {
            removedEntry := this.clipboardHistory.RemoveAt(1)
            this.CleanupClipboardEntry(removedEntry)
        }
    }
    
    static SetSecuritySettings(settings) {
        ; Update security settings
        if (settings.HasProp("maxSize")) {
            this.securitySettings.maxSize := settings.maxSize
        }
        if (settings.HasProp("allowedFormats")) {
            this.securitySettings.allowedFormats := settings.allowedFormats
        }
        if (settings.HasProp("blockedFormats")) {
            this.securitySettings.blockedFormats := settings.blockedFormats
        }
    }
}

; Example usage and demonstrations
; Initialize clipboard management system
ClipboardManager.Initialize()

; Basic clipboard backup and restore
originalClipboard := ClipboardAll()  ; Backup current clipboard
A_Clipboard := "This is temporary content for automation"

; Perform some operations...
Sleep(2000)

; Restore original clipboard
ClipboardAll(originalClipboard)
MsgBox("Clipboard restored to original state")

; Advanced clipboard management
; Save current clipboard with label
clipId1 := ClipboardManager.SaveClipboard("Important data backup")

; Change clipboard content
A_Clipboard := "New temporary content"

; Save again
clipId2 := ClipboardManager.SaveClipboard("Temporary content")

; Get clipboard history
history := ClipboardManager.GetClipboardHistory({maxEntries: 10})

MsgBox("Clipboard history contains " . history.totalEntries . " entries:`n" .
       "Total size: " . Round(history.totalSize / 1024, 1) . " KB")

; Display history entries
for entry in history.entries {
    OutputDebug("Entry: " . entry.label . " (" . entry.formattedTime . ") - " . 
                Round(entry.size / 1024, 1) . " KB")
}

; Restore specific clipboard entry
try {
    restoreResult := ClipboardManager.RestoreClipboard(clipId1)
    MsgBox("Restored clipboard entry: " . restoreResult.label . "`n" .
           "Original timestamp: " . restoreResult.originalTimestamp . "`n" .
           "Restore time: " . restoreResult.restoreTime . "ms")
} catch Error as err {
    MsgBox("Failed to restore clipboard: " . err.Message)
}

; Monitor clipboard changes
monitor := ClipboardManager.MonitorClipboard((changeData) => {
    MsgBox("Clipboard changed!`n" .
           "Time: " . changeData.formattedTime . "`n" .
           "Has text: " . changeData.info.hasText . "`n" .
           "Estimated size: " . Round(changeData.info.estimatedSize / 1024, 1) . " KB" .
           (changeData.HasProp("savedEntryId") ? "`nAuto-saved as: " . changeData.savedEntryId : ""))
}, {
    interval: 1000,
    autoSave: true
})

; Test clipboard monitoring by changing content
A_Clipboard := "Test content 1"
Sleep(1500)
A_Clipboard := "Test content 2"
Sleep(1500)

; Stop monitoring
monitor.stop()

; Register event callbacks
ClipboardManager.RegisterEventCallback("save", (eventData) => {
    OutputDebug("Clipboard saved: " . eventData.label . " (" . Round(eventData.size / 1024, 1) . " KB)")
})

ClipboardManager.RegisterEventCallback("restore", (eventData) => {
    OutputDebug("Clipboard restored: " . eventData.entry.label)
})

ClipboardManager.RegisterEventCallback("clipboard_change", (eventData) => {
    OutputDebug("Clipboard changed at: " . eventData.formattedTime)
})

; Advanced clipboard operations
; Save multiple clipboard states
A_Clipboard := "Document 1 content"
doc1Id := ClipboardManager.SaveClipboard("Document 1")

A_Clipboard := "Document 2 content"
doc2Id := ClipboardManager.SaveClipboard("Document 2")

A_Clipboard := "Document 3 content"
doc3Id := ClipboardManager.SaveClipboard("Document 3")

; Demonstrate restoration sequence
documents := [doc1Id, doc2Id, doc3Id]

for i, docId in documents {
    ClipboardManager.RestoreClipboard(docId)
    MsgBox("Restored to Document " . i . "`nCurrent clipboard: " . SubStr(A_Clipboard, 1, 50))
    Sleep(1000)
}

; Export clipboard history
try {
    exportPath := ClipboardManager.ExportHistory("", "csv")
    MsgBox("Clipboard history exported to: " . exportPath)
} catch Error as err {
    MsgBox("Export failed: " . err.Message)
}

; Set security restrictions
ClipboardManager.SetSecuritySettings({
    maxSize: 10000000,  ; 10MB limit
    blockedFormats: ["executable", "script"]  ; Block certain formats
})

; Configure compression and auto-save
ClipboardManager.EnableCompression()
ClipboardManager.EnableAutoSave()
ClipboardManager.SetMaxHistorySize(25)

; Test large clipboard operation (simulated)
largeContent := ""
Loop 1000 {
    largeContent .= "This is line " . A_Index . " of large clipboard content.`n"
}

A_Clipboard := largeContent
largeId := ClipboardManager.SaveClipboard("Large content test")

MsgBox("Large content saved. Size: " . Round(StrLen(largeContent) / 1024, 1) . " KB")

; Advanced clipboard analysis
A_Clipboard := "Sample text for analysis"
currentClipId := ClipboardManager.SaveClipboard("Analysis sample")

; Get detailed clipboard information
history := ClipboardManager.GetClipboardHistory({maxEntries: 1, includeData: false})
if (history.entries.Length > 0) {
    entry := history.entries[1]
    
    MsgBox("Clipboard Analysis:`n" .
           "Label: " . entry.label . "`n" .
           "Size: " . Round(entry.size / 1024, 1) . " KB`n" .
           "Has text: " . entry.info.hasText . "`n" .
           "Formats: " . entry.info.formats.Length . "`n" .
           "Compressed: " . entry.compressed . "`n" .
           "Access count: " . entry.accessCount)
}

; Performance testing
startTime := A_TickCount

; Perform multiple operations
Loop 10 {
    A_Clipboard := "Performance test " . A_Index
    ClipboardManager.SaveClipboard("Perf test " . A_Index)
}

performanceTime := A_TickCount - startTime

; Get performance report
performanceReport := ClipboardManager.GetPerformanceReport()
; MsgBox(performanceReport . "`n`nPerformance test time: " . performanceTime . "ms", "Performance Report")

; Cleanup demonstration
MsgBox("Clipboard management demonstration complete.`n" .
       "History entries: " . ClipboardManager.clipboardHistory.Length . "`n" .
       "Total operations: " . ClipboardManager.performanceMetrics.saves . " saves, " . 
       ClipboardManager.performanceMetrics.restores . " restores")

; Clear history and reset
ClipboardManager.ClearHistory()
ClipboardManager.StopMonitoring()

; Final clipboard restoration
if (originalClipboard) {
    ClipboardAll(originalClipboard)
    MsgBox("Original clipboard content restored")
}
```

## Implementation Notes

**Memory Management and Performance:**
- ClipboardAll captures can consume significant memory (MB+ for images/rich content), requiring careful resource management
- Large clipboard data should be cleared from variables when no longer needed to prevent memory leaks
- Consider implementing compression and disk-based storage for clipboard history systems to manage memory usage
- Monitor memory consumption when maintaining clipboard history or implementing undo/redo functionality

**Data Format Preservation and Integrity:**
- ClipboardAll preserves all clipboard formats simultaneously, including proprietary application formats
- Format relationships and metadata are maintained, ensuring proper cross-application compatibility
- Some applications may register custom clipboard formats that require specific handling procedures
- Validate data integrity when storing and restoring clipboard contents, especially for long-term storage

**Cross-Application Compatibility:**
- Different applications may interpret clipboard formats differently, affecting restoration behavior
- Some applications lock clipboard access during certain operations, requiring retry mechanisms
- Format priority and selection may vary between applications when multiple formats are available
- Test clipboard operations across target applications to ensure consistent behavior

**Security and Privacy Considerations:**
- Clipboard contents may contain sensitive information including passwords, personal data, and confidential documents
- Implement appropriate security measures for clipboard storage, especially in shared or networked environments
- Consider data encryption for persistent clipboard storage and transmission scenarios
- Be aware of clipboard access restrictions in secure environments or when running with limited privileges

**Timing and Synchronization Issues:**
- Clipboard operations are not atomic and may be interrupted by other applications or system events
- Implement appropriate delays and verification when clipboard operations are part of larger automation sequences
- Handle cases where clipboard access is temporarily unavailable due to system locks or other applications
- Consider clipboard change notifications and monitoring for reactive automation scenarios

## Related AHK Concepts

- [A_Clipboard](../../00_Fundamentals/00-Variables_and_Expressions/variables-and-scope.md) - Basic text clipboard access
- [ClipWait](../../10_Language_Core/01-Functions/Built_In_Functions/clipwait.md) - Waiting for clipboard content availability
- [OnClipboardChange](../../10_Language_Core/01-Functions/Built_In_Functions/onclipboardchange.md) - Monitoring clipboard changes
- [FileRead](../../30_Built_In_Classes/02-File_IO_Classes/File/fileread.md) - Reading data for clipboard operations
- [FileWrite](../../30_Built_In_Classes/02-File_IO_Classes/File/filewrite.md) - Storing clipboard data persistently

## Tags

#AutoHotkey #ClipboardAll #ClipboardManagement #DataExchange #DataBackup #ClipboardHistory #RichFormats #BinaryData #DataIntegrity #CrossApplication
