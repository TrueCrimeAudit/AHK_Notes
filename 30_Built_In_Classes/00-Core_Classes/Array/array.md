# Class: Array

## Category

Built-in Class

## Overview

Array is AutoHotkey v2's dynamic array implementation that provides indexed access to elements with automatic resizing. It serves as the foundation for ordered collections and supports a wide range of operations for managing sequential data.

## Inheritance Hierarchy

```
Object
└── Array
```

## Constructor

```cpp
arr := Array([item1, item2, item3, ...])
arr := Array(Length)
```

### Constructor Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| items | Any | No | Initial items to populate the array |
| Length | Integer | No | Pre-allocate array with specified length |

## Properties

### Instance Properties

| Property | Type | Access | Description |
|----------|------|--------|-------------|
| Length | Integer | Read/Write | Number of elements in the array |
| Capacity | Integer | ReadOnly | Current allocated capacity |

### Static Properties

| Property | Type | Access | Description |
|----------|------|--------|-------------|
| Prototype | Object | ReadOnly | Array prototype object for extending functionality |

## Methods

### Instance Methods

#### Push
```cpp
count := arr.Push([item1, item2, ...])
```
Appends one or more items to the end of the array and returns the new length.

#### Pop
```cpp
item := arr.Pop()
```
Removes and returns the last element from the array.

#### InsertAt
```cpp
arr.InsertAt(pos, value1 [, value2, ...])
```
Inserts one or more values at the specified position.

#### RemoveAt
```cpp
removedItem := arr.RemoveAt(pos [, length])
```
Removes one or more elements starting at the specified position.

#### Has
```cpp
exists := arr.Has(index)
```
Returns true if the array has an element at the specified index.

#### Clone
```cpp
newArray := arr.Clone()
```
Creates a shallow copy of the array.

### Static Methods

#### FromString
```cpp
arr := Array.FromString(str, separator)
```
Creates an array by splitting a string using the specified separator.

## Events

Arrays don't have built-in events, but can be extended to support event-driven patterns.

## Basic Usage Example

```cpp
; Create an empty array
numbers := Array()

; Add elements
numbers.Push(1, 2, 3)
numbers.InsertAt(2, 1.5)  ; Insert at position 2

; Access elements
firstNumber := numbers[1]        ; Gets 1
secondNumber := numbers[2]       ; Gets 1.5

; Get array information
length := numbers.Length         ; Gets 4
hasIndex := numbers.Has(3)       ; Gets true

; Remove elements
removed := numbers.RemoveAt(2)   ; Removes 1.5
lastItem := numbers.Pop()        ; Removes and returns last element
```

## Advanced Usage Examples

```cpp
; Advanced array operations with error handling
class NumberProcessor {
    data := Array()
    
    __New(initialData*) {
        ; Accept variable number of arguments
        this.data := Array(initialData*)
    }
    
    ProcessData() {
        try {
            ; Safe array operations with validation
            if (this.data.Length == 0) {
                throw ValueError("No data to process")
            }
            
            ; Clone array for safe processing
            workingData := this.data.Clone()
            
            ; Process each element
            for index, value in workingData {
                if (!IsNumber(value)) {
                    throw TypeError("All elements must be numbers")
                }
                workingData[index] := value * 2
            }
            
            return workingData
            
        } catch Error as err {
            ; Handle errors gracefully
            OutputDebug("Processing failed: " . err.Message)
            return Array()
        }
    }
    
    GetStatistics() {
        local sum := 0, min := "", max := ""
        
        ; Safe iteration with bounds checking
        for index, value in this.data {
            sum += value
            
            if (min == "" || value < min) {
                min := value
            }
            if (max == "" || value > max) {
                max := value
            }
        }
        
        return {
            sum: sum,
            average: this.data.Length ? sum / this.data.Length : 0,
            min: min,
            max: max,
            count: this.data.Length
        }
    }
}
```

## Real-World Example

```cpp
; Task queue management using Array
class TaskQueue {
    tasks := Array()
    completed := Array()
    
    AddTask(taskName, priority := 5) {
        local task := {
            name: taskName,
            priority: priority,
            created: A_Now,
            status: "pending"
        }
        
        ; Insert based on priority (higher priority first)
        local insertIndex := this.FindInsertPosition(priority)
        this.tasks.InsertAt(insertIndex, task)
        
        return task
    }
    
    FindInsertPosition(priority) {
        ; Find correct position for priority-based insertion
        for index, task in this.tasks {
            if (task.priority < priority) {
                return index
            }
        }
        return this.tasks.Length + 1
    }
    
    ProcessNext() {
        if (this.tasks.Length == 0) {
            return false
        }
        
        ; Get highest priority task (first in array)
        local task := this.tasks.RemoveAt(1)
        task.status := "completed"
        task.completed := A_Now
        
        ; Move to completed array
        this.completed.Push(task)
        
        return task
    }
    
    GetPendingTasks() {
        ; Return clone to prevent external modification
        return this.tasks.Clone()
    }
    
    GetTaskHistory() {
        ; Return statistics about completed tasks
        local stats := {
            total: this.completed.Length,
            byPriority: Map()
        }
        
        for index, task in this.completed {
            local priority := task.priority
            if (!stats.byPriority.Has(priority)) {
                stats.byPriority[priority] := 0
            }
            stats.byPriority[priority]++
        }
        
        return stats
    }
}
```

## Implementation Patterns

### Common Patterns
- **Stack Operations**: Use `Push()` and `Pop()` for LIFO (Last In, First Out) behavior
- **Queue Operations**: Use `Push()` and `RemoveAt(1)` for FIFO (First In, First Out) behavior
- **Safe Iteration**: Always check `Length` before accessing elements by index
- **Bulk Operations**: Use spread operator `*` for adding multiple elements

### Anti-Patterns
- **Direct Length Modification**: Don't manually set `Length` to shrink arrays; use `RemoveAt()` instead
- **Dense Index Assumptions**: Don't assume all indices from 1 to Length exist
- **Memory Waste**: Avoid creating unnecessarily large arrays with pre-allocation

## Performance Considerations

- **Memory Growth**: Arrays automatically resize, doubling capacity when needed
- **Access Time**: O(1) for indexed access, O(n) for search operations
- **Insertion/Deletion**: O(1) at end, O(n) at beginning or middle due to element shifting
- **Memory Usage**: Approximately 16 bytes per element plus array overhead

## Best Practices

1. **Pre-allocate When Size is Known**: Use `Array(knownSize)` for better performance
2. **Use Clone() for Safe Copies**: Prevent unintended modifications to shared arrays
3. **Validate Indices**: Always check bounds before accessing elements
4. **Prefer Push/Pop**: Use stack operations when order doesn't matter for better performance
5. **Handle Empty Arrays**: Always check `Length` before operations that require elements

## Common Pitfalls

### Pitfall 1: Modifying Array During Iteration
**Problem**: Changing array size while iterating can cause skipped elements or errors
**Solution**: Clone the array first or iterate backwards for deletions
```cpp
; Incorrect - modifying during iteration
for index, value in myArray {
    if (value < 0) {
        myArray.RemoveAt(index)  ; This shifts remaining elements!
    }
}

; Correct - iterate backwards for safe removal
for index in Range(myArray.Length, 1, -1) {
    if (myArray[index] < 0) {
        myArray.RemoveAt(index)
    }
}

; Or clone first for complex modifications
workingArray := myArray.Clone()
for index, value in workingArray {
    if (value < 0) {
        ; Safe to modify original
        myArray.RemoveAt(myArray.IndexOf(value))
    }
}
```

### Pitfall 2: Assuming 1-Based Indexing Everywhere
**Problem**: Some operations might use 0-based indexing in certain contexts
**Solution**: Always read documentation and test edge cases
```cpp
; Arrays are 1-based in AutoHotkey v2
arr := Array("a", "b", "c")
firstElement := arr[1]  ; Correct: gets "a"
; arr[0] would be undefined

; But be careful with functions that might expect 0-based
; Always check the specific function documentation
```

## Version History

- **v2.0**: Initial implementation with full array functionality
- **v2.0-a**: Early alpha versions had different method signatures
- **v2.1**: Performance improvements for large arrays

## Related Classes

- [Object](../00-Core_Classes/Object/object.md) - Base class providing fundamental object functionality
- [Map](../00-Core_Classes/Map/map.md) - Key-value pairs for associative collections
- [String](../00-Core_Classes/String/string.md) - String manipulation often works with character arrays

## Related Concepts

- [Iteration and Loops](../../../10_Language_Core/00-Control_Flow/iteration-loops.md) - How to efficiently iterate through arrays
- [Function Parameters](../../../10_Language_Core/01-Functions/parameters-and-arguments.md) - Using arrays as function arguments
- [Object-Oriented Patterns](../../../50_Ecosystem/00-Design_Patterns/collection-patterns.md) - Design patterns for working with collections

## See Also

- [Array Performance Optimization](../../../50_Ecosystem/02-Performance_Optimization/array-optimization.md)
- [Collection Design Patterns](../../../50_Ecosystem/00-Design_Patterns/collection-patterns.md)
- [Data Structure Selection Guide](../../../Index/data-structure-guide.md)

## Tags

#AutoHotkey #Class #Array #Collection #DataStructure #BuiltIn #Indexing #Dynamic