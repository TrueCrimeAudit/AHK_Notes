# Topic: Pointer Arithmetic in AutoHotkey

## Category

Concept

## Overview

AutoHotkey treats all pointers as `byte*` pointers, allowing arithmetic operations to access memory at specific byte offsets. This is fundamental for working with binary data structures, COM interfaces, and Windows APIs where you need to read/write data at precise memory locations.

## Key Points

- AHK treats all pointers uniformly as byte pointers, regardless of the actual data type
- Use NumGet/NumPut with calculated byte offsets to access structured data
- A_PtrSize determines the size of pointer types (4 bytes on 32-bit, 8 bytes on 64-bit)
- Pointer arithmetic is essential for COM object method access and Windows API interaction

## Syntax and Parameters

```cpp
; Basic pointer arithmetic
value := NumGet(pData, offset, "type")
NumPut("type", value, pData, offset)

; Accessing array elements
element := NumGet(pData, index * sizeof_element, "type")

; COM interface method access
method_ptr := NumGet(NumGet(this+0) + method_index * A_PtrSize)
```

## Code Examples

```cpp
; Example 1: Accessing int32 array elements
; For array: int32_t data[] = { 1, 2, 3 }
pData := ; ... pointer to array
value1 := NumGet(pData, 0, "Int")      ; Gets value 1 (offset 0)
value2 := NumGet(pData, 4, "Int")      ; Gets value 2 (offset 4)
value3 := NumGet(pData, 8, "Int")      ; Gets value 3 (offset 8)

; Example 2: COM interface method access
; ISimpleAudioVolume interface method access
VA_ISimpleAudioVolume_SetMasterVolume(this, fLevel, GuidEventContext="") {
    ; Calculate method pointer: vtable + (method_index * pointer_size)
    method_ptr := NumGet(NumGet(this+0) + 3 * A_PtrSize)
    return DllCall(method_ptr, "ptr", this, "float", fLevel, "ptr", VA_GUID(GuidEventContext))
}

; Example 3: Generic array access function
GetArrayElement(pArray, index, elementSize, dataType) {
    return NumGet(pArray, index * elementSize, dataType)
}
```

## Implementation Notes

- **Byte Offset Calculation**: Always calculate offsets in bytes, not in elements
- **Data Type Sizes**: int32_t = 4 bytes, int64_t = 8 bytes, pointer = A_PtrSize bytes
- **Platform Differences**: A_PtrSize is 4 on 32-bit systems, 8 on 64-bit systems
- **Memory Safety**: No bounds checking - ensure offsets are within allocated memory
- **COM Interface Access**: First NumGet gets vtable pointer, second NumGet gets method pointer

## Related AHK Concepts

- NumGet/NumPut functions for memory access
- A_PtrSize for platform-specific pointer sizes
- COM object interaction and vtable navigation
- DllCall for Windows API integration
- Binary data structures and memory layout

## Tags

#AutoHotkey #Memory #Pointers #COM #Windows #API #Binary #DataStructures