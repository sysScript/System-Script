# SystemScript Language Overview

SystemScript is a multi-paradigm systems programming language designed for complete hardware and software control. The language provides direct hardware manipulation capabilities alongside high-level abstractions, enabling development across the entire computing stack.

SYstem Scripts compiler can be [found here](https://github.com/sysScript/Windows-Compiler)

## Language Characteristics

### Design Philosophy

SystemScript bridges the gap between low-level system programming and high-level application development through:

- C-like syntax with extended capabilities
- Recursively enumerable parsing system
- Optional type safety with explicit unsafe operations
- Zero-cost abstractions
- Direct hardware access primitives

### Target Use Cases

- Operating system kernel development
- Device driver implementation
- Network protocol development
- Security research and vulnerability analysis
- Web application development
- Embedded systems programming
- High-performance computing

## Core Features

### Type System

SystemScript implements a strong, static type system with:

- Primitive types (integers, floats, booleans, characters)
- Derived types (pointers, references, arrays, slices, strings)
- User-defined types (structures, enumerations, unions)
- Type inference where unambiguous
- Generic type parameters

### Memory Management

Multiple memory management strategies available:

- Manual allocation and deallocation
- Reference counting
- Region-based allocation
- Stack allocation
- Direct memory mapping for hardware access

### Concurrency Model

Built-in concurrency primitives:

- Native thread support
- Atomic operations
- Synchronization primitives (mutexes, read-write locks)
- Async/await syntax
- Coroutines and generators

### Safety Mechanisms

Safety enforced through:

- Default bounds checking
- Null pointer validation
- Explicit unsafe blocks for unchecked operations
- Memory protection attributes
- Compile-time verification where possible

## Platform Support

### Operating Systems

- Linux (all distributions)
- Windows (7 and later)
- macOS (10.12 and later)
- FreeBSD
- OpenBSD
- Bare metal (no OS)

### Architectures

- x86 (32-bit)
- x86_64 (64-bit)
- ARM (32-bit)
- ARM64 (AArch64)
- RISC-V (32-bit and 64-bit)
- WebAssembly

## Toolchain Components

### Compiler (ssc)

Multi-stage compilation pipeline:

1. Lexical analysis
2. Parsing to AST
3. Semantic analysis
4. Optimization passes
5. Code generation

### Linker (ssl)

Object file combining and executable generation with support for:

- Static linking
- Dynamic linking
- Custom entry points
- Kernel-mode linking

### Debugger (ssd)

Source-level debugging with:

- Breakpoint management
- Variable inspection
- Memory examination
- Register viewing
- Disassembly support

### Build System (ssb)

Project management tool providing:

- Dependency resolution
- Incremental compilation
- Test execution
- Package creation

## Language Comparison

### SystemScript vs C

Advantages over C:

- Modern syntax features (generics, pattern matching)
- Built-in concurrency primitives
- Optional memory safety
- Higher-level abstractions without performance cost

Similarities to C:

- Direct memory access
- Hardware control capabilities
- Manual memory management option
- System call interface

### SystemScript vs Rust

Similarities to Rust:

- Memory safety focus
- Zero-cost abstractions
- Strong type system

Differences from Rust:

- Less restrictive borrow checker
- More flexible unsafe operations
- Direct hardware access without FFI
- Simpler syntax for systems programming

## Getting Started

Basic program structure:

```systemscript
module program_name;

import system.io;

fn main() -> i32 {
    io.println("Program output");
    return 0;
}
```

Compilation:

```bash
ssc -o program source.ss
./program
```

## Documentation Structure

This documentation set includes:

- Language Reference: Complete syntax and semantics
- Standard Library: Available modules and functions
- Instruction Set: Low-level operations
- System Programming Guide: Kernel and driver development
- Network Programming Guide: Protocol implementation
- Security Guide: Safe systems programming practices
- Toolchain Reference: Compiler, linker, and debugger usage
- Examples: Complete working programs
