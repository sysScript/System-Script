# SystemScript Standard Library Reference

The SystemScript standard library provides modules for common programming tasks, system interaction, and low-level hardware access.

## Module Organization

### Core Modules

- `system`: Operating system interfaces
- `memory`: Memory management
- `io`: Input/output operations
- `network`: Networking capabilities
- `hardware`: Hardware access
- `collections`: Data structures
- `concurrency`: Threading and synchronization
- `security`: Cryptography and access control

## system Module

### system.io

#### println

```systemscript
fn println(format: str, args: ...any)
```

Print formatted text to stdout with newline.

**Parameters:**
- `format`: Format string with placeholders
- `args`: Values to substitute

**Example:**
```systemscript
system.io.println("Value: {}", 42);
system.io.println("X: {}, Y: {}", x, y);
```

#### print

```systemscript
fn print(format: str, args: ...any)
```

Print formatted text without newline.

#### read_line

```systemscript
fn read_line() -> str
```

Read line from stdin.

**Returns:** String containing input line

#### write

```systemscript
fn write(data: str) -> i64
```

Write string to stdout.

**Returns:** Number of bytes written

### system.process

#### exit

```systemscript
fn exit(code: i32) -> !
```

Terminate process with exit code.

**Parameters:**
- `code`: Exit status code

**Note:** Never returns

#### spawn

```systemscript
fn spawn(program: str, args: []str) -> Result<Process, Error>
```

Create new process.

**Parameters:**
- `program`: Path to executable
- `args`: Command-line arguments

**Returns:** Process handle or error

#### wait

```systemscript
fn wait(process: &Process) -> i32
```

Wait for process completion.

**Parameters:**
- `process`: Process to wait for

**Returns:** Exit status

#### environment

```systemscript
fn environment() -> HashMap<str, str>
```

Get environment variables.

**Returns:** Map of variable names to values

### system.time

#### now

```systemscript
fn now() -> TimeStamp
```

Get current system time.

**Returns:** Timestamp structure

#### sleep

```systemscript
fn sleep(duration: Duration)
```

Sleep for specified duration.

**Parameters:**
- `duration`: Time to sleep

**Example:**
```systemscript
system.time.sleep(Duration::milliseconds(100));
```

#### Duration Structure

```systemscript
struct Duration {
    fn seconds(s: u64) -> Duration;
    fn milliseconds(ms: u64) -> Duration;
    fn microseconds(us: u64) -> Duration;
    fn nanoseconds(ns: u64) -> Duration;
}
```

## memory Module

### memory.alloc

```systemscript
fn alloc<T>(count: usize) -> *T
```

Allocate memory for count elements.

**Parameters:**
- `T`: Type to allocate
- `count`: Number of elements

**Returns:** Pointer to allocated memory

**Example:**
```systemscript
let buffer = memory.alloc<u8>(1024);
```

### memory.free

```systemscript
fn free<T>(ptr: *T)
```

Free allocated memory.

**Parameters:**
- `ptr`: Pointer to free

### memory.alloc_aligned

```systemscript
fn alloc_aligned<T>(count: usize, alignment: usize) -> *T
```

Allocate aligned memory.

**Parameters:**
- `T`: Type to allocate
- `count`: Number of elements
- `alignment`: Alignment in bytes

**Returns:** Aligned pointer

### memory.set

```systemscript
fn set(dest: *u8, value: u8, count: usize)
```

Fill memory with value.

**Parameters:**
- `dest`: Destination pointer
- `value`: Byte value
- `count`: Number of bytes

### memory.copy

```systemscript
fn copy(dest: *u8, source: *u8, count: usize)
```

Copy memory region.

**Parameters:**
- `dest`: Destination
- `source`: Source
- `count`: Bytes to copy

### memory.compare

```systemscript
fn compare(ptr1: *u8, ptr2: *u8, count: usize) -> bool
```

Compare memory regions.

**Returns:** True if equal

## io Module

### io.File

#### open

```systemscript
fn open(path: str, mode: FileMode) -> Result<File, Error>
```

Open file.

**Parameters:**
- `path`: File path
- `mode`: Open mode (READ, WRITE, APPEND)

**Returns:** File handle or error

**Example:**
```systemscript
let file = io.File::open("data.txt", io.READ)?;
```

#### read

```systemscript
fn read(self, buffer: *u8, count: usize) -> Result<usize, Error>
```

Read bytes from file.

**Parameters:**
- `buffer`: Destination buffer
- `count`: Maximum bytes to read

**Returns:** Bytes read or error

#### write

```systemscript
fn write(self, buffer: *u8, count: usize) -> Result<usize, Error>
```

Write bytes to file.

**Returns:** Bytes written or error

#### close

```systemscript
fn close(self)
```

Close file handle.

#### read_all

```systemscript
fn read_all(self) -> Result<[]u8, Error>
```

Read entire file contents.

**Returns:** File contents as byte array

### io.port

#### read

```systemscript
fn read<T>(port: u16) -> T
```

Read from I/O port.

**Parameters:**
- `T`: Type to read (u8, u16, u32)
- `port`: Port number

**Returns:** Value read

**Example:**
```systemscript
unsafe {
    let value = io.port.read<u8>(0x3F8);
}
```

#### write

```systemscript
fn write<T>(port: u16, value: T)
```

Write to I/O port.

**Parameters:**
- `T`: Type to write
- `port`: Port number
- `value`: Value to write

## network Module

### network.socket

#### create

```systemscript
fn create(family: AddressFamily, type: SocketType) -> Result<Socket, Error>
```

Create socket.

**Parameters:**
- `family`: AF_INET, AF_INET6
- `type`: SOCK_STREAM, SOCK_DGRAM

**Returns:** Socket handle or error

**Example:**
```systemscript
let socket = network.socket.create(
    network.AF_INET,
    network.SOCK_STREAM
)?;
```

#### bind

```systemscript
fn bind(self, address: Address) -> Result<(), Error>
```

Bind socket to address.

#### listen

```systemscript
fn listen(self, backlog: i32) -> Result<(), Error>
```

Listen for connections.

#### accept

```systemscript
fn accept(self) -> Result<Socket, Error>
```

Accept incoming connection.

#### connect

```systemscript
fn connect(self, address: Address) -> Result<(), Error>
```

Connect to remote address.

#### send

```systemscript
fn send(self, data: *u8, length: usize) -> Result<usize, Error>
```

Send data through socket.

#### receive

```systemscript
fn receive(self, buffer: *u8, length: usize) -> Result<usize, Error>
```

Receive data from socket.

### network.http

#### Client

```systemscript
struct Client {
    fn new() -> Client;
    fn get(url: str) -> Result<Response, Error>;
    fn post(url: str, body: *u8, length: usize) -> Result<Response, Error>;
}
```

HTTP client implementation.

**Example:**
```systemscript
let client = network.http.Client::new();
let response = client.get("https://example.com")?;
```

#### Response

```systemscript
struct Response {
    status: u16,
    headers: HashMap<str, str>,
    body: []u8,
    
    fn body_text(self) -> str;
}
```

## hardware Module

### hardware.cpu

#### read_cr0/cr2/cr3/cr4

```systemscript
fn read_cr0() -> u64
fn read_cr2() -> u64
fn read_cr3() -> u64
fn read_cr4() -> u64
```

Read control registers.

**Returns:** Register value

#### write_cr0/cr3/cr4

```systemscript
fn write_cr0(value: u64)
fn write_cr3(value: u64)
fn write_cr4(value: u64)
```

Write control registers.

#### cpuid

```systemscript
fn cpuid(leaf: u32, subleaf: u32) -> (u32, u32, u32, u32)
```

Execute CPUID instruction.

**Returns:** (EAX, EBX, ECX, EDX) values

#### halt

```systemscript
fn halt()
```

Halt processor until interrupt.

#### pause

```systemscript
fn pause()
```

Pause hint for spin loops.

### hardware.interrupts

#### enable

```systemscript
fn enable()
```

Enable interrupts.

#### disable

```systemscript
fn disable() -> u64
```

Disable interrupts.

**Returns:** Previous interrupt state

#### restore

```systemscript
fn restore(flags: u64)
```

Restore interrupt state.

#### register_handler

```systemscript
fn register_handler(vector: u8, handler: fn(&InterruptFrame))
```

Register interrupt handler.

**Parameters:**
- `vector`: Interrupt vector number
- `handler`: Handler function

## collections Module

### collections.Vector

```systemscript
struct Vector<T> {
    fn new() -> Vector<T>;
    fn with_capacity(capacity: usize) -> Vector<T>;
    fn push(self, value: T);
    fn pop(self) -> Option<T>;
    fn get(self, index: usize) -> Option<&T>;
    fn length(self) -> usize;
    fn capacity(self) -> usize;
}
```

Dynamic array implementation.

**Example:**
```systemscript
let vec = collections.Vector<i32>::new();
vec.push(1);
vec.push(2);
vec.push(3);
```

### collections.HashMap

```systemscript
struct HashMap<K, V> {
    fn new() -> HashMap<K, V>;
    fn insert(self, key: K, value: V);
    fn get(self, key: &K) -> Option<&V>;
    fn remove(self, key: &K) -> Option<V>;
    fn contains_key(self, key: &K) -> bool;
}
```

Hash table implementation.

**Example:**
```systemscript
let map = collections.HashMap<str, i32>::new();
map.insert("one", 1);
map.insert("two", 2);
```

### collections.List

```systemscript
struct List<T> {
    fn new() -> List<T>;
    fn push_front(self, value: T);
    fn push_back(self, value: T);
    fn pop_front(self) -> Option<T>;
    fn pop_back(self) -> Option<T>;
}
```

Doubly-linked list.

## concurrency Module

### concurrency.Thread

```systemscript
struct Thread {
    fn spawn(f: fn()) -> Thread;
    fn join(self);
    fn sleep(duration: Duration);
}
```

Thread management.

**Example:**
```systemscript
let thread = concurrency.Thread::spawn(|| {
    io.println("Worker thread");
});
thread.join();
```

### concurrency.Mutex

```systemscript
struct Mutex<T> {
    fn new(value: T) -> Mutex<T>;
    fn lock(self) -> MutexGuard<T>;
}
```

Mutual exclusion lock.

**Example:**
```systemscript
let mutex = concurrency.Mutex::new(0);
{
    let guard = mutex.lock();
    *guard += 1;
}  // Lock released
```

### concurrency.Atomic

```systemscript
struct Atomic<T> {
    fn new(value: T) -> Atomic<T>;
    fn load(self, ordering: Ordering) -> T;
    fn store(self, value: T, ordering: Ordering);
    fn fetch_add(self, value: T, ordering: Ordering) -> T;
    fn fetch_sub(self, value: T, ordering: Ordering) -> T;
    fn compare_exchange(
        self,
        expected: T,
        new: T,
        success: Ordering,
        failure: Ordering
    ) -> Result<T, T>;
}
```

Atomic operations.

## security Module

### security.crypto

#### sha256

```systemscript
fn sha256(data: *u8, length: usize) -> [u8; 32]
```

Compute SHA-256 hash.

**Returns:** 32-byte hash

#### aes_encrypt

```systemscript
fn aes_encrypt(
    data: *u8,
    length: usize,
    key: *u8,
    mode: AesMode
) -> []u8
```

Encrypt data with AES.

**Parameters:**
- `data`: Data to encrypt
- `length`: Data length
- `key`: Encryption key
- `mode`: AES_128, AES_192, AES_256

**Returns:** Encrypted data

## Error Handling

### Result Type

```systemscript
enum Result<T, E> {
    Ok(T),
    Err(E)
}
```

Result type for operations that can fail.

**Example:**
```systemscript
fn divide(a: i32, b: i32) -> Result<i32, str> {
    if (b == 0) {
        return Err("Division by zero");
    }
    return Ok(a / b);
}

match divide(10, 2) {
    Ok(result) => io.println("Result: {}", result),
    Err(msg) => io.println("Error: {}", msg)
}
```

### Option Type

```systemscript
enum Option<T> {
    Some(T),
    None
}
```

Optional value type.

**Example:**
```systemscript
fn find(array: []i32, target: i32) -> Option<usize> {
    for (i in 0..array.length) {
        if (array[i] == target) {
            return Some(i);
        }
    }
    return None;
}
```
