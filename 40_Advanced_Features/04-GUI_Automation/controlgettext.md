# Topic: ControlGetText Function - GUI Control Text Extraction and Data Retrieval

## Category

Function

## Overview

The ControlGetText function retrieves text content from specific GUI controls within windows, providing essential data extraction capabilities for automated testing, form processing, legacy application integration, and accessibility applications. It's crucial for validation, data collection, user interface analysis, and any scenario requiring programmatic access to control text content.

## Key Points

- Extracts text content from GUI controls using control identifiers rather than screen scraping
- Supports comprehensive text retrieval from various control types including edit boxes, labels, buttons, and lists
- Provides reliable data extraction that works regardless of window position, visual state, or display configuration
- Enables sophisticated data validation frameworks and automated form processing systems
- Essential for automated testing, data extraction, legacy system integration, and accessibility applications

## Syntax and Parameters

```cpp
Text := ControlGetText(Control [, WinTitle, WinText, ExcludeTitle, ExcludeText])

; Parameters:
; Control      - Control identifier (ClassNN, text, or control object)
; WinTitle     - Window title or identifier (optional)
; WinText      - Text contained in the window (optional)
; ExcludeTitle - Windows to exclude from matching (optional)
; ExcludeText  - Text in windows to exclude (optional)

; Control identification methods:
; "Edit1"             - ClassNN identifier (class name + instance number)
; "ahk_id " . hwnd    - Control handle (HWND)
; controlObj          - Control object reference

; Window identification:
; "Window Title"      - Exact or partial window title
; "ahk_class Class"   - Window class name
; "ahk_exe Process"   - Process executable name
; "ahk_pid PID"       - Process ID
; If omitted          - Uses last found window

; Return values:
; String containing control text content
; Empty string if control has no text
; Throws exception if control not found
; Throws exception if window not found

; Control type behavior:
; Edit controls       - Returns current text content
; Static controls     - Returns display text
; Button controls     - Returns button text/caption
; ListBox controls    - Returns currently selected item text
; ComboBox controls   - Returns current selection or edit text
; ListView controls   - Returns focused item text (first column)
```

## Code Examples

```cpp
; Basic text retrieval
text := ControlGetText("Edit1", "Notepad")
buttonText := ControlGetText("Button1", "Calculator")
statusText := ControlGetText("StatusBar", "Application")

; Advanced text extraction and processing system
class TextExtractor {
    static extractionCache := Map()
    static cacheTimeout := 5000
    static retryAttempts := 3
    static retryDelay := 500
    static textProcessing := true
    static extractionLog := []
    static dataValidation := true
    static extractionRules := Map()
    
    static Initialize() {
        this.extractionCache.Clear()
        this.extractionLog := []
        this.SetupDefaultRules()
    }
    
    static ExtractText(controlSpec, windowSpec, options := {}) {
        ; Advanced text extraction with caching and error handling
        useCache := options.HasProp("useCache") ? options.useCache : true
        processText := options.HasProp("processText") ? options.processText : this.textProcessing
        validateData := options.HasProp("validate") ? options.validate : this.dataValidation
        retries := options.HasProp("retries") ? options.retries : this.retryAttempts
        timeout := options.HasProp("timeout") ? options.timeout : 5000
        
        ; Create extraction record
        extraction := {
            control: controlSpec,
            window: windowSpec,
            timestamp: A_TickCount,
            text: "",
            success: false,
            cached: false,
            processed: false,
            validated: false,
            error: "",
            attempts: 0
        }
        
        ; Check cache first
        if (useCache) {
            cacheKey := this.GenerateCacheKey(controlSpec, windowSpec)
            if (this.extractionCache.Has(cacheKey)) {
                cacheEntry := this.extractionCache[cacheKey]
                if (A_TickCount - cacheEntry.timestamp < this.cacheTimeout) {
                    extraction.text := cacheEntry.text
                    extraction.success := true
                    extraction.cached := true
                    this.LogExtraction(extraction)
                    return extraction.text
                }
            }
        }
        
        ; Attempt extraction with retries
        for attempt in Range(1, retries + 1) {
            extraction.attempts := attempt
            
            try {
                ; Verify window exists
                if (!WinExist(windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")) {
                    throw Error("Window not found: " . windowSpec.title)
                }
                
                ; Verify control exists
                if (!ControlGetHwnd(controlSpec, windowSpec.title, windowSpec.HasProp("text") ? windowSpec.text : "")) {
                    throw Error("Control not found: " . controlSpec)
                }
                
                ; Extract text
                rawText := ControlGetText(controlSpec, 
                                         windowSpec.title, 
                                         windowSpec.HasProp("text") ? windowSpec.text : "",
                                         windowSpec.HasProp("excludeTitle") ? windowSpec.excludeTitle : "",
                                         windowSpec.HasProp("excludeText") ? windowSpec.excludeText : "")
                
                ; Process text if enabled
                if (processText) {
                    extraction.text := this.ProcessText(rawText, options)
                    extraction.processed := true
                } else {
                    extraction.text := rawText
                }
                
                ; Validate data if enabled
                if (validateData) {
                    this.ValidateExtractedText(extraction, options)
                    extraction.validated := true
                }
                
                ; Cache result
                if (useCache) {
                    this.CacheExtraction(controlSpec, windowSpec, extraction.text)
                }
                
                extraction.success := true
                break
                
            } catch Error as err {
                extraction.error := err.Message
                
                if (attempt < retries) {
                    Sleep(this.retryDelay)
                    continue
                } else {
                    this.LogExtraction(extraction)
                    throw err
                }
            }
        }
        
        this.LogExtraction(extraction)
        return extraction.text
    }
    
    static ProcessText(rawText, options) {
        ; Process extracted text based on options
        processedText := rawText
        
        ; Trim whitespace
        if (!options.HasProp("preserveWhitespace") || !options.preserveWhitespace) {
            processedText := Trim(processedText)
        }
        
        ; Normalize line endings
        if (options.HasProp("normalizeLineEndings") && options.normalizeLineEndings) {
            processedText := StrReplace(processedText, "`r`n", "`n")
            processedText := StrReplace(processedText, "`r", "`n")
        }
        
        ; Remove extra whitespace
        if (options.HasProp("collapseWhitespace") && options.collapseWhitespace) {
            processedText := RegExReplace(processedText, "\s+", " ")
        }
        
        ; Apply text transformation rules
        if (options.HasProp("transform")) {
            switch options.transform {
                case "lowercase":
                    processedText := StrLower(processedText)
                case "uppercase":
                    processedText := StrUpper(processedText)
                case "titlecase":
                    processedText := StrTitle(processedText)
                case "removeNumbers":
                    processedText := RegExReplace(processedText, "\d", "")
                case "numbersOnly":
                    processedText := RegExReplace(processedText, "[^\d\.\-]", "")
                case "alphanumericOnly":
                    processedText := RegExReplace(processedText, "[^\w\s]", "")
            }
        }
        
        ; Apply custom regex transformations
        if (options.HasProp("regexTransforms")) {
            for transform in options.regexTransforms {
                processedText := RegExReplace(processedText, transform.pattern, transform.replacement)
            }
        }
        
        return processedText
    }
    
    static ValidateExtractedText(extraction, options) {
        ; Validate extracted text against rules
        text := extraction.text
        
        ; Check minimum length
        if (options.HasProp("minLength") && StrLen(text) < options.minLength) {
            throw Error("Text validation failed: minimum length not met")
        }
        
        ; Check maximum length
        if (options.HasProp("maxLength") && StrLen(text) > options.maxLength) {
            throw Error("Text validation failed: maximum length exceeded")
        }
        
        ; Check required pattern
        if (options.HasProp("requiredPattern") && !RegExMatch(text, options.requiredPattern)) {
            throw Error("Text validation failed: required pattern not found")
        }
        
        ; Check forbidden pattern
        if (options.HasProp("forbiddenPattern") && RegExMatch(text, options.forbiddenPattern)) {
            throw Error("Text validation failed: forbidden pattern found")
        }
        
        ; Check expected values
        if (options.HasProp("expectedValues") && !options.expectedValues.Find(text)) {
            throw Error("Text validation failed: value not in expected list")
        }
        
        ; Apply custom validation rules
        controlKey := extraction.control
        if (this.extractionRules.Has(controlKey)) {
            rule := this.extractionRules[controlKey]
            if (!this.ApplyValidationRule(text, rule)) {
                throw Error("Text validation failed: custom rule validation")
            }
        }
    }
    
    static ExtractMultipleControls(controlSpecs, windowSpec, options := {}) {
        ; Extract text from multiple controls
        results := Map()
        errors := []
        
        for controlName, controlSpec in controlSpecs {
            try {
                text := this.ExtractText(controlSpec, windowSpec, options)
                results[controlName] := text
            } catch Error as err {
                errors.Push({
                    control: controlName,
                    error: err.Message
                })
            }
        }
        
        return {
            results: results,
            errors: errors,
            successCount: results.Count,
            errorCount: errors.Length
        }
    }
    
    static ExtractFormData(formSpec, windowSpec, options := {}) {
        ; Extract data from entire form
        formData := Map()
        fieldErrors := []
        
        for fieldName, fieldSpec in formSpec.fields {
            try {
                ; Extract field value
                fieldValue := this.ExtractText(fieldSpec.control, windowSpec, fieldSpec.HasProp("options") ? fieldSpec.options : options)
                
                ; Apply field-specific processing
                if (fieldSpec.HasProp("dataType")) {
                    fieldValue := this.ConvertDataType(fieldValue, fieldSpec.dataType)
                }
                
                formData[fieldName] := fieldValue
                
            } catch Error as err {
                fieldErrors.Push({
                    field: fieldName,
                    control: fieldSpec.control,
                    error: err.Message
                })
            }
        }
        
        return {
            formName: formSpec.name,
            data: formData,
            errors: fieldErrors,
            extractedFields: formData.Count,
            totalFields: formSpec.fields.Count,
            success: fieldErrors.Length = 0
        }
    }
    
    static ConvertDataType(text, dataType) {
        ; Convert extracted text to specific data type
        switch dataType {
            case "integer":
                if (!IsInteger(text)) {
                    throw Error("Cannot convert '" . text . "' to integer")
                }
                return Integer(text)
                
            case "float":
                if (!IsFloat(text) && !IsInteger(text)) {
                    throw Error("Cannot convert '" . text . "' to float")
                }
                return Float(text)
                
            case "boolean":
                lowerText := StrLower(Trim(text))
                if (lowerText = "true" || lowerText = "yes" || lowerText = "1" || lowerText = "on") {
                    return true
                } else if (lowerText = "false" || lowerText = "no" || lowerText = "0" || lowerText = "off") {
                    return false
                } else {
                    throw Error("Cannot convert '" . text . "' to boolean")
                }
                
            case "date":
                ; Basic date parsing (would need more sophisticated implementation)
                if (RegExMatch(text, "^\d{1,2}/\d{1,2}/\d{4}$")) {
                    return text  ; Return as formatted date string
                } else {
                    throw Error("Cannot convert '" . text . "' to date")
                }
                
            case "currency":
                ; Remove currency symbols and convert to float
                numericText := RegExReplace(text, "[^\d\.\-]", "")
                if (IsFloat(numericText)) {
                    return Float(numericText)
                } else {
                    throw Error("Cannot convert '" . text . "' to currency")
                }
                
            default:
                return text  ; Return as string
        }
    }
    
    static MonitorTextChanges(controlSpec, windowSpec, callback, options := {}) {
        ; Monitor control for text changes
        checkInterval := options.HasProp("interval") ? options.interval : 1000
        timeout := options.HasProp("timeout") ? options.timeout : 0
        
        lastText := ""
        startTime := A_TickCount
        
        ; Initial text
        try {
            lastText := this.ExtractText(controlSpec, windowSpec, options)
        } catch {
            lastText := ""
        }
        
        ; Start monitoring timer
        timerFunction := () => {
            try {
                currentText := this.ExtractText(controlSpec, windowSpec, options)
                
                if (currentText != lastText) {
                    ; Text changed, call callback
                    if (IsFunc(callback)) {
                        callback.Call(currentText, lastText, controlSpec, windowSpec)
                    }
                    lastText := currentText
                }
                
                ; Check timeout
                if (timeout > 0 && A_TickCount - startTime > timeout) {
                    SetTimer(timerFunction, 0)  ; Stop monitoring
                }
                
            } catch Error {
                ; Control or window no longer available
                SetTimer(timerFunction, 0)  ; Stop monitoring
            }
        }
        
        SetTimer(timerFunction, checkInterval)
        
        return {
            stop: () => SetTimer(timerFunction, 0),
            getLastText: () => lastText
        }
    }
    
    static WaitForTextChange(controlSpec, windowSpec, expectedText := "", timeout := 10000, options := {}) {
        ; Wait for control text to change to expected value
        startTime := A_TickCount
        checkInterval := options.HasProp("checkInterval") ? options.checkInterval : 250
        
        while (A_TickCount - startTime < timeout) {
            try {
                currentText := this.ExtractText(controlSpec, windowSpec, options)
                
                if (expectedText = "") {
                    ; Wait for any change from initial text
                    static initialText := currentText
                    if (currentText != initialText) {
                        return currentText
                    }
                } else {
                    ; Wait for specific text
                    if (currentText = expectedText) {
                        return currentText
                    }
                }
                
            } catch Error {
                ; Continue waiting
            }
            
            Sleep(checkInterval)
        }
        
        throw TimeoutError("Text did not change to expected value within timeout")
    }
    
    static GenerateCacheKey(controlSpec, windowSpec) {
        ; Generate unique cache key for control and window combination
        return windowSpec.title . "|" . controlSpec
    }
    
    static CacheExtraction(controlSpec, windowSpec, text) {
        ; Cache extracted text
        cacheKey := this.GenerateCacheKey(controlSpec, windowSpec)
        this.extractionCache[cacheKey] := {
            text: text,
            timestamp: A_TickCount
        }
        
        ; Limit cache size
        if (this.extractionCache.Count > 100) {
            this.CleanupCache()
        }
    }
    
    static CleanupCache() {
        ; Remove old cache entries
        currentTime := A_TickCount
        
        for cacheKey, entry in this.extractionCache {
            if (currentTime - entry.timestamp > this.cacheTimeout) {
                this.extractionCache.Delete(cacheKey)
            }
        }
    }
    
    static SetupDefaultRules() {
        ; Set up default validation rules
        this.extractionRules := Map()
        
        ; Add common validation rules
        this.AddValidationRule("EmailEdit", {type: "pattern", pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"})
        this.AddValidationRule("PhoneEdit", {type: "pattern", pattern: "^\(\d{3}\)\s?\d{3}-\d{4}$|^\d{3}-\d{3}-\d{4}$"})
        this.AddValidationRule("ZipEdit", {type: "pattern", pattern: "^\d{5}(-\d{4})?$"})
    }
    
    static AddValidationRule(controlIdentifier, rule) {
        ; Add custom validation rule for control
        this.extractionRules[controlIdentifier] := rule
    }
    
    static ApplyValidationRule(text, rule) {
        ; Apply validation rule to text
        switch rule.type {
            case "pattern":
                return RegExMatch(text, rule.pattern) > 0
                
            case "length":
                textLen := StrLen(text)
                if (rule.HasProp("min") && textLen < rule.min) return false
                if (rule.HasProp("max") && textLen > rule.max) return false
                return true
                
            case "numeric":
                if (rule.HasProp("integer") && rule.integer) {
                    return IsInteger(text)
                } else {
                    return IsInteger(text) || IsFloat(text)
                }
                
            case "enum":
                return rule.HasProp("values") && rule.values.Find(text)
                
            default:
                return true
        }
    }
    
    static LogExtraction(extraction) {
        ; Log extraction operation
        this.extractionLog.Push(extraction)
        
        ; Limit log size
        if (this.extractionLog.Length > 500) {
            this.extractionLog.RemoveAt(1)
        }
    }
    
    static GetExtractionStatistics() {
        ; Get extraction operation statistics
        totalExtractions := this.extractionLog.Length
        successfulExtractions := 0
        cachedExtractions := 0
        processedExtractions := 0
        
        for extraction in this.extractionLog {
            if (extraction.success) successfulExtractions++
            if (extraction.cached) cachedExtractions++
            if (extraction.processed) processedExtractions++
        }
        
        return {
            total: totalExtractions,
            successful: successfulExtractions,
            cached: cachedExtractions,
            processed: processedExtractions,
            successRate: totalExtractions > 0 ? Round((successfulExtractions / totalExtractions) * 100, 1) : 0,
            cacheHitRate: totalExtractions > 0 ? Round((cachedExtractions / totalExtractions) * 100, 1) : 0
        }
    }
    
    static GenerateExtractionReport() {
        ; Generate comprehensive extraction report
        stats := this.GetExtractionStatistics()
        
        report := "Text Extraction Report`n"
        report .= "=====================`n`n"
        
        report .= "Summary:`n"
        report .= "Total Extractions: " . stats.total . "`n"
        report .= "Successful: " . stats.successful . "`n"
        report .= "Success Rate: " . stats.successRate . "%`n"
        report .= "Cache Hit Rate: " . stats.cacheHitRate . "%`n"
        report .= "Processed: " . stats.processed . "`n`n"
        
        report .= "Configuration:`n"
        report .= "Text Processing: " . (this.textProcessing ? "Enabled" : "Disabled") . "`n"
        report .= "Data Validation: " . (this.dataValidation ? "Enabled" : "Disabled") . "`n"
        report .= "Cache Timeout: " . this.cacheTimeout . "ms`n"
        report .= "Retry Attempts: " . this.retryAttempts . "`n`n"
        
        ; Recent extractions
        report .= "Recent Extractions:`n"
        recentCount := Min(10, this.extractionLog.Length)
        for i in Range(this.extractionLog.Length - recentCount + 1, this.extractionLog.Length) {
            extraction := this.extractionLog[i]
            status := extraction.success ? "✓" : "✗"
            cached := extraction.cached ? " (cached)" : ""
            report .= status . " " . extraction.control . " -> '" . SubStr(extraction.text, 1, 30) . "'" . cached . "`n"
        }
        
        return report
    }
    
    static ClearCache() {
        this.extractionCache.Clear()
    }
    
    static EnableTextProcessing() {
        this.textProcessing := true
    }
    
    static DisableTextProcessing() {
        this.textProcessing := false
    }
    
    static EnableDataValidation() {
        this.dataValidation := true
    }
    
    static DisableDataValidation() {
        this.dataValidation := false
    }
}

; Data collection and form processing system
class FormProcessor {
    static formDefinitions := Map()
    static collectedData := []
    static processingRules := Map()
    
    static Initialize() {
        this.formDefinitions.Clear()
        this.collectedData := []
        this.SetupDefaultProcessingRules()
    }
    
    static DefineForm(formName, definition) {
        ; Define form structure for automated processing
        this.formDefinitions[formName] := {
            name: formName,
            windowSpec: definition.windowSpec,
            fields: definition.fields,
            validation: definition.HasProp("validation") ? definition.validation : Map(),
            processing: definition.HasProp("processing") ? definition.processing : Map()
        }
    }
    
    static ProcessForm(formName, options := {}) {
        ; Process entire form and collect data
        if (!this.formDefinitions.Has(formName)) {
            throw ValueError("Form definition '" . formName . "' not found")
        }
        
        formDef := this.formDefinitions[formName]
        
        ; Extract form data
        formData := TextExtractor.ExtractFormData(formDef, formDef.windowSpec, options)
        
        ; Apply form-specific processing
        if (formDef.processing.Count > 0) {
            this.ApplyFormProcessing(formData, formDef.processing)
        }
        
        ; Store collected data
        formData.timestamp := A_TickCount
        formData.formattedTime := FormatTime()
        this.collectedData.Push(formData)
        
        return formData
    }
    
    static ApplyFormProcessing(formData, processingRules) {
        ; Apply processing rules to form data
        for fieldName, rule in processingRules {
            if (formData.data.Has(fieldName)) {
                try {
                    originalValue := formData.data[fieldName]
                    processedValue := this.ApplyProcessingRule(originalValue, rule)
                    formData.data[fieldName] := processedValue
                } catch Error as err {
                    ; Log processing error but continue
                    OutputDebug("Form processing error for field " . fieldName . ": " . err.Message)
                }
            }
        }
    }
    
    static ApplyProcessingRule(value, rule) {
        ; Apply individual processing rule
        switch rule.type {
            case "normalize":
                return this.NormalizeValue(value, rule)
            case "calculate":
                return this.CalculateValue(value, rule)
            case "lookup":
                return this.LookupValue(value, rule)
            case "format":
                return this.FormatValue(value, rule)
            default:
                return value
        }
    }
    
    static NormalizeValue(value, rule) {
        ; Normalize value according to rule
        if (rule.HasProp("removeChars")) {
            for char in rule.removeChars {
                value := StrReplace(value, char, "")
            }
        }
        
        if (rule.HasProp("replaceChars")) {
            for replacement in rule.replaceChars {
                value := StrReplace(value, replacement.from, replacement.to)
            }
        }
        
        if (rule.HasProp("case")) {
            switch rule.case {
                case "upper":
                    value := StrUpper(value)
                case "lower":
                    value := StrLower(value)
                case "title":
                    value := StrTitle(value)
            }
        }
        
        return value
    }
    
    static CalculateValue(value, rule) {
        ; Calculate derived value
        if (rule.HasProp("operation")) {
            switch rule.operation {
                case "multiply":
                    return Float(value) * rule.factor
                case "add":
                    return Float(value) + rule.addend
                case "percentage":
                    return Round(Float(value) * 100, rule.HasProp("decimals") ? rule.decimals : 2)
            }
        }
        
        return value
    }
    
    static LookupValue(value, rule) {
        ; Look up value in mapping table
        if (rule.HasProp("mappings") && rule.mappings.Has(value)) {
            return rule.mappings[value]
        }
        
        return rule.HasProp("default") ? rule.default : value
    }
    
    static FormatValue(value, rule) {
        ; Format value according to rule
        if (rule.HasProp("template")) {
            return StrReplace(rule.template, "{value}", value)
        }
        
        return value
    }
    
    static SetupDefaultProcessingRules() {
        ; Set up default processing rules
        this.processingRules := Map()
    }
    
    static ExportCollectedData(format := "CSV", filePath := "") {
        ; Export collected form data
        if (!filePath) {
            filePath := A_ScriptDir . "\collected_data_" . FormatTime(, "yyyyMMdd_HHmmss") . "." . StrLower(format)
        }
        
        switch StrUpper(format) {
            case "CSV":
                this.ExportToCSV(filePath)
            case "JSON":
                this.ExportToJSON(filePath)
            case "XML":
                this.ExportToXML(filePath)
            default:
                throw ValueError("Unsupported export format: " . format)
        }
        
        return filePath
    }
    
    static ExportToCSV(filePath) {
        ; Export data to CSV format
        if (this.collectedData.Length = 0) {
            throw Error("No data to export")
        }
        
        ; Get all unique field names
        allFields := Map()
        for formData in this.collectedData {
            for fieldName in formData.data {
                allFields[fieldName] := true
            }
        }
        
        ; Create CSV header
        csvContent := "Timestamp,FormName"
        for fieldName in allFields {
            csvContent .= "," . fieldName
        }
        csvContent .= "`n"
        
        ; Add data rows
        for formData in this.collectedData {
            csvContent .= formData.formattedTime . "," . formData.formName
            
            for fieldName in allFields {
                value := formData.data.Has(fieldName) ? formData.data[fieldName] : ""
                ; Escape commas and quotes in CSV
                if (InStr(value, ",") || InStr(value, '"')) {
                    value := '"' . StrReplace(value, '"', '""') . '"'
                }
                csvContent .= "," . value
            }
            csvContent .= "`n"
        }
        
        FileWrite(csvContent, filePath)
    }
    
    static ExportToJSON(filePath) {
        ; Export data to JSON format (simplified)
        jsonContent := "["
        
        for i, formData in this.collectedData {
            if (i > 1) {
                jsonContent .= ","
            }
            
            jsonContent .= "`n  {"
            jsonContent .= "`n    ""timestamp"": """ . formData.formattedTime . ""","
            jsonContent .= "`n    ""formName"": """ . formData.formName . ""","
            jsonContent .= "`n    ""data"": {"
            
            fieldCount := 0
            for fieldName, value in formData.data {
                if (fieldCount > 0) {
                    jsonContent .= ","
                }
                jsonContent .= "`n      """ . fieldName . """: """ . StrReplace(String(value), '"', '\"') . """"
                fieldCount++
            }
            
            jsonContent .= "`n    }"
            jsonContent .= "`n  }"
        }
        
        jsonContent .= "`n]"
        
        FileWrite(jsonContent, filePath)
    }
    
    static GetCollectedDataSummary() {
        ; Get summary of collected data
        summary := {
            totalForms: this.collectedData.Length,
            formTypes: Map(),
            fieldsSummary: Map(),
            dateRange: {earliest: "", latest: ""}
        }
        
        for formData in this.collectedData {
            ; Count form types
            formType := formData.formName
            if (!summary.formTypes.Has(formType)) {
                summary.formTypes[formType] := 0
            }
            summary.formTypes[formType]++
            
            ; Update date range
            if (!summary.dateRange.earliest || formData.formattedTime < summary.dateRange.earliest) {
                summary.dateRange.earliest := formData.formattedTime
            }
            if (!summary.dateRange.latest || formData.formattedTime > summary.dateRange.latest) {
                summary.dateRange.latest := formData.formattedTime
            }
            
            ; Count fields
            for fieldName in formData.data {
                if (!summary.fieldsSummary.Has(fieldName)) {
                    summary.fieldsSummary[fieldName] := 0
                }
                summary.fieldsSummary[fieldName]++
            }
        }
        
        return summary
    }
}

; Example usage and demonstrations
; Initialize text extraction system
TextExtractor.Initialize()

; Basic text extraction examples
editText := ControlGetText("Edit1", "Notepad")
buttonCaption := ControlGetText("Button1", "Calculator")
statusMessage := ControlGetText("StatusBar", "Application")

; Advanced text extraction with options
extractedText := TextExtractor.ExtractText("Edit1", {title: "Form"}, {
    processText: true,
    transform: "lowercase",
    collapseWhitespace: true,
    validate: true,
    minLength: 1,
    maxLength: 100
})

; Extract multiple controls
controlSpecs := Map(
    "name", "Edit1",
    "email", "Edit2", 
    "phone", "Edit3",
    "comments", "Edit4"
)

results := TextExtractor.ExtractMultipleControls(controlSpecs, {title: "Contact Form"})

if (results.successCount > 0) {
    for fieldName, text in results.results {
        OutputDebug(fieldName . ": " . text)
    }
}

; Form processing example
FormProcessor.Initialize()

; Define contact form
FormProcessor.DefineForm("ContactForm", {
    windowSpec: {title: "Contact Information"},
    fields: Map(
        "firstName", {control: "Edit1", dataType: "string"},
        "lastName", {control: "Edit2", dataType: "string"},
        "email", {control: "Edit3", dataType: "string"},
        "phone", {control: "Edit4", dataType: "string"},
        "age", {control: "Edit5", dataType: "integer"},
        "salary", {control: "Edit6", dataType: "currency"}
    ),
    processing: Map(
        "firstName", {type: "normalize", case: "title"},
        "lastName", {type: "normalize", case: "title"},
        "email", {type: "normalize", case: "lower"},
        "phone", {type: "normalize", removeChars: ["(", ")", "-", " "]}
    )
})

; Process contact form
try {
    formData := FormProcessor.ProcessForm("ContactForm")
    
    if (formData.success) {
        MsgBox("Form processed successfully! Extracted " . formData.extractedFields . " fields.")
        
        ; Display extracted data
        for fieldName, value in formData.data {
            OutputDebug(fieldName . ": " . value)
        }
    } else {
        MsgBox("Form processing had " . formData.errors.Length . " errors.")
    }
    
} catch Error as err {
    MsgBox("Form processing failed: " . err.Message)
}

; Text change monitoring
monitor := TextExtractor.MonitorTextChanges("Edit1", {title: "Live Data"}, 
    (newText, oldText, control, window) => {
        OutputDebug("Text changed from '" . oldText . "' to '" . newText . "'")
        ToolTip("Text changed: " . newText, 10, 10)
        SetTimer(() => ToolTip(), -2000)
    }, 
    {interval: 500, timeout: 30000}
)

; Wait for specific text change
try {
    finalText := TextExtractor.WaitForTextChange("StatusBar", {title: "Processing"}, "Complete", 15000)
    MsgBox("Operation completed! Final status: " . finalText)
} catch TimeoutError {
    MsgBox("Operation did not complete within timeout")
}

; Data extraction with validation
TextExtractor.AddValidationRule("EmailEdit", {
    type: "pattern", 
    pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
})

try {
    emailText := TextExtractor.ExtractText("EmailEdit", {title: "Registration"}, {
        validate: true,
        transform: "lowercase"
    })
    
    MsgBox("Valid email extracted: " . emailText)
    
} catch Error as err {
    MsgBox("Email validation failed: " . err.Message)
}

; Batch data collection from multiple windows
windows := [
    {title: "Form 1", type: "ContactForm"},
    {title: "Form 2", type: "ContactForm"},
    {title: "Form 3", type: "ContactForm"}
]

collectedData := []

for window in windows {
    try {
        if (WinExist(window.title)) {
            formData := FormProcessor.ProcessForm(window.type)
            formData.windowTitle := window.title
            collectedData.Push(formData)
        }
    } catch Error as err {
        OutputDebug("Failed to process " . window.title . ": " . err.Message)
    }
}

; Export collected data
if (collectedData.Length > 0) {
    try {
        exportPath := FormProcessor.ExportCollectedData("CSV")
        MsgBox("Data exported to: " . exportPath)
    } catch Error as err {
        MsgBox("Export failed: " . err.Message)
    }
}

; Text extraction with caching and performance monitoring
TextExtractor.EnableTextProcessing()
TextExtractor.EnableDataValidation()

; Extract frequently accessed data
Loop 5 {
    text := TextExtractor.ExtractText("StatusBar", {title: "Application"}, {
        useCache: true,
        processText: true,
        transform: "uppercase"
    })
    
    OutputDebug("Iteration " . A_Index . ": " . text)
}

; Get extraction statistics
stats := TextExtractor.GetExtractionStatistics()
MsgBox("Extraction Statistics:`n" .
       "Total: " . stats.total . "`n" .
       "Success Rate: " . stats.successRate . "%`n" .
       "Cache Hit Rate: " . stats.cacheHitRate . "%")

; Generate extraction report
extractionReport := TextExtractor.GenerateExtractionReport()
; MsgBox(extractionReport, "Text Extraction Report")

; Complex text processing example
complexText := TextExtractor.ExtractText("Edit1", {title: "Data Entry"}, {
    processText: true,
    normalizeLineEndings: true,
    collapseWhitespace: true,
    regexTransforms: [
        {pattern: "\d{3}-\d{3}-\d{4}", replacement: "XXX-XXX-XXXX"},  ; Mask phone numbers
        {pattern: "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}", replacement: "***@***.***"}  ; Mask emails
    ],
    validate: true,
    minLength: 10,
    forbiddenPattern: "password|secret|confidential"
})

; Automated data validation across multiple forms
validationResults := []

formTypes := ["ContactForm", "OrderForm", "FeedbackForm"]

for formType in formTypes {
    try {
        formData := FormProcessor.ProcessForm(formType)
        
        validationResult := {
            formType: formType,
            isValid: formData.success,
            fieldCount: formData.extractedFields,
            errorCount: formData.errors.Length,
            errors: formData.errors
        }
        
        validationResults.Push(validationResult)
        
    } catch Error as err {
        validationResults.Push({
            formType: formType,
            isValid: false,
            error: err.Message
        })
    }
}

; Summary of validation results
validCount := 0
for result in validationResults {
    if (result.isValid) {
        validCount++
    }
}

MsgBox("Validation Summary:`n" .
       "Forms Validated: " . validationResults.Length . "`n" .
       "Valid Forms: " . validCount . "`n" .
       "Success Rate: " . Round((validCount / validationResults.Length) * 100, 1) . "%")

; Cleanup and resource management
; monitor.stop()  ; Stop text monitoring
TextExtractor.ClearCache()  ; Clear extraction cache
```

## Implementation Notes

**Control Type Compatibility:**
- Different control types return text in different formats and may have specific behavior patterns
- Edit controls return current input text, while static controls return display text
- List controls may return only selected items, requiring different approaches for full content extraction
- Some controls may not have accessible text content or may require alternative extraction methods

**Text Encoding and Character Handling:**
- Control text may contain special characters, unicode content, or formatting that affects extraction
- Consider text encoding issues when processing extracted content, especially with international applications
- Handle control text that contains newlines, tabs, or other whitespace characters appropriately
- Be aware of text truncation in controls with limited display space versus actual content

**Performance and Caching Considerations:**
- Text extraction can be expensive for large controls or frequent polling operations
- Implement intelligent caching to reduce redundant extraction operations while maintaining data freshness
- Consider the performance impact of text processing and validation on high-frequency extraction scenarios
- Monitor memory usage when caching large amounts of extracted text data

**Error Handling and Reliability:**
- Controls may become unavailable, disabled, or hidden during extraction operations
- Implement robust retry mechanisms for transient failures related to timing or application state
- Handle cases where control text is empty, null, or contains unexpected content gracefully
- Provide meaningful error messages that help identify specific extraction failures

**Data Validation and Security:**
- Validate extracted text against expected patterns and formats to ensure data integrity
- Be cautious when extracting sensitive information like passwords or personal data
- Implement sanitization for extracted text that will be processed or stored to prevent injection attacks
- Consider data privacy implications when logging or caching extracted text content

## Related AHK Concepts

- [ControlClick](./controlclick.md) - Clicking controls for interaction before text extraction
- [ControlSetText](../../10_Language_Core/01-Functions/Built_In_Functions/controlsettext.md) - Setting control text values
- [WinGetText](../../40_Advanced_Features/05-System_Integration/wingettext.md) - Extracting text from entire windows
- [ControlGetPos](../../10_Language_Core/01-Functions/Built_In_Functions/controlgetpos.md) - Getting control position information
- [RegExMatch](../../10_Language_Core/01-Functions/Built_In_Functions/regexmatch.md) - Pattern matching in extracted text

## Tags

#AutoHotkey #ControlGetText #TextExtraction #DataExtraction #GUIAutomation #FormProcessing #DataCollection #TextProcessing #DataValidation #ApplicationIntegration
