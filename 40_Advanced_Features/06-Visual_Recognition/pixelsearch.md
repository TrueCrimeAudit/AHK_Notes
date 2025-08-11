# Topic: PixelSearch Function - Color Detection and Visual Analysis

## Category

Function

## Overview

The PixelSearch function searches for specific colors within a defined screen area, providing essential visual detection capabilities for automation scripts, image analysis, testing applications, and user interface validation. It's fundamental for screen automation that relies on visual cues, color-based triggers, and applications requiring pixel-perfect detection for reliable operation.

## Key Points

- Searches for exact or similar colors within specified rectangular screen regions with configurable tolerance
- Provides coordinates of the first matching pixel found, enabling precise targeting for subsequent automation actions
- Supports color variation tolerance to handle anti-aliasing, color depth differences, and minor display variations
- Essential for visual automation, screen monitoring, game automation, and applications requiring color-based detection
- Can be combined with other detection methods to create robust visual recognition systems

## Syntax and Parameters

```cpp
PixelSearch(&OutputVarX, &OutputVarY, X1, Y1, X2, Y2, ColorID [, Variation, Fast])

; Parameters:
; OutputVarX   - Variable to store X coordinate of found pixel
; OutputVarY   - Variable to store Y coordinate of found pixel
; X1, Y1       - Upper-left corner coordinates of search area
; X2, Y2       - Lower-right corner coordinates of search area
; ColorID      - Color to search for (0xRRGGBB format or decimal)
; Variation    - Optional color tolerance (0-255, default 0)
; Fast         - Optional search mode ("Fast" for faster but less reliable)

; Color format examples:
; 0xFF0000     - Red (hexadecimal)
; 0x00FF00     - Green (hexadecimal)
; 0x0000FF     - Blue (hexadecimal)
; 16711680     - Red (decimal equivalent)
; "Red"        - Named color (converted to RGB)

; Search area definition:
; (0, 0)       - Top-left corner of primary monitor
; (X1, Y1)     - Upper-left corner of search rectangle
; (X2, Y2)     - Lower-right corner of search rectangle
; Coordinates are in screen pixels

; Return behavior:
; Returns 1     - If pixel color found, coordinates stored in output variables
; Returns 0     - If pixel color not found, output variables unchanged
; Throws error  - If search area is invalid or parameters malformed

; Variation parameter:
; 0            - Exact color match only
; 1-255        - Tolerance for color differences (higher = more permissive)
; Useful for   - Anti-aliased text, compressed images, color depth variations

; Fast mode:
; "Fast"       - Uses faster algorithm but may miss some pixels
; Omitted      - Uses thorough search (default, more reliable)
```

## Code Examples

```cpp
; Basic color detection
if (PixelSearch(&foundX, &foundY, 0, 0, A_ScreenWidth, A_ScreenHeight, 0xFF0000)) {
    MsgBox("Red pixel found at: " . foundX . ", " . foundY)
    Click(foundX, foundY)
} else {
    MsgBox("Red pixel not found on screen")
}

; Advanced visual detection and monitoring system
class VisualDetector {
    static detectionCache := Map()
    static cacheTimeout := 3000
    static searchHistory := []
    static monitoringTasks := Map()
    static detectionRules := Map()
    static performanceStats := {searches: 0, hits: 0, misses: 0, cacheHits: 0}
    static coordinateValidation := true
    static colorTolerance := 10
    static searchOptimization := true
    static detectionLogging := true
    
    static Initialize() {
        this.detectionCache.Clear()
        this.searchHistory := []
        this.monitoringTasks.Clear()
        this.ResetPerformanceStats()
        this.SetupDefaultRules()
    }
    
    static SearchColor(searchSpec, options := {}) {
        ; Advanced color search with caching and optimization
        startTime := A_TickCount
        
        ; Extract search parameters
        x1 := searchSpec.HasProp("x1") ? searchSpec.x1 : 0
        y1 := searchSpec.HasProp("y1") ? searchSpec.y1 : 0
        x2 := searchSpec.HasProp("x2") ? searchSpec.x2 : A_ScreenWidth
        y2 := searchSpec.HasProp("y2") ? searchSpec.y2 : A_ScreenHeight
        color := searchSpec.color
        
        ; Process options
        variation := options.HasProp("variation") ? options.variation : this.colorTolerance
        fast := options.HasProp("fast") ? options.fast : false
        useCache := options.HasProp("useCache") ? options.useCache : true
        validateCoords := options.HasProp("validateCoords") ? options.validateCoords : this.coordinateValidation
        searchId := options.HasProp("searchId") ? options.searchId : ""
        
        ; Create search record
        searchRecord := {
            searchId: searchId,
            timestamp: A_TickCount,
            x1: x1, y1: y1, x2: x2, y2: y2,
            color: color,
            variation: variation,
            fast: fast,
            foundX: 0, foundY: 0,
            found: false,
            cached: false,
            searchTime: 0,
            error: ""
        }
        
        try {
            ; Validate search area
            if (validateCoords) {
                this.ValidateSearchArea(x1, y1, x2, y2)
            }
            
            ; Check cache first
            if (useCache) {
                cacheKey := this.GenerateCacheKey(x1, y1, x2, y2, color, variation)
                if (this.detectionCache.Has(cacheKey)) {
                    cacheEntry := this.detectionCache[cacheKey]
                    if (A_TickCount - cacheEntry.timestamp < this.cacheTimeout) {
                        searchRecord.foundX := cacheEntry.x
                        searchRecord.foundY := cacheEntry.y
                        searchRecord.found := cacheEntry.found
                        searchRecord.cached := true
                        searchRecord.searchTime := A_TickCount - startTime
                        
                        this.UpdatePerformanceStats("cache_hit")
                        this.LogSearch(searchRecord)
                        return searchRecord
                    }
                }
            }
            
            ; Perform pixel search
            foundX := 0
            foundY := 0
            
            if (fast && !variation) {
                found := PixelSearch(&foundX, &foundY, x1, y1, x2, y2, color, , "Fast")
            } else if (variation > 0) {
                found := PixelSearch(&foundX, &foundY, x1, y1, x2, y2, color, variation)
            } else {
                found := PixelSearch(&foundX, &foundY, x1, y1, x2, y2, color)
            }
            
            ; Update search record
            searchRecord.found := found
            searchRecord.foundX := foundX
            searchRecord.foundY := foundY
            searchRecord.searchTime := A_TickCount - startTime
            
            ; Cache result
            if (useCache) {
                this.CacheSearchResult(x1, y1, x2, y2, color, variation, found, foundX, foundY)
            }
            
            ; Update statistics
            this.UpdatePerformanceStats(found ? "hit" : "miss")
            
        } catch Error as err {
            searchRecord.error := err.Message
            this.UpdatePerformanceStats("error")
        }
        
        ; Log search operation
        this.LogSearch(searchRecord)
        return searchRecord
    }
    
    static SearchMultipleColors(colorSpecs, searchArea, options := {}) {
        ; Search for multiple colors simultaneously
        results := Map()
        searchResults := []
        
        for colorName, colorSpec in colorSpecs {
            searchSpec := {
                x1: searchArea.x1,
                y1: searchArea.y1,
                x2: searchArea.x2,
                y2: searchArea.y2,
                color: colorSpec.color
            }
            
            ; Merge color-specific options
            colorOptions := options.Clone()
            if (colorSpec.HasProp("variation")) {
                colorOptions.variation := colorSpec.variation
            }
            if (colorSpec.HasProp("fast")) {
                colorOptions.fast := colorSpec.fast
            }
            
            colorOptions.searchId := colorName
            
            searchResult := this.SearchColor(searchSpec, colorOptions)
            results[colorName] := searchResult
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
    
    static MonitorColorChanges(searchSpec, callback, options := {}) {
        ; Monitor area for color changes
        monitorId := options.HasProp("monitorId") ? options.monitorId : "monitor_" . A_TickCount
        checkInterval := options.HasProp("interval") ? options.interval : 1000
        timeout := options.HasProp("timeout") ? options.timeout : 0
        triggerMode := options.HasProp("triggerMode") ? options.triggerMode : "found"  ; "found", "lost", "changed"
        
        ; Initial search to establish baseline
        initialResult := this.SearchColor(searchSpec, options)
        lastFound := initialResult.found
        
        monitorData := {
            monitorId: monitorId,
            searchSpec: searchSpec,
            options: options,
            callback: callback,
            triggerMode: triggerMode,
            lastFound: lastFound,
            startTime: A_TickCount,
            timeout: timeout,
            checkCount: 0,
            triggerCount: 0
        }
        
        ; Create monitoring timer
        timerFunction := () => {
            try {
                monitorData.checkCount++
                currentResult := this.SearchColor(searchSpec, options)
                currentFound := currentResult.found
                
                shouldTrigger := false
                triggerReason := ""
                
                ; Check trigger conditions
                switch triggerMode {
                    case "found":
                        if (currentFound && !monitorData.lastFound) {
                            shouldTrigger := true
                            triggerReason := "Color appeared"
                        }
                    case "lost":
                        if (!currentFound && monitorData.lastFound) {
                            shouldTrigger := true
                            triggerReason := "Color disappeared"
                        }
                    case "changed":
                        if (currentFound != monitorData.lastFound) {
                            shouldTrigger := true
                            triggerReason := currentFound ? "Color appeared" : "Color disappeared"
                        }
                    case "always":
                        if (currentFound) {
                            shouldTrigger := true
                            triggerReason := "Color present"
                        }
                }
                
                ; Trigger callback if conditions met
                if (shouldTrigger && IsFunc(callback)) {
                    monitorData.triggerCount++
                    
                    callbackData := {
                        monitorId: monitorId,
                        searchResult: currentResult,
                        reason: triggerReason,
                        checkCount: monitorData.checkCount,
                        triggerCount: monitorData.triggerCount,
                        elapsedTime: A_TickCount - monitorData.startTime
                    }
                    
                    callback.Call(callbackData)
                }
                
                monitorData.lastFound := currentFound
                
                ; Check timeout
                if (timeout > 0 && A_TickCount - monitorData.startTime > timeout) {
                    this.StopMonitoring(monitorId)
                }
                
            } catch Error as err {
                ; Error in monitoring, stop monitoring
                OutputDebug("Monitor error: " . err.Message)
                this.StopMonitoring(monitorId)
            }
        }
        
        ; Store monitoring data
        this.monitoringTasks[monitorId] := {
            data: monitorData,
            timer: timerFunction
        }
        
        ; Start monitoring timer
        SetTimer(timerFunction, checkInterval)
        
        return {
            monitorId: monitorId,
            stop: () => this.StopMonitoring(monitorId),
            getStatus: () => this.GetMonitorStatus(monitorId)
        }
    }
    
    static StopMonitoring(monitorId) {
        ; Stop color monitoring
        if (this.monitoringTasks.Has(monitorId)) {
            monitorTask := this.monitoringTasks[monitorId]
            SetTimer(monitorTask.timer, 0)
            this.monitoringTasks.Delete(monitorId)
            return true
        }
        return false
    }
    
    static GetMonitorStatus(monitorId) {
        ; Get monitoring task status
        if (this.monitoringTasks.Has(monitorId)) {
            return this.monitoringTasks[monitorId].data
        }
        return false
    }
    
    static WaitForColor(searchSpec, timeout := 10000, options := {}) {
        ; Wait for color to appear
        startTime := A_TickCount
        checkInterval := options.HasProp("checkInterval") ? options.checkInterval : 250
        
        while (A_TickCount - startTime < timeout) {
            searchResult := this.SearchColor(searchSpec, options)
            
            if (searchResult.found) {
                return searchResult
            }
            
            Sleep(checkInterval)
        }
        
        throw TimeoutError("Color not found within timeout period")
    }
    
    static WaitForColorToDisappear(searchSpec, timeout := 10000, options := {}) {
        ; Wait for color to disappear
        startTime := A_TickCount
        checkInterval := options.HasProp("checkInterval") ? options.checkInterval : 250
        
        while (A_TickCount - startTime < timeout) {
            searchResult := this.SearchColor(searchSpec, options)
            
            if (!searchResult.found) {
                return true
            }
            
            Sleep(checkInterval)
        }
        
        throw TimeoutError("Color did not disappear within timeout period")
    }
    
    static AnalyzeColorRegion(x1, y1, x2, y2, options := {}) {
        ; Analyze colors in a specific region
        sampleStep := options.HasProp("sampleStep") ? options.sampleStep : 5
        includeStatistics := options.HasProp("includeStatistics") ? options.includeStatistics : true
        
        colorCounts := Map()
        totalPixels := 0
        sampleCount := 0
        
        ; Sample pixels in the region
        for x in Range(x1, x2, sampleStep) {
            for y in Range(y1, y2, sampleStep) {
                if (x <= x2 && y <= y2) {
                    try {
                        pixelColor := PixelGetColor(x, y)
                        
                        if (!colorCounts.Has(pixelColor)) {
                            colorCounts[pixelColor] := 0
                        }
                        colorCounts[pixelColor]++
                        
                        totalPixels++
                        sampleCount++
                        
                    } catch Error {
                        ; Skip invalid coordinates
                        continue
                    }
                }
            }
        }
        
        ; Generate color analysis
        analysis := {
            region: {x1: x1, y1: y1, x2: x2, y2: y2},
            sampleStep: sampleStep,
            sampleCount: sampleCount,
            uniqueColors: colorCounts.Count,
            colorCounts: colorCounts,
            dominantColors: [],
            statistics: {}
        }
        
        ; Find dominant colors
        colorArray := []
        for color, count in colorCounts {
            colorArray.Push({color: color, count: count, percentage: Round((count / totalPixels) * 100, 2)})
        }
        
        ; Sort by count (descending)
        colorArray.Sort((a, b) => b.count - a.count)
        
        ; Take top colors
        maxColors := options.HasProp("maxDominantColors") ? options.maxDominantColors : 10
        analysis.dominantColors := colorArray.Slice(1, Min(maxColors, colorArray.Length))
        
        ; Calculate statistics if requested
        if (includeStatistics) {
            analysis.statistics := this.CalculateColorStatistics(colorArray, totalPixels)
        }
        
        return analysis
    }
    
    static CalculateColorStatistics(colorArray, totalPixels) {
        ; Calculate color distribution statistics
        if (colorArray.Length = 0) {
            return {variance: 0, entropy: 0, dominanceIndex: 0}
        }
        
        ; Calculate variance in color distribution
        mean := totalPixels / colorArray.Length
        variance := 0
        
        for colorData in colorArray {
            variance += (colorData.count - mean) ** 2
        }
        variance := variance / colorArray.Length
        
        ; Calculate entropy (color diversity measure)
        entropy := 0
        for colorData in colorArray {
            probability := colorData.count / totalPixels
            if (probability > 0) {
                entropy -= probability * Log(probability) / Log(2)
            }
        }
        
        ; Calculate dominance index (how much the top color dominates)
        dominanceIndex := colorArray.Length > 0 ? colorArray[1].percentage : 0
        
        return {
            variance: Round(variance, 2),
            entropy: Round(entropy, 2),
            dominanceIndex: Round(dominanceIndex, 2),
            colorDiversity: Round(entropy / Log(colorArray.Length) * 100, 2)
        }
    }
    
    static CreateColorPattern(colors, positions, options := {}) {
        ; Create a color pattern for complex detection
        patternId := options.HasProp("patternId") ? options.patternId : "pattern_" . A_TickCount
        tolerance := options.HasProp("tolerance") ? options.tolerance : 5
        
        pattern := {
            patternId: patternId,
            colors: colors,
            positions: positions,
            tolerance: tolerance,
            created: A_TickCount
        }
        
        this.detectionRules[patternId] := pattern
        return patternId
    }
    
    static SearchColorPattern(patternId, searchArea, options := {}) {
        ; Search for a color pattern
        if (!this.detectionRules.Has(patternId)) {
            throw ValueError("Color pattern '" . patternId . "' not found")
        }
        
        pattern := this.detectionRules[patternId]
        tolerance := options.HasProp("tolerance") ? options.tolerance : pattern.tolerance
        
        ; Search for anchor color (first color in pattern)
        if (pattern.colors.Length = 0 || pattern.positions.Length = 0) {
            throw ValueError("Invalid color pattern")
        }
        
        anchorColor := pattern.colors[1]
        anchorPos := pattern.positions[1]
        
        ; Search for anchor color
        anchorSearch := {
            x1: searchArea.x1,
            y1: searchArea.y1,
            x2: searchArea.x2,
            y2: searchArea.y2,
            color: anchorColor
        }
        
        anchorResult := this.SearchColor(anchorSearch, {variation: tolerance})
        
        if (!anchorResult.found) {
            return {found: false, patternId: patternId, reason: "Anchor color not found"}
        }
        
        ; Check relative positions of other colors
        baseX := anchorResult.foundX - anchorPos.x
        baseY := anchorResult.foundY - anchorPos.y
        
        for i in Range(2, pattern.colors.Length) {
            expectedX := baseX + pattern.positions[i].x
            expectedY := baseY + pattern.positions[i].y
            expectedColor := pattern.colors[i]
            
            ; Search in small area around expected position
            margin := options.HasProp("positionTolerance") ? options.positionTolerance : 3
            
            colorSearch := {
                x1: expectedX - margin,
                y1: expectedY - margin,
                x2: expectedX + margin,
                y2: expectedY + margin,
                color: expectedColor
            }
            
            colorResult := this.SearchColor(colorSearch, {variation: tolerance})
            
            if (!colorResult.found) {
                return {
                    found: false, 
                    patternId: patternId, 
                    reason: "Pattern color " . i . " not found at expected position",
                    baseX: baseX,
                    baseY: baseY,
                    expectedX: expectedX,
                    expectedY: expectedY
                }
            }
        }
        
        return {
            found: true,
            patternId: patternId,
            baseX: baseX,
            baseY: baseY,
            centerX: baseX + (pattern.positions[pattern.positions.Length].x - pattern.positions[1].x) // 2,
            centerY: baseY + (pattern.positions[pattern.positions.Length].y - pattern.positions[1].y) // 2
        }
    }
    
    static ValidateSearchArea(x1, y1, x2, y2) {
        ; Validate search area coordinates
        if (x1 >= x2 || y1 >= y2) {
            throw ValueError("Invalid search area: coordinates must define a valid rectangle")
        }
        
        if (x1 < 0 || y1 < 0) {
            throw ValueError("Invalid search area: negative coordinates not allowed")
        }
        
        if (x2 > A_ScreenWidth || y2 > A_ScreenHeight) {
            throw ValueError("Invalid search area: coordinates exceed screen bounds")
        }
        
        areaSize := (x2 - x1) * (y2 - y1)
        if (areaSize > 5000000) {  ; Limit very large search areas
            throw ValueError("Invalid search area: area too large (may cause performance issues)")
        }
    }
    
    static GenerateCacheKey(x1, y1, x2, y2, color, variation) {
        ; Generate cache key for search parameters
        return x1 . "," . y1 . "," . x2 . "," . y2 . "," . color . "," . variation
    }
    
    static CacheSearchResult(x1, y1, x2, y2, color, variation, found, foundX, foundY) {
        ; Cache search result
        cacheKey := this.GenerateCacheKey(x1, y1, x2, y2, color, variation)
        
        this.detectionCache[cacheKey] := {
            found: found,
            x: foundX,
            y: foundY,
            timestamp: A_TickCount
        }
        
        ; Limit cache size
        if (this.detectionCache.Count > 200) {
            this.CleanupCache()
        }
    }
    
    static CleanupCache() {
        ; Remove old cache entries
        currentTime := A_TickCount
        
        for cacheKey, entry in this.detectionCache {
            if (currentTime - entry.timestamp > this.cacheTimeout) {
                this.detectionCache.Delete(cacheKey)
            }
        }
    }
    
    static LogSearch(searchRecord) {
        ; Log search operation
        if (this.detectionLogging) {
            this.searchHistory.Push(searchRecord)
            
            ; Limit history size
            if (this.searchHistory.Length > 1000) {
                this.searchHistory.RemoveAt(1)
            }
        }
    }
    
    static UpdatePerformanceStats(operation) {
        ; Update performance statistics
        this.performanceStats.searches++
        
        switch operation {
            case "hit":
                this.performanceStats.hits++
            case "miss":
                this.performanceStats.misses++
            case "cache_hit":
                this.performanceStats.cacheHits++
        }
    }
    
    static ResetPerformanceStats() {
        ; Reset performance statistics
        this.performanceStats := {
            searches: 0,
            hits: 0,
            misses: 0,
            cacheHits: 0
        }
    }
    
    static GenerateSearchSummary(searchResults) {
        ; Generate summary of multiple search results
        summary := {
            total: searchResults.Length,
            found: 0,
            notFound: 0,
            averageSearchTime: 0,
            totalSearchTime: 0,
            cached: 0,
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
        stats := this.performanceStats
        
        report := "Visual Detection Performance Report`n"
        report .= "======================================`n`n"
        
        report .= "Search Statistics:`n"
        report .= "Total Searches: " . stats.searches . "`n"
        report .= "Successful: " . stats.hits . "`n"
        report .= "Not Found: " . stats.misses . "`n"
        report .= "Cache Hits: " . stats.cacheHits . "`n`n"
        
        if (stats.searches > 0) {
            report .= "Success Rate: " . Round((stats.hits / stats.searches) * 100, 1) . "%`n"
            report .= "Cache Hit Rate: " . Round((stats.cacheHits / stats.searches) * 100, 1) . "%`n`n"
        }
        
        report .= "Configuration:`n"
        report .= "Color Tolerance: " . this.colorTolerance . "`n"
        report .= "Cache Timeout: " . this.cacheTimeout . "ms`n"
        report .= "Coordinate Validation: " . (this.coordinateValidation ? "Enabled" : "Disabled") . "`n"
        report .= "Search Optimization: " . (this.searchOptimization ? "Enabled" : "Disabled") . "`n`n"
        
        report .= "Active Monitoring Tasks: " . this.monitoringTasks.Count . "`n"
        report .= "Cached Searches: " . this.detectionCache.Count . "`n"
        report .= "Detection Rules: " . this.detectionRules.Count . "`n`n"
        
        ; Recent searches
        if (this.searchHistory.Length > 0) {
            report .= "Recent Searches:`n"
            recentCount := Min(5, this.searchHistory.Length)
            for i in Range(this.searchHistory.Length - recentCount + 1, this.searchHistory.Length) {
                search := this.searchHistory[i]
                status := search.found ? "✓" : "✗"
                cached := search.cached ? " (cached)" : ""
                report .= status . " Color: " . Format("0x{:06X}", search.color) . " " . search.searchTime . "ms" . cached . "`n"
            }
        }
        
        return report
    }
    
    static SetupDefaultRules() {
        ; Set up default detection rules
        this.detectionRules.Clear()
    }
    
    static ConvertColorFormat(color) {
        ; Convert color to standard format
        if (IsString(color)) {
            ; Handle named colors
            colorMap := Map(
                "red", 0xFF0000,
                "green", 0x00FF00,
                "blue", 0x0000FF,
                "white", 0xFFFFFF,
                "black", 0x000000,
                "yellow", 0xFFFF00,
                "cyan", 0x00FFFF,
                "magenta", 0xFF00FF
            )
            
            lowerColor := StrLower(color)
            if (colorMap.Has(lowerColor)) {
                return colorMap[lowerColor]
            } else {
                throw ValueError("Unknown color name: " . color)
            }
        }
        
        return color  ; Already numeric
    }
    
    static EnableCaching() {
        ; Enable search result caching
        this.detectionCache.Clear()
    }
    
    static DisableCaching() {
        ; Disable search result caching
        this.detectionCache.Clear()
    }
    
    static SetColorTolerance(tolerance) {
        ; Set default color tolerance
        if (tolerance < 0 || tolerance > 255) {
            throw ValueError("Color tolerance must be between 0 and 255")
        }
        this.colorTolerance := tolerance
    }
    
    static SetCacheTimeout(timeout) {
        ; Set cache timeout
        if (timeout < 100) {
            throw ValueError("Cache timeout must be at least 100ms")
        }
        this.cacheTimeout := timeout
    }
    
    static ClearCache() {
        this.detectionCache.Clear()
    }
    
    static ClearHistory() {
        this.searchHistory := []
    }
    
    static StopAllMonitoring() {
        ; Stop all monitoring tasks
        for monitorId in this.monitoringTasks {
            this.StopMonitoring(monitorId)
        }
    }
}

; Color coordination and screen analysis system
class ScreenAnalyzer {
    static analysisCache := Map()
    static regionTemplates := Map()
    static analysisHistory := []
    
    static Initialize() {
        this.analysisCache.Clear()
        this.regionTemplates.Clear()
        this.SetupDefaultRegions()
    }
    
    static AnalyzeScreen(regions := "", options := {}) {
        ; Analyze entire screen or specific regions
        if (!regions) {
            regions := [{
                name: "fullscreen",
                x1: 0, y1: 0, 
                x2: A_ScreenWidth, y2: A_ScreenHeight
            }]
        }
        
        analysis := {
            timestamp: A_TickCount,
            regions: Map(),
            summary: {},
            totalTime: 0
        }
        
        startTime := A_TickCount
        
        for region in regions {
            regionAnalysis := VisualDetector.AnalyzeColorRegion(
                region.x1, region.y1, region.x2, region.y2, options
            )
            
            regionAnalysis.regionName := region.name
            analysis.regions[region.name] := regionAnalysis
        }
        
        analysis.totalTime := A_TickCount - startTime
        analysis.summary := this.GenerateAnalysisSummary(analysis.regions)
        
        this.analysisHistory.Push(analysis)
        
        ; Limit history size
        if (this.analysisHistory.Length > 50) {
            this.analysisHistory.RemoveAt(1)
        }
        
        return analysis
    }
    
    static GenerateAnalysisSummary(regions) {
        ; Generate summary of region analysis
        summary := {
            totalRegions: regions.Count,
            totalUniqueColors: 0,
            averageColorDiversity: 0,
            dominantColorAcrossRegions: 0,
            regionComparisons: []
        }
        
        totalDiversity := 0
        allColors := Map()
        
        for regionName, regionData in regions {
            summary.totalUniqueColors += regionData.uniqueColors
            
            if (regionData.HasProp("statistics")) {
                totalDiversity += regionData.statistics.colorDiversity
            }
            
            ; Collect all colors
            for color, count in regionData.colorCounts {
                if (!allColors.Has(color)) {
                    allColors[color] := 0
                }
                allColors[color] += count
            }
        }
        
        if (regions.Count > 0) {
            summary.averageColorDiversity := Round(totalDiversity / regions.Count, 2)
        }
        
        ; Find most common color across all regions
        if (allColors.Count > 0) {
            maxCount := 0
            dominantColor := 0
            
            for color, count in allColors {
                if (count > maxCount) {
                    maxCount := count
                    dominantColor := color
                }
            }
            
            summary.dominantColorAcrossRegions := dominantColor
        }
        
        return summary
    }
    
    static DefineRegionTemplate(templateName, regions) {
        ; Define reusable region template
        this.regionTemplates[templateName] := regions
    }
    
    static AnalyzeWithTemplate(templateName, options := {}) {
        ; Analyze screen using predefined template
        if (!this.regionTemplates.Has(templateName)) {
            throw ValueError("Region template '" . templateName . "' not found")
        }
        
        return this.AnalyzeScreen(this.regionTemplates[templateName], options)
    }
    
    static CompareScreenStates(analysis1, analysis2) {
        ; Compare two screen analysis states
        comparison := {
            timestamp1: analysis1.timestamp,
            timestamp2: analysis2.timestamp,
            timeDifference: analysis2.timestamp - analysis1.timestamp,
            regionChanges: Map(),
            overallChange: 0
        }
        
        ; Compare each region
        for regionName, region1 in analysis1.regions {
            if (analysis2.regions.Has(regionName)) {
                region2 := analysis2.regions[regionName]
                
                regionComparison := this.CompareRegions(region1, region2)
                comparison.regionChanges[regionName] := regionComparison
            }
        }
        
        ; Calculate overall change metric
        totalChange := 0
        regionCount := 0
        
        for regionName, regionChange in comparison.regionChanges {
            totalChange += regionChange.changeMetric
            regionCount++
        }
        
        if (regionCount > 0) {
            comparison.overallChange := Round(totalChange / regionCount, 2)
        }
        
        return comparison
    }
    
    static CompareRegions(region1, region2) {
        ; Compare two region analyses
        comparison := {
            colorCountDifference: region2.uniqueColors - region1.uniqueColors,
            newColors: [],
            removedColors: [],
            changedColors: [],
            changeMetric: 0
        }
        
        ; Find new and removed colors
        for color in region1.colorCounts {
            if (!region2.colorCounts.Has(color)) {
                comparison.removedColors.Push(color)
            }
        }
        
        for color in region2.colorCounts {
            if (!region1.colorCounts.Has(color)) {
                comparison.newColors.Push(color)
            }
        }
        
        ; Find changed color counts
        for color, count1 in region1.colorCounts {
            if (region2.colorCounts.Has(color)) {
                count2 := region2.colorCounts[color]
                if (count1 != count2) {
                    comparison.changedColors.Push({
                        color: color,
                        oldCount: count1,
                        newCount: count2,
                        difference: count2 - count1
                    })
                }
            }
        }
        
        ; Calculate change metric
        totalChanges := comparison.newColors.Length + comparison.removedColors.Length + comparison.changedColors.Length
        totalColors := Max(region1.uniqueColors, region2.uniqueColors)
        
        if (totalColors > 0) {
            comparison.changeMetric := Round((totalChanges / totalColors) * 100, 2)
        }
        
        return comparison
    }
    
    static SetupDefaultRegions() {
        ; Set up default region templates
        this.DefineRegionTemplate("taskbar", [{
            name: "taskbar",
            x1: 0, y1: A_ScreenHeight - 40,
            x2: A_ScreenWidth, y2: A_ScreenHeight
        }])
        
        this.DefineRegionTemplate("desktop", [{
            name: "desktop",
            x1: 0, y1: 0,
            x2: A_ScreenWidth, y2: A_ScreenHeight - 40
        }])
        
        this.DefineRegionTemplate("corners", [
            {name: "top-left", x1: 0, y1: 0, x2: 100, y2: 100},
            {name: "top-right", x1: A_ScreenWidth - 100, y1: 0, x2: A_ScreenWidth, y2: 100},
            {name: "bottom-left", x1: 0, y1: A_ScreenHeight - 100, x2: 100, y2: A_ScreenHeight},
            {name: "bottom-right", x1: A_ScreenWidth - 100, y1: A_ScreenHeight - 100, x2: A_ScreenWidth, y2: A_ScreenHeight}
        ])
    }
}

; Example usage and demonstrations
; Initialize visual detection system
VisualDetector.Initialize()
ScreenAnalyzer.Initialize()

; Basic color detection examples
if (PixelSearch(&foundX, &foundY, 0, 0, A_ScreenWidth, A_ScreenHeight, 0xFF0000)) {
    MsgBox("Red pixel found at: " . foundX . ", " . foundY)
} else {
    MsgBox("Red pixel not found")
}

; Advanced color search with options
searchResult := VisualDetector.SearchColor({
    x1: 100, y1: 100, x2: 500, y2: 400,
    color: 0x0000FF
}, {
    variation: 20,
    useCache: true,
    validateCoords: true,
    searchId: "blue_button_search"
})

if (searchResult.found) {
    MsgBox("Blue color found at: " . searchResult.foundX . ", " . searchResult.foundY . 
           "`nSearch time: " . searchResult.searchTime . "ms" .
           (searchResult.cached ? " (cached)" : ""))
    
    ; Click on the found color
    Click(searchResult.foundX, searchResult.foundY)
} else {
    MsgBox("Blue color not found in specified area")
}

; Multiple color search
colorSpecs := Map(
    "red_button", {color: 0xFF0000, variation: 15},
    "green_icon", {color: 0x00FF00, variation: 10},
    "blue_text", {color: 0x0000FF, variation: 25}
)

searchArea := {x1: 0, y1: 0, x2: 800, y2: 600}

multiResult := VisualDetector.SearchMultipleColors(colorSpecs, searchArea, {
    useCache: true,
    fast: false
})

MsgBox("Multi-color search results:`n" .
       "Colors found: " . multiResult.foundCount . "/" . multiResult.totalCount . "`n" .
       "Success rate: " . multiResult.summary.successRate . "%")

; Display individual results
for colorName, result in multiResult.results {
    if (result.found) {
        OutputDebug(colorName . " found at: " . result.foundX . ", " . result.foundY)
    } else {
        OutputDebug(colorName . " not found")
    }
}

; Color monitoring example
monitor := VisualDetector.MonitorColorChanges({
    x1: 50, y1: 50, x2: 150, y2: 150,
    color: 0xFFFF00
}, (callbackData) => {
    MsgBox("Color change detected!`n" .
           "Reason: " . callbackData.reason . "`n" .
           "Check count: " . callbackData.checkCount . "`n" .
           "Found at: " . callbackData.searchResult.foundX . ", " . callbackData.searchResult.foundY)
}, {
    monitorId: "yellow_indicator",
    interval: 500,
    timeout: 30000,
    triggerMode: "changed"
})

; Wait for specific color to appear
try {
    waitResult := VisualDetector.WaitForColor({
        x1: 200, y1: 200, x2: 400, y2: 300,
        color: 0x800080  ; Purple
    }, 10000, {
        variation: 30,
        checkInterval: 250
    })
    
    MsgBox("Purple color appeared at: " . waitResult.foundX . ", " . waitResult.foundY)
    
} catch TimeoutError {
    MsgBox("Purple color did not appear within 10 seconds")
}

; Color pattern detection
patternColors := [0xFF0000, 0x00FF00, 0x0000FF]  ; Red, Green, Blue
patternPositions := [{x: 0, y: 0}, {x: 20, y: 0}, {x: 40, y: 0}]  ; Horizontal line

patternId := VisualDetector.CreateColorPattern(patternColors, patternPositions, {
    patternId: "rgb_line",
    tolerance: 15
})

patternResult := VisualDetector.SearchColorPattern(patternId, {
    x1: 0, y1: 0, x2: A_ScreenWidth, y2: A_ScreenHeight
}, {
    tolerance: 20,
    positionTolerance: 5
})

if (patternResult.found) {
    MsgBox("RGB line pattern found at base position: " . patternResult.baseX . ", " . patternResult.baseY)
    
    ; Click on center of pattern
    Click(patternResult.centerX, patternResult.centerY)
} else {
    MsgBox("RGB line pattern not found: " . patternResult.reason)
}

; Screen region analysis
analysis := VisualDetector.AnalyzeColorRegion(100, 100, 300, 200, {
    sampleStep: 3,
    includeStatistics: true,
    maxDominantColors: 5
})

MsgBox("Color analysis results:`n" .
       "Unique colors: " . analysis.uniqueColors . "`n" .
       "Sample count: " . analysis.sampleCount . "`n" .
       "Color diversity: " . analysis.statistics.colorDiversity . "%`n" .
       "Dominant color: " . Format("0x{:06X}", analysis.dominantColors[1].color) . 
       " (" . analysis.dominantColors[1].percentage . "%)")

; Full screen analysis
screenAnalysis := ScreenAnalyzer.AnalyzeScreen([
    {name: "top_half", x1: 0, y1: 0, x2: A_ScreenWidth, y2: A_ScreenHeight // 2},
    {name: "bottom_half", x1: 0, y1: A_ScreenHeight // 2, x2: A_ScreenWidth, y2: A_ScreenHeight}
], {
    sampleStep: 10,
    includeStatistics: true
})

MsgBox("Screen analysis complete:`n" .
       "Regions analyzed: " . screenAnalysis.summary.totalRegions . "`n" .
       "Total unique colors: " . screenAnalysis.summary.totalUniqueColors . "`n" .
       "Average color diversity: " . screenAnalysis.summary.averageColorDiversity . "%`n" .
       "Analysis time: " . screenAnalysis.totalTime . "ms")

; Template-based analysis
ScreenAnalyzer.DefineRegionTemplate("app_window", [
    {name: "title_bar", x1: 100, y1: 100, x2: 500, y2: 130},
    {name: "content_area", x1: 100, y1: 130, x2: 500, y2: 400},
    {name: "status_bar", x1: 100, y1: 400, x2: 500, y2: 420}
])

appAnalysis := ScreenAnalyzer.AnalyzeWithTemplate("app_window", {
    sampleStep: 5,
    includeStatistics: true
})

; Color change detection over time
initialAnalysis := ScreenAnalyzer.AnalyzeScreen([
    {name: "monitor_area", x1: 50, y1: 50, x2: 250, y2: 250}
])

Sleep(5000)  ; Wait 5 seconds

finalAnalysis := ScreenAnalyzer.AnalyzeScreen([
    {name: "monitor_area", x1: 50, y1: 50, x2: 250, y2: 250}
])

comparison := ScreenAnalyzer.CompareScreenStates(initialAnalysis, finalAnalysis)

MsgBox("Screen change analysis:`n" .
       "Time difference: " . comparison.timeDifference . "ms`n" .
       "Overall change: " . comparison.overallChange . "%`n" .
       "New colors: " . comparison.regionChanges["monitor_area"].newColors.Length . "`n" .
       "Removed colors: " . comparison.regionChanges["monitor_area"].removedColors.Length)

; Performance monitoring and optimization
VisualDetector.SetColorTolerance(15)
VisualDetector.SetCacheTimeout(5000)

; Run multiple searches to test performance
Loop 10 {
    randomX := Random(0, A_ScreenWidth - 100)
    randomY := Random(0, A_ScreenHeight - 100)
    randomColor := Random(0, 0xFFFFFF)
    
    searchResult := VisualDetector.SearchColor({
        x1: randomX, y1: randomY, 
        x2: randomX + 100, y2: randomY + 100,
        color: randomColor
    }, {
        variation: 20,
        useCache: true,
        searchId: "performance_test_" . A_Index
    })
}

; Get performance report
performanceReport := VisualDetector.GetPerformanceReport()
; MsgBox(performanceReport, "Performance Report")

; Color-based automation example
automationColors := Map(
    "start_button", {color: 0x00AA00, variation: 20},
    "stop_button", {color: 0xAA0000, variation: 20},
    "status_indicator", {color: 0xFFAA00, variation: 15}
)

automationArea := {x1: 0, y1: 0, x2: A_ScreenWidth, y2: A_ScreenHeight}

automationResult := VisualDetector.SearchMultipleColors(automationColors, automationArea)

if (automationResult.results["start_button"].found) {
    ; Click start button
    startResult := automationResult.results["start_button"]
    Click(startResult.foundX, startResult.foundY)
    
    ; Wait for status indicator to change
    try {
        statusResult := VisualDetector.WaitForColor({
            x1: 0, y1: 0, x2: A_ScreenWidth, y2: A_ScreenHeight,
            color: 0x00FF00  ; Green status
        }, 15000, {variation: 25})
        
        MsgBox("Operation started successfully! Status is green.")
        
    } catch TimeoutError {
        MsgBox("Operation may have failed - status did not turn green")
    }
}

; Cleanup and resource management
; monitor.stop()  ; Stop color monitoring
VisualDetector.StopAllMonitoring()  ; Stop all monitoring tasks
VisualDetector.ClearCache()  ; Clear search cache
VisualDetector.ClearHistory()  ; Clear search history
```

## Implementation Notes

**Performance Considerations:**
- PixelSearch can be computationally expensive on large screen areas, especially with color variation tolerance
- Consider using smaller search regions when possible to improve performance and accuracy
- The "Fast" mode trades some accuracy for speed, which may miss certain pixels in specific display configurations
- Implement caching mechanisms for repeated searches of the same area to reduce computational overhead

**Color Matching and Display Variations:**
- Color values can vary between different monitors, graphics cards, and display settings
- Use appropriate variation tolerances to handle anti-aliasing, compression artifacts, and color depth differences
- Be aware that some applications may use different color profiles or rendering methods that affect pixel colors
- Test color detection across different display configurations and Windows display scaling settings

**Coordinate System and Multi-Monitor Support:**
- Coordinates are relative to the primary monitor's top-left corner (0, 0)
- For multi-monitor setups, coordinates can be negative or exceed primary screen dimensions
- Consider using SysGet or other functions to determine actual screen boundaries for comprehensive detection
- Handle cases where search areas may span multiple monitors with different color characteristics

**Timing and Reliability Issues:**
- Screen content can change rapidly, making pixel detection unreliable for dynamic content
- Implement retry mechanisms and validation to handle timing-sensitive detection scenarios
- Consider the impact of screen refresh rates, animation, and visual effects on detection accuracy
- Use appropriate delays and verification steps when pixel detection is part of a larger automation sequence

**Error Handling and Edge Cases:**
- Handle cases where search areas are partially or completely off-screen gracefully
- Implement proper validation for color format inputs and coordinate boundaries
- Consider scenarios where the target application is minimized, hidden, or covered by other windows
- Provide meaningful error messages and fallback strategies when detection fails consistently

## Related AHK Concepts

- [PixelGetColor](../../10_Language_Core/01-Functions/Built_In_Functions/pixelgetcolor.md) - Getting color values at specific coordinates
- [ImageSearch](./imagesearch.md) - Advanced image pattern recognition
- [Click](../../40_Advanced_Features/00-Hotkeys_and_Input/click.md) - Performing actions at detected pixel locations
- [CoordMode](../../10_Language_Core/01-Functions/Built_In_Functions/coordmode.md) - Setting coordinate reference modes
- [SysGet](../../10_Language_Core/01-Functions/Built_In_Functions/sysget.md) - Getting screen and monitor information

## Tags

#AutoHotkey #PixelSearch #ColorDetection #VisualAutomation #ScreenAutomation #ImageAnalysis #ColorRecognition #VisualTesting #GUIAutomation #ScreenMonitoring
