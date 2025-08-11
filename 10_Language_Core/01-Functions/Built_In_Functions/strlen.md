# Function: StrLen

## Category

Built-in Function

## Overview

StrLen returns the number of characters in a string, providing essential string length measurement functionality. This function is fundamental for string validation, loop boundaries, and text processing operations in AutoHotkey v2.

## Syntax

```cpp
Length := StrLen(String)
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| String | String | Yes | The string whose length you want to determine |

## Return Value

| Type | Description |
|------|-------------|
| Integer | The number of characters in the string, including spaces and special characters but excluding the null terminator |

## Exceptions

| Exception Type | When Thrown |
|----------------|-------------|
| TypeError | When the parameter is not a string or cannot be converted to a string |

## Basic Examples

```cpp
; Simple string length
text := "Hello World"
length := StrLen(text)  ; Returns 11

; Empty string
empty := ""
emptyLength := StrLen(empty)  ; Returns 0

; String with special characters
special := "Hello\nWorld\t!"
specialLength := StrLen(special)  ; Returns 13 (includes \n and \t)
```

## Advanced Examples

```cpp
; String validation function
ValidateInput(userInput) {
    local minLength := 3
    local maxLength := 50
    
    try {
        local inputLength := StrLen(userInput)
        
        if (inputLength < minLength) {
            throw ValueError("Input too short. Minimum " . minLength . " characters required.")
        }
        
        if (inputLength > maxLength) {
            throw ValueError("Input too long. Maximum " . maxLength . " characters allowed.")
        }
        
        return true
        
    } catch TypeError as err {
        throw TypeError("Invalid input type: " . err.Message)
    }
}

; Usage with error handling
try {
    userText := "Example input"
    if (ValidateInput(userText)) {
        ProcessValidInput(userText)
    }
} catch Error as err {
    MsgBox("Validation failed: " . err.Message)
}
```

```cpp
; Text processing with length checks
class TextProcessor {
    static ProcessMessages(messages) {
        local processed := Array()
        local totalLength := 0
        
        for index, message in messages {
            local messageLength := StrLen(message)
            
            ; Skip empty messages
            if (messageLength == 0) {
                continue
            }
            
            ; Truncate long messages
            if (messageLength > 100) {
                message := SubStr(message, 1, 97) . "..."
                messageLength := StrLen(message)  ; Recalculate after truncation
            }
            
            processed.Push({
                content: message,
                length: messageLength,
                index: index
            })
            
            totalLength += messageLength
        }
        
        return {
            messages: processed,
            totalLength: totalLength,
            averageLength: processed.Length ? totalLength / processed.Length : 0
        }
    }
}
```

## Common Use Cases

- **Input Validation**: Checking if user input meets length requirements
- **Text Formatting**: Determining spacing and alignment needs
- **Data Processing**: Analyzing text data for statistics and patterns
- **Loop Control**: Using string length to control iteration boundaries
- **Memory Planning**: Estimating memory requirements for string operations

## Performance Notes

- StrLen is highly optimized and has O(1) time complexity for most strings
- No memory allocation is performed; the function accesses cached length information
- Negligible performance impact even with very large strings
- Consider caching the result if the same string length is needed multiple times in performance-critical code

## Common Pitfalls

### Pitfall 1: Confusion with Byte Length
**Problem**: Expecting StrLen to return byte length rather than character count
**Solution**: StrLen returns character count; use StrPut for byte length calculations
```cpp
; Character count (what StrLen returns)
text := "Hello 世界"
charCount := StrLen(text)  ; Returns 8 characters

; For byte length in specific encoding
byteLength := StrPut(text, "UTF-8") - 1  ; UTF-8 byte count (minus null terminator)
```

### Pitfall 2: Null Character Handling
**Problem**: Assuming strings with embedded null characters behave normally
**Solution**: Be aware that AutoHotkey strings are null-terminated internally
```cpp
; String with embedded null (rare but possible with DllCall)
; AutoHotkey will treat everything after the first null as not part of the string
textWithNull := "Hello" . Chr(0) . "World"
length := StrLen(textWithNull)  ; Returns 5, not 11

; To work with binary data containing nulls, use buffer operations instead
```

## Version History

- **v2.0**: Function introduced with Unicode support and improved performance
- **v2.0-a**: Early alpha versions had different performance characteristics
- **v2.1**: Additional optimizations for very long strings

## Related Functions

- [SubStr](../substr.md) - Extract portions of strings based on length calculations
- [StrReplace](../strreplace.md) - String replacement operations often use length for validation
- [Format](../format.md) - String formatting may require length calculations for padding
- [InStr](../instr.md) - String searching that works with length-based boundaries

## Related Concepts

- [String Data Type](../../../00_Fundamentals/02-Data_Types/string-type.md) - Understanding how strings work in AutoHotkey
- [Unicode Support](../../../40_Advanced_Features/05-System_Integration/unicode.md) - How StrLen handles different character encodings
- [Input Validation Patterns](../../../50_Ecosystem/00-Design_Patterns/validation-patterns.md) - Common uses of StrLen in validation

## See Also

- [String Class Documentation](../../../30_Built_In_Classes/00-Core_Classes/String/string.md)
- [Text Processing Best Practices](../../../50_Ecosystem/01-Best_Practices/text-processing.md)
- [Performance Optimization for Strings](../../../50_Ecosystem/02-Performance_Optimization/string-optimization.md)

## Tags

#AutoHotkey #Function #String #Length #Validation #TextProcessing #BuiltIn #Unicode