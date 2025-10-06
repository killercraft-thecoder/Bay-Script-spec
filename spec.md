# Bay Script Language Specification

## 1. Variables

- **Declaration**: `var <name> = <expression>`
- **Scope**:
  - At top level: `var` creates a global.
  - Inside functions: `var` creates a local.
- **Global keyword**:
  - `global <name> = <expr>` → create/update a global variable.
  - `global <name>` → bind to existing global, or create with value `0` if missing.
  - At top level, `global` is equivalent to `var` (no effect).
- **Assignment**: `set <name> <expr>` updates an existing variable.

## 2. Expressions

- Only **one binary operator** per expression.
- Supported operators (integers only):
  - Arithmetic: `+`, `-`, `*`, `/` (integer division), `%` (modulo), `**` (exponentiation).
  - Bitwise: `<<`, `>>`.
- Example:
var a = 5 + 3 \n var b = [call f] * 2

## 3. Functions

- **Definition**:
function <name> (<args>) <statements> return <expr> end
- **Return values**:
- Must be accessed via `[call ...]` syntax.
- **Nesting**:
- Functions may be defined inside other functions.
- Inner functions cannot access outer locals, but can access globals.

## 4. Inline Calls

- Syntax: `[call <function> <args>]`
- Example: var x = [call double 5]

## 5. Control Flow

- **If/Elif/Else**: 
if <condition> <statements> elif <condition> <statements> else <statements> end
- **Conditions**:
- Allowed operators: `>`, `<`, `==`, `!=`
- Operands may be variables or `[call ...]`.
- Execution:
- First true block runs, others skipped.
- If none true, `else` runs (if present).

## 6. Imports

- **Basic**: `import <module>`
- **From path**: `import <module> from "<path>"`
- Behavior:
- Runs the script/module into the current environment.
- Built‑in modules are provided by the interpreter.
- `.bs` files are Bay Script modules.
- **Circular dependency**:
- Interpreter tracks imports with a 16/32‑bit hash table.
- If already imported, skip execution.

## 7. Scope Model

- **Global scope**: variables declared at top level or with `global`.
- **Local scope**: variables declared with `var` inside functions.
- **No closures**: inner functions cannot see outer locals.

## 8. Performance Notes

- Global lookups are slower than locals:
- Require hash table lookup.
- `global x` without assignment must check existence, creating `x = 0` if missing.

## 9. While Loops

- **Syntax**:
while <condition> <statements> end

- **Condition**:
- Uses the same operators as `if` (`>`, `<`, `==`, `!=`).
- May reference variables or `[call ...]`.

- **Execution**:

1. Evaluate the condition.
2. If true → run the block.
3. At `end`, jump back to the `while` and re‑evaluate.
4. Repeat until the condition is false, then exit.

- **Example**:
var x = 0  \n while x < 5 \n set x x + 1 \n end

## 10. Goto

- **Syntax**:
goto <function> [arguments]

- **Behavior**:
- Transfers control directly to the start of the target function.
- If arguments are given, they are passed normally.
- Does **not** add a new stack frame — it replaces the current one (tail‑call style).
- Execution continues from the target function as if it were entered fresh.

- **Notes**:
- Useful for implementing loops or state machines without recursion overhead.
- Safer than arbitrary jumps: can only jump to function entry points.

- **Example**: 
function countdown (n) if n == 0 return 0 else set n n - 1 goto countdown n end end

## 11. Calls

Regular calling must not use inline calls , inlien calls are for cases like var x = [call calc]
not for use like: [call calc] , instead you just do call calc for that.

# Note:

the snippets may be slighlty inancurate since some forgot the \n's needed , ut its understandable.

## Built‑in Modules

### io

- Provides basic input/output.
- **Functions**:
  - `input()`  
    - Reads an integer from standard input.  
    - **Not guaranteed** to be supported on all platforms.  
    - If unsupported, returns `-1`.  
  - `print(<int>)`  
    - Prints an integer to standard output.

### cstr (C‑style string operations)

- Provides operations for working with raw C‑style strings.
- **Functions**:
  - `getCharCode <cstrVar>, <index>`  
    - Returns the integer code of the character at `index`.  
  - `setCharCode <cstrVar> <index> <value>`  
    - Sets the character at `index` to the given integer code.  
  - `getCStr module.component>`  
    - User‑accessible function.  
    - Takes a backend module component reference (e.g. `somemodule.abc`).  
    - Returns a **linked variable**:
      - A Bay Script variable with no intrinsic value.  
      - Internally linked to a native backend value (C/TS/JS string buffer).  
    - Used for creating and manipulating C‑style strings.
- **Varibles**:
  - `ref`
    - a linked varilbe,  getCSTR can use this by defualt. (as in you can do getCSTR(cstr.ref)) , note: CSTR's have a min length of 32 bytes though actual length must be at leats thta min or bigger.

### module (special backend module)

- A reserved module for **low‑level interfacing** with backend modules and the interpreter.
- Provides utilities that are difficult or inefficient to implement directly in Bay Script.

- **Functions**:
  - `run <cstrVar>`  
    - Executes a Bay Script module (a `.bs` file) or a named module.  
    - Argument must be a **CSTR linked variable** (from `cstr.getCStr`).  
    - The linked variable represents either:
      - A module name, or
      - A path to a Bay Script file.  
    - Runs the specified module in the current environment, similar to `import`, but invoked dynamically at runtime.

  - `getInt <cstrVar>`  
    - Reads an integer value from a linked variable.  
    - Useful for extracting numeric values from backend‑linked data.

  - `setInt <cstrVar> <int>`  
    - Writes an integer value into a linked variable.  
    - Allows Bay Script to update backend‑linked data directly.

  - `exists <cstrVar>`  
    - Checks if a given linked variable or module reference exists.  
    - Returns `1` if it exists, `0` otherwise.

- **Notes**:
  - `run` is the primary way to dynamically load and execute modules at runtime.  
  - `getInt` and `setInt` provide direct access to integer values in backend‑linked variables.  
  - `exists` is a utility for safe checks before attempting to run or access a module.  
  - These utilities are intended for advanced use and backend/module integration, not everyday scripting.

### tables

- Provides access to statically defined, immutable numeric tables.
- **Functions**:
  - `readIndex <table> <index>`  
    - Returns the integer value at the specified `index` in the static `table`.  
    - Behavior for out-of-bounds access is implementation-defined.  
    - Tables must be declared using the `static` keyword at the top level.

- **Static Table Declaration**:
  - `static <name> = [<int>, <int>, ...]`  
    - Declares a globally scoped, immutable table.  
    - Only integer values are allowed.  
    - Table contents cannot be modified after declaration.  
    - Table names must be unique within the module.

---