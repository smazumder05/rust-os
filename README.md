# rust-os
> My experimental os in Rust
# Rust Operating System Roadmap

This roadmap is based on the [Writing an OS in Rust](https://os.phil-opp.com/) series. It breaks the development process into strict **Minimum Viable Products (MVPs)**, each building upon the last to create a functional, 64-bit x86_64 kernel.

---

## 🛠 Toolchain Requirements

Before starting, ensure the following tools are installed and configured:

* **Rust Nightly:** `rustup override set nightly` (Required for features like `abi_x86_interrupt`).
* **Bootimage:** `cargo install bootimage` (Used to create a bootable disk image).
* **QEMU:** To emulate the hardware and run the kernel.

---

## 🚀 Development Phases

### MVP 1: The Freestanding Rust Binary

**Goal:** Boot a minimal Rust kernel on QEMU that loops infinitely, proving we have broken free of the host OS.

* **Core Tasks:**
* Set up a Rust project with a custom target specification (`x86_64-unknown-none`).
* Disable the Rust Standard Library (`#![no_std]`).
* Implement a **Panic Handler** (required without a standard library).
* Overwrite the entry point (`_start`) to prevent the compiler from looking for `main`.
* Integrate the `bootloader` crate to handle the transition from BIOS/UEFI to Long Mode (64-bit).


* **Success Metric:** `cargo run` launches QEMU and the machine does not reboot repeatedly.

---

### MVP 2: VGA Text Mode Driver

**Goal:** Safely print formatted text to the screen.

* **Core Tasks:**
* Create a Rust module to handle the VGA text buffer at memory address `0xb8000`.
* Use the `volatile` crate to prevent the compiler from optimizing away memory writes.
* Implement `core::fmt::Write` to support Rust's formatting macros.
* Use `lazy_static` (or `conquer_once`) and `spin` mutexes to create a safe, global `WRITER` instance.
* Implement custom `print!` and `println!` macros.


* **Success Metric:** Calling `println!("Hello World!");` in `_start` displays text on the screen.

---

### MVP 3: CPU Exception Handling

**Goal:** Prevent the CPU from resetting when an error occurs (e.g., divide by zero).

* **Core Tasks:**
* Initialize the **IDT (Interrupt Descriptor Table)** using the `x86_64` crate.
* Create `extern "x86-interrupt"` calling convention handlers.
* Handle **Breakpoint** exceptions for debugging.
* Handle **Double Faults** to prevent boot loops.
* Set up the **GDT (Global Descriptor Table)** and **TSS (Task State Segment)** to switch stacks on double faults.


* **Success Metric:** Triggering a breakpoint via `int3` prints a message and continues execution.

---

### MVP 4: Hardware Interrupts

**Goal:** Interact with the real world (Keyboard and Timer).

* **Core Tasks:**
* Initialize the **8259 PIC** (Programmable Interrupt Controller) to map hardware IRQs to CPU interrupts.
* Enable interrupts using `x86_64::instructions::interrupts::enable`.
* Implement a **Timer Interrupt** handler for system ticks.
* Implement a **Keyboard Interrupt** handler (reading scancodes from port `0x60`).
* Use the `pc-keyboard` crate to decode raw scancodes into characters.


* **Success Metric:** Physical keyboard input appears on the QEMU screen.

---

### MVP 5: Memory Management & Heap Allocation

**Goal:** Enable dynamic memory usage (`Vec`, `Box`, `String`).

* **Core Tasks:**
* **Paging:** Map virtual addresses to physical addresses.
* Access the memory map provided by the bootloader.
* Implement a **Frame Allocator** to manage physical memory chunks.
* Map the complete physical memory to a virtual offset.
* Implement the `GlobalAlloc` trait.
* Create a **Heap Allocator** (Bump Allocator or Fixed-Size Block Allocator).


* **Success Metric:** `let x = Box::new(42);` runs without crashing.

---

### MVP 6: Cooperative Multitasking

**Goal:** Run multiple tasks concurrently using `async/await` without traditional threads.

* **Core Tasks:**
* Implement the `Future` trait and `poll` method manually.
* Create a `Task` struct (a pinned, heap-allocated future).
* Build an **Executor** that holds a queue of ready tasks.
* Create a **Waker** to notify the executor when a task is ready.
* Convert the Keyboard handler to an async task using an `ArrayQueue`.


* **Success Metric:** A background task prints a dot every second while simultaneously handling interactive keyboard input.

---

> **Note:** This project is for educational purposes and involves low-level systems programming. Ensure you have the `x86_64` architecture targets installed via `rustup`.

