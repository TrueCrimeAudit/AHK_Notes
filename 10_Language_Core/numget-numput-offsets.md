# Topic: NumGet/NumPut Byte Offsets and "Magic Numbers"

## Category

Concept

## Overview

NumGet and NumPut functions use byte offsets to access specific memory locations. These offsets are often called "magic numbers" but are actually calculated based on data structure layouts and memory architecture. Understanding how to calculate these offsets is essential for working with binary data, structures, and APIs.

## Key Points

- Offsets are byte positions in memory, not array indices
- Calculate offsets based on data type sizes and structure layout
- Use A_PtrSize for pointer calculations to maintain cross-platform compatibility
- Memory alignment may affect actual structure layout

## Syntax and Parameters

```cpp
; Basic NumGet/NumPut with offset
value := NumGet(ptr, offset, "type")
NumPut("type", value, ptr, offset)

; Alternative syntax (adding offset to pointer)
value := NumGet(ptr + offset, "type")
NumPut("type", value, ptr + offset)
```

## Code Examples

```cpp
; Example 1: Simple array access
; int32_t numbers[] = { 10, 20, 30, 40 }
pNumbers := ; ... pointer to array
first := NumGet(pNumbers, 0, "Int")      ; 10 (offset 0)
second := NumGet(pNumbers, 4, "Int")     ; 20 (offset 4)
third := NumGet(pNumbers, 8, "Int")      ; 30 (offset 8)
fourth := NumGet(pNumbers, 12, "Int")    ; 40 (offset 12)

; Example 2: Structure access
; struct Person { char name[32]; int age; float height; }
ReadPersonStruct(pPerson) {
    name := StrGet(pPerson, 32, "UTF-8")        ; name at offset 0
    age := NumGet(pPerson, 32, "Int")           ; age at offset 32
    height := NumGet(pPerson, 36, "Float")      ; height at offset 36
    return {name: name, age: age, height: height}
}

; Example 3: Mixed pointer and data structure
; struct Node { int data; struct Node* next; int id; }
ReadNodeStruct(pNode) {
    data := NumGet(pNode, 0, "Int")                    ; data at offset 0
    next := NumGet(pNode, 4, "Ptr")                    ; next at offset 4
    id := NumGet(pNode, 4 + A_PtrSize, "Int")         ; id after pointer
    return {data: data, next: next, id: id}
}

; Example 4: Helper function for offset calculation
CalculateOffset(fieldIndex, fieldSizes*) {
    offset := 0
    loop fieldIndex {
        offset += fieldSizes[A_Index]
    }
    return offset
}

; Usage with field sizes
sizes := [4, A_PtrSize, 4]  ; int, pointer, int
dataOffset := CalculateOffset(1, sizes*)      ; 0
nextOffset := CalculateOffset(2, sizes*)      ; 4
idOffset := CalculateOffset(3, sizes*)        ; 4 + A_PtrSize

; Example 5: COM vtable navigation
; Each method is a pointer in the vtable
GetCOMMethod(object_ptr, method_index) {
    vtable_ptr := NumGet(object_ptr, 0, "Ptr")
    method_offset := method_index * A_PtrSize
    return NumGet(vtable_ptr, method_offset, "Ptr")
}
```

## Implementation Notes

- **Data Type Sizes**: int=4, int64=8, float=4, double=8, char=1, ptr=A_PtrSize
- **Memory Alignment**: Compilers may add padding between structure members
- **Platform Differences**: Pointer sizes vary between 32-bit and 64-bit systems
- **String Handling**: Use StrGet/StrPut for string data in structures
- **Endianness**: AHK follows system endianness (little-endian on x86/x64)
- **Bounds Checking**: No automatic bounds checking - ensure offsets are valid

## Related AHK Concepts

- Pointer arithmetic and memory access
- A_PtrSize for cross-platform compatibility
- StrGet/StrPut for string operations
- DllCall for Windows API integration
- Binary data handling and structures

## Tags

#AutoHotkey #Memory #NumGet #NumPut #BinaryData #Structures #Offsets #DataTypes