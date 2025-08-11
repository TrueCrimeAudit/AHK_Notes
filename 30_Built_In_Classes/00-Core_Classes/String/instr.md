# Function: InStr

## Category

Built-in Function

## Overview

InStr searches for a substring within a string and returns the position of the first occurrence. It's essential for string analysis, data validation, and text processing operations, providing the foundation for conditional string manipulation and parsing logic.

## Syntax

```cpp
Position := InStr(Haystack, Needle [, CaseSensitive?, StartingPos])
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Haystack | String | Yes | The string to search within |
| Needle | String | Yes | The substring to search for |
| CaseSensitive | Boolean | No | true for case-sensitive, false for case-insensitive (default) |
| StartingPos | Integer | No | 1-based position to start searching from (default: 1) |

## Return Value

| Type | Description |
|------|-------------|
| Integer | Position of first occurrence (1-based), or 0 if not found |

## Search Behavior

### Case Sensitivity
| CaseSensitive | Behavior | Example |
|---------------|----------|---------|
| false (default) | Case-insensitive search | `InStr("Hello", "HELLO")` → 1 |
| true | Case-sensitive search | `InStr("Hello", "HELLO", true)` → 0 |

### Starting Position
| StartingPos | Behavior | Example with "Hello World" |
|-------------|----------|----------------------------|
| Positive | Search forward from position | `InStr("Hello World", "o", false, 5)` → 8 |
| Negative | Search backward from end | `InStr("Hello World", "o", false, -1)` → 8 |
| Zero/Omitted | Start from beginning | `InStr("Hello World", "o")` → 5 |

## Exceptions

| Exception Type | When Thrown |
|----------------|-------------|
| TypeError | When Haystack or Needle parameters are not strings |
| ValueError | When StartingPos is invalid (rare) |

## Basic Examples

```cpp
; Basic string searching
text := "Hello World"
pos := InStr(text, "World")        ; Returns 7
pos2 := InStr(text, "world")       ; Returns 7 (case-insensitive by default)
pos3 := InStr(text, "world", true) ; Returns 0 (case-sensitive)

; Check if string contains substring
if (InStr(text, "Hello")) {
    MsgBox("Text contains 'Hello'")
}

; Find character positions
firstO := InStr(text, "o")          ; Returns 5 (first 'o')
lastO := InStr(text, "o", false, -1) ; Returns 8 (last 'o' searching backward)

; Search from specific position
secondO := InStr(text, "o", false, 6) ; Returns 8 (first 'o' after position 6)

; Handle not found
pos := InStr(text, "xyz")
if (pos == 0) {
    MsgBox("Substring not found")
}
```

## Advanced Examples

```cpp
; Advanced string searching with multiple criteria and validation
class StringSearcher {
    static FindAll(haystack, needle, caseSensitive := false) {
        local positions := Array()
        local searchPos := 1
        
        while (searchPos <= StrLen(haystack)) {
            local foundPos := InStr(haystack, needle, caseSensitive, searchPos)
            
            if (foundPos == 0) {
                break
            }
            
            positions.Push(foundPos)
            searchPos := foundPos + StrLen(needle)
        }
        
        return positions
    }
    
    static FindBetween(text, startDelim, endDelim, caseSensitive := false) {
        local results := Array()
        local searchPos := 1
        
        while (searchPos <= StrLen(text)) {
            ; Find start delimiter
            local startPos := InStr(text, startDelim, caseSensitive, searchPos)
            if (startPos == 0) {
                break
            }
            
            ; Find end delimiter after start
            local searchAfterStart := startPos + StrLen(startDelim)
            local endPos := InStr(text, endDelim, caseSensitive, searchAfterStart)
            if (endPos == 0) {
                break
            }
            
            ; Extract content between delimiters
            local contentStart := startPos + StrLen(startDelim)
            local contentLength := endPos - contentStart
            local content := SubStr(text, contentStart, contentLength)
            
            results.Push({
                content: content,
                startPos: startPos,
                endPos: endPos + StrLen(endDelim) - 1,
                fullMatch: SubStr(text, startPos, endPos + StrLen(endDelim) - startPos)
            })
            
            searchPos := endPos + StrLen(endDelim)
        }
        
        return results
    }
    
    static ContainsAny(text, needles, caseSensitive := false) {
        for index, needle in needles {
            if (InStr(text, needle, caseSensitive) > 0) {
                return {
                    found: true,
                    needle: needle,
                    position: InStr(text, needle, caseSensitive),
                    index: index
                }
            }
        }
        
        return {found: false, needle: "", position: 0, index: 0}
    }
    
    static ContainsAll(text, needles, caseSensitive := false) {
        local results := Array()
        local allFound := true
        
        for index, needle in needles {
            local position := InStr(text, needle, caseSensitive)
            local found := position > 0
            
            results.Push({
                needle: needle,
                found: found,
                position: position
            })
            
            if (!found) {
                allFound := false
            }
        }
        
        return {
            allFound: allFound,
            results: results,
            foundCount: this.CountTrue(results, "found")
        }
    }
    
    static CountOccurrences(text, needle, caseSensitive := false, allowOverlap := false) {
        local count := 0
        local searchPos := 1
        local increment := allowOverlap ? 1 : StrLen(needle)
        
        while (searchPos <= StrLen(text)) {
            local foundPos := InStr(text, needle, caseSensitive, searchPos)
            
            if (foundPos == 0) {
                break
            }
            
            count++
            searchPos := foundPos + increment
        }
        
        return count
    }
    
    static ReplaceWithTracking(text, searchText, replaceText, caseSensitive := false) {
        local newText := text
        local replacements := Array()
        local searchPos := 1
        local offset := 0
        
        while (searchPos <= StrLen(text)) {
            local foundPos := InStr(text, searchText, caseSensitive, searchPos)
            
            if (foundPos == 0) {
                break
            }
            
            ; Track replacement details
            replacements.Push({
                originalPos: foundPos,
                newPos: foundPos + offset,
                originalText: searchText,
                replacementText: replaceText
            })
            
            ; Perform replacement
            newText := SubStr(newText, 1, foundPos + offset - 1) . 
                      replaceText . 
                      SubStr(newText, foundPos + offset + StrLen(searchText))
            
            ; Update offset for position tracking
            offset += StrLen(replaceText) - StrLen(searchText)
            searchPos := foundPos + StrLen(searchText)
        }
        
        return {
            text: newText,
            replacements: replacements,
            count: replacements.Length
        }
    }
    
    static CountTrue(array, property) {
        local count := 0
        for index, item in array {
            if (item.%property%) {
                count++
            }
        }
        return count
    }
}

; Usage examples
text := "The quick brown fox jumps over the lazy dog"

; Find all occurrences of "the"
positions := StringSearcher.FindAll(text, "the")  ; [1, 32] (case-insensitive)
caseSensitivePos := StringSearcher.FindAll(text, "the", true)  ; [32] (only lowercase)

; Extract content between delimiters
htmlText := "<div>Content 1</div><span>Content 2</span><div>Content 3</div>"
divContents := StringSearcher.FindBetween(htmlText, "<div>", "</div>")
; Returns array with content: ["Content 1", "Content 3"]

; Check for multiple conditions
keywords := ["fox", "cat", "bird"]
result := StringSearcher.ContainsAny(text, keywords)
if (result.found) {
    MsgBox("Found '" . result.needle . "' at position " . result.position)
}

; Count occurrences
count := StringSearcher.CountOccurrences("banana", "na")  ; Returns 2 (non-overlapping)
overlapCount := StringSearcher.CountOccurrences("banana", "na", false, true)  ; Returns 2
```

```cpp
; Data validation and parsing using InStr
class DataValidator {
    static ValidateEmail(email) {
        local validation := {
            isValid: false,
            errors: Array(),
            parts: {local: "", domain: ""}
        }
        
        ; Basic structure check
        local atPos := InStr(email, "@")
        if (atPos == 0) {
            validation.errors.Push("Missing @ symbol")
            return validation
        }
        
        if (InStr(email, "@", false, atPos + 1) > 0) {
            validation.errors.Push("Multiple @ symbols")
            return validation
        }
        
        ; Extract parts
        validation.parts.local := SubStr(email, 1, atPos - 1)
        validation.parts.domain := SubStr(email, atPos + 1)
        
        ; Validate local part
        if (StrLen(validation.parts.local) == 0) {
            validation.errors.Push("Empty local part")
        } else {
            if (InStr(validation.parts.local, "..") > 0) {
                validation.errors.Push("Consecutive dots in local part")
            }
            if (SubStr(validation.parts.local, 1, 1) == "." || 
                SubStr(validation.parts.local, -1) == ".") {
                validation.errors.Push("Local part cannot start or end with dot")
            }
        }
        
        ; Validate domain part
        if (StrLen(validation.parts.domain) == 0) {
            validation.errors.Push("Empty domain part")
        } else {
            local dotPos := InStr(validation.parts.domain, ".")
            if (dotPos == 0) {
                validation.errors.Push("Domain missing top-level domain")
            } else {
                local tld := SubStr(validation.parts.domain, dotPos + 1)
                if (StrLen(tld) < 2) {
                    validation.errors.Push("Top-level domain too short")
                }
            }
        }
        
        validation.isValid := validation.errors.Length == 0
        return validation
    }
    
    static ParseURL(url) {
        local parsed := {
            protocol: "",
            domain: "",
            port: "",
            path: "",
            query: "",
            fragment: "",
            isValid: false
        }
        
        try {
            local remaining := url
            
            ; Extract protocol
            local protocolPos := InStr(remaining, "://")
            if (protocolPos > 0) {
                parsed.protocol := SubStr(remaining, 1, protocolPos - 1)
                remaining := SubStr(remaining, protocolPos + 3)
            }
            
            ; Extract fragment (anchor)
            local fragmentPos := InStr(remaining, "#")
            if (fragmentPos > 0) {
                parsed.fragment := SubStr(remaining, fragmentPos + 1)
                remaining := SubStr(remaining, 1, fragmentPos - 1)
            }
            
            ; Extract query string
            local queryPos := InStr(remaining, "?")
            if (queryPos > 0) {
                parsed.query := SubStr(remaining, queryPos + 1)
                remaining := SubStr(remaining, 1, queryPos - 1)
            }
            
            ; Extract path
            local pathPos := InStr(remaining, "/")
            if (pathPos > 0) {
                parsed.path := SubStr(remaining, pathPos)
                remaining := SubStr(remaining, 1, pathPos - 1)
            }
            
            ; Extract port
            local portPos := InStr(remaining, ":")
            if (portPos > 0) {
                parsed.port := SubStr(remaining, portPos + 1)
                remaining := SubStr(remaining, 1, portPos - 1)
            }
            
            ; Remaining is domain
            parsed.domain := remaining
            
            ; Basic validation
            parsed.isValid := StrLen(parsed.domain) > 0
            
            return parsed
            
        } catch Error as err {
            parsed.error := err.Message
            return parsed
        }
    }
    
    static ValidateIPAddress(ip) {
        local validation := {
            isValid: false,
            type: "",
            octets: Array(),
            errors: Array()
        }
        
        ; Count dots for IPv4 validation
        local dotCount := StringSearcher.CountOccurrences(ip, ".")
        
        if (dotCount == 3) {
            ; IPv4 validation
            validation.type := "IPv4"
            local parts := StrSplit(ip, ".")
            
            if (parts.Length != 4) {
                validation.errors.Push("IPv4 must have exactly 4 octets")
                return validation
            }
            
            for index, part in parts {
                if (!IsNumber(part)) {
                    validation.errors.Push("Octet " . index . " is not a number: " . part)
                    continue
                }
                
                local num := Number(part)
                if (num < 0 || num > 255) {
                    validation.errors.Push("Octet " . index . " out of range (0-255): " . num)
                }
                
                validation.octets.Push(num)
            }
            
        } else if (InStr(ip, ":") > 0) {
            ; Basic IPv6 detection
            validation.type := "IPv6"
            validation.errors.Push("IPv6 validation not implemented")
            
        } else {
            validation.errors.Push("Unknown IP address format")
        }
        
        validation.isValid := validation.errors.Length == 0
        return validation
    }
    
    static ParseCSVLine(line, delimiter := ",") {
        local fields := Array()
        local currentField := ""
        local inQuotes := false
        local pos := 1
        
        while (pos <= StrLen(line)) {
            local char := SubStr(line, pos, 1)
            
            if (char == '"') {
                if (inQuotes) {
                    ; Check for escaped quote
                    if (pos < StrLen(line) && SubStr(line, pos + 1, 1) == '"') {
                        currentField .= '"'
                        pos += 2
                        continue
                    } else {
                        inQuotes := false
                    }
                } else {
                    inQuotes := true
                }
            } else if (char == delimiter && !inQuotes) {
                fields.Push(currentField)
                currentField := ""
            } else {
                currentField .= char
            }
            
            pos++
        }
        
        ; Add final field
        fields.Push(currentField)
        
        return {
            fields: fields,
            count: fields.Length,
            hasQuotedFields: InStr(line, '"') > 0
        }
    }
}

; Usage examples
emailResult := DataValidator.ValidateEmail("user@domain.com")
if (emailResult.isValid) {
    MsgBox("Valid email: " . emailResult.parts.local . " @ " . emailResult.parts.domain)
} else {
    MsgBox("Invalid email: " . StrJoin(emailResult.errors, ", "))
}

urlResult := DataValidator.ParseURL("https://www.example.com:8080/path?query=value#fragment")
MsgBox("Protocol: " . urlResult.protocol . ", Domain: " . urlResult.domain . ", Port: " . urlResult.port)

ipResult := DataValidator.ValidateIPAddress("192.168.1.1")
if (ipResult.isValid) {
    MsgBox("Valid " . ipResult.type . " address")
}

csvResult := DataValidator.ParseCSVLine('Name,"John, Jr.",25,"New York"')
for index, field in csvResult.fields {
    MsgBox("Field " . index . ": " . field)
}
```

## Real-World Example

```cpp
; Log analysis system using InStr for pattern detection
class LogAnalyzer {
    static patterns := Map(
        "error", ["ERROR", "FATAL", "EXCEPTION", "FAILED"],
        "warning", ["WARN", "WARNING", "DEPRECATED"],
        "security", ["LOGIN", "LOGOUT", "AUTH", "PERMISSION", "ACCESS"],
        "performance", ["SLOW", "TIMEOUT", "MEMORY", "CPU", "PERFORMANCE"]
    )
    
    static AnalyzeLogFile(filePath, options := "") {
        local analysis := {
            fileName: filePath,
            totalLines: 0,
            patterns: Map(),
            timeline: Array(),
            summary: "",
            errors: Array()
        }
        
        ; Initialize pattern counters
        for category, patterns in this.patterns {
            analysis.patterns[category] := {
                count: 0,
                lines: Array(),
                examples: Array()
            }
        }
        
        try {
            local content := FileRead(filePath, "UTF-8")
            local lines := StrSplit(content, "`n")
            analysis.totalLines := lines.Length
            
            ; Analyze each line
            for lineNum, line in lines {
                line := Trim(line)
                if (line == "") {
                    continue
                }
                
                local lineAnalysis := this.AnalyzeLine(line, lineNum)
                
                ; Add to timeline
                if (lineAnalysis.timestamp != "") {
                    analysis.timeline.Push({
                        lineNumber: lineNum,
                        timestamp: lineAnalysis.timestamp,
                        categories: lineAnalysis.categories,
                        line: line
                    })
                }
                
                ; Update pattern counters
                for category in lineAnalysis.categories {
                    analysis.patterns[category].count++
                    analysis.patterns[category].lines.Push(lineNum)
                    
                    ; Store examples (limit to 5 per category)
                    if (analysis.patterns[category].examples.Length < 5) {
                        analysis.patterns[category].examples.Push({
                            line: lineNum,
                            text: line,
                            timestamp: lineAnalysis.timestamp
                        })
                    }
                }
            }
            
            ; Generate summary
            analysis.summary := this.GenerateAnalysisSummary(analysis)
            
        } catch Error as err {
            analysis.errors.Push("Failed to analyze file: " . err.Message)
        }
        
        return analysis
    }
    
    static AnalyzeLine(line, lineNumber) {
        local result := {
            lineNumber: lineNumber,
            timestamp: this.ExtractTimestamp(line),
            categories: Array(),
            severity: 0
        }
        
        ; Check each pattern category
        for category, patterns in this.patterns {
            for index, pattern in patterns {
                if (InStr(line, pattern, false) > 0) {
                    result.categories.Push(category)
                    
                    ; Assign severity scores
                    switch category {
                        case "error":
                            result.severity := Max(result.severity, 3)
                        case "warning":
                            result.severity := Max(result.severity, 2)
                        case "security":
                            result.severity := Max(result.severity, 2)
                        case "performance":
                            result.severity := Max(result.severity, 1)
                    }
                    
                    break  ; Found pattern in this category
                }
            }
        }
        
        return result
    }
    
    static ExtractTimestamp(line) {
        ; Look for common timestamp patterns
        local timestampPatterns := [
            "\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}",  ; 2024-01-15 10:30:45
            "\d{2}/\d{2}/\d{4} \d{2}:\d{2}:\d{2}",  ; 01/15/2024 10:30:45
            "\[\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}\]"  ; [2024-01-15T10:30:45]
        ]
        
        for index, pattern in timestampPatterns {
            if (RegExMatch(line, pattern, &match)) {
                return match[0]
            }
        }
        
        return ""
    }
    
    static GenerateAnalysisSummary(analysis) {
        local summary := "Log Analysis Summary\n"
        summary .= "===================\n"
        summary .= "File: " . analysis.fileName . "\n"
        summary .= "Total Lines: " . analysis.totalLines . "\n\n"
        
        ; Pattern summary
        local totalIssues := 0
        for category, data in analysis.patterns {
            if (data.count > 0) {
                summary .= StrUpper(SubStr(category, 1, 1)) . SubStr(category, 2) . " Issues: " . data.count . "\n"
                totalIssues += data.count
            }
        }
        
        summary .= "\nTotal Issues Found: " . totalIssues . "\n"
        
        ; Timeline summary
        if (analysis.timeline.Length > 0) {
            summary .= "\nTime Range: " . analysis.timeline[1].timestamp . " to " . analysis.timeline[-1].timestamp . "\n"
        }
        
        ; Top issues
        summary .= "\nTop Issue Categories:\n"
        local sortedCategories := Array()
        for category, data in analysis.patterns {
            if (data.count > 0) {
                sortedCategories.Push({category: category, count: data.count})
            }
        }
        
        ; Simple bubble sort by count (descending)
        for i in Range(1, sortedCategories.Length - 1) {
            for j in Range(1, sortedCategories.Length - i) {
                if (sortedCategories[j].count < sortedCategories[j + 1].count) {
                    local temp := sortedCategories[j]
                    sortedCategories[j] := sortedCategories[j + 1]
                    sortedCategories[j + 1] := temp
                }
            }
        }
        
        for index, item in sortedCategories {
            summary .= "  " . index . ". " . item.category . ": " . item.count . " occurrences\n"
            if (index >= 3) break  ; Top 3 only
        }
        
        return summary
    }
    
    static SearchLogForPattern(filePath, searchPattern, caseSensitive := false, maxResults := 100) {
        local results := {
            pattern: searchPattern,
            matches: Array(),
            totalMatches: 0,
            searchTime: 0
        }
        
        local startTime := A_TickCount
        
        try {
            local content := FileRead(filePath, "UTF-8")
            local lines := StrSplit(content, "`n")
            
            for lineNum, line in lines {
                local position := InStr(line, searchPattern, caseSensitive)
                if (position > 0) {
                    results.totalMatches++
                    
                    if (results.matches.Length < maxResults) {
                        results.matches.Push({
                            lineNumber: lineNum,
                            position: position,
                            line: Trim(line),
                            timestamp: this.ExtractTimestamp(line)
                        })
                    }
                }
            }
            
        } catch Error as err {
            results.error := err.Message
        }
        
        results.searchTime := A_TickCount - startTime
        return results
    }
    
    static GenerateLogReport(analysisResults) {
        local report := analysisResults.summary . "\n\n"
        
        ; Add detailed examples
        report .= "Detailed Examples:\n"
        report .= "==================\n"
        
        for category, data in analysisResults.patterns {
            if (data.count > 0 && data.examples.Length > 0) {
                report .= "\n" . StrUpper(category) . " Examples:\n"
                
                for index, example in data.examples {
                    report .= "  Line " . example.line . ": " . SubStr(example.text, 1, 80)
                    if (StrLen(example.text) > 80) {
                        report .= "..."
                    }
                    report .= "\n"
                }
            }
        }
        
        return report
    }
}

; Usage examples
analysis := LogAnalyzer.AnalyzeLogFile("application.log")
MsgBox(analysis.summary)

; Search for specific patterns
searchResults := LogAnalyzer.SearchLogForPattern("application.log", "DATABASE", false, 50)
if (searchResults.totalMatches > 0) {
    MsgBox("Found " . searchResults.totalMatches . " database-related entries in " . searchResults.searchTime . "ms")
    
    ; Show first few matches
    for index, match in searchResults.matches {
        if (index > 3) break  ; Show first 3
        MsgBox("Line " . match.lineNumber . ": " . match.line)
    }
}

; Generate detailed report
if (analysis.patterns["error"].count > 0) {
    report := LogAnalyzer.GenerateLogReport(analysis)
    FileAppend(report, "log_analysis_report.txt", "UTF-8")
    MsgBox("Detailed report saved to log_analysis_report.txt")
}
```

## Common Use Cases

- **Data Validation**: Checking if strings contain required substrings or patterns
- **Content Detection**: Finding specific keywords or patterns in text
- **Parsing Operations**: Locating delimiters and separators for data extraction
- **URL and Path Processing**: Finding protocol separators, file extensions, directory markers
- **Configuration Processing**: Locating key-value separators and section markers
- **Text Analysis**: Finding word boundaries, sentence markers, and formatting elements

## Performance Notes

- InStr is highly optimized with efficient string searching algorithms
- Case-insensitive searches have minimal additional overhead
- Performance scales well with string length for most practical applications
- Multiple searches on the same string benefit from string caching

## Common Pitfalls

### Pitfall 1: Not Handling "Not Found" Results
**Problem**: Assuming InStr always finds something and not checking for 0 return
**Solution**: Always validate InStr results before using positions
```cpp
; Problematic - assumes substring exists
pos := InStr(text, "target")
result := SubStr(text, pos, 5)  ; Error if pos is 0

; Correct - check if found
pos := InStr(text, "target")
if (pos > 0) {
    result := SubStr(text, pos, 5)
} else {
    result := ""  ; Handle not found case
}
```

### Pitfall 2: Case Sensitivity Confusion
**Problem**: Unexpected results due to case sensitivity settings
**Solution**: Explicitly specify case sensitivity when it matters
```cpp
; Problematic - relying on default case insensitivity
if (InStr(userInput, "YES")) {
    ; This matches "yes", "Yes", "YES", etc.
}

; Better - explicit case sensitivity for exact matching
if (InStr(userInput, "YES", true)) {
    ; Only matches exact "YES"
}
```

### Pitfall 3: Incorrect Starting Position Usage
**Problem**: Misunderstanding how starting position affects search results
**Solution**: Understand that starting position is where search begins, not offset to result
```cpp
; Confusing - thinking position is offset
text := "Hello World Hello"
pos := InStr(text, "Hello", false, 10)  ; Returns 13, not 3

; Clear understanding - position 10 is where search starts
; So it finds the second "Hello" at position 13
```

## Version History

- **v2.0**: Enhanced Unicode support and improved search performance
- **v2.0-a**: Better handling of edge cases with starting positions
- **v2.1**: Optimized case-insensitive search algorithms

## Related Functions

- [SubStr](../substr.md) - Extract substrings based on positions found by InStr
- [StrReplace](../strreplace.md) - Replace substrings often used with InStr for conditional replacement
- [RegExMatch](../../../10_Language_Core/01-Functions/Built_In_Functions/regexmatch.md) - Pattern-based searching for complex patterns
- [StrSplit](../strsplit.md) - Split strings using delimiters found by InStr

## Related Concepts

- [String Processing Patterns](../../../50_Ecosystem/00-Design_Patterns/string-patterns.md) - Advanced string manipulation using search operations
- [Data Parsing](../../../50_Ecosystem/00-Design_Patterns/data-parsing.md) - Using InStr for structured data extraction
- [Text Analysis](../../../50_Ecosystem/01-Best_Practices/text-analysis.md) - Analyzing text content using search operations

## See Also

- [String Processing Guide](../../../Index/learning-paths.md#string-processing) - Complete string manipulation techniques
- [Search and Replace Patterns](../../../50_Ecosystem/00-Design_Patterns/search-replace.md) - Combining search and replacement operations
- [Text Processing Performance](../../../50_Ecosystem/02-Performance_Optimization/text-performance.md) - Optimizing string search operations

## Tags

#AutoHotkey #Function #String #Search #Position #TextProcessing #DataValidation #Essential #BuiltIn #StringAnalysis