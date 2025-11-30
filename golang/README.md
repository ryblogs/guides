# Go for Python Developers

**Target Audience:** Python developers used to modern tooling (`uv`, `ruff`, `zed`).
**Goal:** Map your existing mental models to Go's philosophy and tooling.

-----

## 1\. The Mental Model Shift

| Concept | Python | Go |
| :--- | :--- | :--- |
| **Philosophy** | "Magic" (Implicit, dynamic typing). | "Explicit" (Static typing, no magic). |
| **Virtual Envs** | Essential (`.venv`). | **Non-existent.** Deps go to global cache. |
| **Entry Point** | Script runs top-to-bottom. | `func main()` inside `package main`. |
| **Async** | Cooperative (`async`/`await`). | Preemptive (`go` routines). |
| **Classes** | `class` with inheritance. | `struct` with composition. |

-----

## 2\. Tooling (The `uv` Mapping)

Go ships as a single binary. There is no separate package manager; it is built-in.

| Python (`uv`) | Go Command | Notes |
| :--- | :--- | :--- |
| `uv init myproj` | `mkdir myproj && cd myproj`<br>`go mod init name/repo` | Creates `go.mod` (like `pyproject.toml`). |
| `uv add pkg` | `go get github.com/user/repo` | Downloads dep & updates `go.mod`. |
| `uv sync` | `go mod tidy` | **Crucial.** Removes unused deps, adds missing ones. |
| `uv run main.py` | `go run .` | Compiles and runs in memory. |
| `uv build` | `go build` | Compiles a binary executable. |
| `ruff format` | `go fmt` | The industry standard formatter. |

### Quick Start: Scaffolding a Project

```bash
# 1. Create directory
mkdir my-go-proj
cd my-go-proj

# 2. Initialize Module (Use your GitHub path usually)
go mod init github.com/yourname/my-go-proj

# 3. Add a dependency (e.g., color printing)
go get github.com/fatih/color

# 4. Create main.go
touch main.go
```

**Setup for Zed Editor:**
Zed has great Go support, but it needs the Language Server.

```bash
go install golang.org/x/tools/gopls@latest
```

-----

## 3\. Interactive Go (REPL & Notebooks)

Since Go is compiled, it does not have a native REPL. Use these tools to bridge the gap.

### The Jupyter Solution (Best for VS Code / Zed)

Use **Gonb**. This lets you run Go in `.ipynb` files with state preservation.

```bash
go install github.com/janpfeifer/gonb@latest
go install golang.org/x/tools/cmd/goimports@latest
gonb --install
```

*Usage:* Open any `.ipynb` in VS Code and select the "Go (gonb)" kernel.

### The CLI Solution (Best for Terminal)

Use **Yaegi** (Better than `gore` for Python devs). It is a true interpreter.

```bash
go install github.com/traefik/yaegi/cmd/yaegi@latest
# Run it
yaegi
> import "fmt"
> fmt.Println("Hello")
```

-----

## 4\. Syntax: The "Rosetta Stone"

### Variables

Go requires explicit types or short-declaration (`:=`). **Unused variables cause compiler errors.**

| Feature | Python | Go |
| :--- | :--- | :--- |
| **Dynamic** | `x = "hello"` | `x := "hello"` (Inferred inside functions) |
| **Explicit** | `x: str = "hello"` | `var x string = "hello"` |
| **Constants** | `MAX = 10` | `const Max = 10` |
| **Visibility** | `_private_var` | **Lowercase** = Private (`var secret`). |
| | `public_var` | **Uppercase** = Public (`var Exported`). |

### Loops

Go has no `while`. It only has `for`.

**Standard Loop:**

```go
items := []string{"a", "b"}
for i, v := range items {
    fmt.Println(i, v) // i=index, v=value
}
```

**While Loop equivalent:**

```go
count := 0
for count < 5 {
    count++
}
```

### Functions

Go functions accept types and return types. They can return multiple values.

```go
//                Args      Return types
func calculate(a, b int) (int, int) {
    sum := a + b
    diff := a - b
    return sum, diff
}

// Usage
s, d := calculate(10, 5)
```

### Error Handling (The Big Shift)

**Python:** Try / Except (Jump logic).
**Go:** Check values immediately (Linear logic).

```go
// Typical Go Pattern
content, err := os.ReadFile("config.txt")

if err != nil {
    // Handle error immediately
    log.Fatal("Could not read file:", err)
}

// If we get here, 'content' is safe to use
fmt.Println(string(content))
```

-----

## 5\. Structs & Interfaces (OOP Replacement)

Go does not have `class`. It separates Data (Structs) from Behavior (Methods).

### Structs (The Data)

```go
type User struct {
    Name string
    Age  int
}

// Initialization
u := User{Name: "Ryan", Age: 30}
```

### Methods (The Logic)

You attach functions to structs using a "Receiver".

  * `(u User)` = Copy (Read-only).
  * `(u *User)` = Pointer (Read/Write).

<!-- end list -->

```go
// Pointer receiver allows modifying the struct
func (u *User) HaveBirthday() {
    u.Age++
}
```

### Interfaces (Polymorphism)

Go uses implicit interfaces. You don't say `implements`. If you have the methods, you fit the interface.

```go
// The Contract
type Speaker interface {
    Speak() string
}

type Dog struct {}
func (d Dog) Speak() string { return "Woof" }

type Cat struct {}
func (c Cat) Speak() string { return "Meow" }

// Accepts anything that can Speak()
func MakeNoise(s Speaker) {
    fmt.Println(s.Speak())
}
```

-----

## 6\. Concurrency: Goroutines & Channels

**Python (`asyncio`):** Single-threaded event loop. You must `await`. One heavy task blocks everything.
**Go (`goroutines`):** Multi-threaded scheduler. Fire and forget. No `await`.

### 1\. Goroutines

Launch a lightweight thread instantly.

```go
func work() { fmt.Println("Working...") }

func main() {
    go work() // Runs in background
    time.Sleep(time.Second) // Wait for it (naive approach)
}
```

### 2\. Channels

Channels are typed pipes to communicate between routines without locks.

```go
func main() {
    // Create a channel that carries strings (buffer size 2)
    messages := make(chan string, 2)

    // Anonymous function running in background
    go func() {
        messages <- "Ping" // Send data
        messages <- "Pong"
    }()

    // Receive data (Blocks until data arrives)
    msg1 := <-messages
    msg2 := <-messages
    fmt.Println(msg1, msg2)
}
```

-----

## 7\. File Paths & I/O (`pathlib` equivalent)

**The Mental Shift:**

  * **Python (`pathlib`):** You create a `Path` **Object**. You call methods on it (`p.joinpath()`, `p.name`).
  * **Go (`path/filepath`):** You manipulate **Strings**. You pass strings into helper functions.

**Crucial Warning:** Go has two packages: `path` and `path/filepath`.

  * `path` = For URLs or virtual paths (always uses `/`).
  * `path/filepath` = For OS file systems (handles `\` on Windows and `/` on Mac/Linux). **Always use this for files.**

### Common Operations Cheat Sheet

| Task | Python (`pathlib`) | Go (`path/filepath` + `os`) |
| :--- | :--- | :--- |
| **Import** | `from pathlib import Path` | `import "path/filepath"` <br> `import "os"` |
| **Join Paths** | `p = Path("data") / "file.txt"` | `p := filepath.Join("data", "file.txt")` |
| **Get Filename** | `p.name` | `filepath.Base(p)` |
| **Get Directory** | `p.parent` | `filepath.Dir(p)` |
| **Get Extension** | `p.suffix` | `filepath.Ext(p)` |
| **Absolute Path** | `p.resolve()` | `abs, err := filepath.Abs(p)` |
| **Read File** | `p.read_text()` | `data, err := os.ReadFile(p)` |
| **Write File** | `p.write_text("content")` | `err := os.WriteFile(p, []byte("content"), 0644)` |

### The "Check if Exists" Pattern

This is the most jarring difference. Go does not have a simple `.exists()` boolean helper because checking existence is an OS system call that can fail (permissions, disk errors).

**Python:**

```python
if Path("config.json").exists():
    print("Found it")
```

**Go:**

```go
// We try to get stats on the file
if _, err := os.Stat("config.json"); err == nil {
    fmt.Println("Found it")
} else if os.IsNotExist(err) {
    fmt.Println("Does not exist")
} else {
    // Some other error (e.g., permission denied)
    fmt.Println("Error checking file:", err)
}
```

### Full Example: Scanning a Directory (Globbing)

How to find all `.json` files in a folder.

**Python:**

```python
for f in Path("./data").glob("*.json"):
    print(f.name)
```

**Go:**

```go
package main

import (
	"fmt"
	"path/filepath"
)

func main() {
    // Glob returns a slice of strings (paths) matches
    // pattern: "./data/*.json"
    matches, err := filepath.Glob(filepath.Join("data", "*.json"))
    
    if err != nil {
        fmt.Println(err)
        return
    }

    for _, match := range matches {
        fmt.Println("Found:", match)
        fmt.Println("Just name:", filepath.Base(match))
    }
}
```

----------

## 8\. Best Practices / Pro-Tips

1.  **Format on Save:** Ensure your editor runs `go fmt` (or `gopls`) on save. Go code has one standard style.
2.  **Linting:** Install `golangci-lint`. It is the definitive linter.
3.  **Tidy Often:** Run `go mod tidy` whenever you delete imports or change dependencies.
4.  **Embrace the `if err != nil`:** Do not try to create "helper functions" to hide error handling. Go code is verbose by design to make control flow obvious.
5.  **Use `air` for Hot Reloading:**
    `go install github.com/air-verse/air@latest`
    Run `air` in your terminal to auto-restart your app on file save.
