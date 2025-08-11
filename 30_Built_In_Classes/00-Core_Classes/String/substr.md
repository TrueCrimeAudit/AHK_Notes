# Function: SubStr

## Category

Built-in Function

## Overview

SubStr extracts a substring from a string, providing precise control over text extraction and manipulation. It's fundamental for data parsing, text processing, and string analysis, enabling developers to isolate specific portions of text based on position and length.

## Syntax

```cpp
NewString := SubStr(String, StartingPos [, Length])
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| String | String | Yes | The source string from which to extract the substring |
| StartingPos | Integer | Yes | The 1-based position where extraction begins. Negative values count from the end |
| Length | Integer | No | Number of characters to extract. If omitted, extracts to end of string |

## Return Value

| Type | Description |
|------|-------------|
| String | The extracted substring based on the specified parameters |

## Position and Length Behavior

### Starting Position Rules
| StartingPos | Behavior | Example with "Hello World" |
|-------------|----------|----------------------------|
| Positive (1+) | Count from beginning | `SubStr("Hello World", 7)` → "World" |
| Zero (0) | Treated as position 1 | `SubStr("Hello World", 0)` → "Hello World" |
| Negative (-1, -2...) | Count from end | `SubStr("Hello World", -5)` → "World" |

### Length Rules
| Length | Behavior | Example |
|--------|----------|---------|
| Positive | Extract specified number of characters | `SubStr("Hello", 1, 3)` → "Hel" |
| Zero | Return empty string | `SubStr("Hello", 1, 0)` → "" |
| Negative | Extract up to N characters before end | `SubStr("Hello", 1, -1)` → "Hell" |
| Omitted | Extract to end of string | `SubStr("Hello", 3)` → "llo" |

## Exceptions

| Exception Type | When Thrown |
|----------------|-------------|
| TypeError | When String parameter is not a string type |
| ValueError | When StartingPos or Length parameters are invalid |

## Basic Examples

```cpp
; Extract from beginning
text := "Hello World"
first5 := SubStr(text, 1, 5)      ; Returns "Hello"
greeting := SubStr(text, 1, 5)    ; Returns "Hello"

; Extract from middle
middle := SubStr(text, 7, 5)      ; Returns "World"
partial := SubStr(text, 3, 3)     ; Returns "llo"

; Extract from end
last4 := SubStr(text, -4)         ; Returns "orld"
lastWord := SubStr(text, -5)      ; Returns "World"

; Extract to end
fromPos := SubStr(text, 7)        ; Returns "World"
remaining := SubStr(text, 3)      ; Returns "llo World"

; Edge cases
empty := SubStr(text, 1, 0)       ; Returns ""
outOfBounds := SubStr(text, 20)   ; Returns ""
```

## Advanced Examples

```cpp
; Advanced string parsing with validation and error handling
ParseFixedWidthData(dataLine, fieldDefinitions) {
    local parsedData := Map()
    local errors := Array()
    
    try {
        ; Validate input
        if (StrLen(dataLine) == 0) {
            throw ValueError("Data line cannot be empty")
        }
        
        ; Parse each field according to definition
        for fieldName, fieldDef in fieldDefinitions {
            local startPos := fieldDef.start
            local length := fieldDef.length
            local dataType := fieldDef.hasOwnProp("type") ? fieldDef.type : "string"
            
            ; Validate field definition
            if (startPos < 1) {
                errors.Push("Invalid start position for field '" . fieldName . "': " . startPos)
                continue
            }
            
            ; Extract field value
            local rawValue := SubStr(dataLine, startPos, length)
            local cleanValue := Trim(rawValue)
            
            ; Type conversion
            local finalValue := cleanValue
            switch dataType {
                case "number":
                    if (IsNumber(cleanValue)) {
                        finalValue := Number(cleanValue)
                    } else {
                        errors.Push("Invalid number format in field '" . fieldName . "': " . cleanValue)
                        continue
                    }
                case "date":
                    ; Basic date validation (YYYY-MM-DD format)
                    if (RegExMatch(cleanValue, "^\d{4}-\d{2}-\d{2}$")) {
                        finalValue := cleanValue
                    } else {
                        errors.Push("Invalid date format in field '" . fieldName . "': " . cleanValue)
                        continue
                    }
                case "boolean":
                    finalValue := (cleanValue = "Y" || cleanValue = "1" || cleanValue = "true")
            }
            
            parsedData[fieldName] := finalValue
        }
        
        return {
            success: true,
            data: parsedData,
            errors: errors,
            originalLine: dataLine
        }
        
    } catch Error as err {
        return {
            success: false,
            data: Map(),
            errors: [err.Message],
            originalLine: dataLine
        }
    }
}

; Usage example with fixed-width data format
fieldDefs := Map()
fieldDefs["customerID"] := {start: 1, length: 10, type: "string"}
fieldDefs["lastName"] := {start: 11, length: 20, type: "string"}
fieldDefs["firstName"] := {start: 31, length: 15, type: "string"}
fieldDefs["birthDate"] := {start: 46, length: 10, type: "date"}
fieldDefs["balance"] := {start: 56, length: 12, type: "number"}
fieldDefs["active"] := {start: 68, length: 1, type: "boolean"}

dataLine := "CUST001234Smith               John           1985-03-15    1234.56Y"
result := ParseFixedWidthData(dataLine, fieldDefs)

if (result.success) {
    for field, value in result.data {
        MsgBox(field . ": " . value)
    }
}
```

```cpp
; String extraction utility class with advanced features
class StringExtractor {
    static ExtractBetween(text, startDelim, endDelim, includeDelimiters := false) {
        local results := Array()
        local searchPos := 1
        
        while (searchPos <= StrLen(text)) {
            ; Find start delimiter
            local startPos := InStr(text, startDelim, , searchPos)
            if (startPos == 0) {
                break
            }
            
            ; Find end delimiter after start
            local endPos := InStr(text, endDelim, , startPos + StrLen(startDelim))
            if (endPos == 0) {
                break
            }
            
            ; Extract content
            local extractStart := includeDelimiters ? startPos : startPos + StrLen(startDelim)
            local extractLength := includeDelimiters ? 
                (endPos + StrLen(endDelim) - startPos) : 
                (endPos - startPos - StrLen(startDelim))
            
            local extracted := SubStr(text, extractStart, extractLength)
            results.Push(extracted)
            
            ; Move search position past this match
            searchPos := endPos + StrLen(endDelim)
        }
        
        return results
    }
    
    static ExtractWords(text, maxWords := 0) {
        local words := Array()
        local currentWord := ""
        local inWord := false
        local wordCount := 0
        
        for i in Range(1, StrLen(text)) {
            local char := SubStr(text, i, 1)
            
            if (RegExMatch(char, "\w")) {  ; Word character
                currentWord .= char
                inWord := true
            } else {  ; Non-word character
                if (inWord && StrLen(currentWord) > 0) {
                    words.Push(currentWord)
                    wordCount++
                    
                    if (maxWords > 0 && wordCount >= maxWords) {
                        break
                    }
                    
                    currentWord := ""
                    inWord := false
                }
            }
        }
        
        ; Add final word if exists
        if (inWord && StrLen(currentWord) > 0) {
            words.Push(currentWord)
        }
        
        return words
    }
    
    static ExtractNumbers(text, includeDecimals := true) {
        local numbers := Array()
        local currentNumber := ""
        local inNumber := false
        local hasDecimal := false
        
        for i in Range(1, StrLen(text)) {
            local char := SubStr(text, i, 1)
            
            if (IsDigit(char)) {
                currentNumber .= char
                inNumber := true
            } else if (char == "." && includeDecimals && inNumber && !hasDecimal) {
                currentNumber .= char
                hasDecimal := true
            } else {
                if (inNumber && StrLen(currentNumber) > 0) {
                    ; Validate and add number
                    if (IsNumber(currentNumber)) {
                        numbers.Push(Number(currentNumber))
                    }
                    currentNumber := ""
                    inNumber := false
                    hasDecimal := false
                }
            }
        }
        
        ; Add final number if exists
        if (inNumber && StrLen(currentNumber) > 0 && IsNumber(currentNumber)) {
            numbers.Push(Number(currentNumber))
        }
        
        return numbers
    }
    
    static ExtractByPattern(text, pattern, groupIndex := 0) {
        local matches := Array()
        local pos := 1
        
        while (pos <= StrLen(text)) {
            if (RegExMatch(text, pattern, &match, pos)) {
                local extractedText := groupIndex > 0 ? match[groupIndex] : match[0]
                matches.Push({
                    text: extractedText,
                    position: match.Pos,
                    length: match.Len
                })
                pos := match.Pos + match.Len
            } else {
                break
            }
        }
        
        return matches
    }
    
    static TruncateWords(text, maxLength, ellipsis := "...") {
        if (StrLen(text) <= maxLength) {
            return text
        }
        
        ; Find last complete word within limit
        local truncated := SubStr(text, 1, maxLength - StrLen(ellipsis))
        local lastSpace := InStr(truncated, " ", , -1)  ; Last space position
        
        if (lastSpace > 0) {
            truncated := SubStr(truncated, 1, lastSpace - 1)
        }
        
        return truncated . ellipsis
    }
    
    static ExtractFileComponents(filePath) {
        ; Extract directory, filename, name, and extension
        local lastSlash := InStr(filePath, "\", , -1)
        local lastDot := InStr(filePath, ".", , -1)
        
        local directory := lastSlash > 0 ? SubStr(filePath, 1, lastSlash - 1) : ""
        local filename := lastSlash > 0 ? SubStr(filePath, lastSlash + 1) : filePath
        
        local name := ""
        local extension := ""
        
        if (lastDot > lastSlash) {
            name := SubStr(filename, 1, lastDot - lastSlash - 1)
            extension := SubStr(filename, lastDot - lastSlash)
        } else {
            name := filename
        }
        
        return {
            fullPath: filePath,
            directory: directory,
            filename: filename,
            name: name,
            extension: extension
        }
    }
}

; Usage examples
text := "The price is $123.45 and the discount is 15% off"
numbers := StringExtractor.ExtractNumbers(text)  ; [123.45, 15]

htmlText := "<div>Content 1</div><span>Content 2</span><div>Content 3</div>"
divContents := StringExtractor.ExtractBetween(htmlText, "<div>", "</div>")
; Returns ["Content 1", "Content 3"]

longText := "This is a very long sentence that needs to be truncated for display purposes"
short := StringExtractor.TruncateWords(longText, 30)  ; "This is a very long..."

filePath := "C:\Users\John\Documents\report.pdf"
components := StringExtractor.ExtractFileComponents(filePath)
; Returns: {fullPath: "C:\Users\John\Documents\report.pdf", directory: "C:\Users\John\Documents", 
;          filename: "report.pdf", name: "report", extension: ".pdf"}
```

## Real-World Example

```cpp
; Log file parser using SubStr for structured data extraction
class LogParser {
    static ParseLogFile(logPath, logFormat) {
        local entries := Array()
        local errors := Array()
        local lineNumber := 0
        
        try {
            local logContent := FileRead(logPath, "UTF-8")
            local lines := StrSplit(logContent, "`n")
            
            for index, line in lines {
                lineNumber := index
                line := Trim(line)
                
                if (line == "" || SubStr(line, 1, 1) == "#") {
                    continue  ; Skip empty lines and comments
                }
                
                local parsedEntry := this.ParseLogLine(line, logFormat, lineNumber)
                
                if (parsedEntry.success) {
                    entries.Push(parsedEntry.data)
                } else {
                    errors.Push(parsedEntry)
                }
            }
            
            return {
                success: true,
                entries: entries,
                errors: errors,
                totalLines: lines.Length,
                validEntries: entries.Length
            }
            
        } catch Error as err {
            return {
                success: false,
                error: err.Message,
                lineNumber: lineNumber,
                entries: entries,
                errors: errors
            }
        }
    }
    
    static ParseLogLine(line, format, lineNumber) {
        try {
            local entry := Map()
            
            switch format.type {
                case "common":
                    return this.ParseCommonLogFormat(line, lineNumber)
                case "combined":
                    return this.ParseCombinedLogFormat(line, lineNumber)
                case "custom":
                    return this.ParseCustomLogFormat(line, format.pattern, lineNumber)
                case "csv":
                    return this.ParseCSVLogFormat(line, format.fields, lineNumber)
                default:
                    throw ValueError("Unknown log format: " . format.type)
            }
        } catch Error as err {
            return {
                success: false,
                error: err.Message,
                line: line,
                lineNumber: lineNumber
            }
        }
    }
    
    static ParseCommonLogFormat(line, lineNumber) {
        ; Common Log Format: IP - - [timestamp] "request" status size
        local entry := Map()
        
        ; Extract IP address (first field)
        local firstSpace := InStr(line, " ")
        if (firstSpace == 0) {
            throw ValueError("Invalid log format - no spaces found")
        }
        entry["ip"] := SubStr(line, 1, firstSpace - 1)
        
        ; Find timestamp in brackets
        local bracketStart := InStr(line, "[")
        local bracketEnd := InStr(line, "]", , bracketStart)
        if (bracketStart == 0 || bracketEnd == 0) {
            throw ValueError("Invalid log format - timestamp brackets not found")
        }
        entry["timestamp"] := SubStr(line, bracketStart + 1, bracketEnd - bracketStart - 1)
        
        ; Extract request in quotes
        local quoteStart := InStr(line, '"', , bracketEnd)
        local quoteEnd := InStr(line, '"', , quoteStart + 1)
        if (quoteStart == 0 || quoteEnd == 0) {
            throw ValueError("Invalid log format - request quotes not found")
        }
        entry["request"] := SubStr(line, quoteStart + 1, quoteEnd - quoteStart - 1)
        
        ; Extract status and size (remaining fields)
        local remainingText := Trim(SubStr(line, quoteEnd + 1))
        local fields := StrSplit(remainingText, " ")
        
        if (fields.Length >= 2) {
            entry["status"] := fields[1]
            entry["size"] := fields[2]
        }
        
        return {
            success: true,
            data: entry,
            lineNumber: lineNumber
        }
    }
    
    static ParseCustomLogFormat(line, pattern, lineNumber) {
        ; Parse using custom field positions
        local entry := Map()
        
        for fieldName, fieldConfig in pattern {
            local value := ""
            
            if (fieldConfig.hasOwnProp("start") && fieldConfig.hasOwnProp("length")) {
                ; Fixed-width field
                value := SubStr(line, fieldConfig.start, fieldConfig.length)
            } else if (fieldConfig.hasOwnProp("delimiter")) {
                ; Delimited field
                local parts := StrSplit(line, fieldConfig.delimiter)
                if (fieldConfig.index <= parts.Length) {
                    value := parts[fieldConfig.index]
                }
            } else if (fieldConfig.hasOwnProp("regex")) {
                ; Regex extraction
                if (RegExMatch(line, fieldConfig.regex, &match)) {
                    value := match[0]
                }
            }
            
            ; Apply transformations
            value := Trim(value)
            if (fieldConfig.hasOwnProp("type")) {
                switch fieldConfig.type {
                    case "number":
                        value := IsNumber(value) ? Number(value) : 0
                    case "date":
                        ; Keep as string, but validate format
                        if (!RegExMatch(value, "^\d{4}-\d{2}-\d{2}")) {
                            value := ""
                        }
                }
            }
            
            entry[fieldName] := value
        }
        
        return {
            success: true,
            data: entry,
            lineNumber: lineNumber
        }
    }
    
    static AnalyzeParsedLogs(entries) {
        local analysis := {
            totalEntries: entries.Length,
            ipAddresses: Map(),
            statusCodes: Map(),
            requestMethods: Map(),
            timeRange: {earliest: "", latest: ""},
            topIPs: Array(),
            errorEntries: Array()
        }
        
        ; Analyze each entry
        for index, entry in entries {
            ; Count IP addresses
            if (entry.Has("ip")) {
                local ip := entry["ip"]
                analysis.ipAddresses[ip] := analysis.ipAddresses.Has(ip) ? analysis.ipAddresses[ip] + 1 : 1
            }
            
            ; Count status codes
            if (entry.Has("status")) {
                local status := entry["status"]
                analysis.statusCodes[status] := analysis.statusCodes.Has(status) ? analysis.statusCodes[status] + 1 : 1
                
                ; Track errors (4xx, 5xx)
                if (SubStr(status, 1, 1) == "4" || SubStr(status, 1, 1) == "5") {
                    analysis.errorEntries.Push(entry)
                }
            }
            
            ; Extract request methods
            if (entry.Has("request")) {
                local method := SubStr(entry["request"], 1, InStr(entry["request"], " ") - 1)
                analysis.requestMethods[method] := analysis.requestMethods.Has(method) ? analysis.requestMethods[method] + 1 : 1
            }
            
            ; Track time range
            if (entry.Has("timestamp")) {
                local timestamp := entry["timestamp"]
                if (analysis.timeRange.earliest == "" || timestamp < analysis.timeRange.earliest) {
                    analysis.timeRange.earliest := timestamp
                }
                if (analysis.timeRange.latest == "" || timestamp > analysis.timeRange.latest) {
                    analysis.timeRange.latest := timestamp
                }
            }
        }
        
        ; Generate top IPs list
        for ip, count in analysis.ipAddresses {
            analysis.topIPs.Push({ip: ip, count: count})
        }
        
        return analysis
    }
}

; Usage examples
commonLogFormat := {type: "common"}
result := LogParser.ParseLogFile("access.log", commonLogFormat)

if (result.success) {
    MsgBox("Parsed " . result.validEntries . " entries from " . result.totalLines . " lines")
    
    if (result.entries.Length > 0) {
        analysis := LogParser.AnalyzeParsedLogs(result.entries)
        MsgBox("Found " . analysis.ipAddresses.Count . " unique IP addresses")
    }
}

; Custom format example
customFormat := {
    type: "custom",
    pattern: Map(
        "timestamp", {start: 1, length: 19},
        "level", {start: 21, length: 5},
        "component", {start: 27, length: 10},
        "message", {start: 38, length: -1}
    )
}
customResult := LogParser.ParseLogFile("application.log", customFormat)
```

## Common Use Cases

- **Data Parsing**: Extracting specific fields from structured text data
- **String Cleaning**: Removing unwanted prefixes, suffixes, or middle portions
- **Text Analysis**: Isolating words, sentences, or specific patterns for analysis
- **File Path Processing**: Extracting filenames, extensions, or directory paths
- **Format Conversion**: Converting between different text formats and structures
- **Content Truncation**: Creating previews or summaries of longer text

## Performance Notes

- SubStr is highly optimized with O(1) time complexity for position calculation
- Very large strings may require memory considerations for the returned substring
- Negative positions require calculation from string end, adding minimal overhead
- Multiple SubStr operations on the same string are efficient and can be chained

## Common Pitfalls

### Pitfall 1: Off-by-One Position Errors
**Problem**: Confusion between 0-based and 1-based indexing
**Solution**: Remember AutoHotkey uses 1-based indexing for strings
```cpp
; Problematic - thinking 0-based
text := "Hello"
first := SubStr(text, 0, 1)  ; Returns "H" (0 treated as 1)

; Correct - use 1-based indexing
text := "Hello"
first := SubStr(text, 1, 1)  ; Returns "H"
second := SubStr(text, 2, 1) ; Returns "e"
```

### Pitfall 2: Out of Bounds Extraction
**Problem**: Not handling cases where positions exceed string length
**Solution**: Validate positions or handle empty results gracefully
```cpp
; Problematic - no bounds checking
text := "Short"
result := SubStr(text, 10, 5)  ; Returns "" (empty string)

; Better - check bounds first
text := "Short"
if (StrLen(text) >= 10) {
    result := SubStr(text, 10, 5)
} else {
    result := ""  ; Or handle appropriately
}

; Best - use safe extraction function
SafeSubStr(str, start, length := -1) {
    if (start < 1 || start > StrLen(str)) {
        return ""
    }
    if (length == -1) {
        return SubStr(str, start)
    }
    return SubStr(str, start, Min(length, StrLen(str) - start + 1))
}
```

### Pitfall 3: Negative Length Confusion
**Problem**: Misunderstanding negative length behavior
**Solution**: Understand that negative length removes characters from the end
```cpp
; Confusing - negative length behavior
text := "Hello World"
result := SubStr(text, 1, -2)  ; Returns "Hello Wor" (removes 2 from end)

; Clear alternative using StrLen calculation
text := "Hello World"
result := SubStr(text, 1, StrLen(text) - 2)  ; Same result, clearer intent
```

## Version History

- **v2.0**: Enhanced Unicode support and improved performance for large strings
- **v2.0-a**: Better handling of edge cases with negative positions and lengths
- **v2.1**: Optimized memory usage for substring operations

## Related Functions

- [InStr](../instr.md) - Find position of substrings for use with SubStr
- [StrLen](../strlen.md) - Get string length for position calculations
- [StrSplit](../strsplit.md) - Split strings into arrays as alternative to multiple SubStr calls
- [Mid](../mid.md) - Alternative function for substring extraction (v1 compatibility)

## Related Concepts

- [String Processing Patterns](../../../50_Ecosystem/00-Design_Patterns/string-patterns.md) - Advanced string manipulation techniques
- [Data Parsing](../../../50_Ecosystem/00-Design_Patterns/data-parsing.md) - Using SubStr for structured data extraction
- [Text Analysis](../../../50_Ecosystem/01-Best_Practices/text-analysis.md) - Analyzing text content using substring operations

## See Also

- [String Processing Guide](../../../Index/learning-paths.md#string-processing) - Complete string manipulation techniques
- [Data Extraction Patterns](../../../50_Ecosystem/00-Design_Patterns/data-extraction.md) - Patterns for extracting data from text
- [Text Processing Performance](../../../50_Ecosystem/02-Performance_Optimization/text-performance.md) - Optimizing string operations

## Tags

#AutoHotkey #Function #String #SubString #Extraction #TextProcessing #DataParsing #Essential #BuiltIn #StringManipulation