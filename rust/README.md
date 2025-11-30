# Rust for Python Developers

**Target Audience:** Python developers used to modern tooling (`uv`, `ruff`, `zed`).
**Goal:** Map your mental models to Rust's Ownership model and Cargo ecosystem.

-----

## 1\. The Mental Model Shift

| Concept | Python | Rust |
| :--- | :--- | :--- |
| **Philosophy** | "Development Speed" (Dynamic, Interpreter). | "Reliability & Speed" (Zero-cost abstractions). |
| **Memory** | Garbage Collector (Automatic). | **Ownership/Borrowing** (Compile-time management). |
| **Variables** | Mutable by default. | **Immutable by default.** |
| **Async** | Cooperative (`async`/`await`). | Cooperative (`async`/`await`) but needs a Runtime (`Tokio`). |
| **Classes** | `class` with inheritance. | `struct` (Data) + `trait` (Behavior) + `enum` (State). |

-----

## 2\. Tooling (The `uv` Mapping)

Rust uses `cargo`. It handles everything: building, deps, testing, docs, and publishing. It is the inspiration for tools like `uv` and `poetry`.

| Python (`uv`) | Rust (`cargo`) | Notes |
| :--- | :--- | :--- |
| `uv init myproj` | `cargo new myproj` | Creates directory + `Cargo.toml`. |
| `uv add pkg` | `cargo add pkg` | Adds dep to `Cargo.toml` & downloads. |
| `uv sync` | `cargo build` | Resolves deps, updates `Cargo.lock`, compiles. |
| `uv run main.py` | `cargo run` | Compiles debug build and runs it. |
| `uv build` | `cargo build --release` | **Critical.** `-r` turns on optimizations (10-100x faster). |
| `ruff format` | `cargo fmt` | Standard formatter. |
| `ruff check` | `cargo clippy` | **The secret weapon.** Catches bugs and teaches you Rust. |

### Quick Start: Scaffolding

```bash
# 1. Create project (Standard binary layout)
cargo new my-rust-proj
cd my-rust-proj

# 2. Add a dependency (e.g., colored text)
cargo add colored

# 3. Edit main.rs
# Zed has excellent Rust support via rust-analyzer (auto-installed usually)
zed src/main.rs
```

-----

## 3\. Interactive Rust (REPL & Notebooks)

Rust is compiled, but Google has built an amazing Jupyter kernel for it.

### The Jupyter Solution (Best for VS Code / Zed)

Use **Evcxr**.

```bash
# Install the component
rustup component add rust-src
# Install the kernel binary
cargo install evcxr_jupyter
# Register it
evcxr_jupyter --install
```

*Usage:* Open a `.ipynb` file and select the **Rust** kernel.

### The CLI Solution

```bash
cargo install evcxr_repl
evcxr
>> println!("Hello world");
```

-----

## 4\. Syntax: The "Rosetta Stone"

### Variables (Mutability)

In Rust, variables are constant by default. You must opt-in to change them.

| Feature | Python | Rust |
| :--- | :--- | :--- |
| **Immutable** | Impossible (strictly speaking). | `let x = 5;` (Cannot change `x`). |
| **Mutable** | `x = 5` | `let mut x = 5;` |
| **Explicit** | `x: int = 5` | `let x: i32 = 5;` |
| **Constants** | `MAX = 10` | `const MAX: i32 = 10;` |
| **Shadowing** | N/A (Reassigns variable). | `let x = "5"; let x = x.parse();` (Allowed\!). |

### Loops

Rust has `loop` (infinite), `while`, and `for`.

**Standard Loop:**

```rust
let items = vec!["a", "b"];
for (i, v) in items.iter().enumerate() {
    println!("Index: {}, Value: {}", i, v);
}
```

**Infinite Loop (Preferred over `while true`):**

```rust
loop {
    println!("Spinning...");
    break; // You can also return values: break 10;
}
```

### Functions & Implicit Return

Rust is expression-based. The last line of a function (without a semicolon) is the return value.

```rust
// Snake case function names
fn calculate(a: i32, b: i32) -> i32 {
    let sum = a + b;
    sum // No semicolon = "return sum"
}
```

### Error Handling (Result & Option)

Rust has **no exceptions** and **no `None`** (null).

  * `Option<T>`: Maybe there is a value (`Some(v)`), maybe not (`None`).
  * `Result<T, E>`: Maybe success (`Ok(v)`), maybe error (`Err(e)`).

**The `?` Operator (The Magic):**
If a function returns a Result, putting `?` after it means: *"If Error, return error immediately. If Ok, give me the value."*

```rust
use std::fs;

fn read_file() -> Result<String, std::io::Error> {
    // Reads file. If fails, function exits here returning Err.
    // If succeeds, 'content' gets the string.
    let content = fs::read_to_string("config.txt")?;
    
    Ok(content)
}
```

-----

## 5\. Structs & Traits (OOP Replacement)

### Structs (The Data)

```rust
struct User {
    name: String,
    age: u8, // u8 = unsigned 8-bit integer
}

// Initialization
let u = User {
    name: String::from("Ryan"),
    age: 30,
};
```

### Impl (The Methods)

```rust
impl User {
    // "new" is just a convention, like __init__
    fn new(name: &str) -> Self {
        User { name: name.to_string(), age: 0 }
    }

    // &self = Read (self)
    // &mut self = Read/Write (self)
    fn have_birthday(&mut self) {
        self.age += 1;
    }
}
```

### Traits (Polymorphism)

Traits are like Interfaces, but more powerful.

```rust
trait Speaker {
    fn speak(&self) -> String;
}

// Implement for User
impl Speaker for User {
    fn speak(&self) -> String {
        format!("{} says hello", self.name)
    }
}

// Generic function accepting any Speaker
fn make_noise(s: &impl Speaker) {
    println!("{}", s.speak());
}
```

-----

## 6\. Concurrency: Async & Tokio

Rust's async looks very similar to Python's, but Rust doesn't have a built-in runtime. You almost always use **Tokio**.

**Add Tokio:**

```bash
cargo add tokio --features full
```

**The Code:**

```rust
use tokio::time::{sleep, Duration};

async fn do_work() {
    println!("Working...");
    sleep(Duration::from_secs(1)).await;
    println!("Done!");
}

#[tokio::main] // The macro that starts the event loop
async fn main() {
    // JoinHandle, similar to asyncio.create_task
    let task = tokio::spawn(do_work());
    
    println!("Main thread continues...");
    
    // Await the task (like await task)
    let _ = task.await; 
}
```

-----

## 7\. File Paths (`pathlib` equivalent)

Rust is strict about strings vs paths.

  * `Path`: A "slice" of a path (borrowed, like `&str`).
  * `PathBuf`: An owned path (mutable, grows, like `String`).

| Task | Python (`pathlib`) | Rust (`std::path`) |
| :--- | :--- | :--- |
| **Import** | `from pathlib import Path` | `use std::path::{Path, PathBuf};` |
| **Join** | `p = Path("a") / "b"` | `let p: PathBuf = Path::new("a").join("b");` |
| **Exists** | `p.exists()` | `p.exists()` (Returns bool) |
| **Filename** | `p.name` | `p.file_name()` |
| **Extension** | `p.suffix` | `p.extension()` |
| **To String** | `str(p)` | `p.to_str().unwrap()` (Can fail if UTF-8 invalid) |

**Example: Scanning a Directory**

```rust
use std::fs;

fn main() {
    // read_dir returns an iterator of Results
    if let Ok(entries) = fs::read_dir("./data") {
        for entry in entries {
            if let Ok(entry) = entry {
                let path = entry.path();
                if path.extension().unwrap_or_default() == "json" {
                    println!("Found JSON: {:?}", path);
                }
            }
        }
    }
}
```

-----

## 8\. The "Borrow Checker" (The Hard Part)

In Python, variables are references. In Rust, variables **own** their data.

**The Rule:** A value can have:

1.  **Many** readers (immutable borrows `&T`).
2.  **OR**
3.  **One** writer (mutable borrow `&mut T`).
4.  **NEVER BOTH.**

**Python (Fine):**

```python
l = [1, 2, 3]
a = l # Reference
b = l # Reference
l.append(4) # Both a and b see the change
```

**Rust (Compile Error):**

```rust
let mut l = vec![1, 2, 3];
let a = &l;     // Immutable borrow
let b = &mut l; // ERROR: Cannot borrow mutable because it is already borrowed!
println!("{:?}", a); 
```

*Why?* To prevent "Data Races." Rust ensures memory safety without a Garbage Collector.

-----

## 9\. Best Practices / Pro-Tips

1.  **Listen to the Compiler:** Rust error messages are famous for being helpful. They usually tell you exactly how to fix the code.
2.  **Use `cargo clippy`:** It's like having a senior Rust engineer looking over your shoulder. It will suggest "idiomatic" improvements.
3.  **Don't fight the Borrow Checker:** If you are struggling with lifetimes (`'a`), try to use `.clone()` (copies data) or `String` (owned) instead of `&str` (borrowed) until you understand the system better. It's okay to be slightly inefficient while learning.
4.  **`unwrap()` vs `expect()`:** When prototyping, use `.unwrap()` to crash on errors. In production, handle the `Result` properly.
5.  **Documentation:** Run `cargo doc --open`. It builds a local website of documentation for **your specific dependencies**. It is incredible.
