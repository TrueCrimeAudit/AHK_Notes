# Exploring AHK v2: How It Compares to Other Programming Languages

I thought it’d be helpful to explore how AHK v2 mirrors features from popular languages like JavaScript, Python, and others. Whether you’re coming from another language or just curious about AHK’s design, this post should give you some useful insights. 

## Object-Oriented Programming (OOP)

AHK v2 supports **prototype-based programming**, which is similar to JavaScript. If you’re used to classical OOP (like in Python), this might feel a little different, but it’s super powerful once you get the hang of it.

AHK v2:

```cpp
class MyClass {
    static Config := "default"
    __New(param) {
        this.value := param
    }
    Method() => "result"
}
```

JavaScript:

```javascript
class MyClass {
    static config = "default";
    constructor(param) {
        this.value = param;
    }
    method() { return "result"; }
}
```

## First-Class Functions

One of my favorite features in AHK v2 is how it treats functions as one of the most mobile and reusable items. This means you can pass functions around as arguments, return them from other functions, or assign them to variables—just like in JavaScript.

AHK v2:

```cpp
callback := (x) => x * 2
myFunc(callback)
```

JavaScript:

```javascript
const callback = x => x * 2;
myFunc(callback);
```

AHK v2 also supports different ways to define functions, like named functions, anonymous functions, and arrow functions. 

For example:

#### Named Function:

AHK v2:
```cpp
callback(x) {
    return x * 2
}
myFunc(callback)
```

Python:
```Python
def callback(x):
    return x * 2
myFunc(callback)
```
Javascript:
```Javascript
function callback(x) {
    return x * 2;
}
myFunc(callback);
```

C#:
```cs
int Callback(int x) {
    return x * 2;
}
myFunc(Callback);
```

#### Anonymous Function:

AHK v2:
```cpp
foo := (x) {
    return x * 2
}
```
Rust:
```Rust
let foo = |x| x * 2;
```

Kotlin:
```Kotlin
val blah = { x: Int -> x * 2 }
```

#### Arrow Function:

AHK v2:
```cpp
foo := (x) => x * 2
```

Python:
```python
foo = lambda x: x * 2
```

Typescript:
```Typescript
const foo = (x: number) => x * 2;
```

Rust:
```rust
let foo = |x| x * 2;
```

Go:
```Go
blah := func(x int) int { return x * 2 }
```

This flexibility makes AHK v2 feel very modern and expressive like Python & Javascript, obviously. This flexibility is much like the next tier of popular languages like Kotlin, Rust, and Go. 

## Array Methods

If you’re familiar with JavaScript arrays, you’ll notice the simularity with AHK v2’s arrays. They come with handy methods like `Push` and properties like `Length`.

AHK v2:
```cpp
array := [1, 2, 3]
array.Push(4)
array.Length
```

JavaScript:
```javascript
let array = [1, 2, 3];
array.push(4);
array.length;
```
## Map Objects

AHK v2’s `Map` objects are similar to Python dictionaries or JavaScript Maps. They’re great for storing key-value pairs. 

AHK v2:

```cpp
map := Map("key", "value")
map["key"] := "newValue"
```

Python:

```python
dict = {"key": "value"}
dict["key"] = "newValue"
```

Javascript:

```javascript
let map = new Map();
map.set("key", "value");
map.set("key", "newValue");
```

## String Concatenation

AHK v2 uses **implicit string concatenation**, which is a bit different from string interpolation in languages like JavaScript. It’s a small distinction, but it’s good to be aware of.

AHK v2:

```cpp
name := "World"
msg := "Hello " name
```

JavaScript:

```javascript
let name = "World";
let msg = `Hello ${name}`;
```

While they achieve the same result, the way they work under the hood is different. Here's how some other languages do it:

| Language   | Concatenation Syntax                          | Interpolation Syntax         |
|------------|-----------------------------------------------|---------------------------------------------|
| **AHK v2** | `msg := "Hello " name`                        | N/A (implicit concatenation)                |
| **Python** | `msg = "Hello " + name`                       | `msg = f"Hello {name}"`                     |
| **JavaScript** | `msg = "Hello " + name`                   | ``msg = `Hello ${name}` ``                  |
| **Java**   | `msg = "Hello " + name`                       | N/A (use `String.format` or `StringBuilder`)|
| **C#**     | `msg = "Hello " + name`                       | `msg = $"Hello {name}"`                     |{name}"`                     |
| **PHP**    | `msg = "Hello " . $name`                      | `msg = "Hello $name"`                       |
| **Go**     | `msg = "Hello " + name`                       | `msg = fmt.Sprintf("Hello %s", name)`       |
| **Rust**   | `msg = "Hello ".to_string() + &name`          | `msg = format!("Hello {}", name)`           |(name)"`                     |
| **Kotlin** | `msg = "Hello " + name`                       | `msg = "Hello $name"`                       |

## Closures

AHK v2 supports **implicit closures**, which means variables from the outer scope are automatically enclosed within a function. This is similar to JavaScript.

AHK v2:

```cpp
addNumFactory(num1) {
    return (num2) => num1 + num2
}
addFive := addNumFactory(5)
result := addFive(10) ; Output: 15
```

JavaScript:

```javascript
function addNumFactory(num1) {
    return (num2) => num1 + num2;
}
let addFive = addNumFactory(5);
let result = addFive(10); // Output: 15
```

Closures are super useful for creating functions that “remember” their environment.

## Inspiration

AHK v2 draws inspiration from a variety of languages, which is part of what makes it so versatile:

- **C/C++**: Features like `#Directives` (e.g., `#Include`) and the `%%` syntax for variable dereferencing come from C/C++ and DOS/Batch.
- **JavaScript**: The prototype-based OOP and first-class functions are very JavaScript-like.
- **VBScript**: The way AHK v2 handles function calls without parentheses is reminiscent of VBScript.
- **PHP**: The global functions in AHK’s standard library feel similar to PHP’s approach.

## What Makes AHK v2 Unique

While AHK v2 borrows from other languages, it also has some unique features that set it apart:

- **1-Indexing**: Like Lua, AHK uses 1-based indexing for arrays.
- **Reference Counting**: AHK uses reference counting for garbage collection, similar to Python and PHP.
- **RegExMatch/RegExReplace**: These functions have roots in Unix `ed` and Perl, making AHK a great tool for text processing.
- **AHK2EXE**: The ability to compile scripts into executables is one of AHK’s standout features. It’s incredibly user-friendly and something I wish more scripting languages had!

## Final Thoughts

AHK v2 is a fantastic language that combines the best of many worlds. Whether you’re coming from JavaScript, Python, or even Kotlin, you’ll find a lot of familiar concepts in AHK v2. At the same time, it has its own unique flavor of simplicity. 

**Shoutout**: Big thanks to [g.ahk](https://github.com/G33kDude), [Descolada](https://github.com/Descolada), and [Panaku](https://github.com/The-CoDingman) and everyone else who contributed to the original discussion.
