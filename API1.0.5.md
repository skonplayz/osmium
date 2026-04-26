# Osmium API Reference

This document describes the current Osmium runtime, syntax, and built-in extension APIs.

## 1. Language Overview

Osmium is a device-based scripting runtime for `.osd` files.

Core ideas:

- `onStart { ... }` is the only entry point.
- `device Name { on run { ... } }` defines reusable singleton logic.
- `Name.run()` explicitly executes a device.
- `use filesys` exposes `run("./other.osd")` for nested file execution.
- `use cli` exposes terminal output helpers.
- `cli.run("ipconfig")` can execute host commands from the CLI extension.
- `use math` exposes trigonometry, angle conversion, rounding, and random helpers.
- `use string` exposes string helpers for length and casing.
- `use colors` can load a repo-backed extension folder that exposes ANSI color helpers.
- `settings.toml` stores project metadata like name, version, entry file, and terminal visibility.
- Variables are dynamically typed and use plain assignment.
- Arrays use `[...]`.
- Objects use `{ key: value }`.

## 2. File Structure

Typical file:

```julia
use filesys
use cli

dollars = 4

device Counter {
    on run {
        cli.print("counter running")
    }
}

onStart {
    run("./inventory.osd")
    Counter.run()
    dollars = (dollars + 1)
    cli.print("You have", dollars, "dollars.")
}
```

Loop examples:

```julia
while 4 = 4 do {
    break()
}

repeat(5) {
    cli.print("tick")
}

while condition do {
    cli.print("looping")
} else {
    cli.print("finished")
}

if condition then {
    cli.print("yes")
} else {
    cli.print("no")
}
```

Rules:

- Braces are required for blocks.
- Newlines separate statements.
- `#` begins a line comment.
- `let` and `var` are not used in v1.
- `continue()` skips to the next loop iteration.

## 3. Indentation

Indentation is not syntax-sensitive, but the recommended style is:

- 4 spaces per block level
- no tabs
- align nested statements under their parent block

Example:

```julia
onStart {
    cli.print("Level 1")
    if true then {
        cli.print("Level 2")
    }
}
```

## 4. Variables

Variables are created by assignment:

```julia
dollars = 4
```

Reassignment updates the current module variable:

```julia
dollars = (dollars + 1)
```

Rules:

- Variables are dynamically typed.
- Assignment creates the variable if it does not already exist.
- Reassignment updates the existing variable in the current execution context.
- Variables live in the current module unless exported later with a future feature.

- Top-level assignments must be plain value literals only.
- Calls, expressions, and runtime operations are not allowed at the top level.
- Use onStart to assign variables from calls or computed values.

### CORRECT
```julia
name = ""
onStart {
    name = cli.input("What is your name? ")
}
```

### WRONG - will raise OSM200 ResolveError
```julia
name = cli.input("What is your name? ")
```

## 5. Expressions

Supported expression forms:

- literals: strings, numbers, booleans, `null`
- identifiers
- parenthesized expressions
- arrays: `[1, 2, 3]`
- objects: `{ health: 100, name: "Hero" }`
- indexing: `items[0]`
- arithmetic: `+ - * /`
- comparison: `== != < > <= >=`
- boolean: `and`, `or`, `not`
- member access: `value.name`
- calls: `thing(arg1, arg2)`

Important rule:

- Text inside quotes is always a string literal.
- A bare name inside a call, like `cli.print(dollars)`, resolves as a variable or imported symbol.
- Parentheses are for expression grouping and function calls, not for turning text into a command.

Examples:

```julia
(dollars + 1)
(5 * 2)
(dollars - tax)
cli.print(dollars)
cli.print("dollars")
enemies[0]
player.health
```

## 6. Execution Rules

- `onStart` is the only script entry point.
- Devices do not run automatically.
- `device.run()` is always explicit.
- `run("./file.osd")` comes from `filesys`.
- Nested file execution is synchronous.
- Relative paths are resolved from the current script file.
- Nested execution gets its own module context, with no global leakage unless you intentionally share values through exports or the runtime namespace.
- `break()` exits all active loops in the current execution context.
- `continue()` skips the rest of the current loop iteration.

## 7. Devices

Devices are singleton runtime objects.

Example:

```julia
device SundayCheck {
    on run {
        cli.print("Checking...")
    }
}
```

Rules:

- Each device name must be unique per file.
- Devices can be called with `.run()`.
- Device state persists across repeated calls in the same runtime.
- `this` inside a device points to the device state object.

## 8. Built-in Extensions

### `cli`

Available members:

- `cli.print(...)`
- `cli.edit(...)`
- `cli.clear()`
- `cli.input(prompt)`
- `cli.run(command)`

`cli.print`:

- stringifies all arguments
- joins them with spaces
- writes a newline
- with no arguments, prints a blank line

`cli.edit`:

- replaces the most recent `cli.print(...)` output line
- is meant for loading indicators, progress text, and live updates
- requires that a previous print has already happened

`cli.clear`:

- clears the terminal display
- resets the last printed line tracking

`cli.input`:

- writes the prompt to the terminal
- returns the text typed by the user
- is synchronous in v1

`cli.run`:

- executes a host shell command synchronously
- streams the command output through the runtime
- returns the command output as text

Example:

```julia
cli.print("Loading")
cli.edit("Loaded")
```

### `filesys`

Available members:

- `run(path)`
- `read(path)`
- `write(path, data)`
- `list(path)`
- `exists(path)`

Current v1 behavior:

- `run(path)` executes another `.osd` file synchronously.
- `run(path)` resolves relative paths against the current script file.
- `read(path)` returns the UTF-8 text contents of a file.
- `write(path, data)` writes UTF-8 text to a file and creates parent folders as needed.
- `list(path)` and `exists(path)` remain placeholders for now.

### `time`

Available members:

- `time.dayOfWeek`
- `time.hour`
- `time.now()`
- `time.wait(seconds)`

Current v1 behavior:

- `dayOfWeek` and `hour` are deterministic placeholder values.
- `now()` is a placeholder API.
- `wait(seconds)` blocks synchronously for the requested number of seconds.

Example:

```julia
time.wait(5)
```

### `string`

Available members:

- `string.length(value)`
- `string.upper(value)`
- `string.lower(value)`
- `string.contains(value, search)`
- `string.split(value, separator)`
- `string.trim(value)`

Examples:

```julia
cli.print(string.length("Hero"))
cli.print(string.upper("hello"))
cli.print(string.contains("inventory", "vent"))
cli.print(string.split("red,green", ",")[0])
cli.print(string.trim("  hello  "))
```

### `math`

Available members:

- `math.random(min, max)`
- `math.sin(angle)`
- `math.cos(angle)`
- `math.rad(degrees)`
- `math.floor(value)`
- `math.ceil(value)`
- `math.round(value)`
- `math.abs(value)`
- `math.min(a, b, ...)`
- `math.max(a, b, ...)`
- `math.clamp(value, min, max)`
- `math.pi`

Notes:

- `math.sin` and `math.cos` expect radians.
- `math.rad` converts degrees to radians.
- `math.random` returns a floating-point number between `min` and `max`.
- `math.floor` and `math.ceil` behave like standard rounding helpers.
- `math.round` rounds to the nearest integer.
- `math.abs`, `math.min`, `math.max`, and `math.clamp` follow standard math behavior.

Example:

```julia
use math

onStart {
    cli.print(math.floor(1.9))
    cli.print(math.ceil(1.1))
    cli.print(math.rad(180))
}
```

### `tdefinition`

Current behavior:

- `tdefinition(...) { ... }` acts as a context wrapper.
- It is intended for terminal/execution-context shaping.
- It is still extension-layer driven and not a full terminal API.
- The wrapper is decorate-only in v1 and does not execute logic by itself.

## 9. Scope Rules

- Top-level assignments only accept plain literals (strings, numbers, booleans, null).
- Calling functions or extensions at the top level raises OSM200 ResolveError: Calls are not allowed in this context.
- To assign a variable from a call result, declare it as a plain value at the top level, then reassign it inside onStart.
- Top-level assignments create module variables.
- Imported modules are available by their `use` alias.
- Exported device names are available from the module namespace.
- Exported variables can be declared with `export name = value` or `name = value` followed by `export name`.
- Bare `run()` is only available if `use filesys` is present.
- Devices and variables share the same module namespace for lookup purposes.
- Imported modules expose both the module object and the exported symbols.

## 10. Module Loading

`use name` resolution order:

1. local `.osd` module in the current project tree
2. official repo-backed extension folder
3. local extension in `extensions/`
4. built-in extension

This means `use helper` can load `helper.osd`, while `use colors` can load a folder-based extension package such as `extensions/colors/index.js`.

Extension repositories:

- The shipped runtime has the official extension repo URL and branch baked in.
- `use name` first checks local `.osd` modules, then the local `extensions/` folder, then the official repo-backed extension packages.
- Each top-level folder in the official extension repo is treated as an extension package.
- Extension folders should export `index.js` at the folder root.

Folder layout in the official repo:

```text
colors/index.js
mem/index.js
```

Project settings:

```toml
name = "demo-project"
version = "0.1.0"
entry = "master.osd"
terminal-visibility = true
```

`terminal-visibility` defaults to `true` and is used by the build command when creating a release launcher.

## 11. Errors

Error classes:

- parse errors
- resolve errors
- runtime errors

Behavior:

- execution stops on the first error
- CLI exit codes are `0`, `1`, and `2`
- missing APIs fail clearly

## 12. Example Project

Project manifest:

```toml
name = "demo-project"
version = "0.1.0"
entry = "master.osd"
terminal-visibility = true
```

Run it:

```bash
osmium run demo-project
```

Build it:

```bash
osmium build demo-project
```

Register the Windows context menu:

```bash
osmium register
```

## 13. Notes

- Official filesystem and terminal APIs are intentionally conservative in v1.
- Community extensions are loaded through the extension system, not syntax changes.
- ANSI color helpers are provided by the `colors` extension example.
- This document should be updated whenever language behavior changes.
