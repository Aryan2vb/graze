# TGL Compiler: A Six-Phase Compiler for Turtle Graphics

A complete mini compiler for **TGL (Turtle Graphics Language)**, a small language for drawing shapes and patterns with a turtle. Every program passes through all six classical compiler phases: lexical analysis, syntax analysis, semantic analysis, intermediate code generation, code optimization and target code generation. The output of each phase can be inspected, and the final drawing is animated step by step.

```
PEN BLUE;
REPEAT 4 {
  MOVE 100;
  TURN 90;
}
```

This program compiles to a blue square.

---

## Team

**Course project: Compiler Design**

| Name | Roll No. |
| --- | --- |
| Aryan Soni | 230049 |
| Abhijeet Kumar Shah | 230036 |
| Karthik Reddy | 230035 |
| Kanhaiya Kumar | 230062 |
| Fathal Koul | 230043 |


---

## Contents

1. [Features](#features)
2. [How to run](#how-to-run)
3. [Project structure](#project-structure)
4. [The TGL language](#the-tgl-language)
5. [Compiler architecture](#compiler-architecture)
6. [Phase 1: Lexical analysis](#phase-1-lexical-analysis)
7. [Phase 2: Syntax analysis](#phase-2-syntax-analysis)
8. [Phase 3: Semantic analysis](#phase-3-semantic-analysis)
9. [Phase 4: Intermediate code generation](#phase-4-intermediate-code-generation)
10. [Phase 5: Code optimization](#phase-5-code-optimization)
11. [Phase 6: Target code generation](#phase-6-target-code-generation)
12. [Error and warning codes](#error-and-warning-codes)
13. [Sample programs](#sample-programs)
14. [Testing](#testing)
15. [Limitations and future work](#limitations-and-future-work)

---

## Features

- **All six compiler phases**, each with its own output that can be viewed separately.
- **Source editor** with syntax highlighting, line numbers and 14 ready-made sample programs.
- **Pipeline view** showing the status of every phase: Success, Warnings, Failed or Skipped.
- **Diagnostics** with an error code, the phase, the line and column, and a suggested fix. Clicking a diagnostic jumps to that line.
- **Phase inspector** with six tabs: Tokens, Syntax tree, Semantic report with symbol table, Intermediate code, Optimizer (before and after), and Target code.
- **Six optimization passes**, run until nothing more changes. Each optimized program is checked automatically: the original and the optimized code are both executed and must produce the same drawing.
- **Drawing sheet** with zoom, pan, fit-to-screen, and step-by-step playback with speed control. The turtle's position, heading, pen state and color are shown after every step, and the current source line is highlighted.
- **Language reference** built into the tool: commands, tokens, grammar, rules, colors and every error code.
- **Safe by design:** a program with any error is never executed, and the amount of work a program does is limited before it runs.

---

## How to run

No installation is needed. The whole compiler runs in the browser.

1. Download or clone this repository (**Code → Download ZIP**, or `git clone`).
2. Open [`TGL Compiler Visualiser (2).html`](<TGL Compiler Visualiser (2).html>) in Chrome, Edge or Firefox (double-click the file).
3. Choose a program from the **Sample** list, or type your own. The program recompiles as you type. You can also press **Compile**.
4. Use the player under the drawing sheet to step through the drawing, and the tabs at the bottom to see what each phase produced.
5. Open **Language reference** (top right) for the full language documentation.

An internet connection is only used to load fonts. Without one, the page works the same with system fonts.

---

## Project structure

```
.
├── README.md                          This file
└── TGL Compiler Visualiser (2).html     The compiler and the visualiser (one self-contained file)
```

The whole project is a single HTML file with no dependencies. Inside `TGL Compiler Visualiser (2).html`, the compiler core and the user interface are kept separate:

| Part | Responsibility |
| --- | --- |
| `lex()` | Phase 1: characters to tokens |
| `parse()` | Phase 2: tokens to Abstract Syntax Tree |
| `analyze()` | Phase 3: semantic rules, symbol table, limits |
| `genIR()` | Phase 4: AST to structured intermediate code |
| `optimize()` | Phase 5: six optimization passes |
| `codegen()` and `execute()` | Phase 6: flat drawing instructions, run by the turtle |
| `equivalent()` | Checks that the optimized program draws exactly the same as the original |
| `compile()` | Runs the phases in order and stops at the first phase with an error |
| User interface | Editor, pipeline, diagnostics, inspector tabs, drawing sheet, reference |

The compiler core never touches the page, and the drawing sheet draws only the lines recorded by phase 6. The source code is never interpreted by the user interface itself.

---

## The TGL language

### Commands

| Command | Argument | What it does | Example |
| --- | --- | --- | --- |
| `PEN` | a color | Sets the pen color | `PEN BLUE;` |
| `MOVE` | distance, 0 to 1000 | Moves forward along the heading; draws if the pen is down | `MOVE 100;` |
| `TURN` | angle, 0 to 360 | Turns clockwise by the angle, relative to the current heading | `TURN 90;` |
| `REPEAT` | whole number, 1 to 100 | Runs the block in braces that many times; blocks can be nested up to 10 deep | `REPEAT 4 { ... }` |
| `PENUP` | none | Lifts the pen; later moves do not draw | `PENUP;` |
| `PENDOWN` | none | Lowers the pen; later moves draw | `PENDOWN;` |
| `CLEAR` | none | Erases everything drawn so far; position, heading, color and pen stay the same | `CLEAR;` |

Every command ends with `;`. Keywords and color names are case-insensitive. Comments start with `//` and run to the end of the line.

`SETX`, `SETY`, `GOTO`, `CIRCLE` and `WAIT` are reserved as planned extensions. They are recognized and parsed, then rejected by the semantic phase with error S001.

### Colors

`BLACK`, `WHITE`, `RED`, `GREEN`, `BLUE`, `YELLOW`, `ORANGE`, `PURPLE`, `PINK`, `CYAN`, `MAGENTA`, `GRAY` (also `GREY`), and any hex color written as `#RRGGBB`, for example `PEN #1E90FF;`.

### The turtle and the coordinate system

| Property | Value |
| --- | --- |
| Sheet size | 800 × 600 units |
| Origin | (0, 0) at the top-left corner; x grows right, y grows down |
| Start position | (400, 300), the center of the sheet |
| Start heading | 0°, facing right |
| Start pen | down, color BLACK |
| Turning | `TURN a` sets heading = (heading + a) mod 360, which is clockwise on screen |

Movement formula for `MOVE d`:

```
θ     = heading × π / 180
x_new = x + d × cos θ
y_new = y + d × sin θ        (no sign change is needed, because y grows downward)
```

Values within 0.000000001 of a whole number are rounded to it, so cos 90° is exactly 0 and closed shapes such as a square end exactly where they started.

---

## Compiler architecture

```
 Source code
     │
     ▼
 Phase 1  Lexical analysis          characters        →  tokens
     │
     ▼
 Phase 2  Syntax analysis           tokens            →  Abstract Syntax Tree (AST)
     │
     ▼
 Phase 3  Semantic analysis         AST               →  validated AST + symbol table + diagnostics
     │
     ▼
 Phase 4  Intermediate code         validated AST     →  structured IR (loops kept)
     │
     ▼
 Phase 5  Code optimization         IR                →  smaller IR, same drawing (verified)
     │
     ▼
 Phase 6  Target code generation    optimized IR      →  flat drawing instructions → turtle → lines
     │
     ▼
 Drawing sheet
```

**Front end (phases 1 to 3)** checks that the program is correct. **Back end (phases 4 to 6)** turns it into efficient drawing instructions. If any phase reports an error, compilation stops, every later phase is marked Skipped, and nothing is drawn. Warnings are shown but do not stop compilation.

---

## Phase 1: Lexical analysis

The lexer reads the source one character at a time and groups the characters into **tokens**. It skips spaces, tabs, new lines and `//` comments. Each token records its type, its exact text (lexeme), its line and column, and its start and end position in the source.

### Token types

| Category | Token types |
| --- | --- |
| Keywords | `PEN` `MOVE` `TURN` `REPEAT` `PENUP` `PENDOWN` `CLEAR` (and the extension keywords `SETX` `SETY` `GOTO` `CIRCLE` `WAIT`) |
| Literals | `INTEGER` (100, -50), `FLOAT` (12.5), `HEX_COLOR` (#1E90FF) |
| Names | `IDENTIFIER` (color names such as BLUE) |
| Symbols | `LBRACE` `{`, `RBRACE` `}`, `SEMICOLON` `;`, `COMMA` `,` |
| Marker | `EOF` (end of input) |

### Example

```
MOVE 100;   →   [MOVE 'MOVE' 1:1]  [INTEGER '100' 1:6]  [SEMICOLON ';' 1:9]
```

### Design decisions

- **Longest match:** the lexer reads a whole word before checking the keyword table, so `REPEATER` is an identifier while `REPEAT` is a keyword.
- **Color names are identifiers:** `PEN INVISIBLE;` passes the lexer and is rejected later by the semantic phase, which can give a precise message.
- **Negative numbers are read as numbers:** `MOVE -50;` reaches the semantic phase and gets a clear out-of-range error instead of a confusing character error.

---

## Phase 2: Syntax analysis

The parser checks that the tokens follow the grammar and builds the **Abstract Syntax Tree (AST)**.

### Grammar (EBNF)

```
program       ::= statement* EOF ;

statement     ::= pen_stmt | move_stmt | turn_stmt | repeat_stmt
                | penup_stmt | pendown_stmt | clear_stmt ;

pen_stmt      ::= PEN color SEMICOLON ;
move_stmt     ::= MOVE number SEMICOLON ;
turn_stmt     ::= TURN number SEMICOLON ;
repeat_stmt   ::= REPEAT number LBRACE statement* RBRACE ;
penup_stmt    ::= PENUP SEMICOLON ;
pendown_stmt  ::= PENDOWN SEMICOLON ;
clear_stmt    ::= CLEAR SEMICOLON ;

color         ::= IDENTIFIER | HEX_COLOR ;
number        ::= INTEGER | FLOAT ;
```

The parser is a **recursive descent parser**, with one function per grammar rule. Every statement starts with a different keyword, so the grammar is **LL(1)**: one token of lookahead is always enough and the parser never backtracks. Nested `REPEAT` blocks work because the REPEAT rule contains `statement*` again.

`REPEAT number` (not only integers) and `PEN IDENTIFIER` (not only known colors) are intentional. They let `REPEAT 1.5` and `PEN INVISIBLE` be reported by the semantic phase with an exact explanation.

### AST for the square program

```
ProgramNode
|-- PenCommandNode
|   `-- ColorNode BLUE
`-- RepeatNode
    |-- NumberNode 4 (count)
    |-- MoveCommandNode
    |   `-- NumberNode 100
    `-- TurnCommandNode
        `-- NumberNode 90
```

### Syntax errors detected

| Input | Code | Message |
| --- | --- | --- |
| `MOVE;` | P002 | MOVE needs a distance |
| `REPEAT { MOVE 100; }` | P002 | REPEAT needs a count before `{` |
| `REPEAT 4 MOVE 100;` | P004 | Expected `{` after REPEAT 4 |
| `REPEAT 4 { MOVE 100; TURN 90; }}` | P005 | This `}` does not close any REPEAT block |
| `REPEAT 4 { MOVE 100;` | P006 | The REPEAT block is never closed |
| `MOVE 100 200;` | P007 | MOVE takes exactly one value |
| `PEN BLUE RED;` | P007 | PEN takes exactly one color |
| `MOVE 100` (no `;`) | P003 | Missing `;` after MOVE 100 |
| `MOVEE 100;` | P001 | MOVEE is not a command. Did you mean MOVE? |
| `MOVE BLUE;` | P008 | MOVE expects a number |

---

## Phase 3: Semantic analysis

Semantic analysis checks whether a correctly formed program is **allowed**. Unlike the lexer and parser, it does not stop at the first problem. It walks the whole tree and reports every semantic error and warning together.

### Semantic rules

| Rule | Check | Error code |
| --- | --- | --- |
| S1 | Every command is supported in TGL v1 | S001 |
| S2 | MOVE has a numeric distance | guaranteed by the grammar |
| S3 | MOVE distance is from 0 to 1000 | S003 |
| S4 | TURN has a numeric angle | guaranteed by the grammar |
| S5 | TURN angle is from 0 to 360 degrees | S005 |
| S6 | PEN uses a supported color | S006 |
| S7 | REPEAT count is a whole number from 1 to 100 | S007 |
| S8 | REPEAT nesting is at most 10 levels deep | S008 |
| S9 | REPEAT bodies contain only valid statements | reported by the rules above |
| S10 | Every number is finite | S010 |
| S11 | Every command has the right number of arguments | guaranteed by the grammar |
| S12 | At most 10,000 operations are executed | S012 |
| S13 | Extension commands follow their own rules | S001 |

### Counting operations before running

TGL has only counted loops, so the exact number of operations is known before anything runs. A simple command counts 1, and `REPEAT n { body }` counts n × (operations in the body).

```
REPEAT 100 { REPEAT 100 { MOVE 1; TURN 1; } }   =  100 × 100 × 2  =  20,000   →  error S012
```

A program that is too large is rejected at compile time and never executed.

### Symbol table

TGL has no user variables, so the symbol table records the built-in names a program uses:

- every **command** used, with its signature, how often it is written, how often it will execute, and the lines it appears on;
- every **color** used, with its hex value and how often it is set.

### All limits in one place

| Limit | Value |
| --- | --- |
| MOVE distance | 0 to 1000 |
| TURN angle | 0 to 360 degrees |
| REPEAT count | whole number, 1 to 100 |
| REPEAT nesting depth | 10 |
| Executed operations | 10,000 |
| Token length | 64 characters |
| Program size | 65,536 characters |
| Optimizer rounds | 20 |

---

## Phase 4: Intermediate code generation

The validated AST is converted into a **structured intermediate representation (IR)**: a linear list of simple instructions. Loops are kept as `LOOP_BEGIN` / `LOOP_END` pairs instead of being copied out, which keeps the code small.

| Instruction | Operand | Meaning |
| --- | --- | --- |
| `SET_PEN` | color | set the pen color |
| `MOVE` | distance | move forward |
| `TURN` | angle | turn clockwise |
| `PEN_UP` | none | lift the pen |
| `PEN_DOWN` | none | lower the pen |
| `CLEAR` | none | erase the drawing |
| `LOOP_BEGIN` | count, label | start of a repeated block |
| `LOOP_END` | label | end of the block with the same label |

IR for the square program:

```
#1  SET_PEN     BLUE            line 1
#2  LOOP_BEGIN  4    [L2]       line 2
#3    MOVE      100             line 3
#4    TURN      90              line 4
#5  LOOP_END         [L2]       line 5
```

Every instruction has a unique id and the source line it came from.

---

## Phase 5: Code optimization

The optimizer makes the IR smaller **without changing the drawing**. Six passes run in order and are repeated until a full round changes nothing (a fixed point), with a limit of 20 rounds.

| Pass | Rule | Example |
| --- | --- | --- |
| 1. No-op elimination | Remove `MOVE 0` and turns by 0 or a multiple of 360 | `TURN 360;` → removed |
| 2. Redundant pen state | Remove a PEN, PENUP or PENDOWN that sets what is already set | `PEN RED; PEN RED;` → `PEN RED;` |
| 3. Turn combination | Merge adjacent turns, using mod 360 | `TURN 30; TURN 60;` → `TURN 90;` |
| 4. Move combination | Merge adjacent moves | `MOVE 40; MOVE 60;` → `MOVE 100;` |
| 5. Loop optimization | Unwrap `REPEAT 1`, remove empty loops, flatten a loop that only holds another loop, collapse a loop of one MOVE or TURN | `REPEAT 3 { REPEAT 4 { ... } }` → `REPEAT 12 { ... }` |
| 6. Dead instruction | Remove a pen setting that is overwritten before any MOVE | `PENUP; PENDOWN;` → `PENDOWN;` |

### Safety rules

- Moves are **never** merged across a TURN, a pen color change, PENUP or PENDOWN, CLEAR, or a loop boundary.
- Loops are **never** unrolled in general; they are only expanded in phase 6.
- Inside a loop that changes the pen, the pen state is treated as unknown, so no pen command is removed by mistake.

### Verified by execution

After optimizing, the compiler executes both the original and the optimized code and compares them. They must have the **same final turtle state** (position, heading, pen and color) and the **same visible lines** (after joining straight lines that continue each other in the same color). The result is shown as a **Verified** badge on the Optimizer tab. The "Optimizer demo" sample uses all six passes and goes from **23 instructions to 9**.

---

## Phase 6: Target code generation

The optimized IR is translated into a flat list of **drawing instructions**. This is the only place where loops are expanded.

| Drawing instruction | From IR | Effect |
| --- | --- | --- |
| `SET_COLOR #hex` | `SET_PEN` | color = hex |
| `FORWARD d` | `MOVE` | move d units; add a line if the pen is down |
| `ROTATE_CW a` | `TURN` | heading = (heading + a) mod 360 |
| `LIFT_PEN` | `PEN_UP` | pen up |
| `LOWER_PEN` | `PEN_DOWN` | pen down |
| `CLEAR_CANVAS` | `CLEAR` | hide all lines drawn so far |

The turtle executes these instructions and records its state before and after every step. The drawing sheet and the step player use only this record.

### Trace of the square program

| Step | Instruction | Position after | Heading after |
| --- | --- | --- | --- |
| 1 | SET_COLOR #1971C2 (BLUE) | (400, 300) | 0° |
| 2 | FORWARD 100 | (500, 300) | 0° |
| 3 | ROTATE_CW 90 | (500, 300) | 90° |
| 4 | FORWARD 100 | (500, 400) | 90° |
| 5 | ROTATE_CW 90 | (500, 400) | 180° |
| 6 | FORWARD 100 | (400, 400) | 180° |
| 7 | ROTATE_CW 90 | (400, 400) | 270° |
| 8 | FORWARD 100 | (400, 300) | 270° |
| 9 | ROTATE_CW 90 | (400, 300) | 0° |

The turtle ends at its starting point, so the square closes exactly. A drawing that goes outside the 800 × 600 sheet is still completed, with warning W010.

---

## Error and warning codes

### Lexical errors

| Code | Meaning | Example |
| --- | --- | --- |
| L001 | Invalid character | `MOVE 100 & 50;` |
| L002 | Minus sign without a number | `MOVE - 5;` |
| L003 | Single slash | `MOVE 5 / 2;` |
| L004 | Block comment not supported | `/* note */` |
| L005 | Malformed number | `MOVE 12abc;`, `MOVE .5;`, `MOVE 1.;` |
| L006 | Number has two decimal points | `MOVE 1.2.3;` |
| L007 | Invalid hex color | `PEN #12345;` |
| L008 | Token too long | a name over 64 characters |
| L009 | Program too long | source over 65,536 characters |
| L010 | Empty program | only spaces or comments |

### Syntax errors

| Code | Meaning |
| --- | --- |
| P001 | Unknown command (with a "did you mean" suggestion) |
| P002 | Missing argument |
| P003 | Missing semicolon |
| P004 | Missing `{` after the REPEAT count |
| P005 | Unexpected `}` |
| P006 | REPEAT block never closed |
| P007 | Too many arguments |
| P008 | Wrong kind of argument |
| P009 | Unexpected token |

### Semantic errors

| Code | Meaning | Example |
| --- | --- | --- |
| S001 | Command not available in TGL v1 | `GOTO 10, 20;` |
| S003 | MOVE distance out of range | `MOVE 5000;`, `MOVE -50;` |
| S005 | TURN angle out of range | `TURN 500;`, `TURN -90;` |
| S006 | Unknown color | `PEN INVISIBLE;` |
| S007 | Invalid REPEAT count | `REPEAT 0`, `REPEAT 1.5`, `REPEAT 101` |
| S008 | REPEAT nested too deeply | 11 nested REPEAT blocks |
| S010 | Number is not finite | |
| S012 | Too many operations | more than 10,000 |

### Warnings (the program still compiles)

| Code | Meaning |
| --- | --- |
| W001 | `MOVE 0` does nothing |
| W002 | `TURN 0` or `TURN 360` does not change the heading |
| W004 | Empty REPEAT block |
| W005 | REPEAT block never moves the turtle |
| W006 | Program draws nothing |
| W007 | Same color set twice in a row |
| W010 | Drawing leaves the sheet |

---

## Sample programs

All samples are available from the **Sample** list in the visualiser.

| Sample | Program | Expected result |
| --- | --- | --- |
| Square | `PEN BLUE; REPEAT 4 { MOVE 100; TURN 90; }` | blue square |
| Triangle | `PEN RED; REPEAT 3 { MOVE 120; TURN 120; }` | red triangle |
| Hexagon | `PEN GREEN; REPEAT 6 { MOVE 80; TURN 60; }` | green hexagon |
| Star | `PEN PURPLE; REPEAT 5 { MOVE 150; TURN 144; }` | purple five-pointed star |
| Nested pattern | `PEN BLUE; REPEAT 6 { REPEAT 4 { MOVE 50; TURN 90; } TURN 60; }` | six squares turned 60° apart |
| Multicolor | a red square, then a blue triangle | two shapes in two colors |
| Pen up | a red square, then `PENUP; MOVE 150; PENDOWN;` | turtle moves away without drawing |
| Flower | 36 squares, each turned 10° | 144-line pattern |
| Optimizer demo | uses every optimization pass | 23 IR instructions reduced to 9 |
| Error: distance | `MOVE 5000;` | semantic error S003 |
| Error: color | `PEN INVISIBLE;` | semantic error S006 |
| Error: repeat | `REPEAT 0 { MOVE 100; }` | semantic error S007 |
| Error: syntax | a MOVE without its semicolon | syntax error P003 |
| Error: lexical | `MOVE 100 & 50;` | lexical error L001 |

---

## Testing

The compiler core was tested separately from the user interface.

| Test | What was checked | Result |
| --- | --- | --- |
| Sample programs | All 14 samples give the expected success or error code | passed |
| Closed shapes | Square, triangle, hexagon and star end exactly at (400, 300) facing 0° | passed |
| Edge cases | The cases from the project brief (`MOVE;`, `REPEAT 4 MOVE 100;`, an extra `}`, `MOVE -50`, `TURN 500`, `PEN INVISIBLE`, `REPEAT 1.5`, 11 nested loops and others) each give the right error code | passed |
| Boundaries | MOVE 1000 and REPEAT 100 accepted; MOVE 5000, REPEAT 0 and REPEAT 101 rejected | passed |
| Property-based | 3,000 randomly generated valid programs, each optimized and checked to draw exactly the same as the original | 3,000 of 3,000 passed |

---

## Limitations and future work

**Current limitations**

- No variables, arithmetic, procedures or conditions.
- `TURN` only turns clockwise; negative angles are rejected.
- The parser stops at the first syntax error.
- `SETX`, `SETY`, `GOTO`, `CIRCLE` and `WAIT` are reserved but not enabled.

**Possible extensions**

| Extension | Phases affected |
| --- | --- |
| Variables (`SET size = 50;`) | new tokens and grammar rule; the symbol table becomes a real variable table with scopes |
| Procedures (`TO square ... END`) | parser, symbol table for procedure names, IR call and return |
| `CIRCLE r` and `GOTO x, y` | semantic rules, new IR operations, new drawing instructions |
| Error recovery | parser skips to the next `;` or `}` to report several syntax errors at once |
| New output targets | only phase 6 changes, for example SVG export or G-code for a pen plotter |


