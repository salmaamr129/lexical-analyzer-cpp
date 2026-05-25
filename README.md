# Lexical Analyzer (C++)

A hand-written lexical analyzer (scanner) in C++ that tokenises simple
arithmetic / assignment expressions. Implemented as a finite-state
analyser driven directly from input characters — no regex engine, no
lexer-generator (no Flex), just the classic textbook implementation.

Originally written for a **Concepts of Programming Languages** course
assignment based on the lexer presented in Sebesta's textbook (chapter 4).

## What it recognises

The scanner recognises the following token types:

| Token | Description | Examples |
|---|---|---|
| `IDENT`        | identifiers (letter then letters/digits)             | `x`, `y`, `count1` |
| `INT_LIT`      | integer literals                                      | `5`, `10`, `42`    |
| `ASSIGN_OP`    | assignment operator                                   | `=`                |
| `ADD_OP`       | addition                                              | `+`                |
| `SUB_OP`       | subtraction                                           | `-`                |
| `MULT_OP`      | multiplication                                        | `*`                |
| `DIV_OP`       | division                                              | `/`                |
| `LEFT_PAREN`   | open parenthesis                                      | `(`                |
| `RIGHT_PAREN`  | close parenthesis                                     | `)`                |
| `EOF`          | end-of-file sentinel                                  | (end of input)     |

Whitespace is skipped via `getNonBlank()`.

## How it works

The scanner is a state machine driven by `charClass`, which is one of
`LETTER`, `DIGIT`, `UNKNOWN`, or `END_OF_FILE`. The main `lex()` loop:

1. Skip whitespace.
2. If the next char is a letter, consume letters/digits and emit `IDENT`.
3. If it's a digit, consume digits and emit `INT_LIT`.
4. Otherwise look up the char in the operator table and emit the matching
   single-character token (`+`, `-`, `*`, `/`, `=`, `(`, `)`).
5. Stop when `END_OF_FILE` is reached.

Each token's name and lexeme is printed to stdout.

## Build & run

### Linux / macOS
```bash
g++ -std=c++17 -Wall -o lexer concept.cpp
./lexer
```

### Windows (MSVC, from Developer Command Prompt)
```cmd
cl /EHsc /std:c++17 concept.cpp /Fe:lexer.exe
lexer.exe
```

The program reads from a file called **`front.in`** in the current
working directory. Edit `front.in` to scan a different expression.

## Sample input

`front.in`:

```
x = 5 + 10 * (y - 2)
```

## Expected output

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

## Files

```
concept.cpp     The lexer source (single-file program, ~155 lines)
front.in        Sample input expression
README.md       This file
```

## Reference

- Sebesta, *Concepts of Programming Languages*, Chapter 4 (Lexical Analysis).
- The state-machine structure and the constant codes
  (`LETTER=0`, `DIGIT=1`, `INT_LIT=10`, `IDENT=11`, …) follow the
  canonical implementation from that chapter.

## License

[MIT](LICENSE)
