# AutoHotkey v2 OOP Debugger Agent Prompt

Help debug AHK v2 OOP issue: $ARGUMENTS

## Debugging Steps:

### 1. Understand the problem:
   * Parse AHK v2 error messages and stack traces
   * **Error Types**: Analyze error categories
      * Type errors (wrong object types, invalid method calls)
      * Property errors (undefined properties, getter/setter issues)
      * Method errors (missing methods, incorrect signatures)
      * Inheritance errors (super calls, constructor chains)
   * Locate exact line number and file from error output
   * Check OutputDebug statements and ListVars output
   * Review A_LastError for system-level issues
   * Trace through recent script modifications

### 2. Analyze the code path:
   * Trace object instantiation and lifecycle
   * Map inheritance hierarchy and method resolution order
   * Identify property definitions and access patterns
   * Check constructor parameters and initialization
   * Review static vs instance members
   * Verify proper use of `this` and `super`
   * Examine event handler bindings and callbacks

### 3. Suggest debugging strategies:
**AHK v2 Specific:**
   * Use #Warn directives for early detection:
      * `#Warn All` - Enable all warnings
      * `#Warn VarUnset` - Catch uninitialized variables
      * `#Warn UseUnset` - Detect use of unset variables
   * Strategic OutputDebug placement:
      ```ahk
      OutputDebug("Object Type: " Type(obj))
      OutputDebug("Properties: " . ObjOwnPropCount(obj))
      OutputDebug("HasMethod: " . HasMethod(obj, "MethodName"))
      ```
   * ListVars for runtime inspection
   * Try-Catch blocks with detailed error info:
      ```ahk
      try {
          ; Suspicious code
      } catch Error as e {
          OutputDebug("Error: " e.Message "`nFile: " e.File "`nLine: " e.Line)
          OutputDebug("Stack: " e.Stack)
      }
      ```
   * Property debugging with descriptors:
      ```ahk
      for prop in obj.OwnProps() {
          desc := obj.GetOwnPropDesc(prop)
          OutputDebug(prop ": " Type(desc))
      }
      ```
   * Visual Studio Code debugging with zero-plusplus.vscode-autohotkey-debug
   * SciTE4AutoHotkey debugging features

### 4. Identify common OOP pitfalls:
**Class & Object Issues:**
   * Incorrect constructor syntax (missing __New or wrong parameters)
   * Property initialization timing problems
   * Missing property definitions causing runtime errors
   * Circular references in object properties
   * Memory leaks from event handler references

**Inheritance Problems:**
   * Super() call placement and parameters
   * Method override signature mismatches
   * Property shadowing in derived classes
   * Static member inheritance confusion
   * Base class constructor not called

**Property & Method Issues:**
   * Getter/Setter infinite recursion
   * Property descriptor conflicts
   * Method binding context loss (this reference)
   * Fat arrow vs regular function confusion
   * Missing return statements in getters

**Type & Reference Errors:**
   * Null/undefined object access
   * Type coercion problems
   * Reference vs value semantics confusion
   * Weak reference cleanup issues
   * Object comparison mistakes (= vs ==)

**Event & Callback Problems:**
   * Event handler memory leaks
   * Incorrect callback binding (lost this context)
   * Async timing issues with GUI events
   * Multiple event registration
   * Missing event cleanup in __Delete

**GUI OOP Issues:**
   * Control event binding to class methods
   * GUI object lifecycle management
   * Parent-child relationship errors
   * Custom control inheritance problems
   * Event propagation issues

### 5. Create minimal reproduction:
   * Isolate the problematic class/object
   * Remove unrelated methods and properties
   * Create simple test harness:
      ```ahk
      ; Test isolated class
      class TestClass {
          __New() {
              ; Minimal constructor
          }
          
          TestMethod() {
              ; Isolated problem method
          }
      }
      
      ; Reproduce issue
      obj := TestClass()
      obj.TestMethod()
      ```
   * Document exact steps and inputs that trigger the error
   * Include AHK version and system info

### 6. Propose solutions:
   * Provide specific code fixes with proper v2 syntax:
      ```ahk
      ; Before (problematic)
      class MyClass {
          prop := unset  ; Wrong initialization
      }
      
      ; After (fixed)
      class MyClass {
          prop := ""  ; Proper initialization
          
          __New() {
              this.prop := "initialized"
          }
      }
      ```
   
   * **Defensive programming techniques:**
      * Validate object types: `if (obj is MyClass)`
      * Check property existence: `if (obj.HasOwnProp("prop"))`
      * Verify method availability: `if (HasMethod(obj, "method"))`
      * Safe navigation with try-catch
      * Default parameter values in methods
   
   * **Error handling patterns:**
      ```ahk
      class SafeClass {
          SafeMethod(param := "") {
              if (!param)
                  throw ValueError("Parameter required", -1)
              
              try {
                  return this._ProcessParam(param)
              } catch Error as e {
                  this._LogError(e)
                  return this._GetDefault()
              }
          }
      }
      ```
   
   * **Add comprehensive validation:**
      * Type checking in setters
      * Parameter validation in methods
      * State verification before operations
      * Invariant checks in critical methods
   
   * **Implement proper cleanup:**
      ```ahk
      __Delete() {
          ; Remove event handlers
          if (this.HasOwnProp("gui"))
              this.gui.Destroy()
          
          ; Clear circular references
          this.parent := ""
          
          ; Release COM objects
          if (this.HasOwnProp("comObj"))
              this.comObj := ""
      }
      ```

## Output format:

### Root Cause Analysis
* Specific AHK v2 OOP issue identified
* Why the error occurs in the object model
* Impact on script execution

### Step-by-step Debugging Plan
1. Enable #Warn directives
2. Add strategic OutputDebug statements
3. Implement error catching
4. Test with isolated examples
5. Verify fix with edge cases

### Specific Code Changes Needed
* Exact line modifications with before/after
* Proper v2 syntax corrections
* OOP pattern improvements

### Test Cases to Verify Fix
```ahk
; Test case 1: Normal operation
TestNormalOperation() {
    ; Test code
}

; Test case 2: Edge case
TestEdgeCase() {
    ; Test code
}

; Test case 3: Error handling
TestErrorHandling() {
    ; Test code
}
```

### Prevention Strategies
* **Code review checklist:**
   * All classes have proper __New methods
   * Properties are initialized correctly
   * Event handlers are properly cleaned up
   * No circular references
   * Proper use of static vs instance members

* **Development practices:**
   * Use #Warn All during development
   * Implement unit tests for classes
   * Document expected object interfaces
   * Use consistent naming conventions
   * Regular code reviews focusing on OOP patterns

* **Monitoring & logging:**
   * Implement debug mode with verbose logging
   * Track object creation/destruction
   * Monitor memory usage for leaks
   * Log all caught exceptions

* **Documentation updates:**
   * Document class interfaces and contracts
   * Provide usage examples for each class
   * Note any special initialization requirements
   * List known limitations and workarounds