# Topic: Format Function - String Formatting and Interpolation

## Category

Function

## Overview

The Format function provides powerful string formatting and interpolation capabilities in AutoHotkey v2. It supports printf-style format specifiers, numeric formatting, padding, alignment, and precision controls, making it essential for creating well-formatted output, reports, and user interfaces.

## Key Points

- Supports printf-style format specifiers for precise string construction
- Handles numeric formatting with padding, precision, and alignment options
- Provides automatic type conversion and validation for safety
- Supports positional arguments for flexible parameter ordering
- Essential for creating formatted output, reports, and data presentation

## Syntax and Parameters

```cpp
Format(FormatStr, Values*)

; FormatStr - Format string with placeholders and specifiers
; Values*   - Variadic arguments to insert into format string

; Format Specifiers:
; {:d}      - Decimal integer
; {:f}      - Floating point
; {:s}      - String
; {:x}      - Hexadecimal (lowercase)
; {:X}      - Hexadecimal (uppercase)
; {:o}      - Octal
; {:e/:E}   - Scientific notation
; {:g/:G}   - General numeric format
; {:c}      - Character from numeric code
; {:p}      - Pointer (hexadecimal with 0x prefix)
```

## Code Examples

```cpp
; Basic string interpolation
name := "Alice"
age := 25
result := Format("Hello {}, you are {} years old!", name, age)
; Output: "Hello Alice, you are 25 years old!"

; Numeric formatting with precision
pi := 3.14159265359
formatted := Format("Pi to 2 decimal places: {:.2f}", pi)
; Output: "Pi to 2 decimal places: 3.14"

; Integer formatting with padding
number := 42
result := Format("Padded number: {:08d}", number)
; Output: "Padded number: 00000042"

; Hexadecimal representation
value := 255
lower := Format("Lowercase hex: {:x}", value)    ; "lowercase hex: ff"
upper := Format("Uppercase hex: {:X}", value)    ; "Uppercase hex: FF"

; Alignment and width control
text := "Center"
left := Format("{:.<20s}", text)      ; "Center.............."
right := Format("{:.>20s}", text)     ; "..............Center"
center := Format("{:.^20s}", text)    ; ".......Center......."

; Scientific notation
large := 1234567.89
scientific := Format("Scientific: {:.2e}", large)
; Output: "Scientific: 1.23e+06"

; Multiple format specifiers
price := 29.99
quantity := 3
total := price * quantity
invoice := Format("Item: ${:.2f} x {:d} = ${:.2f}", price, quantity, total)
; Output: "Item: $29.99 x 3 = $89.97"

; Positional arguments (1-indexed)
template := "The {2:s} is {1:d} years old"
result := Format(template, 5, "cat")
; Output: "The cat is 5 years old"

; Character codes
ascii := 65
char := Format("ASCII {:d} is '{:c}'", ascii, ascii)
; Output: "ASCII 65 is 'A'"

; Complex formatting example
function GenerateReport(sales) {
    header := Format("{:^50s}", "SALES REPORT")
    separator := Format("{:-^50s}", "")
    
    report := header . "`n" . separator . "`n"
    
    for index, sale in sales {
        line := Format("Item {:2d}: {:.<30s} ${:>8.2f}", 
                      index, sale.name, sale.amount)
        report .= line . "`n"
    }
    
    return report
}
```

## Implementation Notes

**Format Specifier Details:**
- Width specifier: `{:10s}` sets minimum field width to 10 characters
- Precision specifier: `{:.2f}` sets decimal places for floating point
- Alignment: `<` (left), `>` (right), `^` (center)
- Fill characters: `{:0>8d}` fills with zeros, `{:.>10s}` fills with periods

**Type Conversion:**
- Automatic conversion between compatible types (string ↔ number)
- String formatting preserves original value type when possible
- Invalid conversions throw TypeError exceptions

**Performance Considerations:**
- More efficient than string concatenation for complex formatting
- Format string parsing happens at runtime - avoid in tight loops
- Consider caching format strings for repeated operations

**Common Pitfalls:**
- Missing closing braces cause formatting errors
- Incorrect argument count doesn't always throw errors (may show placeholders)
- Precision specifiers only work with appropriate numeric formats

**Internationalization Notes:**
- Decimal separator follows system locale settings
- Number formatting respects regional settings
- Consider locale-specific formatting for international applications

## Related AHK Concepts

- [String Concatenation](../string-concatenation.md) - Alternative string building methods
- [StrReplace](./strreplace.md) - Pattern-based string replacement
- [RegExReplace](../../10_Language_Core/01-Functions/Built_In_Functions/regexreplace.md) - Advanced pattern replacement
- [ToString](./tostring.md) - Object to string conversion
- [Locale Functions](../../10_Language_Core/01-Functions/Built_In_Functions/locale.md) - Regional formatting settings

## Tags

#AutoHotkey #String #Formatting #Printf #Interpolation #Numeric #Alignment #Precision