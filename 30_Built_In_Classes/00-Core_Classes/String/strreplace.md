# Function: StrReplace

## Category

Built-in Function

## Overview

StrReplace searches for and replaces occurrences of a substring within a string, providing essential text manipulation capabilities. It's fundamental for data processing, text formatting, and content transformation in AutoHotkey applications, supporting both simple text replacement and advanced pattern matching.

## Syntax

```cpp
NewString := StrReplace(Haystack, SearchText [, ReplaceText, CaseSensitive?, &OutputVarCount])
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Haystack | String | Yes | The string to search within |
| SearchText | String | Yes | The text to search for |
| ReplaceText | String | No | The replacement text. If omitted, SearchText is removed |
| CaseSensitive | Boolean/Integer | No | true/1 for case-sensitive, false/0 for case-insensitive (default) |
| &OutputVarCount | VarRef | No | Variable to receive the number of replacements made |

## Return Value

| Type | Description |
|------|-------------|
| String | A new string with the specified replacements made |

## Exceptions

| Exception Type | When Thrown |
|----------------|-------------|
| ValueError | When SearchText is empty string |
| TypeError | When parameters are not valid string types |

## Basic Examples

```cpp
; Simple text replacement
text := "Hello World"
newText := StrReplace(text, "World", "AutoHotkey")  ; Returns "Hello AutoHotkey"

; Remove text (omit replacement)
filename := "document.txt.backup"
cleanName := StrReplace(filename, ".backup")  ; Returns "document.txt"

; Case-insensitive replacement (default)
message := "ERROR: File not found"
fixed := StrReplace(message, "error", "Warning")  ; Returns "Warning: File not found"

; Case-sensitive replacement
message := "ERROR: File not found"
fixed := StrReplace(message, "ERROR", "Warning", true)  ; Returns "Warning: File not found"

; Count replacements
data := "apple,banana,apple,orange,apple"
result := StrReplace(data, "apple", "fruit", false, &count)
; result = "fruit,banana,fruit,orange,fruit"
; count = 3
```

## Advanced Examples

```cpp
; Advanced text processing with validation and error handling
ProcessTextData(inputText, replacements) {
    local processedText := inputText
    local totalReplacements := 0
    local errors := Array()
    
    try {
        ; Validate input
        if (StrLen(inputText) == 0) {
            throw ValueError("Input text cannot be empty")
        }
        
        ; Process each replacement
        for index, replacement in replacements {
            local searchText := replacement.search
            local replaceText := replacement.replace
            local caseSensitive := replacement.hasOwnProp("caseSensitive") ? replacement.caseSensitive : false
            local count := 0
            
            ; Validate replacement data
            if (StrLen(searchText) == 0) {
                errors.Push("Empty search text at index " . index)
                continue
            }
            
            ; Perform replacement
            processedText := StrReplace(processedText, searchText, replaceText, caseSensitive, &count)
            totalReplacements += count
            
            ; Log significant replacements
            if (count > 0) {
                OutputDebug("Replaced '" . searchText . "' " . count . " times")
            }
        }
        
        return {
            text: processedText,
            replacements: totalReplacements,
            errors: errors,
            success: true
        }
        
    } catch Error as err {
        return {
            text: inputText,
            replacements: 0,
            errors: [err.Message],
            success: false
        }
    }
}

; Usage example
replacements := [
    {search: "color", replace: "colour", caseSensitive: false},
    {search: "center", replace: "centre", caseSensitive: false},
    {search: "FILE_PATH", replace: "C:\Documents\", caseSensitive: true}
]

result := ProcessTextData("The color of the center file is at FILE_PATH.", replacements)
if (result.success) {
    MsgBox("Processed: " . result.text . "`nReplacements: " . result.replacements)
}
```

```cpp
; Template processing system using StrReplace
class TemplateProcessor {
    static placeholderPattern := "{{{([^}]+)}}}"
    
    static ProcessTemplate(template, data) {
        local processedTemplate := template
        local replacementCount := 0
        local missingVars := Array()
        
        ; Replace each placeholder with actual data
        for placeholder, value in data {
            local searchPattern := "{{{" . placeholder . "}}}"
            local count := 0
            
            processedTemplate := StrReplace(processedTemplate, searchPattern, value, false, &count)
            replacementCount += count
        }
        
        ; Check for unprocessed placeholders
        local remainingPlaceholders := this.FindUnprocessedPlaceholders(processedTemplate)
        
        return {
            result: processedTemplate,
            replacements: replacementCount,
            missingPlaceholders: remainingPlaceholders,
            isComplete: remainingPlaceholders.Length == 0
        }
    }
    
    static FindUnprocessedPlaceholders(text) {
        local placeholders := Array()
        local pos := 1
        
        ; Simple search for remaining {{{placeholder}}} patterns
        while (pos := InStr(text, "{{{", , pos)) {
            local endPos := InStr(text, "}}}", , pos + 3)
            if (endPos > 0) {
                local placeholder := SubStr(text, pos, endPos - pos + 3)
                placeholders.Push(placeholder)
                pos := endPos + 3
            } else {
                break
            }
        }
        
        return placeholders
    }
    
    static SanitizeForTemplate(text) {
        ; Escape characters that might interfere with template processing
        text := StrReplace(text, "{", "{{")
        text := StrReplace(text, "}", "}}")
        return text
    }
    
    static CreateTemplate(text, variables) {
        ; Convert text with variables into a template
        local template := text
        
        for variable, placeholder in variables {
            template := StrReplace(template, variable, "{{{" . placeholder . "}}}")
        }
        
        return template
    }
}

; Usage examples
templateText := "Dear {{{name}}}, your order #{{{orderNumber}}} for ${{{amount}}} has been {{{status}}}."

templateData := Map()
templateData["name"] := "John Smith"
templateData["orderNumber"] := "12345"
templateData["amount"] := "99.99"
templateData["status"] := "shipped"

result := TemplateProcessor.ProcessTemplate(templateText, templateData)
MsgBox("Result: " . result.result . "`nComplete: " . result.isComplete)
```

## Real-World Example

```cpp
; Data cleaning and normalization system
class DataCleaner {
    static CleanPhoneNumbers(text) {
        ; Standardize phone number formats
        local cleaned := text
        local count := 0
        
        ; Remove common phone number separators and normalize
        local phoneReplacements := [
            {search: "(", replace: ""},
            {search: ")", replace: ""},
            {search: "-", replace: ""},
            {search: " ", replace: ""},
            {search: ".", replace: ""}
        ]
        
        for index, replacement in phoneReplacements {
            cleaned := StrReplace(cleaned, replacement.search, replacement.replace, false, &tempCount)
            count += tempCount
        }
        
        return {text: cleaned, changes: count}
    }
    
    static CleanEmailData(emailText) {
        local cleaned := emailText
        local totalChanges := 0
        
        ; Common email cleaning operations
        local emailCleaners := [
            {search: "mailto:", replace: "", desc: "Remove mailto prefix"},
            {search: " ", replace: "", desc: "Remove spaces"},
            {search: Tab, replace: "", desc: "Remove tabs"},
            {search: "`r", replace: "", desc: "Remove carriage returns"},
            {search: "`n", replace: "", desc: "Remove line feeds"}
        ]
        
        local cleaningLog := Array()
        
        for index, cleaner in emailCleaners {
            local count := 0
            cleaned := StrReplace(cleaned, cleaner.search, cleaner.replace, false, &count)
            
            if (count > 0) {
                cleaningLog.Push(cleaner.desc . ": " . count . " changes")
                totalChanges += count
            }
        }
        
        ; Validate email format (basic check)
        local isValid := InStr(cleaned, "@") > 0 && InStr(cleaned, ".") > 0
        
        return {
            email: cleaned,
            changes: totalChanges,
            log: cleaningLog,
            isValid: isValid
        }
    }
    
    static CleanCSVData(csvText) {
        local cleaned := csvText
        local issues := Array()
        local fixCount := 0
        
        ; Common CSV data issues
        local csvFixes := [
            {search: '""', replace: '"', desc: "Fix double quotes"},
            {search: ",  ,", replace: ",,", desc: "Remove extra spaces in empty fields"},
            {search: ", ,", replace: ",,", desc: "Remove spaces around empty fields"},
            {search: "`r`n", replace: "`n", desc: "Normalize line endings"},
            {search: "`r", replace: "`n", desc: "Convert Mac line endings"}
        ]
        
        for index, fix in csvFixes {
            local count := 0
            cleaned := StrReplace(cleaned, fix.search, fix.replace, false, &count)
            
            if (count > 0) {
                issues.Push(fix.desc . ": " . count . " fixes")
                fixCount += count
            }
        }
        
        ; Count rows and estimate columns
        local lines := StrSplit(cleaned, "`n")
        local rowCount := lines.Length
        local avgColumns := 0
        
        if (rowCount > 0) {
            local totalCommas := 0
            for index, line in lines {
                totalCommas += StrLen(line) - StrLen(StrReplace(line, ",", ""))
            }
            avgColumns := Round(totalCommas / rowCount) + 1
        }
        
        return {
            data: cleaned,
            fixes: fixCount,
            issues: issues,
            rows: rowCount,
            estimatedColumns: avgColumns
        }
    }
    
    static ProcessBatchData(dataArray, cleaningType) {
        local results := Array()
        local totalChanges := 0
        
        for index, dataItem in dataArray {
            local result := ""
            
            switch cleaningType {
                case "phone":
                    result := this.CleanPhoneNumbers(dataItem)
                case "email":
                    result := this.CleanEmailData(dataItem)
                case "csv":
                    result := this.CleanCSVData(dataItem)
                default:
                    result := {text: dataItem, changes: 0, error: "Unknown cleaning type"}
            }
            
            results.Push(result)
            totalChanges += result.hasOwnProp("changes") ? result.changes : 0
        }
        
        return {
            results: results,
            totalChanges: totalChanges,
            itemsProcessed: dataArray.Length
        }
    }
}

; Usage examples
phoneData := ["(555) 123-4567", "555.987.6543", "555 111 2222"]
phoneResults := DataCleaner.ProcessBatchData(phoneData, "phone")

emailData := ["mailto:user@domain.com ", "  contact@site.org`n", "info@company.co.uk`t"]
emailResults := DataCleaner.ProcessBatchData(emailData, "email")

; Display results
for index, result in phoneResults.results {
    MsgBox("Phone " . index . ": " . result.text . " (" . result.changes . " changes)")
}
```

## Common Use Cases

- **Data Cleaning**: Removing unwanted characters and formatting inconsistencies
- **Text Normalization**: Standardizing text formats across datasets
- **Template Processing**: Replacing placeholders with actual values
- **File Path Manipulation**: Modifying file paths and extensions
- **Configuration Processing**: Updating configuration strings and parameters
- **Content Sanitization**: Removing or replacing problematic characters

## Performance Notes

- StrReplace has O(n*m) complexity where n is haystack length and m is search text length
- Multiple replacements on the same text should be batched when possible
- Case-insensitive searches are slightly slower than case-sensitive ones
- Very large strings may benefit from chunked processing for memory efficiency

## Common Pitfalls

### Pitfall 1: Empty Search String
**Problem**: Attempting to replace an empty string causes an error
**Solution**: Always validate search strings before replacement
```cpp
; Problematic - empty search string
searchText := ""
result := StrReplace("Hello World", searchText, "X")  ; Throws ValueError

; Correct - validate before replacement
if (StrLen(searchText) > 0) {
    result := StrReplace("Hello World", searchText, "X")
} else {
    result := "Hello World"  ; Return original if nothing to search
}
```

### Pitfall 2: Overlapping Replacements
**Problem**: Replacement text contains the search text, causing unintended recursive replacements
**Solution**: Plan replacement order carefully or use different approach
```cpp
; Problematic - replacement contains search text
text := "cat"
result := StrReplace(text, "cat", "catch")  ; Works fine: "catch"
result := StrReplace(result, "cat", "dog")   ; Becomes "dogh" (unexpected)

; Better - plan replacements carefully or use intermediate values
text := "cat"
temp := StrReplace(text, "cat", "TEMP_CAT")
result := StrReplace(temp, "TEMP_CAT", "dog")  ; Result: "dog"
```

### Pitfall 3: Case Sensitivity Confusion
**Problem**: Unexpected results due to case sensitivity settings
**Solution**: Explicitly specify case sensitivity when it matters
```cpp
; Problematic - relying on default case sensitivity
text := "HTML and html are different"
result := StrReplace(text, "html", "XML")  ; Replaces both HTML and html

; Better - explicit case sensitivity
text := "HTML and html are different"
result := StrReplace(text, "html", "XML", true)   ; Only replaces lowercase "html"
result := StrReplace(result, "HTML", "XML", true) ; Then replace "HTML"
```

## Version History

- **v2.0**: Enhanced Unicode support and improved performance for large strings
- **v2.0-a**: Added output count parameter for replacement tracking
- **v2.1**: Optimized memory usage for multiple consecutive replacements

## Related Functions

- [RegExReplace](../../../10_Language_Core/01-Functions/Built_In_Functions/regexreplace.md) - Pattern-based replacement using regular expressions
- [SubStr](../substr.md) - Extract portions of strings for targeted replacement
- [InStr](../instr.md) - Find positions for conditional replacement
- [Format](../../../10_Language_Core/01-Functions/Built_In_Functions/format.md) - String formatting as alternative to template replacement

## Related Concepts

- [String Manipulation Patterns](../../../50_Ecosystem/00-Design_Patterns/string-patterns.md) - Common string processing techniques
- [Data Validation](../../../50_Ecosystem/01-Best_Practices/data-validation.md) - Validating strings before processing
- [Regular Expressions](../../../10_Language_Core/01-Functions/Built_In_Functions/regex-overview.md) - Advanced pattern matching for complex replacements

## See Also

- [String Processing Guide](../../../Index/learning-paths.md#string-processing) - Comprehensive string manipulation techniques
- [Text Processing Best Practices](../../../50_Ecosystem/01-Best_Practices/text-processing.md) - Efficient text processing strategies
- [Data Cleaning Patterns](../../../50_Ecosystem/00-Design_Patterns/data-cleaning.md) - Common data cleaning approaches

## Tags

#AutoHotkey #Function #String #TextProcessing #Replace #DataCleaning #Manipulation #BuiltIn #Essential #TextTransformation