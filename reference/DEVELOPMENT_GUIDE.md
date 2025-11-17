# SystemScript Kernel Development Guide

This guide covers operating system kernel development using SystemScript, including initialization, memory management, process scheduling, and device drivers.

## Kernel Architecture

### Minimal Kernel Structure

```systemscript
module kernel;

import hardware.cpu;
import hardware.interrupts;
import memory.physical;
import memory.virtual;

#[no_mangle]
#[export]
fn kernel_main(boot_info: *BootInfo) {
    // Initialize early subsystems
    early_init();
    
    // Initialize memory management
    memory_init(boot_info);
    
    // Initialize interrupt handling
    interrupts_init();
    
    // Initialize process scheduler
    scheduler_init();
    
    // Start first process
    start_init_process();
    
    // Enter scheduler
    scheduler_run();
}
```

### Boot Sequence

#### Entry Point

```systemscript
#[link_section = ".boot"]
#[naked]
#[no_mangle]
extern "C" fn _start() {
    unsafe {
        asm {
            "cli"                    // Disable interrupts
            "mov esp, stack_top"     // Set up stack
            "push ebx"               // Multiboot info
            "call kernel_main"       // Jump to kernel
            "1: hlt"                 // Halt if return
            "jmp 1b"
        }
    }
}
```

#### Early Initialization

```systemscript
fn early_init() {
    // Initialize serial port for debugging
    serial.init();
    
    // Initialize basic VGA output
    vga.init();
    
    // Print boot message
    serial.write("Kernel starting...\n");
    
    // Detect CPU features
    cpu.detect_features();
    
    // Set up basic exception handlers
    setup_early_exceptions();
}
```

## Memory Management

### Physical Memory Manager

```systemscript
struct PhysicalMemoryManager {
    bitmap: *u8,
    total_frames: usize,
    free_frames: usize,
    
    fn init(memory_map: *MemoryMap, map_entries: usize) {
        // Calculate total memory
        let total_memory: usize = 0;
        for (i in 0..map_entries) {
            let entry = &memory_map[i];
            if (entry.type == MEMORY_AVAILABLE) {
                total_memory += entry.length;
            }
        }
        
        // Calculate number of frames
        total_frames = total_memory / PAGE_SIZE;
        free_frames = total_frames;
        
        // Allocate bitmap
        let bitmap_size = (total_frames + 7) / 8;
        bitmap = allocate_early(bitmap_size);
        
        // Mark all frames as free
        memory.set(bitmap, 0xFF, bitmap_size);
        
        // Mark used regions
        mark_used_regions(memory_map, map_entries);
    }
    
    fn alloc_frame() -> *void {
        if (free_frames == 0) {
            return null;
        }
        
        // Find free frame
        for (i in 0..total_frames) {
            let byte_index = i / 8;
            let bit_index = i % 8;
            
            if (bitmap[byte_index] & (1 << bit_index)) {
                // Frame is free, allocate it
                bitmap[byte_index] &= ~(1 << bit_index);
                free_frames -= 1;
                return (i * PAGE_SIZE) as *void;
            }
        }
        
        return null;
    }
    
    fn free_frame(address: *void) {
        let frame = (address as usize) / PAGE_SIZE;
        let byte_index = frame / 8;
        let bit_index = frame % 8;
        
        if (!(bitmap[byte_index] & (1 << bit_index))) {
            // Frame is allocated, free it
            bitmap[byte_index] |= (1 << bit_index);
            free_frames += 1;
        }
    }
}
```

### Virtual Memory Manager

```systemscript
struct PageTable {
    entries: [PageTableEntry; 512]
}

struct PageTableEntry {
    value: u64,
    
    fn present(self) -> bool {
        return (value & 0x1) != 0;
    }
    
    fn writable(self) -> bool {
        return (value & 0x2) != 0;
    }
    
    fn user_accessible(self) -> bool {
        return (value & 0x4) != 0;
    }
    
    fn frame(self) -> u64 {
        return value & 0xFFFFFFFFFF000;
    }
    
    fn set_frame(self, address: u64, flags: u64) {
        value = (address & 0xFFFFFFFFFF000) | flags;
    }
}

struct VirtualMemoryManager {
    kernel_page_table: *PageTable,
    
    fn init() {
        // Create kernel page table
        kernel_page_table = physical.alloc_frame() as *PageTable;
        memory.set(kernel_page_table as *u8, 0, PAGE_SIZE);
        
        // Map kernel sections
        map_kernel_sections();
        
        // Load page table
        cpu.write_cr3(kernel_page_table as u64);
    }
    
    fn map_page(virtual_addr: u64, physical_addr: u64, flags: u64) {
        let pml4_index = (virtual_addr >> 39) & 0x1FF;
        let pdp_index = (virtual_addr >> 30) & 0x1FF;
        let pd_index = (virtual_addr >> 21) & 0x1FF;
        let pt_index = (virtual_addr >> 12) & 0x1FF;
        
        // Get or create PML4 entry
        let pml4 = kernel_page_table;
        if (!pml4.entries[pml4_index].present()) {
            let new_table = physical.alloc_frame();
            memory.set(new_table, 0, PAGE_SIZE);
            pml4.entries[pml4_index].set_frame(
                new_table as u64,
                PAGE_PRESENT | PAGE_WRITABLE
            );
        }
        
        // Get or create PDP entry
        let pdp = pml4.entries[pml4_index].frame() as *PageTable;
        if (!pdp.entries[pdp_index].present()) {
            let new_table = physical.alloc_frame();
            memory.set(new_table, 0, PAGE_SIZE);
            pdp.entries[pdp_index].set_frame(
                new_table as u64,
                PAGE_PRESENT | PAGE_WRITABLE
            );
        }
        
        // Get or create PD entry
        let pd = pdp.entries[pdp_index].frame() as *PageTable;
        if (!pd.entries[pd_index].present()) {
            let new_table = physical.alloc_frame();
            memory.set(new_table, 0, PAGE_SIZE);
            pd.entries[pd_index].set_frame(
                new_table as u64,
                PAGE_PRESENT | PAGE_WRITABLE
            );
        }
        
        // Set PT entry
        let pt = pd.entries[pd_index].frame() as *PageTable;
        pt.entries[pt_index].set_frame(physical_addr, flags);
        
        // Flush TLB entry
        cpu.invlpg(virtual_addr);
    }
}
```

## Interrupt Handling

### IDT Setup

```systemscript
struct IDTEntry {
    offset_low: u16,
    selector: u16,
    ist: u8,
    flags: u8,
    offset_mid: u16,
    offset_high: u32,
    reserved: u32
}

struct IDT {
    entries: [IDTEntry; 256]
}

static mut idt: IDT;

fn interrupts_init() {
    // Clear IDT
    memory.set(&mut idt as *u8, 0, sizeof(IDT));
    
    // Set up exception handlers
    set_idt_entry(0, divide_by_zero_handler, 0x8E);
    set_idt_entry(1, debug_handler, 0x8E);
    set_idt_entry(3, breakpoint_handler, 0x8E);
    set_idt_entry(6, invalid_opcode_handler, 0x8E);
    set_idt_entry(8, double_fault_handler, 0x8E);
    set_idt_entry(13, general_protection_fault_handler, 0x8E);
    set_idt_entry(14, page_fault_handler, 0x8E);
    
    // Set up IRQ handlers
    for (i in 0..16) {
        set_idt_entry(32 + i, irq_stub_table[i], 0x8E);
    }
    
    // Load IDT
    let idtr = IDTR {
        limit: (sizeof(IDT) - 1) as u16,
        base: &idt as u64
    };
    
    unsafe {
        asm {
            "lidt [%0]"
            :
            : "r"(&idtr)
            : "memory"
        }
    }
    
    // Enable interrupts
    cpu.enable_interrupts();
}

fn set_idt_entry(index: u8, handler: fn(), flags: u8) {
    let entry = &mut idt.entries[index];
    let handler_addr = handler as u64;
    
    entry.offset_low = (handler_addr & 0xFFFF) as u16;
    entry.selector = 0x08;  // Kernel code segment
    entry.ist = 0;
    entry.flags = flags;
    entry.offset_mid = ((handler_addr >> 16) & 0xFFFF) as u16;
    entry.offset_high = ((handler_addr >> 32) & 0xFFFFFFFF) as u32;
    entry.reserved = 0;
}
```

### Exception Handlers

```systemscript
#[interrupt_handler]
fn page_fault_handler(frame: &InterruptFrame) {
    let fault_address = cpu.read_cr2();
    let error_code = frame.error_code;
    
    // Decode error code
    let present = (error_code & 0x1) != 0;
    let write = (error_code & 0x2) != 0;
    let user = (error_code & 0x4) != 0;
    let reserved = (error_code & 0x8) != 0;
    let instruction = (error_code & 0x10) != 0;
    
    serial.write("Page fault at address: 0x");
    serial.write_hex(fault_address);
    serial.write("\n");
    
    if (!present) {
        // Page not present, allocate it
        let physical_frame = physical.alloc_frame();
        if (physical_frame != null) {
            virtual_memory.map_page(
                fault_address & ~0xFFF,
                physical_frame as u64,
                PAGE_PRESENT | PAGE_WRITABLE | PAGE_USER
            );
            return;
        }
    }
    
    // Unhandled page fault
    panic("Unhandled page fault");
}

#[interrupt_handler]
fn general_protection_fault_handler(frame: &InterruptFrame) {
    serial.write("General protection fault\n");
    serial.write("Error code: ");
    serial.write_hex(frame.error_code as u64);
    serial.write("\n");
    
    print_registers(frame);
    
    panic("General protection fault");
}
```

## Process Management

### Process Structure

```systemscript
struct Process {
    id: u32,
    name: [char; 32],
    state: ProcessState,
    context: ProcessContext,
    page_table: *PageTable,
    kernel_stack: *u8,
    user_stack: *u8,
    priority: u8,
    time_slice: u32,
    next: *Process
}

enum ProcessState {
    Ready,
    Running,
    Blocked,
    Terminated
}

struct ProcessContext {
    rax: u64,
    rbx: u64,
    rcx: u64,
    rdx: u64,
    rsi: u64,
    rdi: u64,
    rbp: u64,
    rsp: u64,
    r8: u64,
    r9: u64,
    r10: u64,
    r11: u64,
    r12: u64,
    r13: u64,
    r14: u64,
    r15: u64,
    rip: u64,
    rflags: u64,
    cs: u64,
    ss: u64
}
```

### Scheduler

```systemscript
struct Scheduler {
    ready_queue: *Process,
    current_process: *Process,
    next_pid: u32,
    
    fn init() {
        ready_queue = null;
        current_process = null;
        next_pid = 1;
        
        // Create idle process
        let idle = create_process("idle", idle_task);
        idle.priority = 0;
        add_to_ready_queue(idle);
    }
    
    fn create_process(name: str, entry_point: fn()) -> *Process {
        let process = kalloc(sizeof(Process)) as *Process;
        
        process.id = next_pid++;
        memory.copy(process.name, name, name.length);
        process.state = ProcessState::Ready;
        
        // Allocate kernel stack
        process.kernel_stack = kalloc(KERNEL_STACK_SIZE);
        
        // Allocate user stack
        process.user_stack = kalloc(USER_STACK_SIZE);
        
        // Set up initial context
        setup_initial_context(process, entry_point);
        
        // Create page table
        process.page_table = create_page_table();
        
        return process;
    }
    
    fn schedule() {
        if (ready_queue == null) {
            return;
        }
        
        // Save current process context if running
        if (current_process != null && 
            current_process.state == ProcessState::Running) {
            save_context(current_process);
            current_process.state = ProcessState::Ready;
            add_to_ready_queue(current_process);
        }
        
        // Select next process
        current_process = ready_queue;
        ready_queue = ready_queue.next;
        current_process.next = null;
        current_process.state = ProcessState::Running;
        
        // Switch to process
        switch_to_process(current_process);
    }
    
    fn switch_to_process(process: *Process) {
        // Load page table
        cpu.write_cr3(process.page_table as u64);
        
        // Restore context
        restore_context(process);
    }
}
```

### Context Switching

```systemscript
fn save_context(process: *Process) {
    unsafe {
        asm {
            "mov [%0 + 0], rax"
            "mov [%0 + 8], rbx"
            "mov [%0 + 16], rcx"
            "mov [%0 + 24], rdx"
            "mov [%0 + 32], rsi"
            "mov [%0 + 40], rdi"
            "mov [%0 + 48], rbp"
            "mov [%0 + 56], rsp"
            "mov [%0 + 64], r8"
            "mov [%0 + 72], r9"
            "mov [%0 + 80], r10"
            "mov [%0 + 88], r11"
            "mov [%0 + 96], r12"
            "mov [%0 + 104], r13"
            "mov [%0 + 112], r14"
            "mov [%0 + 120], r15"
            :
            : "r"(&process.context)
            : "memory"
        }
    }
}

fn restore_context(process: *Process) {
    unsafe {
        asm {
            "mov rax, [%0 + 0]"
            "mov rbx, [%0 + 8]"
            "mov rcx, [%0 + 16]"
            "mov rdx, [%0 + 24]"
            "mov rsi, [%0 + 32]"
            "mov rdi, [%0 + 40]"
            "mov rbp, [%0 + 48]"
            "mov rsp, [%0 + 56]"
            "mov r8, [%0 + 64]"
            "mov r9, [%0 + 72]"
            "mov r10, [%0 + 80]"
            "mov r11, [%0 + 88]"
            "mov r12, [%0 + 96]"
            "mov r13, [%0 + 104]"
            "mov r14, [%0 + 112]"
            "mov r15, [%0 + 120]"
            :
            : "r"(&process.context)
            : "memory"
        }
    }
}
```

## System Calls

### System Call Interface

```systemscript
const SYSCALL_READ: u64 = 0;
const SYSCALL_WRITE: u64 = 1;
const SYSCALL_OPEN: u64 = 2;
const SYSCALL_CLOSE: u64 = 3;
const SYSCALL_EXIT: u64 = 60;

fn syscall_handler(frame: &InterruptFrame) {
    let syscall_number = frame.rax;
    
    let result = match syscall_number {
        SYSCALL_READ => sys_read(
            frame.rdi as i32,
            frame.rsi as *u8,
            frame.rdx as usize
        ),
        SYSCALL_WRITE => sys_write(
            frame.rdi as i32,
            frame.rsi as *u8,
            frame.rdx as usize
        ),
        SYSCALL_OPEN => sys_open(
            frame.rdi as *char,
            frame.rsi as i32
        ),
        SYSCALL_CLOSE => sys_close(frame.rdi as i32),
        SYSCALL_EXIT => sys_exit(frame.rdi as i32),
        _ => -1  // Invalid syscall
    };
    
    // Return result in rax
    frame.rax = result as u64;
}

fn sys_write(fd: i32, buffer: *u8, count: usize) -> i64 {
    // Validate parameters
    if (!validate_user_pointer(buffer, count)) {
        return -1;
    }
    
    // Handle different file descriptors
    match fd {
        1 => {  // stdout
            for (i in 0..count) {
                terminal.putchar(buffer[i] as char);
            }
            return count as i64;
        },
        _ => return -1
    }
}
```

## Device Drivers

### Driver Interface

```systemscript
trait Driver {
    fn probe() -> bool;
    fn init() -> Result<(), Error>;
    fn shutdown();
    fn read(buffer: *u8, count: usize) -> i64;
    fn write(buffer: *u8, count: usize) -> i64;
}
```

### Example Driver Implementation

```systemscript
struct SerialDriver {
    port: u16,
    
    fn probe() -> bool {
        // Check if serial port exists
        return port.read<u8>(0x3F8 + 5) != 0xFF;
    }
    
    fn init() -> Result<(), Error> {
        port = 0x3F8;
        
        // Disable interrupts
        port.write<u8>(port + 1, 0x00);
        
        // Set baud rate
        port.write<u8>(port + 3, 0x80);
        port.write<u8>(port + 0, 0x03);
        port.write<u8>(port + 1, 0x00);
        
        // Configure port
        port.write<u8>(port + 3, 0x03);
        port.write<u8>(port + 2, 0xC7);
        port.write<u8>(port + 4, 0x0B);
        
        return Ok(());
    }
    
    fn write(buffer: *u8, count: usize) -> i64 {
        for (i in 0..count) {
            // Wait for transmit buffer empty
            while ((port.read<u8>(port + 5) & 0x20) == 0) {
                cpu.pause();
            }
            
            // Write character
            port.write<u8>(port, buffer[i]);
        }
        
        return count as i64;
    }
}
```
