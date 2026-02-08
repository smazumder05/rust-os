# rust-os
> My experimental os in Rust
# Rust Operating System Roadmap

This roadmap is based on the [Writing an OS in Rust](https://os.phil-opp.com/) series. It breaks the development process into strict **Minimum Viable Products (MVPs)**, each building upon the last to create a functional, 64-bit x86_64 kernel.

---

##  Toolchain Requirements

Before starting, ensure the following tools are installed and configured:

* **Rust Nightly:** `rustup override set nightly` (Required for features like `abi_x86_interrupt`).
* **Bootimage:** `cargo install bootimage` (Used to create a bootable disk image).
* **QEMU:** To emulate the hardware and run the kernel.

---

##  Development Phases

### MVP 2: VGA Text Mode Driver

**Goal:** Safely print formatted text to the screen.

* **Core Tasks:**
* Create a Rust module to handle the VGA text buffer at memory address `0xb8000`.
* Use the `volatile` crate to prevent the compiler from optimizing away memory writes.
* Implement `core::fmt::Write` to support Rust's formatting macros.
* Use `lazy_static` (or `conquer_once`) and `spin` mutexes to create a safe, global `WRITER` instance.
* Implement custom `print!` and `println!` macros.


* **Success Metric:** Calling `println!("Hello World!");` in `_start` displays text on the screen.


> **Note:** This project is for educational purposes and involves low-level systems programming. Ensure you have the `x86_64` architecture targets installed via `rustup`.

