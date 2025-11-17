# SystemScript Syntax Reference

## Lexical Structure

### Comments

#### Single-line Comments

```systemscript
// This is a single-line comment
let x = 5;  // Comment after code
```

#### Multi-line Comments

```systemscript
/*
 * This is a multi-line comment
 * spanning several lines
 */
```

#### Nested Comments

```systemscript
/* Outer comment /* inner comment */ still in outer */
```

### Identifiers

Valid identifier rules:

- Start with letter or underscore
- Contain letters, digits, underscores
- Case-sensitive

```systemscript
let valid_identifier = 0;
let ValidIdentifier = 1;
let _underscore_start = 2;
let name123 = 3;
```

Reserved keywords cannot be identifiers.

### Keywords

#### Declaration Keywords

```
module import export
fn struct enum union type
let const static
pub priv
```

#### Control Flow Keywords

```
if else switch case default
while do for loop
break continue return
goto
```

#### Type Keywords

```
i8 i16 i32 i64
u8 u16 u32 u64
f32 f64
bool char void
str
```

#### Operator Keywords

```
as
sizeof
typeof
```

#### Safety Keywords

```
unsafe
defer
```

#### Async Keywords

```
async await yield
```

#### Other Keywords

```
true false
null
in
```

### Literals

#### Integer Literals

```systemscript
let decimal = 42;
let hex = 0x2A;
let octal = 0o52;
let binary = 0b101010;
let with_underscores = 1_000_000;
```

Type suffixes:

```systemscript
let i32_value = 42i32;
let u64_value = 100u64;
let i8_value = -5i8;
```

#### Floating-Point Literals

```systemscript
let float = 3.14;
let scientific = 1.5e10;
let with_suffix = 2.5f32;
```

#### Boolean Literals

```systemscript
let t = true;
let f = false;
```

#### Character Literals

```systemscript
let ch = 'A';
let newline = '\n';
let tab = '\t';
let backslash = '\\';
let quote = '\'';
let unicode = '\u{1F600}';
```

#### String Literals

```systemscript
let simple = "Hello, world";
let with_escapes = "Line 1\nLine 2";
let raw = r"No escapes: \n stays as \n";
let multiline = """
    Multiple lines
    of text
""";
```

#### Null Literal

```systemscript
let ptr: *i32 = null;
```

### Operators

#### Arithmetic Operators

```systemscript
+   // Addition
-   // Subtraction
*   // Multiplication
/   // Division
%   // Remainder
```

#### Bitwise Operators

```systemscript
&   // Bitwise AND
|   // Bitwise OR
^   // Bitwise XOR
~   // Bitwise NOT
<<  // Left shift
>>  // Right shift
```

#### Comparison Operators

```systemscript
==  // Equal
!=  // Not equal
<   // Less than
<=  // Less than or equal
>   // Greater than
>=  // Greater than or equal
```

#### Logical Operators

```systemscript
&&  // Logical AND
||  // Logical OR
!   // Logical NOT
```

#### Assignment Operators

```systemscript
=   // Assignment
+=  // Add and assign
-=  // Subtract and assign
*=  // Multiply and assign
/=  // Divide and assign
%=  // Remainder and assign
&=  // AND and assign
|=  // OR and assign
^=  // XOR and assign
<<= // Left shift and assign
>>= // Right shift and assign
```

#### Other Operators

```systemscript
&   // Reference
*   // Dereference
.   // Member access
->  // Pointer member access
::  // Scope resolution
..  // Range (exclusive)
... // Range (inclusive)
?   // Error propagation
|>  // Pipeline
```

## Module System

### Module Declaration

```systemscript
module module_name;
```

Nested modules:

```systemscript
module network.protocols.http;
```

### Imports

Basic import:

```systemscript
import module_name;
```

Import specific items:

```systemscript
import module.{item1, item2, item3};
```

Import with alias:

```systemscript
import long_module_name as short;
```

Import all:

```systemscript
import module.*;
```

### Exports

Export function:

```systemscript
pub fn public_function() { }
```

Export type:

```systemscript
pub struct PublicStruct { }
```

Export with restrictions:

```systemscript
pub(crate) fn crate_public() { }
pub(module) fn module_public() { }
```

## Variable Declarations

### Immutable Variables

```systemscript
let x = 10;
let name: str = "value";
```

### Mutable Variables

```systemscript
let mut counter = 0;
counter += 1;
```

### Constants

```systemscript
const MAX_SIZE: u32 = 1024;
const PI: f64 = 3.14159;
```

### Static Variables

```systemscript
static mut GLOBAL_COUNTER: i32 = 0;
```

### Type Inference

```systemscript
let x = 42;           // Inferred as i32
let y = 3.14;         // Inferred as f64
let z = "text";       // Inferred as str
```

## Type Declarations

### Primitive Types

```systemscript
let i: i32 = 0;
let u: u64 = 100;
let f: f32 = 1.5;
let b: bool = true;
let c: char = 'A';
```

### Pointer Types

```systemscript
let ptr: *i32 = &x;
let mut_ptr: *mut i32 = &mut x;
let void_ptr: *void = null;
```

### Reference Types

```systemscript
let ref: &i32 = &x;
let mut_ref: &mut i32 = &mut x;
```

### Array Types

Fixed-size arrays:

```systemscript
let array: [i32; 5] = [1, 2, 3, 4, 5];
let zeroed: [u8; 100] = [0; 100];
```

### Slice Types

```systemscript
let slice: []i32 = array[1..4];
```

### Structure Types

```systemscript
struct Point {
    x: f32,
    y: f32
}

let p = Point { x: 1.0, y: 2.0 };
```

### Enumeration Types

```systemscript
enum Status {
    Ok,
    Error,
    Pending
}

let s = Status::Ok;
```

### Union Types

```systemscript
union Value {
    i: i32,
    f: f32,
    bytes: [u8; 4]
}
```

### Function Types

```systemscript
let func: fn(i32, i32) -> i32 = add;
```

### Type Aliases

```systemscript
type Result<T> = Result<T, Error>;
type Callback = fn(i32) -> void;
```

## Function Declarations

### Basic Function

```systemscript
fn function_name(param1: Type1, param2: Type2) -> ReturnType {
    // function body
    return value;
}
```

### Function Without Return

```systemscript
fn procedure(x: i32) {
    // no return statement needed
}
```

### Generic Functions

```systemscript
fn generic<T>(value: T) -> T {
    return value;
}
```

With constraints:

```systemscript
fn constrained<T: Comparable>(a: T, b: T) -> T {
    return if (a < b) a else b;
}
```

### Variadic Functions

```systemscript
fn variadic(format: str, args: ...any) {
    // process arguments
}
```

### Function Attributes

```systemscript
#[inline(always)]
fn always_inlined() { }

#[no_return]
fn exits_program() { }

#[export_name = "c_function"]
fn exported() { }
```

### Method Syntax

```systemscript
struct Type {
    fn method(self, param: i32) { }
    fn static_method() { }
}
```

## Control Flow Statements

### If Statements

```systemscript
if (condition) {
    // code
}

if (condition) {
    // code
} else {
    // code
}

if (condition1) {
    // code
} else if (condition2) {
    // code
} else {
    // code
}
```

If expression:

```systemscript
let value = if (condition) 1 else 2;
```

### Switch Statements

```systemscript
switch (value) {
    case 1:
        // code
        break;
    case 2:
    case 3:
        // code for 2 or 3
        break;
    case 4...10:
        // range match
        break;
    default:
        // default case
}
```

### Match Expressions

```systemscript
match value {
    1 => action1(),
    2 | 3 => action2(),
    4...10 => action3(),
    _ => default_action()
}
```

Pattern matching:

```systemscript
match object {
    Type1 { field } => use_field(field),
    Type2 => other_action(),
    _ => ()
}
```

### While Loops

```systemscript
while (condition) {
    // code
}
```

### Do-While Loops

```systemscript
do {
    // code
} while (condition);
```

### For Loops

C-style:

```systemscript
for (let i = 0; i < 10; i++) {
    // code
}
```

Iterator-style:

```systemscript
for (item in collection) {
    // use item
}
```

Range:

```systemscript
for (i in 0..10) {
    // i from 0 to 9
}

for (i in 0...10) {
    // i from 0 to 10
}
```

### Loop Statement

Infinite loop:

```systemscript
loop {
    // code
    if (condition) break;
}
```

### Break and Continue

```systemscript
while (condition) {
    if (skip_condition) continue;
    if (exit_condition) break;
}
```

Named breaks:

```systemscript
outer: for (i in 0..10) {
    inner: for (j in 0..10) {
        if (condition) break outer;
    }
}
```

### Return Statement

```systemscript
return;              // Return from void function
return value;        // Return with value
return (a, b, c);    // Return multiple values
```

### Defer Statement

```systemscript
fn example() {
    let resource = acquire();
    defer release(resource);
    // resource will be released on function exit
}
```

## Expressions

### Literal Expressions

```systemscript
42
3.14
"text"
true
```

### Variable Expressions

```systemscript
x
some_variable
```

### Binary Expressions

```systemscript
a + b
x * y
value == 10
```

### Unary Expressions

```systemscript
-x
!flag
~bits
*ptr
&value
```

### Function Call Expressions

```systemscript
function()
function(arg1, arg2)
object.method(arg)
```

### Array Index Expressions

```systemscript
array[0]
array[i]
array[1..5]
```

### Member Access Expressions

```systemscript
object.field
pointer->field
module::function
```

### Cast Expressions

```systemscript
value as i32
ptr as *u8
```

### Sizeof Expressions

```systemscript
sizeof(i32)
sizeof(Type)
```

### Typeof Expressions

```systemscript
typeof(variable)
```

### Lambda Expressions

```systemscript
|x| x * 2
|x, y| x + y
|| { /* code */ }
```

### Array Literal Expressions

```systemscript
[1, 2, 3, 4, 5]
[0; 100]  // 100 zeros
```

### Struct Literal Expressions

```systemscript
Point { x: 1.0, y: 2.0 }
Person { name: "John", ..defaults }
```

## Unsafe Blocks

```systemscript
unsafe {
    // unchecked operations allowed
    let ptr = 0x1000 as *u32;
    *ptr = 42;
}
```

## Async/Await Syntax

```systemscript
async fn async_function() -> i32 {
    let result = await other_async();
    return result;
}
```

## Attributes

### Function Attributes

```systemscript
#[inline(always)]
#[inline(never)]
#[no_mangle]
#[export_name = "name"]
#[calling_convention(cdecl)]
#[no_return]
```

### Type Attributes

```systemscript
#[repr(C)]
#[repr(packed)]
#[repr(align(16))]
```

### Conditional Compilation

```systemscript
#[cfg(target_os = "linux")]
#[cfg(target_arch = "x86_64")]
#[cfg(feature = "advanced")]
```

### Compiler Directives

```systemscript
#[allow(unused)]
#[warn(deprecated)]
#[deny(unsafe_code)]
```

## Macros

### Simple Macros

```systemscript
macro_name!()
macro_name!(args)
```

### Declarative Macros

```systemscript
macro_rules! name {
    (pattern) => { expansion };
}
```

## Comments and Documentation

### Documentation Comments

```systemscript
/// Documentation for following item
fn documented_function() { }

/** 
 * Multi-line documentation
 * for following item
 */
struct DocumentedStruct { }
```

### Module Documentation

```systemscript
//! Module-level documentation
```
