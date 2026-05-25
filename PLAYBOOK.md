# Playbook -- Lexical Analyzer (C++)

Step-by-step guide to build and run the lexer locally, scan a sample
expression, and verify it works end-to-end. If you've never compiled a
C++ program before, this is the file to follow.

## 1. Prerequisites

Install one C++17-capable toolchain for your platform, plus `git`.
Pinned versions are what the project was developed against -- newer
versions also work.

| Tool | Version | Where to get it |
|---|---|---|
| g++ (Linux)             | 9+    | `sudo apt install g++` (Ubuntu/Debian) or `sudo dnf install gcc-c++` (Fedora) |
| clang (macOS)           | any   | `xcode-select --install` (Xcode Command Line Tools)                          |
| MSVC (Windows)          | 2019+ | Visual Studio Build Tools -- https://visualstudio.microsoft.com/downloads/   |
| MinGW-w64 (Windows alt) | 11+   | https://www.mingw-w64.org/ or via MSYS2                                      |
| Git                     | any   | https://git-scm.com/downloads                                                |

Verify the compiler is installed:

```bash
g++ --version
# should print "g++ (...) 9.x" or newer
```

On Windows with MSVC, open the **Developer Command Prompt for VS**
(not the regular `cmd.exe`) and run:

```cmd
cl
:: should print the Microsoft (R) C/C++ Optimizing Compiler banner
```

On Windows you can also use **WSL** and follow the Linux instructions
verbatim.

## 2. Clone the repo

```bash
git clone https://github.com/salmaamr129/lexical-analyzer-cpp.git
cd lexical-analyzer-cpp
```

The repo contains three files: `concept.cpp` (the lexer source),
`front.in` (the sample input), and `README.md`.

## 3. Build

### Linux / macOS (g++ or clang)

```bash
g++ -std=c++17 -Wall -o lexer concept.cpp
```

On macOS, `g++` is aliased to `clang++` by default -- the same command
works.

### Windows -- MSVC (Developer Command Prompt for VS)

```cmd
cl /EHsc /std:c++17 concept.cpp /Fe:lexer.exe
```

### Windows -- MinGW-w64

```cmd
g++ -std=c++17 -Wall -o lexer.exe concept.cpp
```

A successful build produces a single executable (`lexer` on Linux/macOS,
`lexer.exe` on Windows) in the current directory and prints nothing
(warnings, if any, are harmless).

## 4. Run

The program reads from a file called **`front.in`** in the current
working directory -- the path is hardcoded and relative, so run the
binary from the same directory the file lives in.

### Linux / macOS

```bash
./lexer
```

### Windows

```cmd
.\lexer.exe
```

## 5. Verify it works

With the default `front.in` (which contains `x = 5 + 10 * (y - 2)`),
running `./lexer` should print exactly:

```
Next token is: IDENT, Next lexeme is x
Next token is: ASSIGN_OP, Next lexeme is =
Next token is: INT_LIT, Next lexeme is 5
Next token is: ADD_OP, Next lexeme is +
Next token is: INT_LIT, Next lexeme is 10
Next token is: MULT_OP, Next lexeme is *
Next token is: LEFT_PAREN, Next lexeme is (
Next token is: IDENT, Next lexeme is y
Next token is: SUB_OP, Next lexeme is -
Next token is: INT_LIT, Next lexeme is 2
Next token is: RIGHT_PAREN, Next lexeme is )
Next token is: EOF, Next lexeme is EOF
[LOOP] Current Token: EOF
```

If the first line you see is
`Next token is: IDENT, Next lexeme is x`, the program is working.

## 6. Edit `front.in` to test different inputs

`front.in` is a plain text file -- open it in any editor, replace the
contents, save, and re-run `./lexer` (no rebuild needed).

### Example A -- pure identifier addition

`front.in`:

```
a = b + c
```

Expected token stream:

```
Next token is: IDENT, Next lexeme is a
Next token is: ASSIGN_OP, Next lexeme is =
Next token is: IDENT, Next lexeme is b
Next token is: ADD_OP, Next lexeme is +
Next token is: IDENT, Next lexeme is c
Next token is: EOF, Next lexeme is EOF
[LOOP] Current Token: EOF
```

### Example B -- integer division

`front.in`:

```
count1 = 100 / 4
```

Expected token stream:

```
Next token is: IDENT, Next lexeme is count1
Next token is: ASSIGN_OP, Next lexeme is =
Next token is: INT_LIT, Next lexeme is 100
Next token is: DIV_OP, Next lexeme is /
Next token is: INT_LIT, Next lexeme is 4
Next token is: EOF, Next lexeme is EOF
[LOOP] Current Token: EOF
```

Note `count1` is a single `IDENT` -- identifiers may contain digits
after the first letter.

## 7. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `ERROR - cannot open front.in` (or no output) | Running the binary from a different directory than `front.in` | `cd` into the repo root before running `./lexer`; the path is hardcoded and relative |
| `g++: command not found` | Compiler isn't installed | `sudo apt install g++` / `sudo dnf install gcc-c++` / `xcode-select --install` |
| `'cl' is not recognized as an internal or external command` | Using the regular `cmd.exe` instead of the Developer Command Prompt for VS | Open **Developer Command Prompt for VS** from the Start menu and rebuild |
| `error: 'auto' return without trailing return type` or similar standard errors | Compiler defaulting to an older C++ standard | Pass `-std=c++17` explicitly; upgrade g++ to 9 or newer |
| Crash, garbage output, or hangs | `front.in` is empty, binary, or contains non-ASCII characters | Replace `front.in` with a plain ASCII expression and re-run |
| Output stops mid-expression | An unsupported character (e.g. `;`, `{`, `==`) hit the operator table and was emitted as `UNKNOWN` | Stick to the documented token set (`+ - * / = ( )`, identifiers, integers) |

## 8. Stop

There's nothing to stop. The program reads `front.in`, prints one line
per token, and exits on its own when it reaches end-of-file. If you
want to interrupt a misbehaving run, `Ctrl+C` in the terminal.

## 9. What to look at next

- **`concept.cpp` -> `lex()`** is the entry point and main loop. It
  dispatches on `charClass` (`LETTER`, `DIGIT`, `UNKNOWN`,
  `END_OF_FILE`) and emits one token per call.
- **`concept.cpp` -> `lookup()`** is the operator table -- a `switch`
  on the current character that maps `+ - * / = ( )` to their token
  codes.
- To add a new token (for example, a separate `EQ_OP` for `==`), you
  need to touch two places:
  1. Add a `#define EQ_OP <code>` near the top of `concept.cpp`
     alongside the other token constants.
  2. Extend the `lookup()` `switch` so that when it sees `=` it peeks
     at the next character, and if it's also `=` it consumes both and
     returns `EQ_OP` instead of `ASSIGN_OP`.
- The token-naming logic that prints `Next token is: IDENT, ...` lives
  in the same file -- search for the `switch` on `nextToken` to add a
  human-readable name for any new token you introduce.
