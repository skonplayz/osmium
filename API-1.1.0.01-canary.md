# Osmium API Reference for 1.1.0.01-canary

This document describes the 1.1.0.01-canary Osmium runtime, syntax, and built-in extension APIs.

## 1. Language Overview

Osmium is a device-based scripting runtime for `.osd` files.

Core ideas:

- `onStart { ... }` is the only entry point.
- `device Name { on run { ... } }` defines reusable singleton logic.
- `Name.run()` explicitly executes a device.
- `fn name(params) { ... }` defines reusable functions with parameters and return values.
- `Name()` calls a function.
- `use filesys` exposes `run("./other.osd")` for nested file execution.
- `use cli` exposes terminal output helpers.
- `cli.run("ipconfig")` can execute host commands from the CLI extension.
- `use math` exposes trigonometry, angle conversion, rounding, and random helpers.
- `use string` exposes string helpers for length and casing.
- `use colors` can load a repo-backed extension folder that exposes ANSI color helpers.
- `settings.toml` stores project configuration including window appearance, app identity, runtime behavior, build settings, and permissions. See Section 12 for full details.
- Variables are dynamically typed and use plain assignment.
- Arrays use `[...]`.
- Objects use `{ key: value }`.

## 1.1 Reserved Words

The following words are reserved and cannot be used as variable names:

- `use`
- `device`
- `on`
- `onStart`
- `if`
- `then`
- `else`
- `while`
- `do`
- `repeat`
- `for`
- `in`
- `try`
- `catch`
- `return`
- `continue`
- `true`
- `false`
- `null`
- `and`
- `or`
- `not`
- `as`
- `export`
- `this`
- `fn`
- `delete`
- `input` (reserved for cli.input)

## 2. File Structure

Typical file:

```osd
use filesys
use cli

dollars = 4

fn add(a, b) {
    return (a + b)
}

device Counter {
    on run {
        cli.print("counter running")
    }
}

onStart {
    run("./inventory.osd")
    Counter.run()
    dollars = add(dollars, 1)
    cli.print("You have {dollars} dollars.")
}
```

You can also use brace notation to import multiple extensions in a single line:

```osd
use { cli, time, filesys, colors }

dollars = 4

onStart {
    cli.print("Hello from {dollars} dollars.")
}
```

Both syntaxes are valid and can be mixed in the same file:

```osd
use { cli, time }
use math
```

The brace notation is purely for code organization and readability. The rules for brace notation:
- Space before closing brace and after opening brace: `{ cli, time, filesys }`
- Items must be comma-separated
- Last item has no comma, just a space before the closing brace
- Aliases are not supported in brace notation (use single-line `use` with `as` for aliases)

Multi-file example:

`main.osd`:
```osd
use { cli, time, filesys }

onStart {
    cli.print("Starting main module")
    run("./helper.osd")
    cli.print("Helper module finished")
    cli.print("Total time: {time.now()}")
    cli.print("Hello World")
    cli.edit("Loading")
    cli.clear()
    cli.warn("Low disk space")
    cli.error("Connection failed")
    cli.success("Build complete")
    name = cli.prompt("Enter your name", "Guest")
    confirmed = cli.confirm("Continue?")
}
```

`helper.osd`:
```osd
use cli

onStart {
    cli.print("Helper module running")
    cli.print("Performing some work...")
}
```

The module system allows you to split your code into multiple `.osd` files and execute them using `run("./path/to/file.osd")`. This is useful for organizing large projects and reusing common functionality.

Loop examples:

```osd
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

if condition then {
    cli.print("yes")
} else if otherCondition then {
    cli.print("maybe")
} else {
    cli.print("no")
}

for item in array do {
    cli.print(item)
}

try {
    riskyOperation()
} catch error {
    cli.print("Error:", error)
}

delete object.key
delete array[0]
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

```osd
onStart {
    cli.print("Level 1")
    if true then {
        cli.print("Level 2")
    }
}
```

## 4. Variables

Variables are created by assignment:

```osd
dollars = 4
```

Reassignment updates the current module variable:

```osd
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
- string interpolation: `"Hello {name}"`
- array length: `length(array)` or `array.length()`
- object deletion: `delete object.key` or `delete array[index]`
- array concatenation: `array + [item]` or `array1 + array2`
- array push: `push(array, item)` adds an item to the end of an array
- array pop: `pop(array)` removes and returns the last item from an array
- array first: `first(array)` returns the first item of an array
- array last: `last(array)` returns the last item of an array
- array includes: `includes(array, item)` checks if an array contains a value
- array reverse: `reverse(array)` reverses an array in-place
- array join: `join(array, separator)` joins array elements into a string
- array slice: `slice(array, start, end)` returns a sub-array
- array indexOf: `indexOf(array, item)` returns the index of an item
- array isEmpty: `isEmpty(array)` checks if an array is empty
- compound assignment: `+=`, `-=`, `*=`, `/=` for shorthand math assignment
- increment/decrement: `++` and `--` for prefix and postfix increment/decrement
- modulo: `%` for remainder operation
- ternary: `condition ? value_if_true : value_if_false` for conditional expressions
- multiline strings: `"""text"""` for strings spanning multiple lines

Important rule:

- Text inside quotes is always a string literal.
- String interpolation uses `{}` to embed expressions: `"Hello {name}"` evaluates `name` and inserts it.
- Triple-quoted strings preserve newlines: `"""line1\nline2"""` is a literal multiline string.
- A bare name inside a call, like `cli.print(dollars)`, resolves as a variable or imported symbol.
- Parentheses are for expression grouping and function calls, not for turning text into a command.
- `length(value)` returns the length of strings, arrays, and objects.
- `delete object.key` removes a property from an object.
- `delete array[index]` removes an element from an array.
- Arrays can be concatenated using the `+` operator: `items + [newItem]` creates a new array with the item appended.
- `push(array, item)` modifies the array in-place and returns it.
- `pop(array)` modifies the array in-place and returns the removed item.
- Compound assignment operators modify the variable in-place: `x += 1` is equivalent to `x = (x + 1)`.
- `++x` increments x and returns the new value; `x++` returns the old value then increments.
- `--x` decrements x and returns the new value; `x--` returns the old value then decrements.
- `%` returns the remainder of division: `10 % 3` returns `1`.
- Ternary expressions: `x > 0 ? "positive" : "negative"` returns `"positive"` if x > 0, else `"negative"`.

Examples:

```osd
(dollars + 1)
(5 * 2)
(dollars - tax)
cli.print(dollars)
cli.print("dollars")
cli.print("Hello {name}")
enemies[0]
player.health
len = length(items)
len = items.length()
delete player.health
delete items[0]
items = items + [newItem]
combined = list1 + list2
items = push(items, newItem)
last = pop(items)
x += 1
x -= 5
x *= 2
x /= 3
count++
result = count++
total = (x > 10 ? "high" : "low")
remainder = 10 % 3
multiline = """This is a
multiline string"""
firstItem = first(items)
lastItem = last(items)
hasValue = includes(items, "value")
items = reverse(items)
joined = join(items, ", ")
sub = slice(items, 0, 3)
index = indexOf(items, "value")
empty = isEmpty(items)
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

```osd
device SundayCheck {
    on run {
        cli.print("Checking...")
    }
}
```

Rules:

- Each device name must be unique per file.
- Devices can be called with `.run()` or `.runAsync()` for non-blocking execution.
- Device state persists across repeated calls in the same runtime.
- `this` inside a device points to the device state object.

## 7.1 Functions

Functions are reusable blocks of code with parameters and return values.

Functions can be defined at the top level of a module:

```osd
fn add(a, b) {
    return (a + b)
}

fn greet(name) {
    return "Hello, " + name
}

export fn multiply(x, y) {
    return (x * y)
}

onStart {
    result = add(5, 3)
    message = greet("World")
    product = multiply(4, 7)
    cli.print("{result}, {message}, {product}")
}
```

Rules:

- Functions are defined with `fn name(params) { ... }`.
- Functions can be defined at the top level of a module (outside of any block).
- Functions can be exported with `export fn name(params) { ... }`.
- Functions are called with `name(arg1, arg2)`.
- `return` exits the function and returns a value.
- Functions do not persist state between calls (unlike devices).
- Functions can be called from anywhere in the module.

## 8. Built-in Extensions

### `cli`

Available members:

- `cli.print(...)`
- `cli.edit(...)`
- `cli.clear()`
- `cli.input(prompt)`
- `cli.run(command)`
- `cli.warn(...)`
- `cli.error(...)`
- `cli.success(...)`
- `cli.prompt(question, default)`
- `cli.confirm(question)`

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

```osd
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
- `delete(path)`
- `readJSON(path)`
- `writeJSON(path, data)`
- `append(path, data)`
- `copy(src, dest)`
- `move(src, dest)`
- `extension(path)`
- `filename(path)`
- `dirname(path)`

Current v1 behavior:

- `run(path)` executes another `.osd` file synchronously.
- `run(path)` resolves relative paths against the current script file.
- `read(path)` returns the UTF-8 text contents of a file.
- `write(path, data)` writes UTF-8 text to a file and creates parent folders as needed.
- `list(path)` returns an array of file/directory names in the given directory.
- `exists(path)` returns true if the path exists, false otherwise.
- `delete(path)` deletes a file or directory. Directories are deleted recursively.
- `readJSON(path)` reads and parses a JSON file, returning the parsed object.
- `writeJSON(path, data)` writes data as formatted JSON to a file.
- `append(path, data)` appends text to a file (creates if it doesn't exist).
- `copy(src, dest)` copies a file from src to dest.
- `move(src, dest)` moves/renames a file from src to dest.
- `extension(path)` returns the file extension (e.g., ".osd").
- `filename(path)` returns the filename from a path (e.g., "file.osd").
- `dirname(path)` returns the directory portion of a path.

### `time`

Available members:

- `time.dayOfWeek`
- `time.hour`
- `time.now()`
- `time.wait(seconds)`
- `time.format(timestamp, pattern)`
- `time.date(pattern)`
- `time.timestamp()`
- `time.sleep(ms)`

Current v1 behavior:

- `dayOfWeek` returns the current day of week (0-6, Sunday is 0).
- `hour` returns the current hour (0-23).
- `now()` returns the current Unix timestamp in seconds.
- `wait(seconds)` blocks synchronously for the requested number of seconds.
- `format(timestamp, pattern)` formats a Unix timestamp as a string. Patterns: `iso`, `date`, `time`, `compact` (default: compact).
- `date(pattern)` returns today's date formatted with the given pattern (default: date).
- `timestamp()` is an alias for `now()`, returning the current Unix timestamp in seconds.
- `sleep(ms)` blocks synchronously for the requested number of milliseconds.

Example:

```osd
timestamp = time.now()
cli.print("Current time:", timestamp)
formatted = time.format(timestamp, "compact")
cli.print("Formatted:", formatted)
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
- `string.replace(value, search, replacement)`
- `string.startsWith(value, prefix)`
- `string.endsWith(value, suffix)`
- `string.isEmpty(value)`
- `string.repeat(value, n)`
- `string.pad(value, length, char)`
- `string.reverse(value)`
- `string.toNumber(value)`
- `string.isNumber(value)`

Examples:

```osd
cli.print(string.length("Hero"))
cli.print(string.upper("hello"))
cli.print(string.contains("inventory", "vent"))
cli.print(string.split("red,green", ",")[0])
cli.print(string.trim("  hello  "))
cli.print(string.replace("Hello World", "World", "Osmium"))
cli.print(string.startsWith("filename.osd", "file"))
cli.print(string.endsWith("filename.osd", ".osd"))
cli.print(string.isEmpty(""))
cli.print(string.repeat("=", 10))
cli.print(string.pad("5", 3, "0"))
cli.print(string.reverse("hello"))
num = string.toNumber("42")
cli.print(string.isNumber("123"))
```

### `math`

Available members:

- `math.random(min, max)`
- `math.randomInt(min, max)`
- `math.sin(angle)`
- `math.cos(angle)`
- `math.rad(degrees)`
- `math.deg(radians)`
- `math.floor(value)`
- `math.ceil(value)`
- `math.round(value)`
- `math.abs(value)`
- `math.min(...)`
- `math.max(...)`
- `math.clamp(value, min, max)`
- `math.pi`
- `math.even(n)`
- `math.odd(n)`
- `math.percent(value, total)`
- `math.pow(base, exp)`
- `math.sqrt(value)`

Notes:

- `math.sin` and `math.cos` expect radians.
- `math.rad` converts degrees to radians.
- `math.random` returns a floating-point number between `min` and `max`.
- `math.randomInt` returns a random integer between `min` and `max` (inclusive).
- `math.floor` and `math.ceil` behave like standard rounding helpers.
- `math.round` rounds to the nearest integer.
- `math.abs`, `math.min`, `math.max`, and `math.clamp` follow standard math behavior.
- `math.even(n)` returns true if n is even, false otherwise.
- `math.odd(n)` returns true if n is odd, false otherwise.
- `math.percent(value, total)` returns (value / total) * 100.
- `math.pow(base, exp)` returns base raised to the power of exp.
- `math.sqrt(value)` returns the square root of value (must be non-negative).

Example:

```osd
use math

onStart {
    cli.print(math.floor(1.9))
    cli.print(math.ceil(1.1))
    cli.print(math.rad(180))
    cli.print(math.randomInt(1, 10))
    cli.print(math.even(4))
    cli.print(math.odd(5))
    cli.print(math.percent(25, 100))
    cli.print(math.pow(2, 3))
    cli.print(math.sqrt(16))
}
```
}
```

### `tdefinition`

Current behavior:

- `tdefinition(...) { ... }` acts as a context wrapper.
- It is intended for terminal/execution-context shaping.
- It is still extension-layer driven and not a full terminal API.
- The wrapper is decorate-only in v1 and does not execute logic by itself.

### `net`

Available members:

- `net.get(url)`
- `net.listen(port)`

Current v1 behavior:

- `net.get(url)` makes an HTTP GET request and returns the response body as a string.
- `net.listen(port)` starts an HTTP server on the specified port.

Example:

```osd
use net

onStart {
    response = net.get("https://example.com")
    cli.print(response)
}
```

### `json`

Available members:

- `json.parse(text)`
- `json.stringify(value)`

Current v1 behavior:

- `json.parse(text)` parses a JSON string and returns the corresponding object/array.
- `json.stringify(value)` converts a value to a JSON string.

Example:

```osd
use json

onStart {
    data = json.parse('{"name": "test"}')
    text = json.stringify(data)
    cli.print(text)
}
```

### `mem`

Available members:

- `mem.alloc(size)`
- `mem.from(data)`
- `mem.toString(buffer)`
- `mem.get(buffer, index)`
- `mem.set(buffer, index, value)`
- `mem.length(buffer)`

Current v1 behavior:

- `mem.alloc(size)` creates a buffer of the specified size.
- `mem.from(data)` creates a buffer from an array or string.
- `mem.toString(buffer)` converts a buffer to a UTF-8 string.
- `mem.get(buffer, index)` reads a byte from the buffer at the specified index.
- `mem.set(buffer, index, value)` writes a byte (0-255) to the buffer at the specified index.
- `mem.length(buffer)` returns the length of the buffer.

Example:

```osd
use mem

onStart {
    buf = mem.alloc(10)
    mem.set(buf, 0, 65)
    value = mem.get(buf, 0)
    cli.print(value)
}
```

### `sqlite`

Available members:

- `sqlite.open(path)`
- `sqlite.exec(db, sql)`
- `sqlite.query(db, sql, params)`
- `sqlite.close(db)`

Current v1 behavior:

- `sqlite.open(path)` opens a SQLite database file and returns a database handle.
- `sqlite.exec(db, sql)` executes a SQL statement without returning results.
- `sqlite.query(db, sql, params)` executes a parameterized query and returns results as an array of objects.
- `sqlite.close(db)` closes the database connection.

Example:

```osd
use sqlite

onStart {
    db = sqlite.open("test.db")
    sqlite.exec(db, "CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)")
    sqlite.exec(db, "INSERT INTO users (name) VALUES ('Alice')")
    results = sqlite.query(db, "SELECT * FROM users")
    cli.print(results)
    sqlite.close(db)
}
```

### `tui`

Available members:

- `tui.table(data, headers)`
- `tui.progress(value, total)`
- `tui.spinner(message)`

Current v1 behavior:

- `tui.table(data, headers)` displays a formatted table in the terminal.
- `tui.progress(value, total)` displays a progress bar.
- `tui.spinner(message)` displays a loading spinner with a message.

Example:

```osd
use tui

onStart {
    data = [
        { name: "Alice", age: 30 },
        { name: "Bob", age: 25 }
    ]
    tui.table(data, ["Name", "Age"])
    tui.progress(50, 100)
}
```

### `test`

Available members:

- `test.assert(condition, message)`
- `test.assertEqual(actual, expected, message)`
- `test.assertTrue(value, message)`
- `test.assertFalse(value, message)`

Current v1 behavior:

- `test.assert(condition, message)` asserts that a condition is true.
- `test.assertEqual(actual, expected, message)` asserts that two values are equal.
- `test.assertTrue(value, message)` asserts that a value is truthy.
- `test.assertFalse(value, message)` asserts that a value is falsy.

Example:

```osd
use test

onStart {
    test.assertEqual(1 + 1, 2, "Addition works")
    test.assertTrue(true, "True is truthy")
    cli.print("All tests passed")
}
```

### `git`

Available members:

- `git.init(path)`
- `git.clone(url, path)`
- `git.commit(path, message)`
- `git.push(path)`
- `git.pull(path)`
- `git.status(path)`

Current v1 behavior:

- `git.init(path)` initializes a git repository at the specified path.
- `git.clone(url, path)` clones a repository from URL to path.
- `git.commit(path, message)` creates a commit with the given message.
- `git.push(path)` pushes changes to the remote repository.
- `git.pull(path)` pulls changes from the remote repository.
- `git.status(path)` returns the git status of the repository.

Example:

```osd
use git

onStart {
    git.init("./myproject")
    git.commit("./myproject", "Initial commit")
    status = git.status("./myproject")
    cli.print(status)
}
```

### `xml`

Available members:

- `xml.parse(text)`
- `xml.stringify(obj)`

Current v1 behavior:

- `xml.parse(text)` parses an XML string and returns the corresponding object.
- `xml.stringify(obj)` converts an object to an XML string.

Example:

```osd
use xml

onStart {
    xmlText = "<root><item>test</item></root>"
    obj = xml.parse(xmlText)
    cli.print(obj)
}
```

### `yaml`

Available members:

- `yaml.parse(text)`
- `yaml.stringify(obj)`

Current v1 behavior:

- `yaml.parse(text)` parses a YAML string and returns the corresponding object.
- `yaml.stringify(obj)` converts an object to a YAML string.

Example:

```osd
use yaml

onStart {
    yamlText = "name: test\nvalue: 123"
    obj = yaml.parse(yamlText)
    cli.print(obj)
}
```

### `toml`

Available members:

- `toml.parse(text)`
- `toml.stringify(obj)`

Current v1 behavior:

- `toml.parse(text)` parses a TOML string and returns the corresponding object.
- `toml.stringify(obj)` converts an object to a TOML string.

Example:

```osd
use toml

onStart {
    tomlText = "[section]\nkey = \"value\""
    obj = toml.parse(tomlText)
    cli.print(obj)
}
```

### `smtp`

Available members:

- `smtp.send(host, port, from, to, subject, body, user, password)`

Current v1 behavior:

- `smtp.send(host, port, from, to, subject, body, user, password)` sends an email via SMTP.

Example:

```osd
use smtp

onStart {
    smtp.send("smtp.example.com", 587, "from@example.com", "to@example.com", "Test", "Body", "user", "pass")
    cli.print("Email sent")
}
```

### `chart`

Available members:

- `chart.bar(data, labels, outputPath)`
- `chart.line(data, labels, outputPath)`
- `chart.pie(data, labels, outputPath)`

Current v1 behavior:

- `chart.bar(data, labels, outputPath)` creates a bar chart and saves it to the output path.
- `chart.line(data, labels, outputPath)` creates a line chart and saves it to the output path.
- `chart.pie(data, labels, outputPath)` creates a pie chart and saves it to the output path.

Example:

```osd
use chart

onStart {
    data = [10, 20, 30]
    labels = ["A", "B", "C"]
    chart.bar(data, labels, "chart.png")
    cli.print("Chart created")
}
```

### `websocket`

Available members:

- `websocket.connect(url)`
- `websocket.send(ws, message)`
- `websocket.close(ws)`

Current v1 behavior:

- `websocket.connect(url)` connects to a WebSocket server and returns a WebSocket handle.
- `websocket.send(ws, message)` sends a message through the WebSocket.
- `websocket.close(ws)` closes the WebSocket connection.

Example:

```osd
use websocket

onStart {
    ws = websocket.connect("ws://example.com")
    websocket.send(ws, "Hello")
    websocket.close(ws)
}
```

### `cache`

Available members:

- `cache.set(key, value, ttl)`
- `cache.get(key)`
- `cache.delete(key)`
- `cache.clear()`

Current v1 behavior:

- `cache.set(key, value, ttl)` stores a value with an optional time-to-live in seconds.
- `cache.get(key)` retrieves a value from the cache.
- `cache.delete(key)` removes a value from the cache.
- `cache.clear()` clears all cached values.

Example:

```osd
use cache

onStart {
    cache.set("mykey", "myvalue", 60)
    value = cache.get("mykey")
    cli.print(value)
}
```

### `ini`

Available members:

- `ini.parse(text)`
- `ini.stringify(obj)`

Current v1 behavior:

- `ini.parse(text)` parses an INI string and returns the corresponding object.
- `ini.stringify(obj)` converts an object to an INI string.

Example:

```osd
use ini

onStart {
    iniText = "[section]\nkey = value"
    obj = ini.parse(iniText)
    cli.print(obj)
}
```

### `markdown`

Available members:

- `markdown.parse(text)`
- `markdown.render(text)`

Current v1 behavior:

- `markdown.parse(text)` parses markdown text and returns the AST.
- `markdown.render(text)` converts markdown to HTML.

Example:

```osd
use markdown

onStart {
    md = "# Hello\n\nThis is **bold**."
    html = markdown.render(md)
    cli.print(html)
}
```

### `dns`

Available members:

- `dns.lookup(hostname)`
- `dns.resolve(hostname, type)`

Current v1 behavior:

- `dns.lookup(hostname)` resolves a hostname to an IP address.
- `dns.resolve(hostname, type)` resolves a DNS record of the specified type (A, AAAA, MX, TXT, etc.).

Example:

```osd
use dns

onStart {
    ip = dns.lookup("example.com")
    cli.print(ip)
}
```

### `jwt`

Available members:

- `jwt.sign(payload, secret)`
- `jwt.verify(token, secret)`
- `jwt.decode(token)`

Current v1 behavior:

- `jwt.sign(payload, secret)` creates a JWT token from the payload.
- `jwt.verify(token, secret)` verifies a JWT token and returns the payload.
- `jwt.decode(token)` decodes a JWT token without verification.

Example:

```osd
use jwt

onStart {
    token = jwt.sign({ user: "alice" }, "secret")
    payload = jwt.verify(token, "secret")
    cli.print(payload)
}
```

### `bcrypt`

Available members:

- `bcrypt.hash(password, rounds)`
- `bcrypt.compare(password, hash)`

Current v1 behavior:

- `bcrypt.hash(password, rounds)` hashes a password using bcrypt.
- `bcrypt.compare(password, hash)` compares a password against a hash.

Example:

```osd
use bcrypt

onStart {
    hash = bcrypt.hash("mypassword", 10)
    match = bcrypt.compare("mypassword", hash)
    cli.print(match)
}
```

### `socket`

Available members:

- `socket.createServer(port, handler)`
- `socket.connect(host, port)`

Current v1 behavior:

- `socket.createServer(port, handler)` creates a TCP server on the specified port.
- `socket.connect(host, port)` connects to a TCP server.

Example:

```osd
use socket

onStart {
    socket.createServer(8080, (conn) => {
        cli.print("Client connected")
    })
}
```

### `power`

Available members:

- `power.shutdown()`
- `power.restart()`
- `power.sleep()`

Current v1 behavior:

- `power.shutdown()` initiates system shutdown.
- `power.restart()` initiates system restart.
- `power.sleep()` puts the system to sleep.

Example:

```osd
use power

onStart {
    power.shutdown()
}
```

### `registry`

Available members:

- `registry.get(key, valueName)`
- `registry.set(key, valueName, value, type)`

Current v1 behavior:

- `registry.get(key, valueName)` reads a value from the Windows registry.
- `registry.set(key, valueName, value, type)` writes a value to the Windows registry.
- This extension is only available on Windows.

Example:

```osd
use registry

onStart {
    value = registry.get("HKLM\\Software\\MyApp", "Version")
    cli.print(value)
}
```

### `service`

Available members:

- `service.start(name)`
- `service.stop(name)`
- `service.status(name)`

Current v1 behavior:

- `service.start(name)` starts a system service.
- `service.stop(name)` stops a system service.
- `service.status(name)` returns the status of a system service.

Example:

```osd
use service

onStart {
    service.start("nginx")
    status = service.status("nginx")
    cli.print(status)
}
```

### `display`

Available members:

- `display.brightness(level)`

Current v1 behavior:

- `display.brightness(level)` sets the display brightness (0-100).

Example:

```osd
use display

onStart {
    display.brightness(50)
}
```

### `input`

Available members:

- `input.read(prompt)`
- `input.password(prompt)`

Current v1 behavior:

- `input.read(prompt)` reads user input from the terminal.
- `input.password(prompt)` reads a password (hidden input).

Example:

```osd
use input

onStart {
    name = input.read("Enter your name: ")
    cli.print("Hello, {name}")
}
```

### `ftp`

Available members:

- `ftp.upload(host, user, password, localFile, remotePath)`
- `ftp.download(host, user, password, remotePath, localFile)`

Current v1 behavior:

- `ftp.upload(host, user, password, localFile, remotePath)` uploads a file to an FTP server.
- `ftp.download(host, user, password, remotePath, localFile)` downloads a file from an FTP server.

Example:

```osd
use ftp

onStart {
    ftp.upload("ftp.example.com", "user", "pass", "local.txt", "/remote.txt")
    cli.print("Upload complete")
}
```

### `ssh`

Available members:

- `ssh.exec(host, command, user)`

Current v1 behavior:

- `ssh.exec(host, command, user)` executes a command on a remote server via SSH.

Example:

```osd
use ssh

onStart {
    output = ssh.exec("example.com", "ls -la", "user")
    cli.print(output)
}
```

### `proxy`

Available members:

- `proxy.request(url, proxyUrl)`

Current v1 behavior:

- `proxy.request(url, proxyUrl)` makes an HTTP request through a proxy server.

Example:

```osd
use proxy

onStart {
    response = proxy.request("https://example.com", "http://proxy.example.com:8080")
    cli.print(response)
}
```

### `tls`

Available members:

- `tls.certInfo(host, port)`

Current v1 behavior:

- `tls.certInfo(host, port)` retrieves TLS certificate information for a host.

Example:

```osd
use tls

onStart {
    cert = tls.certInfo("example.com", 443)
    cli.print(cert)
}
```

### `vault`

Available members:

- `vault.encrypt(data, password)`
- `vault.decrypt(encrypted, password)`

Current v1 behavior:

- `vault.encrypt(data, password)` encrypts data using AES-256-CBC.
- `vault.decrypt(encrypted, password)` decrypts data encrypted with vault.encrypt.

Example:

```osd
use vault

onStart {
    encrypted = vault.encrypt("secret data", "password")
    decrypted = vault.decrypt(encrypted, "password")
    cli.print(decrypted)
}
```

### `image`

Available members:

- `image.resize(inputPath, outputPath, width, height)`
- `image.info(path)`

Current v1 behavior:

- `image.resize(inputPath, outputPath, width, height)` resizes an image.
- `image.info(path)` returns metadata about an image.

Example:

```osd
use image

onStart {
    image.resize("input.jpg", "output.jpg", 800, 600)
    info = image.info("input.jpg")
    cli.print(info)
}
```

### `pdf`

Available members:

- `pdf.text(path)`
- `pdf.info(path)`

Current v1 behavior:

- `pdf.text(path)` extracts text from a PDF file.
- `pdf.info(path)` returns metadata about a PDF file.

Example:

```osd
use pdf

onStart {
    text = pdf.text("document.pdf")
    cli.print(text)
}
```

### `qr`

Available members:

- `qr.generate(text, outputPath)`

Current v1 behavior:

- `qr.generate(text, outputPath)` generates a QR code from text.

Example:

```osd
use qr

onStart {
    qr.generate("https://example.com", "qrcode.png")
    cli.print("QR code generated")
}
```

### `barcode`

Available members:

- `barcode.generate(text, outputPath)`

Current v1 behavior:

- `barcode.generate(text, outputPath)` generates a barcode from text.

Example:

```osd
use barcode

onStart {
    barcode.generate("123456", "barcode.png")
    cli.print("Barcode generated")
}
```

### `video`

Available members:

- `video.info(path)`
- `video.thumbnail(inputPath, outputPath, timestamp)`

Current v1 behavior:

- `video.info(path)` returns metadata about a video file.
- `video.thumbnail(inputPath, outputPath, timestamp)` extracts a thumbnail from a video.

Example:

```osd
use video

onStart {
    info = video.info("video.mp4")
    cli.print(info)
}
```

### `audio`

Available members:

- `audio.play(path)`
- `audio.record(outputPath, duration)`
- `audio.convert(inputPath, outputPath, format)`
- `audio.info(path)`
- `audio.beep()`

Current v1 behavior:

- `audio.play(path)` plays an audio file.
- `audio.record(outputPath, duration)` records audio for the specified duration in seconds.
- `audio.convert(inputPath, outputPath, format)` converts an audio file to a different format.
- `audio.info(path)` returns metadata about an audio file.
- `audio.beep()` plays a system beep sound.

Example:

```osd
use audio

onStart {
    audio.play("sound.mp3")
    audio.beep()
}
```

### `inspect`

Available members:

- `inspect.value(value)`
- `inspect.type(value)`

Current v1 behavior:

- `inspect.value(value)` returns a detailed string representation of a value.
- `inspect.type(value)` returns the type of a value.

Example:

```osd
use inspect

onStart {
    obj = { name: "test", value: 123 }
    cli.print(inspect.value(obj))
    cli.print(inspect.type(obj))
}
```

### `bench`

Available members:

- `bench.run(fn, iterations)`
- `bench.start()`
- `bench.end()`

Current v1 behavior:

- `bench.run(fn, iterations)` runs a function multiple times and returns the average execution time.
- `bench.start()` starts a timer.
- `bench.end()` ends the timer and returns the elapsed time.

Example:

```osd
use bench

onStart {
    time = bench.run(() => {
        cli.print("test")
    }, 100)
    cli.print("Average time: {time}ms")
}
```

### `lorem`

Available members:

- `lorem.word()`
- `lorem.sentence()`
- `lorem.paragraph()`

Current v1 behavior:

- `lorem.word()` returns a random lorem ipsum word.
- `lorem.sentence()` returns a random lorem ipsum sentence.
- `lorem.paragraph()` returns a random lorem ipsum paragraph.

Example:

```osd
use lorem

onStart {
    cli.print(lorem.word())
    cli.print(lorem.sentence())
}
```

### `ascii`

Available members:

- `ascii.fromText(text)`
- `ascii.toText(ascii)`

Current v1 behavior:

- `ascii.fromText(text)` converts text to ASCII art.
- `ascii.toText(ascii)` converts ASCII art back to text.

Example:

```osd
use ascii

onStart {
    art = ascii.fromText("Hello")
    cli.print(art)
}
```

### `matrix`

Available members:

- `matrix.add(a, b)`
- `matrix.subtract(a, b)`
- `matrix.multiply(a, b)`
- `matrix.transpose(m)`

Current v1 behavior:

- `matrix.add(a, b)` adds two matrices.
- `matrix.subtract(a, b)` subtracts two matrices.
- `matrix.multiply(a, b)` multiplies two matrices.
- `matrix.transpose(m)` transposes a matrix.

Example:

```osd
use matrix

onStart {
    a = [[1, 2], [3, 4]]
    b = [[5, 6], [7, 8]]
    result = matrix.add(a, b)
    cli.print(result)
}
```

### `cowsay`

Available members:

- `cowsay.say(text)`

Current v1 behavior:

- `cowsay.say(text)` displays text with a cow saying it.

Example:

```osd
use cowsay

onStart {
    cowsay.say("Hello World")
}
```

### `figlet`

Available members:

- `figlet.render(text)`

Current v1 behavior:

- `figlet.render(text)` renders text as ASCII art using FIGlet fonts.

Example:

```osd
use figlet

onStart {
    art = figlet.render("Hello")
    cli.print(art)
}
```

### `confetti`

Available members:

- `confetti.celebrate()`

Current v1 behavior:

- `confetti.celebrate()` displays a confetti animation in the terminal.

Example:

```osd
use confetti

onStart {
    confetti.celebrate()
}
```

### `weather`

Available members:

- `weather.current(location)`
- `weather.forecast(location, days)`

Current v1 behavior:

- `weather.current(location)` returns current weather for a location.
- `weather.forecast(location, days)` returns weather forecast for the specified number of days.

Example:

```osd
use weather

onStart {
    current = weather.current("London")
    cli.print(current)
}
```

### `joke`

Available members:

- `joke.random()`

Current v1 behavior:

- `joke.random()` returns a random joke.

Example:

```osd
use joke

onStart {
    joke = joke.random()
    cli.print(joke)
}
```

### `quote`

Available members:

- `quote.random()`
- `quote.byAuthor(author)`

Current v1 behavior:

- `quote.random()` returns a random quote.
- `quote.byAuthor(author)` returns a quote by a specific author.

Example:

```osd
use quote

onStart {
    q = quote.random()
    cli.print(q)
}
```

### `tree`

Available members:

- `tree.list(path)`

Current v1 behavior:

- `tree.list(path)` displays a directory tree structure.

Example:

```osd
use tree

onStart {
    tree.list("./")
}
```

### `diff`

Available members:

- `diff.compare(text1, text2)`
- `diff.compareFiles(path1, path2)`

Current v1 behavior:

- `diff.compare(text1, text2)` compares two text strings and returns the differences.
- `diff.compareFiles(path1, path2)` compares two files and returns the differences.

Example:

```osd
use diff

onStart {
    result = diff.compare("Hello World", "Hello Osmium")
    cli.print(result)
}
```

### `mock`

Available members:

- `mock.fn(implementation)`
- `mock.object(spec)`

Current v1 behavior:

- `mock.fn(implementation)` creates a mock function.
- `mock.object(spec)` creates a mock object with the specified methods.

Example:

```osd
use mock

onStart {
    fn = mock.fn(() => {
        return "mocked"
    })
    result = fn()
    cli.print(result)
}
```

### `lint`

Available members:

- `lint.check(code)`
- `lint.checkFile(path)`

Current v1 behavior:

- `lint.check(code)` lints Osmium code and returns issues.
- `lint.checkFile(path)` lints an Osmium file and returns issues.

Example:

```osd
use lint

onStart {
    issues = lint.checkFile("main.osd")
    cli.print(issues)
}
```

### `repl`

Available members:

- `repl.start()`

Current v1 behavior:

- `repl.start()` starts an interactive REPL session.

Example:

```osd
use repl

onStart {
    repl.start()
}
```

### `menu`

Available members:

- `menu.show(options, prompt)`

Current v1 behavior:

- `menu.show(options, prompt)` displays an interactive menu and returns the selected option.

Example:

```osd
use menu

onStart {
    choice = menu.show(["Option 1", "Option 2", "Option 3"], "Select an option:")
    cli.print("You chose: {choice}")
}
```

### `docker`

Available members:

- `docker.run(image, command)`
- `docker.build(path, tag)`
- `docker.ps()`
- `docker.images()`

Current v1 behavior:

- `docker.run(image, command)` runs a Docker container.
- `docker.build(path, tag)` builds a Docker image.
- `docker.ps()` lists running containers.
- `docker.images()` lists available images.

Example:

```osd
use docker

onStart {
    containers = docker.ps()
    cli.print(containers)
}
```

### `npm`

Available members:

- `npm.install(package)`
- `npm.run(script)`
- `npm.list()`

Current v1 behavior:

- `npm.install(package)` installs an npm package.
- `npm.run(script)` runs an npm script.
- `npm.list()` lists installed packages.

Example:

```osd
use npm

onStart {
    npm.install("lodash")
    cli.print("Package installed")
}
```

### `slack`

Available members:

- `slack.sendMessage(webhook, text)`
- `slack.sendAttachment(webhook, attachment)`

Current v1 behavior:

- `slack.sendMessage(webhook, text)` sends a message to a Slack channel via webhook.
- `slack.sendAttachment(webhook, attachment)` sends an attachment to a Slack channel.

Example:

```osd
use slack

onStart {
    slack.sendMessage("https://hooks.slack.com/...", "Hello from Osmium")
}
```

### `discord`

Available members:

- `discord.sendMessage(webhook, text)`

Current v1 behavior:

- `discord.sendMessage(webhook, text)` sends a message to a Discord channel via webhook.

Example:

```osd
use discord

onStart {
    discord.sendMessage("https://discord.com/api/webhooks/...", "Hello from Osmium")
}
```

### `telegram`

Available members:

- `telegram.sendMessage(token, chatId, text)`

Current v1 behavior:

- `telegram.sendMessage(token, chatId, text)` sends a message via the Telegram Bot API.

Example:

```osd
use telegram

onStart {
    telegram.sendMessage("bot_token", "chat_id", "Hello from Osmium")
}
```

### `github`

Available members:

- `github.repo(owner, repo)`
- `github.user(username)`

Current v1 behavior:

- `github.repo(owner, repo)` fetches information about a GitHub repository.
- `github.user(username)` fetches information about a GitHub user.

Example:

```osd
use github

onStart {
    repo = github.repo("facebook", "react")
    cli.print(repo)
}
```

## 9. Scope Rules

- Top-level assignments only accept plain literals (strings, numbers, booleans, null).
- Calling functions or extensions at the top level raises OSM200 ResolveError with the message "Function calls are not allowed at the top level. Move the call inside onStart or a function."
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

Extension versioning:

- Extensions can be pinned to specific versions using `@version` syntax: `use colors@1.2`
- Version comparison uses semantic versioning (major.minor.patch).
- If a version is specified, the extension must match or be compatible with the requested version.
- Built-in extensions have version `0.1.0` unless otherwise specified.

Example:

```osd
use colors@1.2
use json
```

Extension repositories:

- The shipped runtime has the official extension repo URL and branch baked in.
- `use name` first checks local `.osd` modules, then the local `extensions/` folder, then the official repo-backed extension packages.
- Each top-level folder in the official extension repo is treated as an extension package.
- Extension folders should export `index.js` at the folder root.

Folder layout in the official repo:

```text
colors/index.js
mem/index.js
net/index.js
json/index.js
```

## 11. Errors

Error classes:

- parse errors
- resolve errors
- runtime errors

Behavior:

- execution stops on the first error
- CLI exit codes are `0`, `1`, and `2`
- missing APIs fail clearly

## 12. Project Setup

Before running or building an Osmium project, you must initialize it using the interactive setup process.

### Initialization

Run the init command to create a new project:

```bash
osmium init [path]
```

If no path is provided, the current directory is used.

The init command displays an interactive menu:

```
What files to generate?

  settings.toml
> master.osd + settings.toml
  master.osd (Empty) + settings.toml
  cancel

Use arrow keys to navigate, Enter to select
```

**Options:**
- `settings.toml` - Creates only the settings.toml file with all required configuration
- `master.osd + settings.toml` - Creates both files with example code in master.osd
- `master.osd (Empty) + settings.toml` - Creates both files with minimal template in master.osd
- `cancel` - Exits without creating files

### Required Settings

The `settings.toml` file is required for all Osmium projects. It contains the following required settings:

#### Window & Terminal Appearance
- `terminal-title` - Window title for the terminal (e.g., "My App")
- `terminal-width` - Terminal width in columns (e.g., 120)
- `terminal-height` - Terminal height in rows (e.g., 30)
- `terminal-theme` - Terminal theme (e.g., "dark", "light")
- `terminal-font` - Terminal font name (e.g., "Cascadia Code")
- `terminal-font-size` - Font size in pixels (e.g., 14)
- `terminal-opacity` - Window opacity 0.0-1.0 (e.g., 0.95)
- `terminal-cursor` - Cursor style (e.g., "block", "underline", "bar")
- `terminal-scrollback` - Scrollback buffer size (e.g., 1000)

#### App Identity
- `name` - Project name (e.g., "my-app")
- `version` - Semantic version (e.g., "1.0.0")
- `description` - Project description (e.g., "A cool Osmium app")
- `author` - Author name (e.g., "your name")
- `license` - License identifier (e.g., "MIT")
- `homepage` - Project homepage URL
- `repository` - Repository URL
- `keywords` - Array of keywords (e.g., ["cli", "tool", "osmium"])

#### Runtime Behavior
- `entry` - Entry file to run (e.g., "master.osd")
- `terminal-visibility` - Whether to show terminal window (true/false)
- `strict-mode` - Enable strict mode (true/false)
- `max-memory` - Memory limit (e.g., "256mb", "1gb")
- `timeout` - Execution timeout in seconds (e.g., 30)
- `log-level` - Logging level (e.g., "info", "debug", "error")
- `log-file` - Log file path (e.g., "app.log")
- `encoding` - File encoding (e.g., "utf-8")
- `locale` - Locale setting (e.g., "en-US")
- `timezone` - Timezone (e.g., "UTC")

#### Build & Distribution
- `build-output` - Output directory for builds (e.g., "./dist")
- `build-target` - Target platform (e.g., "windows", "linux", "macos")
- `build-icon` - Icon file path (e.g., "./assets/icon.ico")
- `build-single-file` - Build as single executable (true/false)

#### Environment & Secrets
- `env-file` - Path to .env file (e.g., ".env")
- `dotenv` - Enable dotenv loading (true/false)
- `secrets-file` - Path to secrets.toml file

#### Updates & Versioning
- `auto-update` - Enable auto-updates (true/false)
- `update-channel` - Update channel (e.g., "stable", "beta")
- `update-url` - Update check URL
- `check-update-on-start` - Check for updates on startup (true/false)

#### Permissions (Security Model)
```toml
[permissions]
allow-shell = true
allow-network = true
allow-filesystem = true
allow-clipboard = false
allow-notifications = false
allow-registry = false
```

#### Build Metadata
```toml
[build.metadata]
company = "Your Company"
copyright = "2026"
product-name = "My App"
file-description = "A cool CLI tool"
```

### Example settings.toml

```toml
# Window & terminal appearance
terminal-title = "My App"
terminal-width = 120
terminal-height = 30
terminal-theme = "dark"
terminal-font = "Cascadia Code"
terminal-font-size = 14
terminal-opacity = 0.95
terminal-cursor = "block"
terminal-scrollback = 1000

# App identity
name = "my-app"
version = "1.0.0"
description = "A cool Osmium app"
author = "your name"
license = "MIT"
homepage = "https://github.com/you/your-app"
repository = "https://github.com/you/your-app"
keywords = ["cli", "tool", "osmium"]

# Runtime behaviour
entry = "master.osd"
terminal-visibility = true
strict-mode = true
max-memory = "256mb"
timeout = 30
log-level = "info"
log-file = "app.log"
encoding = "utf-8"
locale = "en-US"
timezone = "UTC"

# Build & distribution
build-output = "./dist"
build-target = "windows"
build-icon = "./assets/icon.ico"
build-single-file = true

# Environment & secrets
env-file = ".env"
dotenv = true
secrets-file = "secrets.toml"

# Updates & versioning
auto-update = true
update-channel = "stable"
update-url = "https://yourapp.com/releases"
check-update-on-start = true

# Permissions (security model)
[permissions]
allow-shell = true
allow-network = true
allow-filesystem = true
allow-clipboard = false
allow-notifications = false
allow-registry = false

# App metadata for built executables
[build.metadata]
company = "Your Company"
copyright = "2026"
product-name = "My App"
file-description = "A cool CLI tool"
```

### Running and Building

After initialization, you can run or build your project:

```bash
# Run the project (uses entry file from settings.toml)
osmium run [path]

# Build the project
osmium build [path]

# Watch for changes
osmium watch [path]

# Debug mode
osmium debug [path]
```

**Important:** All commands require a valid `settings.toml` file with all required settings. If settings are missing, you will be prompted to run `osmium init` again.

## 13. Example Project

After running `osmium init` and selecting "master.osd + settings.toml", your project will have:

**settings.toml** (with all required settings - see Section 12 for full details)

**master.osd** (example entry file):

```osd
use filesys
use cli

dollars = 4

onStart {
    cli.print("Starting program...")
    dollars = (dollars + 1)
    cli.print("You have {dollars} dollars.")
}
```

Run it:

```bash
osmium run [path]
```

Build it:

```bash
osmium build [path]
```

Register the Windows context menu:

```bash
osmium register
```

## 14. Error Codes

All Osmium errors use the format `LxLxN` where:
- First letter: Error category (G=General, P=Parse, R=Resolve, E=Execution)
- Second letter: Subcategory
- 'x': Literal separator
- Third letter: Specific error type
- Number: Error variant

### General Errors (GxLxN)

| Code | Name | Description | Fix |
|------|------|-------------|-----|
| GxA0 | OsmiumError | Base error class | Contact support |
| GxA1 | UnknownError | Unidentified error occurred | Check error details |

### Parse Errors (PxLxN)

| Code | Name | Description | Fix |
|------|------|-------------|-----|
| PxA0 | ParseError | Base parse error | Check syntax |
| PxB1 | UnterminatedComment | Multiline comment not closed | Add `*/` to close comment |
| PxB2 | UnterminatedInterpolation | String interpolation not closed | Add `}` to close interpolation |
| PxB3 | UnterminatedString | String literal not closed | Add closing quote |
| PxB4 | InvalidNumber | Malformed number literal | Fix number format |
| PxB5 | UnexpectedCharacter | Invalid character in source | Remove or fix character |
| PxC1 | ExpectedIdentifier | Identifier expected but not found | Add valid identifier |
| PxC2 | ExpectedOperator | Operator expected but not found | Add expected operator |
| PxC3 | ExpectedBrace | `{` expected to start block | Add opening brace |
| PxC4 | ExpectedDevice | Device keyword expected | Add `device` keyword |
| PxC5 | ExpectedFunction | Function keyword expected | Add `fn` keyword |
| PxC6 | ExpectedOnStart | onStart keyword expected | Add `onStart` keyword |
| PxC7 | ExpectedAssignmentTarget | Valid assignment target expected | Check left side of assignment |
| PxD1 | ExpectedDo | `do` keyword expected after while | Add `do` keyword |
| PxD2 | ExpectedThen | `then` keyword expected after if | Add `then` keyword |
| PxD3 | ExpectedCatch | `catch` keyword expected after try | Add `catch` keyword |
| PxD4 | ExpectedIn | `in` keyword expected in for loop | Add `in` keyword |
| PxE1 | ExpectedClosingBrace | `}` expected to close block | Add closing brace |
| PxE2 | ExpectedClosingParen | `)` expected to close parentheses | Add closing parenthesis |
| PxE3 | ExpectedClosingBracket | `]` expected to close array/index | Add closing bracket |
| PxF1 | InvalidExport | Invalid export syntax | Check export syntax |
| PxF2 | InvalidUse | Invalid use statement | Check use syntax |
| PxF3 | InvalidBraceNotation | Invalid brace notation in use | Fix brace syntax |

### Resolve Errors (RxLxN)

| Code | Name | Description | Fix |
|------|------|-------------|-----|
| RxA0 | ResolveError | Base resolve error | Check references |
| RxB1 | UndefinedSymbol | Referenced symbol not found | Define or import symbol |
| RxB2 | UndefinedDevice | Device not found | Define device or check spelling |
| RxB3 | UndefinedFunction | Function not found | Define function or check spelling |
| RxB4 | UndefinedVariable | Variable not found | Define variable or check spelling |
| RxC1 | CannotExportUndefined | Cannot export undefined symbol | Define symbol before export |
| RxC2 | CannotRedefineVariable | Variable already defined | Use different name |
| RxC3 | InvalidModule | Invalid module reference | Check module path |
| RxD1 | FileNotFound | Source file not found | Check file path |
| RxD2 | ExtensionNotFound | Extension not found | Install extension or check spelling |
| RxD3 | InvalidExtension | Extension module invalid | Check extension implementation |

### Execution Errors (ExLxN)

| Code | Name | Description | Fix |
|------|------|-------------|-----|
| ExA0 | RuntimeError | Base runtime error | Check runtime state |
| ExB1 | UnsupportedOperator | Operator not supported | Use valid operator |
| ExB2 | UnsupportedUnaryOperator | Unary operator not supported | Use valid unary operator |
| ExB3 | UnsupportedPostfixOperator | Postfix operator not supported | Use valid postfix operator |
| ExB4 | UnknownCompoundAssignment | Compound assignment operator unknown | Use valid compound operator |
| ExC1 | DivisionByZero | Division by zero attempted | Check divisor value |
| ExC2 | InvalidAssignmentTarget | Cannot assign to target | Check assignment target |
| ExC3 | InvalidSpread | Spread operator requires array | Use array with spread |
| ExD1 | ArrayIndexOutOfBounds | Array index out of range | Check array index |
| ExD2 | InvalidArrayLength | Invalid array length | Use valid length |
| ExD3 | ArrayMethodFailed | Array method operation failed | Check array and method |
| ExE1 | InvalidNumberConversion | Cannot convert to number | Check value type |
| ExE2 | InvalidStringConversion | Cannot convert to string | Check value type |
| ExE3 | InvalidBooleanConversion | Cannot convert to boolean | Check value type |
| ExF1 | NullReference | Operation on null value | Check for null before operation |
| ExF2 | UndefinedReference | Operation on undefined value | Check for undefined before operation |
| ExG1 | FunctionCallFailed | Function call failed | Check function arguments |
| ExG2 | InvalidArguments | Invalid function arguments | Check argument types/count |
| ExG3 | ReturnOutsideFunction | Return statement outside function | Move return inside function |
| ExH1 | BreakOutsideLoop | Break statement outside loop | Move break inside loop |
| ExH2 | ContinueOutsideLoop | Continue statement outside loop | Move continue inside loop |
| ExI1 | InvalidThisContext | Invalid 'this' context | Check context usage |
| ExI2 | InvalidMemberAccess | Invalid member access | Check object structure |
| ExJ1 | CLIError | CLI operation failed | Check CLI command |
| ExJ2 | InputError | Input operation failed | Check input handling |
| ExJ3 | KeyPressError | Key press detection failed | Check terminal support |
| ExK1 | FileReadError | File read failed | Check file path and permissions |
| ExK2 | FileWriteError | File write failed | Check file path and permissions |
| ExK3 | FileDeleteError | File delete failed | Check file path and permissions |
| ExK4 | DirectoryNotFoundError | Directory not found | Check directory path |
| ExK5 | InvalidPathError | Invalid file path | Fix path format |
| ExL1 | JSONParseError | JSON parsing failed | Check JSON format |
| ExL2 | JSONStringifyError | JSON serialization failed | Check object structure |
| ExM1 | NetworkError | Network operation failed | Check network connection |
| ExM2 | InvalidURLError | Invalid URL format | Fix URL format |
| ExN1 | CryptoError | Cryptographic operation failed | Check crypto parameters |
| ExO1 | MathError | Math operation failed | Check math parameters |
| ExO2 | InvalidRangeError | Invalid numeric range | Fix range values |
| ExP1 | StringError | String operation failed | Check string parameters |
| ExP2 | InvalidEncodingError | Invalid encoding | Use valid encoding |
| ExQ1 | TimeError | Time operation failed | Check time parameters |
| ExQ2 | InvalidDurationError | Invalid duration value | Fix duration format |
| ExR1 | ExtensionError | Extension operation failed | Check extension implementation |
| ExR2 | InvalidExtensionError | Invalid extension module | Fix extension code |
| ExS1 | PermissionError | Permission denied | Check permissions settings |
| ExS2 | SecurityError | Security violation | Check security settings |
| ExT1 | MemoryError | Memory limit exceeded | Reduce memory usage |
| ExT2 | TimeoutError | Operation timed out | Increase timeout or optimize |
| ExU1 | UpdateError | Update operation failed | Check update settings |
| ExU2 | VersionError | Version mismatch | Check version compatibility |

## 15. Notes

- Official filesystem and terminal APIs are intentionally conservative in v1.
- Community extensions are loaded through the extension system, not syntax changes.
- ANSI color helpers are provided by the `colors` extension example.
- This document should be updated whenever language behavior changes.
