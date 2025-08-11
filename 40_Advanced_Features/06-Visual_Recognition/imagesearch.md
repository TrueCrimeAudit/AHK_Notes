# Topic: ImageSearch Function - Advanced Image Pattern Recognition

## Category

Function

## Overview

The ImageSearch function locates specific image patterns within screen regions, providing sophisticated visual recognition capabilities for complex automation, testing applications, and user interface detection. It enables scripts to identify UI elements, icons, text patterns, and graphical content regardless of exact positioning, making it essential for robust automation that must adapt to interface changes and variations.

## Key Points

- Searches for image files within specified screen regions with configurable similarity tolerance
- Provides coordinates of matching image locations, enabling precise targeting for automation actions
- Supports various image formats and handles size variations, rotation, and transparency differences
- Essential for cross-platform automation, UI testing, game automation, and applications requiring visual element detection
- More robust than color-based detection for complex graphical interfaces and dynamic content

## Syntax and Parameters

```cpp
ImageSearch(&OutputVarX, &OutputVarY, X1, Y1, X2, Y2, ImageFile)

; Parameters:
; OutputVarX   - Variable to store X coordinate of found image's top-left corner
; OutputVarY   - Variable to store Y coordinate of found image's top-left corner
; X1, Y1       - Upper-left corner coordinates of search area
; X2, Y2       - Lower-right corner coordinates of search area
; ImageFile    - Path to image file to search for

; Supported image formats:
; .bmp, .jpg, .jpeg, .png, .gif, .tif, .tiff, .ico

; Image file path options:
; "button.png"           - Relative to script directory
; "C:\\Images\\icon.bmp" - Absolute path
; "*50 image.png"        - Variation tolerance (0-255)
; "*Trans0xFF0000 img.png" - Transparent color
; "*w100 *h50 image.png"   - Scale image to specific size

; Advanced options in filename:
; *n              - Variation tolerance (0-255, where 0 = exact match)
; *TransN         - Transparent color (N = RGB value)
; *wN *hN         - Scale to width/height N
; *Icon2          - Use specific icon from .ico/.exe/.dll file

; Search area definition:
; (0, 0)          - Top-left corner of primary monitor
; (X1, Y1)        - Upper-left corner of search rectangle
; (X2, Y2)        - Lower-right corner of search rectangle
; Coordinates are in screen pixels

; Return behavior:
; Returns 1        - If image found, coordinates stored in output variables
; Returns 0        - If image not found, output variables unchanged
; Throws error     - If image file not found or search area invalid

; Coordinate result:
; OutputVarX, OutputVarY contain top-left corner of found image
; Add image dimensions to get center or other reference points
```

## Code Examples

```cpp
; Basic image detection
if (ImageSearch(&foundX, &foundY, 0, 0, A_ScreenWidth, A_ScreenHeight, "button.png")) {
    MsgBox("Button found at: " . foundX . ", " . foundY)
    ; Click center of found image
    Click(foundX + 50, foundY + 25)  ; Assuming button is 100x50 pixels
} else {
    MsgBox("Button not found on screen")
}

; Advanced image recognition and template management system
class ImageRecognition {
    static imageTemplates := Map()
    static searchCache := Map()
    static cacheTimeout := 5000
    static searchHistory := []
    static recognitionRules := Map()
    static performanceMetrics := {searches: 0, hits: 0, misses: 0, cacheHits: 0, totalTime: 0}
    static imageScaling := true
    static transparencyHandling := true
    static multiScaleSearch := false
    static recognitionLogging := true
    static templateValidation := true
    
    static Initialize() {
        this.imageTemplates.Clear()
        this.searchCache.Clear()
        this.searchHistory := []
        this.ResetPerformanceMetrics()
        this.SetupDefaultTemplates()
    }
    
    static RegisterTemplate(templateId, imagePath, options := {}) {
        ; Register image template for recognition
        if (this.templateValidation && !FileExist(imagePath)) {
            throw FileNotFoundError("Template image file not found: " . imagePath)
        }
        
        template := {
            templateId: templateId,
            imagePath: imagePath,
            options: options,
            registered: A_TickCount,
            searchCount: 0,
            hitCount: 0,
            lastFound: 0,
            averageSearchTime: 0
        }
        
        ; Process template options
        if (options.HasProp("variation")) {
            template.variation := options.variation
        }
        
        if (options.HasProp("transparent")) {
            template.transparent := options.transparent
        }
        
        if (options.HasProp("scaling")) {
            template.scaling := options.scaling
        }
        
        if (options.HasProp("description")) {
            template.description := options.description
        }
        
        this.imageTemplates[templateId] := template
        return templateId
    }
    
    static SearchImage(templateId, searchArea, options := {}) {
        ; Advanced image search with caching and optimization
        if (!this.imageTemplates.Has(templateId)) {
            throw ValueError("Image template '" . templateId . "' not registered")
        }
        
        template := this.imageTemplates[templateId]
        startTime := A_TickCount
        
        ; Create search record
        searchRecord := {
            templateId: templateId,
            timestamp: A_TickCount,
            searchArea: searchArea,
            foundX: 0, foundY: 0,
            found: false,
            cached: false,
            searchTime: 0,
            variation: 0,
            scaled: false,
            error: ""
        }
        
        try {
            ; Extract search parameters
            x1 := searchArea.HasProp("x1") ? searchArea.x1 : 0
            y1 := searchArea.HasProp("y1") ? searchArea.y1 : 0
            x2 := searchArea.HasProp("x2") ? searchArea.x2 : A_ScreenWidth
            y2 := searchArea.HasProp("y2") ? searchArea.y2 : A_ScreenHeight
            
            ; Process search options
            useCache := options.HasProp("useCache") ? options.useCache : true
            variation := options.HasProp("variation") ? options.variation : (template.HasProp("variation") ? template.variation : 0)
            allowScaling := options.HasProp("allowScaling") ? options.allowScaling : this.imageScaling
            
            searchRecord.variation := variation
            
            ; Check cache first
            if (useCache) {
                cacheKey := this.GenerateCacheKey(templateId, x1, y1, x2, y2, variation)
                if (this.searchCache.Has(cacheKey)) {
                    cacheEntry := this.searchCache[cacheKey]
                    if (A_TickCount - cacheEntry.timestamp < this.cacheTimeout) {
                        searchRecord.foundX := cacheEntry.x
                        searchRecord.foundY := cacheEntry.y
                        searchRecord.found := cacheEntry.found
                        searchRecord.cached := true
                        searchRecord.searchTime := A_TickCount - startTime
                        
                        this.UpdatePerformanceMetrics("cache_hit", searchRecord.searchTime)
                        this.LogSearch(searchRecord)
                        return searchRecord
                    }
                }
            }
            
            ; Build image search path with options
            imagePath := this.BuildImagePath(template, options)
            
            ; Perform image search
            foundX := 0
            foundY := 0
            found := ImageSearch(&foundX, &foundY, x1, y1, x2, y2, imagePath)
            
            ; Update search record
            searchRecord.found := found
            searchRecord.foundX := foundX
            searchRecord.foundY := foundY
            searchRecord.searchTime := A_TickCount - startTime
            
            ; Try multi-scale search if enabled and initial search failed
            if (!found && allowScaling && this.multiScaleSearch) {
                scaleResult := this.MultiScaleSearch(template, x1, y1, x2, y2, options)
                if (scaleResult.found) {
                    searchRecord.found := true
                    searchRecord.foundX := scaleResult.x
                    searchRecord.foundY := scaleResult.y
                    searchRecord.scaled := true
                    searchRecord.searchTime := A_TickCount - startTime
                }
            }
            
            ; Update template statistics
            template.searchCount++
            if (searchRecord.found) {
                template.hitCount++
                template.lastFound := A_TickCount
            }
            
            if (template.searchCount > 0) {
                template.averageSearchTime := (template.averageSearchTime * (template.searchCount - 1) + searchRecord.searchTime) / template.searchCount
            }
            
            ; Cache result
            if (useCache) {
                this.CacheSearchResult(templateId, x1, y1, x2, y2, variation, searchRecord)
            }
            
            ; Update performance metrics
            this.UpdatePerformanceMetrics(found ? "hit" : "miss", searchRecord.searchTime)
            
        } catch Error as err {
            searchRecord.error := err.Message
            this.UpdatePerformanceMetrics("error", A_TickCount - startTime)
        }
        
        ; Log search operation
        this.LogSearch(searchRecord)
        return searchRecord
    }
    
    static SearchMultipleImages(templateIds, searchArea, options := {}) {
        ; Search for multiple images simultaneously
        results := Map()
        searchResults := []
        
        for templateId in templateIds {
            searchResult := this.SearchImage(templateId, searchArea, options)
            results[templateId] := searchResult
            searchResults.Push(searchResult)
        }
        
        return {
            results: results,
            searchResults: searchResults,
            foundCount: searchResults.Filter(r => r.found).Length,
            totalCount: searchResults.Length,
            summary: this.GenerateSearchSummary(searchResults)
        }
    }
    
    static FindBestMatch(templateIds, searchArea, options := {}) {
        ; Find the best matching image from multiple templates
        multiResult := this.SearchMultipleImages(templateIds, searchArea, options)
        
        if (multiResult.foundCount = 0) {
            return {found: false, reason: "No templates matched"}
        }
        
        ; Find template with best match (could be enhanced with confidence scoring)
        bestMatch := ""
        bestResult := ""
        shortestSearchTime := 999999
        
        for templateId, result in multiResult.results {
            if (result.found && result.searchTime < shortestSearchTime) {
                shortestSearchTime := result.searchTime
                bestMatch := templateId
                bestResult := result
            }
        }
        
        return {
            found: true,
            bestMatch: bestMatch,
            result: bestResult,
            allResults: multiResult
        }
    }
    
    static WaitForImage(templateId, searchArea, timeout := 10000, options := {}) {
        ; Wait for image to appear
        startTime := A_TickCount
        checkInterval := options.HasProp("checkInterval") ? options.checkInterval : 500
        
        while (A_TickCount - startTime < timeout) {
            searchResult := this.SearchImage(templateId, searchArea, options)
            
            if (searchResult.found) {
                return searchResult
            }
            
            Sleep(checkInterval)
        }
        
        throw TimeoutError("Image '" . templateId . "' not found within timeout period")
    }
    
    static WaitForImageToDisappear(templateId, searchArea, timeout := 10000, options := {}) {
        ; Wait for image to disappear
        startTime := A_TickCount
        checkInterval := options.HasProp("checkInterval") ? options.checkInterval : 500
        
        while (A_TickCount - startTime < timeout) {
            searchResult := this.SearchImage(templateId, searchArea, options)
            
            if (!searchResult.found) {
                return true
            }
            
            Sleep(checkInterval)
        }
        
        throw TimeoutError("Image '" . templateId . "' did not disappear within timeout period")
    }
    
    static MonitorImageChanges(templateIds, searchArea, callback, options := {}) {
        ; Monitor area for image changes
        monitorId := options.HasProp("monitorId") ? options.monitorId : "monitor_" . A_TickCount
        checkInterval := options.HasProp("interval") ? options.interval : 2000
        timeout := options.HasProp("timeout") ? options.timeout : 0
        
        ; Initial search to establish baseline
        lastResults := Map()
        for templateId in templateIds {
            result := this.SearchImage(templateId, searchArea, options)
            lastResults[templateId] := result.found
        }
        
        monitorData := {
            monitorId: monitorId,
            templateIds: templateIds,
            searchArea: searchArea,
            options: options,
            callback: callback,
            lastResults: lastResults,
            startTime: A_TickCount,
            timeout: timeout,
            checkCount: 0,
            changeCount: 0
        }
        
        ; Create monitoring timer
        timerFunction := () => {
            try {
                monitorData.checkCount++
                changes := []
                
                for templateId in templateIds {
                    currentResult := this.SearchImage(templateId, searchArea, options)
                    lastFound := monitorData.lastResults[templateId]
                    currentFound := currentResult.found
                    
                    if (currentFound != lastFound) {
                        changes.Push({
                            templateId: templateId,
                            action: currentFound ? "appeared" : "disappeared",
                            result: currentResult
                        })
                        
                        monitorData.lastResults[templateId] := currentFound
                    }
                }
                
                ; Trigger callback if changes detected
                if (changes.Length > 0 && IsFunc(callback)) {
                    monitorData.changeCount++
                    
                    callbackData := {
                        monitorId: monitorId,
                        changes: changes,
                        checkCount: monitorData.checkCount,
                        changeCount: monitorData.changeCount,
                        elapsedTime: A_TickCount - monitorData.startTime
                    }
                    
                    callback.Call(callbackData)
                }
                
                ; Check timeout
                if (timeout > 0 && A_TickCount - monitorData.startTime > timeout) {
                    this.StopImageMonitoring(monitorId)
                }
                
            } catch Error as err {
                OutputDebug("Image monitor error: " . err.Message)
                this.StopImageMonitoring(monitorId)
            }
        }
        
        ; Store monitoring data and start timer
        if (!this.HasProp("monitoringTasks")) {
            this.monitoringTasks := Map()
        }
        
        this.monitoringTasks[monitorId] := {
            data: monitorData,
            timer: timerFunction
        }
        
        SetTimer(timerFunction, checkInterval)
        
        return {
            monitorId: monitorId,
            stop: () => this.StopImageMonitoring(monitorId),
            getStatus: () => this.GetImageMonitorStatus(monitorId)
        }
    }
    
    static StopImageMonitoring(monitorId) {
        ; Stop image monitoring
        if (this.HasProp("monitoringTasks") && this.monitoringTasks.Has(monitorId)) {
            monitorTask := this.monitoringTasks[monitorId]
            SetTimer(monitorTask.timer, 0)
            this.monitoringTasks.Delete(monitorId)
            return true
        }
        return false
    }
    
    static GetImageMonitorStatus(monitorId) {
        ; Get image monitoring status
        if (this.HasProp("monitoringTasks") && this.monitoringTasks.Has(monitorId)) {
            return this.monitoringTasks[monitorId].data
        }
        return false
    }
    
    static MultiScaleSearch(template, x1, y1, x2, y2, options) {
        ; Search image at multiple scales
        scales := options.HasProp("scales") ? options.scales : [0.8, 0.9, 1.1, 1.2, 1.5]
        
        for scale in scales {
            try {
                scaledOptions := options.Clone()
                scaledOptions.width := Round(100 * scale)
                scaledOptions.height := Round(100 * scale)
                
                imagePath := this.BuildImagePath(template, scaledOptions)
                
                foundX := 0
                foundY := 0
                found := ImageSearch(&foundX, &foundY, x1, y1, x2, y2, imagePath)
                
                if (found) {
                    return {found: true, x: foundX, y: foundY, scale: scale}
                }
                
            } catch Error {
                ; Continue to next scale
                continue
            }
        }
        
        return {found: false}
    }
    
    static BuildImagePath(template, options) {
        ; Build image path with search options
        imagePath := template.imagePath
        pathPrefix := ""
        
        ; Add variation tolerance
        if (options.HasProp("variation") && options.variation > 0) {
            pathPrefix .= "*" . options.variation . " "
        } else if (template.HasProp("variation") && template.variation > 0) {
            pathPrefix .= "*" . template.variation . " "
        }
        
        ; Add transparency
        if (options.HasProp("transparent")) {
            pathPrefix .= "*Trans" . options.transparent . " "
        } else if (template.HasProp("transparent")) {
            pathPrefix .= "*Trans" . template.transparent . " "
        }
        
        ; Add scaling
        if (options.HasProp("width") && options.HasProp("height")) {
            pathPrefix .= "*w" . options.width . " *h" . options.height . " "
        } else if (template.HasProp("scaling")) {
            pathPrefix .= "*w" . template.scaling.width . " *h" . template.scaling.height . " "
        }
        
        ; Add icon index
        if (options.HasProp("iconIndex")) {
            pathPrefix .= "*Icon" . options.iconIndex . " "
        }
        
        return pathPrefix . imagePath
    }
    
    static CreateImageSequence(sequenceId, templateIds, options := {}) {
        ; Create sequence of images to search for in order
        sequence := {
            sequenceId: sequenceId,
            templateIds: templateIds,
            options: options,
            created: A_TickCount
        }
        
        this.recognitionRules[sequenceId] := sequence
        return sequenceId
    }
    
    static SearchImageSequence(sequenceId, searchArea, options := {}) {
        ; Search for image sequence
        if (!this.recognitionRules.Has(sequenceId)) {
            throw ValueError("Image sequence '" . sequenceId . "' not found")
        }
        
        sequence := this.recognitionRules[sequenceId]
        sequenceResults := []
        allFound := true
        
        for templateId in sequence.templateIds {
            searchResult := this.SearchImage(templateId, searchArea, options)
            sequenceResults.Push(searchResult)
            
            if (!searchResult.found) {
                allFound := false
                break  ; Stop at first missing image in sequence
            }
        }
        
        return {
            sequenceId: sequenceId,
            found: allFound,
            results: sequenceResults,
            completedSteps: sequenceResults.Length,
            totalSteps: sequence.templateIds.Length
        }
    }
    
    static AnalyzeScreenRegion(x1, y1, x2, y2, templateIds, options := {}) {
        ; Analyze screen region for multiple images
        analysis := {
            region: {x1: x1, y1: y1, x2: x2, y2: y2},
            timestamp: A_TickCount,
            templateResults: Map(),
            summary: {},
            analysisTime: 0
        }
        
        startTime := A_TickCount
        
        searchArea := {x1: x1, y1: y1, x2: x2, y2: y2}
        
        for templateId in templateIds {
            searchResult := this.SearchImage(templateId, searchArea, options)
            analysis.templateResults[templateId] := searchResult
        }
        
        analysis.analysisTime := A_TickCount - startTime
        analysis.summary := this.GenerateAnalysisSummary(analysis.templateResults)
        
        return analysis
    }
    
    static GenerateAnalysisSummary(templateResults) {
        ; Generate summary of template analysis
        summary := {
            totalTemplates: templateResults.Count,
            foundTemplates: 0,
            notFoundTemplates: 0,
            averageSearchTime: 0,
            totalSearchTime: 0,
            foundImages: []
        }
        
        for templateId, result in templateResults {
            if (result.found) {
                summary.foundTemplates++
                summary.foundImages.Push({
                    templateId: templateId,
                    x: result.foundX,
                    y: result.foundY,
                    searchTime: result.searchTime
                })
            } else {
                summary.notFoundTemplates++
            }
            
            summary.totalSearchTime += result.searchTime
        }
        
        if (templateResults.Count > 0) {
            summary.averageSearchTime := Round(summary.totalSearchTime / templateResults.Count, 2)
        }
        
        summary.detectionRate := templateResults.Count > 0 ? Round((summary.foundTemplates / templateResults.Count) * 100, 1) : 0
        
        return summary
    }
    
    static GetTemplateStatistics(templateId) {
        ; Get statistics for specific template
        if (!this.imageTemplates.Has(templateId)) {
            throw ValueError("Template '" . templateId . "' not found")
        }
        
        template := this.imageTemplates[templateId]
        
        return {
            templateId: templateId,
            imagePath: template.imagePath,
            searchCount: template.searchCount,
            hitCount: template.hitCount,
            hitRate: template.searchCount > 0 ? Round((template.hitCount / template.searchCount) * 100, 1) : 0,
            averageSearchTime: Round(template.averageSearchTime, 2),
            lastFound: template.lastFound,
            timeSinceLastFound: template.lastFound > 0 ? A_TickCount - template.lastFound : -1
        }
    }
    
    static GenerateCacheKey(templateId, x1, y1, x2, y2, variation) {
        ; Generate cache key for search parameters
        return templateId . "|" . x1 . "," . y1 . "," . x2 . "," . y2 . "|" . variation
    }
    
    static CacheSearchResult(templateId, x1, y1, x2, y2, variation, searchRecord) {
        ; Cache search result
        cacheKey := this.GenerateCacheKey(templateId, x1, y1, x2, y2, variation)
        
        this.searchCache[cacheKey] := {
            found: searchRecord.found,
            x: searchRecord.foundX,
            y: searchRecord.foundY,
            timestamp: A_TickCount
        }
        
        ; Limit cache size
        if (this.searchCache.Count > 150) {
            this.CleanupCache()
        }
    }
    
    static CleanupCache() {
        ; Remove old cache entries
        currentTime := A_TickCount
        
        for cacheKey, entry in this.searchCache {
            if (currentTime - entry.timestamp > this.cacheTimeout) {
                this.searchCache.Delete(cacheKey)
            }
        }
    }
    
    static LogSearch(searchRecord) {
        ; Log search operation
        if (this.recognitionLogging) {
            this.searchHistory.Push(searchRecord)
            
            ; Limit history size
            if (this.searchHistory.Length > 500) {
                this.searchHistory.RemoveAt(1)
            }
        }
    }
    
    static UpdatePerformanceMetrics(operation, searchTime := 0) {
        ; Update performance metrics
        this.performanceMetrics.searches++
        this.performanceMetrics.totalTime += searchTime
        
        switch operation {
            case "hit":
                this.performanceMetrics.hits++
            case "miss":
                this.performanceMetrics.misses++
            case "cache_hit":
                this.performanceMetrics.cacheHits++
        }
    }
    
    static ResetPerformanceMetrics() {
        ; Reset performance metrics
        this.performanceMetrics := {
            searches: 0,
            hits: 0,
            misses: 0,
            cacheHits: 0,
            totalTime: 0
        }
    }
    
    static GenerateSearchSummary(searchResults) {
        ; Generate summary of search results
        summary := {
            total: searchResults.Length,
            found: 0,
            notFound: 0,
            cached: 0,
            scaled: 0,
            averageSearchTime: 0,
            totalSearchTime: 0,
            errors: 0
        }
        
        for result in searchResults {
            if (result.found) {
                summary.found++
            } else {
                summary.notFound++
            }
            
            if (result.cached) {
                summary.cached++
            }
            
            if (result.scaled) {
                summary.scaled++
            }
            
            if (result.error) {
                summary.errors++
            }
            
            summary.totalSearchTime += result.searchTime
        }
        
        if (searchResults.Length > 0) {
            summary.averageSearchTime := Round(summary.totalSearchTime / searchResults.Length, 2)
        }
        
        summary.successRate := searchResults.Length > 0 ? Round((summary.found / searchResults.Length) * 100, 1) : 0
        summary.cacheRate := searchResults.Length > 0 ? Round((summary.cached / searchResults.Length) * 100, 1) : 0
        
        return summary
    }
    
    static GetPerformanceReport() {
        ; Get comprehensive performance report
        metrics := this.performanceMetrics
        
        report := "Image Recognition Performance Report`n"
        report .= "=====================================`n`n"
        
        report .= "Search Statistics:`n"
        report .= "Total Searches: " . metrics.searches . "`n"
        report .= "Successful: " . metrics.hits . "`n"
        report .= "Not Found: " . metrics.misses . "`n"
        report .= "Cache Hits: " . metrics.cacheHits . "`n"
        report .= "Total Time: " . metrics.totalTime . "ms`n`n"
        
        if (metrics.searches > 0) {
            report .= "Success Rate: " . Round((metrics.hits / metrics.searches) * 100, 1) . "%`n"
            report .= "Cache Hit Rate: " . Round((metrics.cacheHits / metrics.searches) * 100, 1) . "%`n"
            report .= "Average Search Time: " . Round(metrics.totalTime / metrics.searches, 2) . "ms`n`n"
        }
        
        report .= "Configuration:`n"
        report .= "Image Scaling: " . (this.imageScaling ? "Enabled" : "Disabled") . "`n"
        report .= "Multi-scale Search: " . (this.multiScaleSearch ? "Enabled" : "Disabled") . "`n"
        report .= "Transparency Handling: " . (this.transparencyHandling ? "Enabled" : "Disabled") . "`n"
        report .= "Cache Timeout: " . this.cacheTimeout . "ms`n`n"
        
        report .= "Registered Templates: " . this.imageTemplates.Count . "`n"
        report .= "Cached Searches: " . this.searchCache.Count . "`n"
        report .= "Recognition Rules: " . this.recognitionRules.Count . "`n`n"
        
        ; Template statistics
        if (this.imageTemplates.Count > 0) {
            report .= "Template Performance:`n"
            for templateId, template in this.imageTemplates {
                if (template.searchCount > 0) {
                    hitRate := Round((template.hitCount / template.searchCount) * 100, 1)
                    report .= "• " . templateId . ": " . hitRate . "% (" . template.hitCount . "/" . template.searchCount . ")`n"
                }
            }
        }
        
        return report
    }
    
    static SetupDefaultTemplates() {
        ; Set up default template configurations
        this.recognitionRules.Clear()
    }
    
    static EnableImageScaling() {
        this.imageScaling := true
    }
    
    static DisableImageScaling() {
        this.imageScaling := false
    }
    
    static EnableMultiScaleSearch() {
        this.multiScaleSearch := true
    }
    
    static DisableMultiScaleSearch() {
        this.multiScaleSearch := false
    }
    
    static SetCacheTimeout(timeout) {
        if (timeout < 100) {
            throw ValueError("Cache timeout must be at least 100ms")
        }
        this.cacheTimeout := timeout
    }
    
    static ClearCache() {
        this.searchCache.Clear()
    }
    
    static ClearHistory() {
        this.searchHistory := []
    }
    
    static StopAllImageMonitoring() {
        ; Stop all image monitoring tasks
        if (this.HasProp("monitoringTasks")) {
            for monitorId in this.monitoringTasks {
                this.StopImageMonitoring(monitorId)
            }
        }
    }
}

; UI automation and screen interaction system
class UIAutomation {
    static uiElements := Map()
    static automationRules := Map()
    static interactionHistory := []
    
    static Initialize() {
        this.uiElements.Clear()
        this.automationRules.Clear()
        this.SetupDefaultElements()
    }
    
    static DefineUIElement(elementId, imageTemplate, options := {}) {
        ; Define UI element for automation
        element := {
            elementId: elementId,
            imageTemplate: imageTemplate,
            elementType: options.HasProp("type") ? options.type : "button",
            action: options.HasProp("action") ? options.action : "click",
            offset: options.HasProp("offset") ? options.offset : {x: 0, y: 0},
            searchArea: options.HasProp("searchArea") ? options.searchArea : {x1: 0, y1: 0, x2: A_ScreenWidth, y2: A_ScreenHeight},
            description: options.HasProp("description") ? options.description : "",
            timeout: options.HasProp("timeout") ? options.timeout : 5000
        }
        
        this.uiElements[elementId] := element
        return elementId
    }
    
    static InteractWithElement(elementId, options := {}) {
        ; Interact with UI element
        if (!this.uiElements.Has(elementId)) {
            throw ValueError("UI element '" . elementId . "' not defined")
        }
        
        element := this.uiElements[elementId]
        
        try {
            ; Search for element
            searchResult := ImageRecognition.SearchImage(element.imageTemplate, element.searchArea, options)
            
            if (!searchResult.found) {
                throw Error("UI element '" . elementId . "' not found on screen")
            }
            
            ; Calculate interaction coordinates
            targetX := searchResult.foundX + element.offset.x
            targetY := searchResult.foundY + element.offset.y
            
            ; Perform action
            switch element.action {
                case "click":
                    Click(targetX, targetY)
                case "rightclick":
                    Click(targetX, targetY, "Right")
                case "doubleclick":
                    Click(targetX, targetY, 2)
                case "hover":
                    MouseMove(targetX, targetY)
                default:
                    Click(targetX, targetY)
            }
            
            ; Log interaction
            interaction := {
                elementId: elementId,
                action: element.action,
                x: targetX, y: targetY,
                timestamp: A_TickCount,
                success: true
            }
            
            this.interactionHistory.Push(interaction)
            
            return {
                success: true,
                elementId: elementId,
                action: element.action,
                coordinates: {x: targetX, y: targetY},
                searchResult: searchResult
            }
            
        } catch Error as err {
            ; Log failed interaction
            interaction := {
                elementId: elementId,
                action: element.action,
                timestamp: A_TickCount,
                success: false,
                error: err.Message
            }
            
            this.interactionHistory.Push(interaction)
            throw err
        }
    }
    
    static WaitAndInteract(elementId, timeout := 10000, options := {}) {
        ; Wait for element to appear and interact
        element := this.uiElements[elementId]
        
        try {
            ; Wait for element to appear
            searchResult := ImageRecognition.WaitForImage(element.imageTemplate, element.searchArea, timeout, options)
            
            ; Interact with element
            return this.InteractWithElement(elementId, options)
            
        } catch TimeoutError {
            throw TimeoutError("UI element '" . elementId . "' did not appear within timeout")
        }
    }
    
    static CreateAutomationFlow(flowId, elementSequence, options := {}) {
        ; Create automation flow
        flow := {
            flowId: flowId,
            elementSequence: elementSequence,
            options: options,
            created: A_TickCount
        }
        
        this.automationRules[flowId] := flow
        return flowId
    }
    
    static ExecuteAutomationFlow(flowId, options := {}) {
        ; Execute automation flow
        if (!this.automationRules.Has(flowId)) {
            throw ValueError("Automation flow '" . flowId . "' not found")
        }
        
        flow := this.automationRules[flowId]
        flowResults := []
        
        for step in flow.elementSequence {
            try {
                elementId := step.HasProp("elementId") ? step.elementId : step
                stepOptions := step.HasProp("options") ? step.options : options
                stepDelay := step.HasProp("delay") ? step.delay : 500
                
                ; Execute step
                result := this.InteractWithElement(elementId, stepOptions)
                result.stepIndex := A_Index
                flowResults.Push(result)
                
                ; Wait between steps
                if (stepDelay > 0 && A_Index < flow.elementSequence.Length) {
                    Sleep(stepDelay)
                }
                
            } catch Error as err {
                flowResults.Push({
                    stepIndex: A_Index,
                    elementId: elementId,
                    success: false,
                    error: err.Message
                })
                
                ; Stop flow on error unless continue on error is enabled
                if (!options.HasProp("continueOnError") || !options.continueOnError) {
                    break
                }
            }
        }
        
        return {
            flowId: flowId,
            results: flowResults,
            completedSteps: flowResults.Length,
            totalSteps: flow.elementSequence.Length,
            success: flowResults.Length = flow.elementSequence.Length && flowResults.Filter(r => r.success).Length = flowResults.Length
        }
    }
    
    static SetupDefaultElements() {
        ; Set up default UI elements
        ; Would include common UI elements like close buttons, etc.
    }
    
    static GetInteractionHistory() {
        return this.interactionHistory
    }
    
    static ClearInteractionHistory() {
        this.interactionHistory := []
    }
}

; Example usage and demonstrations
; Initialize image recognition system
ImageRecognition.Initialize()
UIAutomation.Initialize()

; Basic image search
if (ImageSearch(&foundX, &foundY, 0, 0, A_ScreenWidth, A_ScreenHeight, "button.png")) {
    MsgBox("Button found at: " . foundX . ", " . foundY)
    Click(foundX + 25, foundY + 12)  ; Click center of 50x24 button
} else {
    MsgBox("Button not found")
}

; Register image templates
ImageRecognition.RegisterTemplate("login_button", "images\\login_btn.png", {
    variation: 20,
    description: "Main login button"
})

ImageRecognition.RegisterTemplate("logout_button", "images\\logout_btn.png", {
    variation: 15,
    transparent: 0xFF00FF,
    description: "Logout button with magenta transparency"
})

ImageRecognition.RegisterTemplate("app_icon", "images\\app_icon.ico", {
    variation: 10,
    scaling: {width: 32, height: 32}
})

; Advanced image search with registered template
searchArea := {x1: 100, y1: 100, x2: 500, y2: 400}

loginResult := ImageRecognition.SearchImage("login_button", searchArea, {
    variation: 25,
    useCache: true,
    allowScaling: true
})

if (loginResult.found) {
    MsgBox("Login button found at: " . loginResult.foundX . ", " . loginResult.foundY . 
           "`nSearch time: " . loginResult.searchTime . "ms" .
           (loginResult.cached ? " (cached)" : "") .
           (loginResult.scaled ? " (scaled)" : ""))
    
    ; Click on login button
    Click(loginResult.foundX + 50, loginResult.foundY + 15)
} else {
    MsgBox("Login button not found in search area")
}

; Multiple image search
buttonTemplates := ["login_button", "logout_button", "app_icon"]

multiResult := ImageRecognition.SearchMultipleImages(buttonTemplates, {
    x1: 0, y1: 0, x2: A_ScreenWidth, y2: A_ScreenHeight
}, {
    variation: 20,
    useCache: true
})

MsgBox("Multi-image search results:`n" .
       "Images found: " . multiResult.foundCount . "/" . multiResult.totalCount . "`n" .
       "Success rate: " . multiResult.summary.successRate . "%")

; Find best matching template
bestMatch := ImageRecognition.FindBestMatch(buttonTemplates, searchArea)

if (bestMatch.found) {
    MsgBox("Best match: " . bestMatch.bestMatch . "`n" .
           "Found at: " . bestMatch.result.foundX . ", " . bestMatch.result.foundY)
    
    ; Click on best matching element
    Click(bestMatch.result.foundX + 25, bestMatch.result.foundY + 12)
}

; Wait for image to appear
try {
    waitResult := ImageRecognition.WaitForImage("login_button", searchArea, 15000, {
        variation: 30,
        checkInterval: 1000
    })
    
    MsgBox("Login button appeared at: " . waitResult.foundX . ", " . waitResult.foundY)
    Click(waitResult.foundX + 50, waitResult.foundY + 15)
    
} catch TimeoutError {
    MsgBox("Login button did not appear within 15 seconds")
}

; Image monitoring
monitor := ImageRecognition.MonitorImageChanges(["login_button", "logout_button"], searchArea, 
    (callbackData) => {
        changeText := ""
        for change in callbackData.changes {
            changeText .= change.templateId . " " . change.action . "`n"
        }
        
        MsgBox("Image changes detected:`n" . changeText . 
               "Check count: " . callbackData.checkCount)
    }, 
    {
        monitorId: "ui_monitor",
        interval: 2000,
        timeout: 60000
    }
)

; Image sequence search
ImageRecognition.CreateImageSequence("login_flow", ["login_button", "username_field", "password_field"], {
    description: "Complete login interface"
})

sequenceResult := ImageRecognition.SearchImageSequence("login_flow", {
    x1: 0, y1: 0, x2: 800, y2: 600
})

if (sequenceResult.found) {
    MsgBox("Complete login sequence found! All " . sequenceResult.totalSteps . " elements present.")
} else {
    MsgBox("Login sequence incomplete: " . sequenceResult.completedSteps . "/" . sequenceResult.totalSteps . " found")
}

; Screen region analysis
analysisResult := ImageRecognition.AnalyzeScreenRegion(0, 0, 400, 300, 
    ["login_button", "logout_button", "app_icon"], {
    variation: 25
})

MsgBox("Screen analysis complete:`n" .
       "Region: 0,0 to 400,300`n" .
       "Templates found: " . analysisResult.summary.foundTemplates . "/" . analysisResult.summary.totalTemplates . "`n" .
       "Detection rate: " . analysisResult.summary.detectionRate . "%`n" .
       "Analysis time: " . analysisResult.analysisTime . "ms")

; UI automation with image recognition
UIAutomation.DefineUIElement("main_menu_button", "login_button", {
    type: "button",
    action: "click",
    offset: {x: 25, y: 12},  ; Click center of button
    searchArea: {x1: 0, y1: 0, x2: 200, y2: 100},
    description: "Main menu access button"
})

UIAutomation.DefineUIElement("settings_icon", "app_icon", {
    type: "icon",
    action: "doubleclick",
    offset: {x: 16, y: 16},  ; Click center of 32x32 icon
    description: "Settings configuration icon"
})

; Interact with UI elements
try {
    menuResult := UIAutomation.InteractWithElement("main_menu_button", {
        variation: 20
    })
    
    MsgBox("Menu button clicked successfully at: " . menuResult.coordinates.x . ", " . menuResult.coordinates.y)
    
} catch Error as err {
    MsgBox("Failed to click menu button: " . err.Message)
}

; Wait and interact
try {
    waitInteractResult := UIAutomation.WaitAndInteract("settings_icon", 10000, {
        variation: 15
    })
    
    MsgBox("Settings icon found and double-clicked!")
    
} catch TimeoutError {
    MsgBox("Settings icon did not appear")
}

; Automation flow
UIAutomation.CreateAutomationFlow("setup_workflow", [
    {elementId: "main_menu_button", delay: 1000},
    {elementId: "settings_icon", delay: 500},
    {elementId: "login_button", delay: 2000}
], {
    description: "Complete setup workflow"
})

flowResult := UIAutomation.ExecuteAutomationFlow("setup_workflow", {
    variation: 20,
    continueOnError: false
})

if (flowResult.success) {
    MsgBox("Automation flow completed successfully! " . flowResult.completedSteps . "/" . flowResult.totalSteps . " steps executed.")
} else {
    MsgBox("Automation flow failed at step " . flowResult.completedSteps . "/" . flowResult.totalSteps)
}

; Template performance analysis
for templateId in ["login_button", "logout_button", "app_icon"] {
    try {
        stats := ImageRecognition.GetTemplateStatistics(templateId)
        OutputDebug(templateId . " - Hit rate: " . stats.hitRate . "%, Avg time: " . stats.averageSearchTime . "ms")
    } catch Error {
        OutputDebug(templateId . " - No statistics available")
    }
}

; Multi-scale search example
ImageRecognition.EnableMultiScaleSearch()

scaleSearchResult := ImageRecognition.SearchImage("app_icon", {
    x1: 0, y1: 0, x2: A_ScreenWidth, y2: A_ScreenHeight
}, {
    variation: 20,
    allowScaling: true,
    scales: [0.5, 0.75, 1.0, 1.25, 1.5, 2.0]
})

if (scaleSearchResult.found) {
    MsgBox("App icon found" . (scaleSearchResult.scaled ? " (scaled)" : "") . 
           " at: " . scaleSearchResult.foundX . ", " . scaleSearchResult.foundY)
}

; Performance monitoring
performanceReport := ImageRecognition.GetPerformanceReport()
; MsgBox(performanceReport, "Image Recognition Performance")

; Cleanup and resource management
; monitor.stop()  ; Stop image monitoring
ImageRecognition.StopAllImageMonitoring()  ; Stop all monitoring
ImageRecognition.ClearCache()  ; Clear search cache
ImageRecognition.ClearHistory()  ; Clear search history
UIAutomation.ClearInteractionHistory()  ; Clear interaction history
```

## Implementation Notes

**Image File Requirements and Optimization:**
- ImageSearch works best with high-quality, uncompressed images that closely match the target appearance
- Consider image file size versus search performance trade-offs, as larger images require more processing time
- Use appropriate image formats: PNG for screenshots with transparency, BMP for fastest loading, JPG for photographs
- Ensure template images are captured at the same DPI and scaling as the target screen content

**Variation Tolerance and Matching Accuracy:**
- Variation tolerance balances detection reliability with false positive rates
- Lower values (0-30) for exact matches, higher values (50-100) for flexible matching across different display conditions
- Test variation settings across different monitors, scaling factors, and graphics settings to ensure consistent behavior
- Consider that compressed images, anti-aliasing, and color depth differences affect matching requirements

**Performance and Resource Management:**
- ImageSearch is computationally intensive, especially for large search areas and high variation tolerances
- Implement intelligent search area reduction and caching strategies to improve performance
- Consider using multiple smaller template images instead of one large image for better accuracy and speed
- Monitor memory usage when dealing with many template images or frequent searches

**Multi-Monitor and Scaling Considerations:**
- Handle different DPI settings and Windows display scaling that can affect image appearance
- Test template images across different monitor configurations and scaling factors
- Consider creating multiple template variants for different scaling levels if consistent matching is critical
- Account for color profile differences between monitors that may affect color-based matching

**Error Handling and Reliability:**
- Implement robust error handling for missing image files, corrupted templates, and invalid search coordinates
- Use timeout mechanisms and retry logic for time-sensitive automation scenarios
- Provide fallback strategies when primary image templates fail to match
- Consider combining ImageSearch with other detection methods for increased reliability

## Related AHK Concepts

- [PixelSearch](./pixelsearch.md) - Color-based pixel detection for simpler visual recognition
- [PixelGetColor](../../10_Language_Core/01-Functions/Built_In_Functions/pixelgetcolor.md) - Getting color values for image analysis
- [Click](../../40_Advanced_Features/00-Hotkeys_and_Input/click.md) - Performing actions at detected image locations
- [FileExist](../../30_Built_In_Classes/02-File_IO_Classes/File/fileexist.md) - Validating template image file availability
- [CoordMode](../../10_Language_Core/01-Functions/Built_In_Functions/coordmode.md) - Setting coordinate reference systems

## Tags

#AutoHotkey #ImageSearch #ImageRecognition #VisualAutomation #TemplateMatching #UIAutomation #ScreenDetection #PatternRecognition #GUIAutomation #ComputerVision
