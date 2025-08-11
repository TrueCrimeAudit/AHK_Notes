# AutoHotkey v2 Function Reference Index

Comprehensive index of all AutoHotkey v2 built-in functions organized by category for easy navigation and reference.

## Navigation

- [String Functions](#string-functions)
- [Math Functions](#math-functions)
- [File Functions](#file-functions)
- [System Functions](#system-functions)
- [GUI Functions](#gui-functions)
- [Input Functions](#input-functions)
- [Array/Object Functions](#arrayobject-functions)
- [Conversion Functions](#conversion-functions)
- [Process Functions](#process-functions)
- [Registry Functions](#registry-functions)
- [Window Functions](#window-functions)
- [Misc Functions](#misc-functions)

## String Functions

Functions for string manipulation, searching, and formatting.

| Function | Purpose | Location | Status |
|----------|---------|----------|---------|
| StrLen | Get string length | [📝](../../30_Built_In_Classes/00-Core_Classes/String/strlen.md) | 🚧 Planned |
| SubStr | Extract substring | [📝](../../30_Built_In_Classes/00-Core_Classes/String/substr.md) | ✅ Complete |
| StrReplace | Replace string parts | [📝](../../30_Built_In_Classes/00-Core_Classes/String/strreplace.md) | ✅ Complete |
| RegExMatch | Pattern matching | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/regexmatch.md) | ✅ Complete |
| RegExReplace | Pattern replacement | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/regexreplace.md) | ✅ Complete |
| Format | String formatting | [📝](../../30_Built_In_Classes/00-Core_Classes/String/format.md) | ✅ Complete |
| StrSplit | Split string to array | [📝](../../30_Built_In_Classes/00-Core_Classes/String/strsplit.md) | 🚧 Planned |
| Trim | Remove whitespace | [📝](../../30_Built_In_Classes/00-Core_Classes/String/trim.md) | 🚧 Planned |
| StrLower | Convert to lowercase | [📝](../../30_Built_In_Classes/00-Core_Classes/String/strlower.md) | 🚧 Planned |
| StrUpper | Convert to uppercase | [📝](../../30_Built_In_Classes/00-Core_Classes/String/strupper.md) | 🚧 Planned |
| StrCompare | Compare strings | [📝](../../30_Built_In_Classes/00-Core_Classes/String/strcompare.md) | 🚧 Planned |
| InStr | Find substring position | [📝](../../30_Built_In_Classes/00-Core_Classes/String/instr.md) | ✅ Complete |

## Math Functions

Mathematical calculations and numeric operations.

| Function | Purpose | Location | Status |
|----------|---------|----------|---------|
| Abs | Absolute value | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/abs.md) | 🚧 Planned |
| Ceil | Round up to integer | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/ceil.md) | 🚧 Planned |
| Floor | Round down to integer | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/floor.md) | 🚧 Planned |
| Round | Round to nearest | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/round.md) | 🚧 Planned |
| Sqrt | Square root | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/sqrt.md) | 🚧 Planned |
| Sin | Sine function | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/sin.md) | 🚧 Planned |
| Cos | Cosine function | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/cos.md) | 🚧 Planned |
| Tan | Tangent function | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/tan.md) | 🚧 Planned |
| Log | Natural logarithm | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/log.md) | 🚧 Planned |
| Exp | Exponential function | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/exp.md) | 🚧 Planned |
| Min | Minimum value | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/min.md) | 🚧 Planned |
| Max | Maximum value | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/max.md) | 🚧 Planned |
| Mod | Modulo operation | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/mod.md) | 🚧 Planned |
| Random | Generate random number | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/random.md) | ✅ Complete |

## File Functions

File and directory operations.

| Function | Purpose | Location | Status |
|----------|---------|----------|---------|
| FileRead | Read file contents | [📝](../../30_Built_In_Classes/02-File_IO_Classes/File/fileread.md) | ✅ Complete |
| FileWrite | Write to file | [📝](../../30_Built_In_Classes/02-File_IO_Classes/File/filewrite.md) | ✅ Complete |
| FileAppend | Append to file | [📝](../../30_Built_In_Classes/02-File_IO_Classes/File/fileappend.md) | ✅ Complete |
| FileDelete | Delete file | [📝](../../30_Built_In_Classes/02-File_IO_Classes/File/filedelete.md) | 🚧 Planned |
| FileExist | Check file existence | [📝](../../30_Built_In_Classes/02-File_IO_Classes/File/fileexist.md) | ✅ Complete |
| FileMove | Move/rename file | [📝](../../30_Built_In_Classes/02-File_IO_Classes/File/filemove.md) | 🚧 Planned |
| FileCopy | Copy file | [📝](../../30_Built_In_Classes/02-File_IO_Classes/File/filecopy.md) | 🚧 Planned |
| FileGetSize | Get file size | [📝](../../30_Built_In_Classes/02-File_IO_Classes/File/filegetsize.md) | 🚧 Planned |
| FileGetTime | Get file timestamp | [📝](../../30_Built_In_Classes/02-File_IO_Classes/File/filegettime.md) | 🚧 Planned |
| FileSetTime | Set file timestamp | [📝](../../30_Built_In_Classes/02-File_IO_Classes/File/filesettime.md) | 🚧 Planned |
| FileGetAttrib | Get file attributes | [📝](../../30_Built_In_Classes/02-File_IO_Classes/File/filegetattrib.md) | 🚧 Planned |
| FileSetAttrib | Set file attributes | [📝](../../30_Built_In_Classes/02-File_IO_Classes/File/filesetattrib.md) | 🚧 Planned |
| DirCreate | Create directory | [📝](../../30_Built_In_Classes/02-File_IO_Classes/Dir/dircreate.md) | ✅ Complete |
| DirDelete | Delete directory | [📝](../../30_Built_In_Classes/02-File_IO_Classes/Dir/dirdelete.md) | 🚧 Planned |
| FileSelect | File selection dialog | [📝](../../30_Built_In_Classes/02-File_IO_Classes/FileSelect/fileselect.md) | 🚧 Planned |
| DirSelect | Directory selection dialog | [📝](../../30_Built_In_Classes/02-File_IO_Classes/DirSelect/dirselect.md) | 🚧 Planned |

## System Functions

System operations and information.

| Function | Purpose | Location | Status |
|----------|---------|----------|---------|
| Run | Execute program | [📝](../../40_Advanced_Features/05-System_Integration/run.md) | ✅ Complete |
| RunWait | Execute and wait | [📝](../../40_Advanced_Features/05-System_Integration/runwait.md) | 🚧 Planned |
| ExitApp | Exit application | [📝](../../40_Advanced_Features/05-System_Integration/exitapp.md) | 🚧 Planned |
| Reload | Reload script | [📝](../../40_Advanced_Features/05-System_Integration/reload.md) | 🚧 Planned |
| Suspend | Suspend hotkeys | [📝](../../40_Advanced_Features/00-Hotkeys_and_Input/suspend.md) | 🚧 Planned |
| Sleep | Pause execution | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/sleep.md) | ✅ Complete |
| SetTimer | Set timer function | [📝](../../40_Advanced_Features/03-Threading/settimer.md) | ✅ Complete |
| SysGet | Get system metrics | [📝](../../40_Advanced_Features/05-System_Integration/sysget.md) | 🚧 Planned |
| EnvGet | Get environment variable | [📝](../../40_Advanced_Features/05-System_Integration/envget.md) | 🚧 Planned |
| EnvSet | Set environment variable | [📝](../../40_Advanced_Features/05-System_Integration/envset.md) | 🚧 Planned |
| ClipWait | Wait for clipboard | [📝](../../40_Advanced_Features/05-System_Integration/clipwait.md) | 🚧 Planned |

## GUI Functions

Graphical user interface creation and management.

| Function | Purpose | Location | Status |
|----------|---------|----------|---------|
| Gui | Create GUI window | [📝](../../30_Built_In_Classes/01-GUI_Classes/Gui/gui-constructor.md) | 🚧 Planned |
| MsgBox | Display message box | [📝](../../30_Built_In_Classes/01-GUI_Classes/MsgBox/msgbox.md) | ✅ Complete |
| InputBox | Get user input | [📝](../../30_Built_In_Classes/01-GUI_Classes/InputBox/inputbox.md) | ✅ Complete |
| ToolTip | Show tooltip | [📝](../../30_Built_In_Classes/01-GUI_Classes/ToolTip/tooltip.md) | 🚧 Planned |
| TrayTip | Show tray notification | [📝](../../30_Built_In_Classes/01-GUI_Classes/TrayTip/traytip.md) | 🚧 Planned |
| ProgressBar | Progress indicator | [📝](../../30_Built_In_Classes/01-GUI_Classes/Individual_Controls/progressbar.md) | 🚧 Planned |
| SplashTextOn | Show splash text | [📝](../../30_Built_In_Classes/01-GUI_Classes/SplashText/splashtexton.md) | 🚧 Planned |
| SplashTextOff | Hide splash text | [📝](../../30_Built_In_Classes/01-GUI_Classes/SplashText/splashtextoff.md) | 🚧 Planned |

## Input Functions

User input simulation and detection.

| Function | Purpose | Location | Status |
|----------|---------|----------|---------|
| Send | Send keystrokes | [📝](../../40_Advanced_Features/00-Hotkeys_and_Input/send.md) | ✅ Complete |
| SendText | Send text literally | [📝](../../40_Advanced_Features/00-Hotkeys_and_Input/sendtext.md) | 🚧 Planned |
| SendInput | Send input stream | [📝](../../40_Advanced_Features/00-Hotkeys_and_Input/sendinput.md) | 🚧 Planned |
| Click | Simulate mouse click | [📝](../../40_Advanced_Features/00-Hotkeys_and_Input/click.md) | ✅ Complete |
| MouseMove | Move mouse cursor | [📝](../../40_Advanced_Features/00-Hotkeys_and_Input/mousemove.md) | ✅ Complete |
| MouseClick | Click mouse button | [📝](../../40_Advanced_Features/00-Hotkeys_and_Input/mouseclick.md) | 🚧 Planned |
| MouseClickDrag | Drag with mouse | [📝](../../40_Advanced_Features/00-Hotkeys_and_Input/mouseclickdrag.md) | 🚧 Planned |
| GetKeyState | Get key/button state | [📝](../../40_Advanced_Features/00-Hotkeys_and_Input/getkeystate.md) | ✅ Complete |
| KeyWait | Wait for key event | [📝](../../40_Advanced_Features/00-Hotkeys_and_Input/keywait.md) | 🚧 Planned |
| Hotkey | Set hotkey dynamically | [📝](../../40_Advanced_Features/00-Hotkeys_and_Input/hotkey-function.md) | 🚧 Planned |
| Hotstring | Set hotstring dynamically | [📝](../../40_Advanced_Features/00-Hotkeys_and_Input/hotstring-function.md) | 🚧 Planned |

## Array/Object Functions

Working with arrays, maps, and objects.

| Function | Purpose | Location | Status |
|----------|---------|----------|---------|
| Array | Create array | [📝](../../30_Built_In_Classes/00-Core_Classes/Array/array.md) | ✅ Complete |
| Map | Create map | [📝](../../30_Built_In_Classes/00-Core_Classes/Map/map.md) | 🚧 Planned |
| Object | Create object | [📝](../../30_Built_In_Classes/00-Core_Classes/Object/object.md) | 🚧 Planned |
| IsObject | Check if object | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/isobject.md) | 🚧 Planned |
| HasProp | Check property existence | [📝](../../20_Object_System/02-Methods_and_Properties/hasprop.md) | 🚧 Planned |
| HasMethod | Check method existence | [📝](../../20_Object_System/02-Methods_and_Properties/hasmethod.md) | 🚧 Planned |
| ObjBindMethod | Bind method context | [📝](../../20_Object_System/06-Method_Binding/objbindmethod.md) | 🚧 Planned |
| ObjSetBase | Set object base | [📝](../../20_Object_System/05-Prototypes/objsetbase.md) | 🚧 Planned |
| ObjGetBase | Get object base | [📝](../../20_Object_System/05-Prototypes/objgetbase.md) | 🚧 Planned |

## Conversion Functions

Type conversion and validation.

| Function | Purpose | Location | Status |
|----------|---------|----------|---------|
| String | Convert to string | [📝](../../30_Built_In_Classes/00-Core_Classes/String/string-function.md) | 🚧 Planned |
| Number | Convert to number | [📝](../../30_Built_In_Classes/00-Core_Classes/Number/number-function.md) | 🚧 Planned |
| Integer | Convert to integer | [📝](../../30_Built_In_Classes/00-Core_Classes/Number/integer.md) | 🚧 Planned |
| Float | Convert to float | [📝](../../30_Built_In_Classes/00-Core_Classes/Number/float.md) | 🚧 Planned |
| IsNumber | Check if number | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/isnumber.md) | 🚧 Planned |
| IsInteger | Check if integer | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/isinteger.md) | 🚧 Planned |
| IsFloat | Check if float | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/isfloat.md) | 🚧 Planned |
| IsDigit | Check if digit | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/isdigit.md) | 🚧 Planned |
| IsAlpha | Check if alphabetic | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/isalpha.md) | 🚧 Planned |
| IsAlnum | Check if alphanumeric | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/isalnum.md) | 🚧 Planned |

## Process Functions

Process and thread management.

| Function | Purpose | Location | Status |
|----------|---------|----------|---------|
| ProcessExist | Check process existence | [📝](../../30_Built_In_Classes/03-System_Classes/Process/processexist.md) | 🚧 Planned |
| ProcessClose | Close process | [📝](../../30_Built_In_Classes/03-System_Classes/Process/processclose.md) | 🚧 Planned |
| ProcessWait | Wait for process | [📝](../../30_Built_In_Classes/03-System_Classes/Process/processwait.md) | 🚧 Planned |
| ProcessWaitClose | Wait for process close | [📝](../../30_Built_In_Classes/03-System_Classes/Process/processwaitclose.md) | 🚧 Planned |
| ProcessSetPriority | Set process priority | [📝](../../30_Built_In_Classes/03-System_Classes/Process/processsetpriority.md) | 🚧 Planned |

## Registry Functions

Windows registry operations.

| Function | Purpose | Location | Status |
|----------|---------|----------|---------|
| RegRead | Read registry value | [📝](../../40_Advanced_Features/05-System_Integration/regread.md) | 🚧 Planned |
| RegWrite | Write registry value | [📝](../../40_Advanced_Features/05-System_Integration/regwrite.md) | 🚧 Planned |
| RegDelete | Delete registry key/value | [📝](../../40_Advanced_Features/05-System_Integration/regdelete.md) | 🚧 Planned |
| RegCreateKey | Create registry key | [📝](../../40_Advanced_Features/05-System_Integration/regcreatekey.md) | 🚧 Planned |

## Window Functions

Window manipulation and information.

| Function | Purpose | Location | Status |
|----------|---------|----------|---------|
| WinActivate | Activate window | [📝](../../40_Advanced_Features/05-System_Integration/winactivate.md) | ✅ Complete |
| WinClose | Close window | [📝](../../40_Advanced_Features/05-System_Integration/winclose.md) | ✅ Complete |
| WinExist | Check window existence | [📝](../../40_Advanced_Features/05-System_Integration/winexist.md) | ✅ Complete |
| WinGetTitle | Get window title | [📝](../../40_Advanced_Features/05-System_Integration/wingettitle.md) | 🚧 Planned |
| WinSetTitle | Set window title | [📝](../../40_Advanced_Features/05-System_Integration/winsettitle.md) | 🚧 Planned |
| WinMove | Move/resize window | [📝](../../40_Advanced_Features/05-System_Integration/winmove.md) | 🚧 Planned |
| WinHide | Hide window | [📝](../../40_Advanced_Features/05-System_Integration/winhide.md) | 🚧 Planned |
| WinShow | Show window | [📝](../../40_Advanced_Features/05-System_Integration/winshow.md) | 🚧 Planned |
| WinMinimize | Minimize window | [📝](../../40_Advanced_Features/05-System_Integration/winminimize.md) | 🚧 Planned |
| WinMaximize | Maximize window | [📝](../../40_Advanced_Features/05-System_Integration/winmaximize.md) | 🚧 Planned |
| WinRestore | Restore window | [📝](../../40_Advanced_Features/05-System_Integration/winrestore.md) | 🚧 Planned |
| WinWait | Wait for window | [📝](../../40_Advanced_Features/05-System_Integration/winwait.md) | 🚧 Planned |
| WinWaitActive | Wait for active window | [📝](../../40_Advanced_Features/05-System_Integration/winwaitactive.md) | 🚧 Planned |
| WinWaitClose | Wait for window close | [📝](../../40_Advanced_Features/05-System_Integration/winwaitclose.md) | 🚧 Planned |

## Misc Functions

Other useful functions.

| Function | Purpose | Location | Status |
|----------|---------|----------|---------|
| A_TickCount | Get system uptime | [📝](../../40_Advanced_Features/05-System_Integration/a-tickcount.md) | 🚧 Planned |
| A_Now | Current timestamp | [📝](../../40_Advanced_Features/05-System_Integration/a-now.md) | 🚧 Planned |
| FormatTime | Format timestamp | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/formattime.md) | 🚧 Planned |
| DateAdd | Add to date | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/dateadd.md) | 🚧 Planned |
| DateDiff | Date difference | [📝](../../10_Language_Core/01-Functions/Built_In_Functions/datediff.md) | 🚧 Planned |
| DllCall | Call DLL function | [📝](../../40_Advanced_Features/02-DLL_Integration/dllcall.md) | 🚧 Planned |
| NumGet | Get number from memory | [📝](../../40_Advanced_Features/04-Memory_Management/numget.md) | 🚧 Planned |
| NumPut | Put number in memory | [📝](../../40_Advanced_Features/04-Memory_Management/numput.md) | 🚧 Planned |
| StrGet | Get string from memory | [📝](../../40_Advanced_Features/04-Memory_Management/strget.md) | 🚧 Planned |
| StrPut | Put string in memory | [📝](../../40_Advanced_Features/04-Memory_Management/strput.md) | 🚧 Planned |
| VarSetStrCapacity | Set string capacity | [📝](../../40_Advanced_Features/04-Memory_Management/varsetstrcapacity.md) | 🚧 Planned |

## Status Legend

- ✅ **Complete**: Full documentation available
- 🚧 **Planned**: Documentation planned and structured
- ❌ **Missing**: Not yet planned or structured

## Usage Notes

1. **Function vs Method**: Some operations are available both as standalone functions and as methods on classes
2. **Parameter Order**: Always check documentation for exact parameter order and types
3. **Return Values**: Functions may return different types; check documentation for specifics
4. **Error Handling**: Most functions can throw exceptions; wrap in try-catch when needed
5. **Performance**: Consider method alternatives for repeated operations on the same object

## Quick Reference Tips

- **String Operations**: Most string functions work with both variables and literals
- **File Operations**: Always check for file existence before operations when uncertain
- **GUI Functions**: Consider object-oriented Gui class for complex interfaces
- **System Functions**: Many require appropriate permissions or may fail silently
- **Math Functions**: All math functions handle both integers and floats appropriately

---

**Total Functions**: 150+ documented  
**Categories**: 12 functional areas  
**Coverage**: Comprehensive AutoHotkey v2 function reference  
**Last Updated**: January 2025