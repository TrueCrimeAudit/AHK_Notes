# Topic: RegExMatch Function - Advanced Pattern Matching

## Category

Function

## Overview

The RegExMatch function performs regular expression pattern matching against text, providing powerful and flexible text searching, validation, and extraction capabilities. It's essential for complex text processing, data validation, parsing structured text, and implementing sophisticated search functionality.

## Key Points

- Performs pattern matching using full regular expression syntax for complex text operations
- Returns match position and optionally captures matched groups for data extraction
- Supports advanced regex features including lookahead, lookbehind, and named groups
- Provides case-sensitive and case-insensitive matching options with modifier flags
- Essential for data validation, text parsing, log analysis, and content extraction tasks

## Syntax and Parameters

```cpp
Pos := RegExMatch(Haystack, NeedleRegEx [, &OutputVar, StartingPos])

; Parameters:
; Haystack      - String to search within
; NeedleRegEx   - Regular expression pattern with optional flags
; OutputVar     - Variable to receive match details (optional, by reference)
; StartingPos   - 1-based position to start search (optional, default: 1)

; Pattern flags (prefix pattern with "(?flags)"):
; i - Case insensitive matching
; m - Multiline mode (^ and $ match line boundaries)
; s - Single line mode (. matches newlines)
; x - Extended syntax (ignore whitespace in pattern)
; A - Anchored matching (match only at start of string)
; D - Dollar end only ($ matches only at end of string)
; J - Allow duplicate named groups
; U - Ungreedy matching (quantifiers are non-greedy by default)

; Return Value:
; Returns 1-based position of first match, 0 if no match found
; OutputVar receives MatchObject with match details and captured groups
```

## Code Examples

```cpp
; Basic pattern matching
text := "The price is $29.99 for this item"
if (RegExMatch(text, "\$\d+\.\d+")) {
    MsgBox("Price found in text")
}

; Case-insensitive matching with capture groups
email := "Contact us at Support@Example.COM"
if (RegExMatch(email, "(?i)([a-z0-9._%+-]+)@([a-z0-9.-]+\.[a-z]{2,})", &match)) {
    MsgBox("Username: " . match[1] . "`nDomain: " . match[2])
}

; Named capture groups for structured data extraction
logLine := "2024-01-15 14:30:25 [ERROR] Connection timeout"
pattern := "(?P<date>\d{4}-\d{2}-\d{2}) (?P<time>\d{2}:\d{2}:\d{2}) \[(?P<level>\w+)\] (?P<message>.*)"

if (RegExMatch(logLine, pattern, &match)) {
    MsgBox("Date: " . match["date"] . "`nLevel: " . match["level"] . "`nMessage: " . match["message"])
}

; Multiple matches with loop processing
function ExtractAllEmails(text) {
    emails := []
    pattern := "(?i)\b[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}\b"
    startPos := 1
    
    while (pos := RegExMatch(text, pattern, &match, startPos)) {
        emails.Push(match[0])  ; Full match
        startPos := pos + match.Len[0]  ; Move past current match
    }
    
    return emails
}

; Data validation with comprehensive patterns
class DataValidator {
    static patterns := Map(
        "email", "(?i)^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$",
        "phone", "^(\+\d{1,3}[- ]?)?\d{10}$",
        "ssn", "^\d{3}-\d{2}-\d{4}$",
        "zipcode", "^\d{5}(-\d{4})?$",
        "ipv4", "^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$",
        "url", "(?i)^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$",
        "creditcard", "^(?:\d{4}[- ]?){3}\d{4}$",
        "password", "^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$"
    )
    
    static Validate(type, value) {
        if (!this.patterns.Has(type)) {
            throw ValueError("Unknown validation type: " . type)
        }
        
        pattern := this.patterns[type]
        return RegExMatch(value, pattern) > 0
    }
    
    static ValidateWithDetails(type, value) {
        if (!this.patterns.Has(type)) {
            throw ValueError("Unknown validation type: " . type)
        }
        
        pattern := this.patterns[type]
        result := RegExMatch(value, pattern, &match)
        
        return {
            isValid: result > 0,
            match: result > 0 ? match : "",
            position: result
        }
    }
    
    static GetSuggestion(type, value) {
        switch type {
            case "email":
                if (!InStr(value, "@")) return "Email must contain @ symbol"
                if (!RegExMatch(value, "\.")) return "Email must contain domain extension"
                return "Check email format: user@domain.com"
                
            case "phone":
                return "Phone format: (123) 456-7890 or 123-456-7890"
                
            case "password":
                requirements := []
                if (!RegExMatch(value, "[a-z]")) requirements.Push("lowercase letter")
                if (!RegExMatch(value, "[A-Z]")) requirements.Push("uppercase letter")
                if (!RegExMatch(value, "\d")) requirements.Push("number")
                if (!RegExMatch(value, "[@$!%*?&]")) requirements.Push("special character")
                if (StrLen(value) < 8) requirements.Push("minimum 8 characters")
                
                return "Password must contain: " . Join(requirements, ", ")
                
            default:
                return "Please check the format and try again"
        }
    }
}

; Advanced text parsing for structured documents
class LogParser {
    static ParseApacheLog(logLine) {
        ; Combined Log Format pattern
        pattern := '^(\S+) \S+ \S+ \[([^\]]+)\] "(\S+) (\S+) (\S+)" (\d+) (\d+|-) "([^"]*)" "([^"]*)"'
        
        if (RegExMatch(logLine, pattern, &match)) {
            return {
                ip: match[1],
                timestamp: match[2],
                method: match[3],
                url: match[4],
                protocol: match[5],
                status: Integer(match[6]),
                size: match[7] = "-" ? 0 : Integer(match[7]),
                referer: match[8],
                userAgent: match[9]
            }
        }
        
        return false
    }
    
    static ParseCSV(line, delimiter := ",") {
        fields := []
        pattern := '(?:^|' . delimiter . ')("(?:[^"]|"")*"|[^' . delimiter . ']*)'
        startPos := 1
        
        while (pos := RegExMatch(line, pattern, &match, startPos)) {
            field := match[1]
            
            ; Remove quotes and handle escaped quotes
            if (SubStr(field, 1, 1) = '"' && SubStr(field, -1) = '"') {
                field := SubStr(field, 2, -1)
                field := StrReplace(field, '""', '"')
            }
            
            fields.Push(field)
            startPos := pos + match.Len[0]
        }
        
        return fields
    }
    
    static ExtractCodeBlocks(markdown) {
        blocks := []
        pattern := "(?ms)^```(\w+)?\s*\n(.*?)\n```"
        startPos := 1
        
        while (pos := RegExMatch(markdown, pattern, &match, startPos)) {
            blocks.Push({
                language: match[1],
                code: match[2],
                position: pos
            })
            startPos := pos + match.Len[0]
        }
        
        return blocks
    }
}

; Content extraction and transformation
function ExtractLinks(html) {
    links := []
    pattern := '(?i)<a\s+(?:[^>]*\s+)?href\s*=\s*["\']([^"\']+)["\'][^>]*>(.*?)</a>'
    startPos := 1
    
    while (pos := RegExMatch(html, pattern, &match, startPos)) {
        ; Extract text content (remove HTML tags)
        text := RegExReplace(match[2], "<[^>]*>", "")
        
        links.Push({
            url: match[1],
            text: Trim(text),
            fullMatch: match[0]
        })
        
        startPos := pos + match.Len[0]
    }
    
    return links
}

; Search and replace with pattern groups
function SmartReplace(text, searchPattern, replaceTemplate) {
    ; Replace with support for captured groups
    ; Example: SmartReplace("2024-01-15", "(\d{4})-(\d{2})-(\d{2})", "$2/$3/$1")
    ; Result: "01/15/2024"
    
    result := text
    startPos := 1
    offset := 0
    
    while (pos := RegExMatch(text, searchPattern, &match, startPos)) {
        ; Build replacement text
        replacement := replaceTemplate
        
        ; Replace group references ($1, $2, etc.)
        for i in Range(0, match.Count) {
            groupRef := "$" . i
            if (InStr(replacement, groupRef)) {
                groupValue := i = 0 ? match[0] : (i <= match.Count ? match[i] : "")
                replacement := StrReplace(replacement, groupRef, groupValue)
            }
        }
        
        ; Calculate position in result string
        resultPos := pos + offset
        
        ; Perform replacement
        before := SubStr(result, 1, resultPos - 1)
        after := SubStr(result, resultPos + match.Len[0])
        result := before . replacement . after
        
        ; Update offset for next iteration
        offset += StrLen(replacement) - match.Len[0]
        startPos := pos + match.Len[0]
    }
    
    return result
}

; Usage examples
; Email validation
if (DataValidator.Validate("email", "user@example.com")) {
    MsgBox("Valid email address")
}

; Extract all phone numbers from text
phonePattern := "(?:\+\d{1,3}[- ]?)?\(?\d{3}\)?[- ]?\d{3}[- ]?\d{4}"
phones := ExtractAllEmails("Call 555-123-4567 or (555) 987-6543")

; Parse log file
logEntry := '192.168.1.1 - - [10/Oct/2000:13:55:36 -0700] "GET /apache_pb.gif HTTP/1.0" 200 2326'
logData := LogParser.ParseApacheLog(logEntry)

; Smart date format conversion
dateText := "Meeting on 2024-01-15 at 2:30 PM"
reformatted := SmartReplace(dateText, "(\d{4})-(\d{2})-(\d{2})", "$2/$3/$1")
; Result: "Meeting on 01/15/2024 at 2:30 PM"
```

## Implementation Notes

**Pattern Performance:**
- Compile complex patterns once and reuse for better performance
- Use anchors (^ and $) when full string matching is required
- Avoid excessive backtracking with greedy quantifiers in complex patterns
- Consider using atomic groups and possessive quantifiers for optimization

**Capture Group Management:**
- Group 0 always contains the full match
- Numbered groups start from 1, corresponding to parentheses in pattern
- Named groups accessible by name for more readable code
- Non-capturing groups (?:...) don't create numbered captures

**Memory and Resource Usage:**
- Large text processing should be done in chunks when possible
- Match objects contain references to original string data
- Clear match variables when processing large volumes of data
- Consider using RegExReplace for simple substitutions instead of match-and-replace loops

**Error Handling:**
- Invalid patterns throw exceptions at runtime
- Check return value (0) for no match before accessing match object
- Validate input strings for null or empty values
- Handle encoding issues with special characters in patterns

**Cross-Platform Considerations:**
- Line ending differences (\r\n vs \n) affect multiline patterns
- Unicode support varies with pattern complexity
- Case sensitivity behavior may differ with international characters
- File path patterns need OS-specific handling

## Related AHK Concepts

- [RegExReplace](./regexreplace.md) - Pattern-based text replacement
- [StrReplace](../../30_Built_In_Classes/00-Core_Classes/String/strreplace.md) - Simple string replacement
- [InStr](../../30_Built_In_Classes/00-Core_Classes/String/instr.md) - Basic string searching
- [Parse](../../10_Language_Core/00-Control_Flow/Loop_Constructs/parse.md) - Simple text parsing
- [Format](../../30_Built_In_Classes/00-Core_Classes/String/format.md) - String formatting and templates

## Tags

#AutoHotkey #RegExMatch #RegularExpressions #PatternMatching #TextProcessing #Validation #Parsing #DataExtraction