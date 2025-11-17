# SystemScript Instruction Set Reference

This document describes the low-level instruction set available in SystemScript for direct hardware manipulation and systems programming.

## Memory Instructions

### Load Operations

#### Basic Load

```systemscript
let value = *ptr;  // Load from pointer
```

Loads data from memory address specified by pointer.

**Parameters:**
- `ptr`: Pointer to memory location

**Returns:** Value at memory address

**Safety:** Unchecked, requires unsafe block

#### Typed Load

```systemscript
let value = memory.load<T>(address);
```

Load typed value from memory address.

**Parameters:**
- `T`: Type to load
- `address`: Memory address as integer

**Returns:** Typed value

**Example:**
```systemscript
unsafe {
    let value = memory.load<u32>(0x1000);
}
```

#### Atomic Load

```systemscript
let value = memory.load_atomic<T>(ptr, ordering);
```

Load value with specified memory ordering.

**Parameters:**
- `T`: Type to load
- `ptr`: Pointer to memory
- `ordering`: Memory ordering (RELAXED, ACQUIRE, SEQ_CST)

**Returns:** Value with atomic guarantees

### Store Operations

#### Basic Store

```systemscript
*ptr = value;  // Store to pointer
```

Store data to memory address.

**Parameters:**
- `ptr`: Destination pointer
- `value`: Value to store

**Safety:** Unchecked, requires unsafe block

#### Typed Store

```systemscript
memory.store<T>(address, value);
```

Store typed value to memory address.

**Parameters:**
- `T`: Type to store
- `address`: Memory address
- `value`: Value to store

**Example:**
```systemscript
unsafe {
    memory.store<u32>(0x1000, 0x12345678);
}
```

#### Atomic Store

```systemscript
memory.store_atomic<T>(ptr, value, ordering);
```

Store value with memory ordering.

**Parameters:**
- `T`: Type to store
- `ptr`: Destination pointer
- `value`: Value to store
- `ordering`: Memory ordering

### Memory Operations

#### Set

```systemscript
memory.set(dest, value, count);
```

Fill memory region with value.

**Parameters:**
- `dest`: Destination pointer
- `value`: Byte value to set
- `count`: Number of bytes

**Example:**
```systemscript
unsafe {
    memory.set(buffer, 0, 1024);  // Zero 1024 bytes
}
```

#### Copy

```systemscript
memory.copy(dest, source, count);
```

Copy memory region.

**Parameters:**
- `dest`: Destination pointer
- `source`: Source pointer
- `count`: Number of bytes

**Safety:** Overlapping regions undefined

#### Move

```systemscript
memory.move(dest, source, count);
```

Copy memory with overlap support.

**Parameters:**
- `dest`: Destination pointer
- `source`: Source pointer
- `count`: Number of bytes

#### Compare

```systemscript
let equal = memory.compare(ptr1, ptr2, count);
```

Compare memory regions.

**Parameters:**
- `ptr1`: First pointer
- `ptr2`: Second pointer
- `count`: Number of bytes

**Returns:** Boolean indicating equality

## Arithmetic Instructions

### Integer Arithmetic

#### Add

```systemscript
let result = a + b;
```

Add two integers with overflow wrapping.

**Variants:**
- `a.wrapping_add(b)`: Explicit wrapping
- `a.saturating_add(b)`: Saturate at type bounds
- `a.checked_add(b)`: Return Option, None on overflow

#### Subtract

```systemscript
let result = a - b;
```

Subtract with overflow wrapping.

**Variants:**
- `a.wrapping_sub(b)`: Explicit wrapping
- `a.saturating_sub(b)`: Saturate at bounds
- `a.checked_sub(b)`: Return Option

#### Multiply

```systemscript
let result = a * b;
```

Multiply with overflow wrapping.

**Variants:**
- `a.wrapping_mul(b)`: Explicit wrapping
- `a.saturating_mul(b)`: Saturate
- `a.checked_mul(b)`: Return Option

#### Divide

```systemscript
let result = a / b;
```

Integer division (truncated).

**Safety:** Division by zero causes panic

**Variants:**
- `a.checked_div(b)`: Return Option (None if b == 0)

#### Remainder

```systemscript
let result = a % b;
```

Compute remainder.

**Safety:** Division by zero causes panic

### Bitwise Operations

#### AND

```systemscript
let result = a & b;
```

Bitwise AND operation.

#### OR

```systemscript
let result = a | b;
```

Bitwise OR operation.

#### XOR

```systemscript
let result = a ^ b;
```

Bitwise XOR operation.

#### NOT

```systemscript
let result = ~a;
```

Bitwise NOT (one's complement).

#### Shift Left

```systemscript
let result = a << n;
```

Shift bits left by n positions.

**Note:** Undefined if n >= bit width

#### Shift Right

```systemscript
let result = a >> n;
```

Arithmetic shift right (signed) or logical shift right (unsigned).

**Note:** Undefined if n >= bit width

#### Rotate Left

```systemscript
let result = a.rotate_left(n);
```

Rotate bits left.

#### Rotate Right

```systemscript
let result = a.rotate_right(n);
```

Rotate bits right.

## Atomic Instructions

### Compare and Swap

```systemscript
let (old, success) = memory.compare_exchange<T>(
    ptr, 
    expected, 
    new_value,
    success_ordering,
    failure_ordering
);
```

Atomic compare-and-swap operation.

**Parameters:**
- `ptr`: Pointer to atomic variable
- `expected`: Expected current value
- `new_value`: Value to set if match
- `success_ordering`: Ordering if successful
- `failure_ordering`: Ordering if failed

**Returns:** Tuple of (previous value, success boolean)

### Fetch and Add

```systemscript
let old = memory.fetch_add<T>(ptr, value, ordering);
```

Atomically add value and return previous.

**Parameters:**
- `ptr`: Pointer to atomic variable
- `value`: Value to add
- `ordering`: Memory ordering

**Returns:** Previous value

### Fetch and Sub

```systemscript
let old = memory.fetch_sub<T>(ptr, value, ordering);
```

Atomically subtract and return previous.

### Fetch and AND

```systemscript
let old = memory.fetch_and<T>(ptr, value, ordering);
```

Atomically AND and return previous.

### Fetch and OR

```systemscript
let old = memory.fetch_or<T>(ptr, value, ordering);
```

Atomically OR and return previous.

### Fetch and XOR

```systemscript
let old = memory.fetch_xor<T>(ptr, value, ordering);
```

Atomically XOR and return previous.

### Exchange

```systemscript
let old = memory.exchange<T>(ptr, new_value, ordering);
```

Atomically swap values.

**Parameters:**
- `ptr`: Pointer to atomic variable
- `new_value`: New value to set
- `ordering`: Memory ordering

**Returns:** Previous value

## Control Flow Instructions

### Unconditional Jump

Implemented through language constructs:

```systemscript
goto label;  // Not recommended

// Preferred alternatives:
break;
continue;
return;
```

### Conditional Branch

```systemscript
if (condition) {
    // taken branch
} else {
    // not taken branch
}
```

Compiles to conditional branch instruction.

### Function Call

```systemscript
let result = function(arg1, arg2);
```

Call function with arguments.

**Calling conventions:**
- Default: platform-specific
- `#[calling_convention(cdecl)]`
- `#[calling_convention(stdcall)]`
- `#[calling_convention(fastcall)]`

### Return

```systemscript
return value;
```

Return from function with value.

## Hardware Access Instructions

### Port I/O

#### Port Read

```systemscript
let value = port.read<T>(port_number);
```

Read from I/O port.

**Parameters:**
- `T`: Type to read (u8, u16, u32)
- `port_number`: Port address

**Returns:** Value read from port

**Safety:** Requires hardware access permissions

**Example:**
```systemscript
unsafe {
    let data = port.read<u8>(0x3F8);  // Read from serial port
}
```

#### Port Write

```systemscript
port.write<T>(port_number, value);
```

Write to I/O port.

**Parameters:**
- `T`: Type to write
- `port_number`: Port address
- `value`: Value to write

**Example:**
```systemscript
unsafe {
    port.write<u8>(0x3F8, 0x41);  // Write 'A' to serial port
}
```

### CPU Instructions

#### Read MSR

```systemscript
let value = cpu.read_msr(msr_number);
```

Read Model-Specific Register.

**Parameters:**
- `msr_number`: MSR identifier

**Returns:** 64-bit MSR value

**Safety:** Requires kernel mode

#### Write MSR

```systemscript
cpu.write_msr(msr_number, value);
```

Write to Model-Specific Register.

**Parameters:**
- `msr_number`: MSR identifier
- `value`: 64-bit value to write

#### CPUID

```systemscript
let (eax, ebx, ecx, edx) = cpu.cpuid(leaf, subleaf);
```

Execute CPUID instruction.

**Parameters:**
- `leaf`: CPUID function
- `subleaf`: CPUID subfunction

**Returns:** Four 32-bit register values

#### Read TSC

```systemscript
let timestamp = cpu.read_tsc();
```

Read Time-Stamp Counter.

**Returns:** 64-bit cycle count

#### Halt

```systemscript
cpu.halt();
```

Halt processor until interrupt.

#### Pause

```systemscript
cpu.pause();
```

Hint to processor (used in spin loops).

### Interrupt Control

#### Disable Interrupts

```systemscript
let flags = cpu.disable_interrupts();
```

Disable interrupts and return previous flags.

**Returns:** Previous interrupt state

#### Enable Interrupts

```systemscript
cpu.enable_interrupts();
```

Enable interrupts.

#### Restore Interrupts

```systemscript
cpu.restore_interrupts(flags);
```

Restore interrupt state from saved flags.

**Parameters:**
- `flags`: Previously saved interrupt state

## Memory Barrier Instructions

### Full Barrier

```systemscript
memory.barrier();
```

Full memory barrier (all loads and stores).

### Load Barrier

```systemscript
memory.barrier_load();
```

Prevent reordering of loads.

### Store Barrier

```systemscript
memory.barrier_store();
```

Prevent reordering of stores.

### Acquire Barrier

```systemscript
memory.barrier_acquire();
```

Acquire barrier for synchronization.

### Release Barrier

```systemscript
memory.barrier_release();
```

Release barrier for synchronization.

## SIMD Instructions

### Vector Load

```systemscript
let vector = simd.load_f32x4(ptr);
```

Load 4 floats into vector register.

**Parameters:**
- `ptr`: Pointer to aligned data

**Returns:** Vector value

### Vector Store

```systemscript
simd.store_f32x4(ptr, vector);
```

Store vector to memory.

**Parameters:**
- `ptr`: Pointer to destination
- `vector`: Vector value to store

### Vector Operations

#### Add

```systemscript
let result = simd.add_f32x4(a, b);
```

Add two vectors element-wise.

#### Multiply

```systemscript
let result = simd.mul_f32x4(a, b);
```

Multiply two vectors element-wise.

#### Fused Multiply-Add

```systemscript
let result = simd.fma_f32x4(a, b, c);
```

Compute a * b + c with single rounding.

## Cache Control Instructions

### Flush Cache Line

```systemscript
cache.flush_line(address);
```

Flush cache line containing address.

**Parameters:**
- `address`: Memory address

### Invalidate Cache Line

```systemscript
cache.invalidate_line(address);
```

Invalidate cache line.

### Prefetch

```systemscript
cache.prefetch(address, locality);
```

Prefetch data into cache.

**Parameters:**
- `address`: Memory address
- `locality`: Temporal locality hint (0-3)

## Inline Assembly

For direct assembly instruction insertion:

```systemscript
asm {
    "instruction operands"
    : output constraints
    : input constraints
    : clobbers
}
```

**Example:**
```systemscript
let result: u64;
unsafe {
    asm {
        "rdtsc"
        : "=a"(result)
        :
        : "edx"
    }
}
```

## Memory Ordering

Available memory ordering modes:

- `RELAXED`: No ordering constraints
- `ACQUIRE`: Acquire semantics for loads
- `RELEASE`: Release semantics for stores
- `ACQ_REL`: Both acquire and release
- `SEQ_CST`: Sequentially consistent (strongest)

Usage:

```systemscript
use memory::Ordering;

let value = atomic.load(Ordering::ACQUIRE);
atomic.store(42, Ordering::RELEASE);
```
