# Topic: Random Function - Random Number Generation

## Category

Function

## Overview

The Random function generates pseudo-random numbers within specified ranges in AutoHotkey v2. It provides both integer and floating-point random number generation with configurable minimum and maximum bounds, making it essential for simulations, games, testing, and any application requiring unpredictable values.

## Key Points

- Generates pseudo-random integers and floating-point numbers within specified ranges
- Supports both inclusive and exclusive range boundaries for flexible control
- Uses system-seeded random number generator for reasonable unpredictability
- Provides consistent cross-platform behavior for reliable randomization
- Essential for simulations, games, testing scenarios, and probabilistic algorithms

## Syntax and Parameters

```cpp
Random([Min, Max])

; Parameters:
; Min - Minimum value (inclusive, default: 0)
; Max - Maximum value (inclusive for integers, default: 2147483647)

; Returns:
; Integer between Min and Max (inclusive)
; If no parameters: random integer between 0 and 2147483647
; Floating-point numbers require explicit decimal specification
```

## Code Examples

```cpp
; Basic random integer (0 to 2147483647)
randomNum := Random()

; Random integer in specific range
dice := Random(1, 6)              ; Simulate dice roll (1-6)
percent := Random(0, 100)         ; Percentage (0-100)

; Random floating-point numbers
randomFloat := Random(0.0, 1.0)   ; Float between 0.0 and 1.0
temperature := Random(20.5, 25.7) ; Temperature range

; Negative ranges
offset := Random(-10, 10)         ; Range from -10 to 10
variance := Random(-5.5, 5.5)     ; Floating variance

; Single parameter (0 to Max)
smallRandom := Random(5)          ; 0 to 5 inclusive

; Common use cases
function SimulateCoinFlip() {
    return Random(0, 1) ? "Heads" : "Tails"
}

function GeneratePassword(length) {
    chars := "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789"
    password := ""
    
    Loop length {
        index := Random(1, StrLen(chars))
        password .= SubStr(chars, index, 1)
    }
    
    return password
}

; Array element selection
items := ["apple", "banana", "cherry", "date"]
randomItem := items[Random(1, items.Length)]

; Probability-based decisions
function RandomEvent(probability) {
    ; probability should be between 0.0 and 1.0
    return Random(0.0, 1.0) < probability
}

; Monte Carlo simulation example
function EstimatePi(iterations) {
    insideCircle := 0
    
    Loop iterations {
        x := Random(-1.0, 1.0)
        y := Random(-1.0, 1.0)
        
        if (x*x + y*y <= 1.0) {
            insideCircle++
        }
    }
    
    return 4.0 * insideCircle / iterations
}

; Weighted random selection
function WeightedRandom(weights) {
    total := 0
    for weight in weights
        total += weight
    
    target := Random(0.0, total)
    current := 0
    
    for index, weight in weights {
        current += weight
        if (target <= current)
            return index
    }
}

; Random delay for natural timing
function RandomDelay(minMs, maxMs) {
    delay := Random(minMs, maxMs)
    Sleep(delay)
}
```

## Implementation Notes

**Random Number Quality:**
- Uses system-provided pseudo-random number generator
- Automatically seeded at script startup for reasonable unpredictability
- Not cryptographically secure - use system crypto APIs for security purposes
- Sequence is deterministic if manually seeded with same value

**Range Behavior:**
- Both Min and Max parameters are inclusive for integer generation
- Floating-point ranges determined by parameter types (use decimals for floats)
- Single parameter treated as Max value with Min defaulting to 0
- Negative ranges work correctly in both directions

**Performance Considerations:**
- Very fast generation suitable for real-time applications
- No significant overhead for repeated calls
- Consider pre-generating arrays for massive random data needs
- Function call overhead negligible compared to computation

**Type Handling:**
- Integer parameters produce integer results
- Floating-point parameters produce floating-point results
- Mixed types follow AutoHotkey's standard type promotion rules
- Return type matches the most specific parameter type

**Common Pitfalls:**
- Remember ranges are inclusive (both Min and Max can be returned)
- Floating-point precision limitations apply to decimal ranges
- Large integer ranges may have subtle distribution irregularities
- Don't rely on Random() for security-critical randomness

## Related AHK Concepts

- [Loop](../../10_Language_Core/00-Control_Flow/Loop_Constructs/loop.md) - Iteration for multiple random values
- [Array](../array.md) - Storing and selecting from random collections
- [Math Functions](../../10_Language_Core/01-Functions/Built_In_Functions/math.md) - Mathematical operations on random values
- [Sleep](../../10_Language_Core/01-Functions/Built_In_Functions/sleep.md) - Random timing and delays
- [Crypto Functions](../../40_Advanced_Features/05-System_Integration/crypto.md) - Secure random generation

## Tags

#AutoHotkey #Random #NumberGeneration #Simulation #Probability #Testing #Games #Mathematics