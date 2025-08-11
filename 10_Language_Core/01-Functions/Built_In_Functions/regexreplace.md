# Topic: RegExReplace Function - Advanced Pattern-Based String Replacement

## Category

Function

## Overview

The RegExReplace function performs advanced pattern-based string replacement using regular expressions, providing powerful text transformation capabilities. It's essential for complex text processing, data transformation, content filtering, and any application requiring sophisticated pattern matching and replacement operations.

## Key Points

- Performs pattern-based string replacement using full regular expression syntax for complex transformations
- Supports capture groups, backreferences, and replacement patterns for advanced text manipulation
- Provides comprehensive modifier flags for case sensitivity, multiline processing, and global replacement
- Enables complex text processing including data validation, content filtering, and format conversion
- Essential for data transformation, content management, log processing, and automated text manipulation

## Syntax and Parameters

```cpp
NewStr := RegExReplace(Haystack, NeedleRegEx [, Replacement, &OutputVarCount, Limit, StartingPos])

; Parameters:
; Haystack       - String to search within and replace
; NeedleRegEx    - Regular expression pattern with optional flags
; Replacement    - Replacement string with optional backreferences (default: empty string)
; OutputVarCount - Variable to receive count of replacements made (optional, by reference)
; Limit          - Maximum number of replacements to perform (default: -1 = all)
; StartingPos    - 1-based position to start search (default: 1)

; Pattern flags (prefix pattern with "(?flags)"):
; i - Case insensitive matching
; m - Multiline mode (^ and $ match line boundaries)
; s - Single line mode (. matches newlines)
; x - Extended syntax (ignore whitespace in pattern)
; g - Global replacement (replace all occurrences) - default behavior
; U - Ungreedy matching (quantifiers are non-greedy by default)

; Replacement patterns:
; $0 or $& - Entire match
; $1, $2... - Captured groups
; $` - Text before match
; $' - Text after match
; $$ - Literal dollar sign

; Return Value:
; Returns new string with replacements made
; Original string unchanged
```

## Code Examples

```cpp
; Basic pattern replacement
text := "Hello world, wonderful world!"
result := RegExReplace(text, "world", "universe")
; Result: "Hello universe, wonderful universe!"

; Case-insensitive replacement
text := "Apple, APPLE, apple"
result := RegExReplace(text, "(?i)apple", "orange")
; Result: "orange, orange, orange"

; Using capture groups and backreferences
phoneNumbers := "Call 555-123-4567 or 555-987-6543"
formatted := RegExReplace(phoneNumbers, "(\d{3})-(\d{3})-(\d{4})", "($1) $2-$3")
; Result: "Call (555) 123-4567 or (555) 987-6543"

; Limit replacements
text := "test test test test"
result := RegExReplace(text, "test", "demo", , 2)  ; Replace only first 2
; Result: "demo demo test test"

; Count replacements
text := "cat bat rat"
result := RegExReplace(text, ".at", "pet", &count)
; Result: "pet pet pet", count = 3

; Advanced text processor for data transformation
class TextProcessor {
    static rules := Map()
    static globalFlags := "g"  ; Global replacement by default
    
    static AddRule(name, pattern, replacement, flags := "") {
        this.rules[name] := {
            pattern: pattern,
            replacement: replacement,
            flags: flags ? flags : this.globalFlags,
            enabled: true,
            appliedCount: 0
        }
    }
    
    static ProcessText(text, ruleNames := "") {
        if (!ruleNames) {
            ; Apply all enabled rules
            ruleNames := []
            for name, rule in this.rules {
                if (rule.enabled) {
                    ruleNames.Push(name)
                }
            }
        }
        
        processedText := text
        results := Map()
        
        for ruleName in ruleNames {
            if (!this.rules.Has(ruleName)) {
                continue
            }
            
            rule := this.rules[ruleName]
            if (!rule.enabled) {
                continue
            }
            
            ; Apply rule and count replacements
            oldText := processedText
            processedText := RegExReplace(processedText, 
                                        "(?)" . rule.flags . rule.pattern, 
                                        rule.replacement, 
                                        &replacementCount)
            
            rule.appliedCount += replacementCount
            
            results[ruleName] := {
                replacements: replacementCount,
                textChanged: oldText != processedText
            }
        }
        
        return {
            originalText: text,
            processedText: processedText,
            ruleResults: results
        }
    }
    
    static EnableRule(name) {
        if (this.rules.Has(name)) {
            this.rules[name].enabled := true
        }
    }
    
    static DisableRule(name) {
        if (this.rules.Has(name)) {
            this.rules[name].enabled := false
        }
    }
    
    static GetRuleStatistics() {
        stats := Map()
        for name, rule in this.rules {
            stats[name] := {
                enabled: rule.enabled,
                totalApplications: rule.appliedCount,
                pattern: rule.pattern,
                replacement: rule.replacement
            }
        }
        return stats
    }
    
    static ClearStatistics() {
        for name, rule in this.rules {
            rule.appliedCount := 0
        }
    }
}

; Data sanitization and validation processor
class DataSanitizer {
    static Initialize() {
        ; Define common sanitization rules
        
        ; Remove HTML tags
        TextProcessor.AddRule("stripHTML", "<[^>]*>", "")
        
        ; Normalize whitespace
        TextProcessor.AddRule("normalizeWhitespace", "\s+", " ")
        
        ; Remove leading/trailing whitespace
        TextProcessor.AddRule("trimWhitespace", "^\s+|\s+$", "")
        
        ; Normalize phone numbers
        TextProcessor.AddRule("normalizePhone", "(\d{3})[^\d]*(\d{3})[^\d]*(\d{4})", "$1-$2-$3")
        
        ; Normalize email addresses (simple)
        TextProcessor.AddRule("normalizeEmail", "(?i)([a-z0-9._%+-]+)@([a-z0-9.-]+\.[a-z]{2,})", "$1@$2")
        
        ; Remove dangerous characters for SQL/XSS prevention
        TextProcessor.AddRule("removeDangerous", "[<>\"';&]", "")
        
        ; Normalize line endings
        TextProcessor.AddRule("normalizeLineEndings", "\r\n|\r", "`n")
        
        ; Remove excessive punctuation
        TextProcessor.AddRule("normalizeEllipsis", "\.{3,}", "...")
        TextProcessor.AddRule("normalizeExclamation", "!{2,}", "!")
        TextProcessor.AddRule("normalizeQuestion", "\?{2,}", "?")
    }
    
    static SanitizeText(text, level := "basic") {
        switch level {
            case "basic":
                rules := ["normalizeWhitespace", "trimWhitespace"]
            case "web":
                rules := ["stripHTML", "normalizeWhitespace", "trimWhitespace", "removeDangerous"]
            case "data":
                rules := ["normalizePhone", "normalizeEmail", "normalizeWhitespace", "trimWhitespace"]
            case "document":
                rules := ["normalizeLineEndings", "normalizeWhitespace", "normalizeEllipsis", 
                         "normalizeExclamation", "normalizeQuestion", "trimWhitespace"]
            case "all":
                rules := []  ; Use all enabled rules
            default:
                throw ValueError("Unknown sanitization level: " . level)
        }
        
        return TextProcessor.ProcessText(text, rules)
    }
    
    static ValidateData(text, validationType) {
        validationRules := Map(
            "email", "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$",
            "phone", "^\d{3}-\d{3}-\d{4}$|^\(\d{3}\)\s?\d{3}-\d{4}$",
            "url", "^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$",
            "zipcode", "^\d{5}(-\d{4})?$",
            "ssn", "^\d{3}-\d{2}-\d{4}$",
            "creditcard", "^\d{4}[- ]?\d{4}[- ]?\d{4}[- ]?\d{4}$"
        )
        
        if (!validationRules.Has(validationType)) {
            throw ValueError("Unknown validation type: " . validationType)
        }
        
        pattern := validationRules[validationType]
        return RegExMatch(text, pattern) > 0
    }
}

; Log file processor with pattern-based analysis
class LogAnalyzer {
    static logPatterns := Map()
    static Initialize() {
        ; Define common log patterns
        this.logPatterns := Map(
            "timestamp", "\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}",
            "ipAddress", "\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b",
            "httpStatus", "\b[1-5]\d{2}\b",
            "userAgent", "User-Agent: ([^\r\n]+)",
            "errorLevel", "\[(ERROR|WARN|WARNING|FATAL|CRITICAL)\]",
            "sessionId", "session[_-]?id[:\s=]*([a-zA-Z0-9]+)",
            "userId", "user[_-]?id[:\s=]*(\d+)",
            "sqlQuery", "(?i)(SELECT|INSERT|UPDATE|DELETE).*?(?=\s|$|;)"
        )
    }
    
    static AnalyzeLogFile(filePath) {
        if (!FileExist(filePath)) {
            throw FileNotFoundError("Log file not found: " . filePath)
        }
        
        content := FileRead(filePath)
        return this.AnalyzeLogContent(content)
    }
    
    static AnalyzeLogContent(content) {
        analysis := Map()
        
        ; Extract different types of information
        for patternName, pattern in this.logPatterns {
            matches := []
            pos := 1
            
            while (pos := RegExMatch(content, pattern, &match, pos)) {
                matches.Push({
                    match: match[0],
                    position: pos,
                    groups: this.ExtractGroups(match)
                })
                pos += match.Len[0]
            }
            
            analysis[patternName] := {
                count: matches.Length,
                matches: matches
            }
        }
        
        return analysis
    }
    
    static ExtractGroups(match) {
        groups := []
        for i in Range(1, match.Count) {
            groups.Push(match[i])
        }
        return groups
    }
    
    static FilterLogsByPattern(content, filterPattern, include := true) {
        lines := StrSplit(content, "`n", "`r")
        filteredLines := []
        
        for line in lines {
            matchFound := RegExMatch(line, filterPattern) > 0
            
            if ((include && matchFound) || (!include && !matchFound)) {
                filteredLines.Push(line)
            }
        }
        
        return filteredLines.Join("`n")
    }
    
    static ExtractErrorsWithContext(content, contextLines := 2) {
        lines := StrSplit(content, "`n", "`r")
        errors := []
        
        for index, line in lines {
            if (RegExMatch(line, "(?i)(error|exception|fatal|critical)")) {
                ; Extract context around error
                contextStart := Max(1, index - contextLines)
                contextEnd := Min(lines.Length, index + contextLines)
                
                context := []
                for i in Range(contextStart, contextEnd) {
                    context.Push(lines[i])
                }
                
                errors.Push({
                    errorLine: line,
                    lineNumber: index,
                    context: context.Join("`n")
                })
            }
        }
        
        return errors
    }
    
    static GenerateLogReport(analysis) {
        report := "Log Analysis Report`n"
        report .= "==================`n`n"
        
        for patternName, data in analysis {
            report .= patternName . ": " . data.count . " occurrences`n"
            
            if (data.count > 0 && data.count <= 10) {
                ; Show examples for small counts
                report .= "  Examples:`n"
                for match in data.matches {
                    report .= "    " . match.match . "`n"
                }
            }
            report .= "`n"
        }
        
        return report
    }
}

; Content transformation engine
class ContentTransformer {
    static transformations := Map()
    
    static RegisterTransformation(name, patterns) {
        ; patterns should be array of {search: "pattern", replace: "replacement"}
        this.transformations[name] := patterns
    }
    
    static Transform(content, transformationName) {
        if (!this.transformations.Has(transformationName)) {
            throw ValueError("Unknown transformation: " . transformationName)
        }
        
        patterns := this.transformations[transformationName]
        result := content
        appliedCount := 0
        
        for pattern in patterns {
            result := RegExReplace(result, pattern.search, pattern.replace, &count)
            appliedCount += count
        }
        
        return {
            originalContent: content,
            transformedContent: result,
            transformationsApplied: appliedCount
        }
    }
    
    static InitializeCommonTransformations() {
        ; Markdown to HTML conversion (basic)
        this.RegisterTransformation("markdownToHTML", [
            {search: "^# (.+)$", replace: "<h1>$1</h1>"},
            {search: "^## (.+)$", replace: "<h2>$1</h2>"},
            {search: "^### (.+)$", replace: "<h3>$1</h3>"},
            {search: "\*\*(.+?)\*\*", replace: "<strong>$1</strong>"},
            {search: "\*(.+?)\*", replace: "<em>$1</em>"},
            {search: "`(.+?)`", replace: "<code>$1</code>"},
            {search: "^- (.+)$", replace: "<li>$1</li>"}
        ])
        
        ; Clean phone number formatting
        this.RegisterTransformation("cleanPhone", [
            {search: "\((\d{3})\)\s*(\d{3})-(\d{4})", replace: "$1-$2-$3"},
            {search: "(\d{3})\.(\d{3})\.(\d{4})", replace: "$1-$2-$3"},
            {search: "(\d{3})\s+(\d{3})\s+(\d{4})", replace: "$1-$2-$3"}
        ])
        
        ; Normalize currency formatting
        this.RegisterTransformation("normalizeCurrency", [
            {search: "\$\s*(\d+)\.(\d{2})", replace: "$$$1.$2"},
            {search: "\$\s*(\d+)", replace: "$$$1.00"}
        ])
        
        ; Remove extra spacing and formatting
        this.RegisterTransformation("cleanText", [
            {search: "\s{2,}", replace: " "},
            {search: "^\s+|\s+$", replace: ""},
            {search: "\n\s*\n\s*\n", replace: "`n`n"}
        ])
    }
}

; Example usage with comprehensive text processing
; Initialize processors
DataSanitizer.Initialize()
LogAnalyzer.Initialize()
ContentTransformer.InitializeCommonTransformations()

; Basic text processing
text := "Contact us at (555) 123-4567 or email support@example.com"
processed := RegExReplace(text, "(\d{3})\D*(\d{3})\D*(\d{4})", "$1-$2-$3")
; Result: "Contact us at 555-123-4567 or email support@example.com"

; Advanced data sanitization
userInput := "<script>alert('xss')</script>Hello    World!   "
sanitized := DataSanitizer.SanitizeText(userInput, "web")
; Result: Clean text without HTML tags and normalized whitespace

; Log analysis
logContent := FileRead("application.log")
analysis := LogAnalyzer.AnalyzeLogContent(logContent)
report := LogAnalyzer.GenerateLogReport(analysis)

; Content transformation
markdown := "# Header`n## Subheader`n**Bold text** and *italic text*"
html := ContentTransformer.Transform(markdown, "markdownToHTML")

; Custom text processing rules
TextProcessor.AddRule("replacePasswords", "password[:\s=]+\S+", "password: [REDACTED]", "gi")
TextProcessor.AddRule("maskCreditCards", "\b\d{4}[- ]?\d{4}[- ]?\d{4}[- ]?\d{4}\b", "****-****-****-****")

secureText := TextProcessor.ProcessText("User password: secret123 and card 1234-5678-9012-3456")
; Result: Text with sensitive data masked
```

## Implementation Notes

**Pattern Performance:**
- Complex patterns with extensive backtracking can be slow
- Use atomic groups and possessive quantifiers for optimization
- Test patterns with large text samples to identify performance issues
- Consider breaking complex patterns into multiple simpler operations

**Backreference Handling:**
- Numbered groups ($1, $2, etc.) correspond to parentheses in pattern
- $0 or $& represents the entire match
- Named groups can be referenced using $+{name} syntax
- Escape literal dollar signs with $$ in replacement strings

**Memory Considerations:**
- Large string replacements create new string objects
- Process very large texts in chunks when possible
- Consider streaming approaches for file-based processing
- Monitor memory usage with extensive pattern matching operations

**Unicode and Encoding:**
- Patterns work with Unicode characters in AutoHotkey v2
- Be aware of normalization differences in Unicode text
- Test patterns with international character sets
- Consider locale-specific character classes for comprehensive matching

**Error Handling:**
- Invalid regex patterns throw exceptions at runtime
- Validate patterns before use in production code
- Handle edge cases where text might not match expected format
- Provide fallback mechanisms for critical text processing operations

## Related AHK Concepts

- [RegExMatch](./regexmatch.md) - Pattern matching without replacement
- [StrReplace](../../30_Built_In_Classes/00-Core_Classes/String/strreplace.md) - Simple string replacement
- [InStr](../../30_Built_In_Classes/00-Core_Classes/String/instr.md) - Basic string searching
- [Format](../../30_Built_In_Classes/00-Core_Classes/String/format.md) - String formatting and templating
- [FileRead](../../30_Built_In_Classes/02-File_IO_Classes/File/fileread.md) - Reading text files for processing

## Tags

#AutoHotkey #RegExReplace #RegularExpressions #TextProcessing #StringManipulation #PatternReplacement #DataTransformation #ContentProcessing