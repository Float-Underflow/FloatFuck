# Floatfuck - A Brainfuck Specification

### This Specification is a work in progress, and subject to frequent changes!

## Quick Specification 

Below is a table specifying the attributes most commonly altered between brainfuck variations, and the values for these attribues that are compliant with this spec.

| Specification           | Value                      |
|-------------------------|----------------------------|
| Version                 | 1.0.0                      |
| Minimum Cell Count      | 30,000 (30k)               |
| Data Pointer Wrapping   | None (Undefined Behaviour) |
| Cell Overflow Behaviour | Wrapping (Mod 256)         |
| Cell Format             | Unsigned                   |
| Cell Size               | 8 bits                     |

### Quick Commands

| Command | Name          | Behaviour                                                                                                                |
|---------|---------------|--------------------------------------------------------------------------------------------------------------------------|
| `<`     | Move-Left     | Move the data pointer one cell to the left                                                                               |
| `>`     | Move-Right    | Move the data pointer one cell to the right                                                                              |
| `+`     | Increment     | Increase the cell value at the data pointer by one                                                                       |
| `-`     | Decrement     | Decrease the cell value at the data pointer by one                                                                       |
| `,`     | Input         | Receive one byte from stdin and store it in the current cell                                                             |
| `.`     | Output        | Send the value of the current cell to stdout                                                                             |
| `[`     | Jump If Zero  | If the byte at the current cell is zero, jump the intruction pointer forward to the matching ']' command                 |
| `]`     | Jump Not Zero | If the byte at the current cell is non-zero, jump the instruction pointer backward to the matching preceding '[' command |

## Justification

The closest thing to an official brainfuck spec is probably the [esolang article on brainfuck](https://esolangs.org/wiki/Brainfuck). It contains an excellent writeup on the history and conventions of brainfuck, its variants, its many compilers/interpreters, and it provides a spec of it. However, for the most aggressive optimisations and analysis we need to be clear about the behavioural expectations of brainfuck-related software, so we have to draw a line in the sand about which of the countless conventions from the many brainfuck interpreters/compilers it makes sense to bring forward into future projects. 
This is not _the_ brainfuck standard; the intent of this spec is just to nail down a _specific_ model of brainfuck that maintains its traditional mental model and attempts to match the expectations of modern brainfuck enthusiasts while allowing compilers and interpreters implementing the spec to set a clear expecation for behaviour.

## The Floatfuck Standard

### 1. Definitions 
In this section we establish the terminology used throught the rest of the specification.

#### 1.1 Interpretation Phases

##### 1.1.1 Initialisation Phase
The **initialisation phase** is the phase where an interpreter or binary performs the perphieral work necessary to begin interpreting a brainfuck source-code sequence, but excludes any behaviour that is the result of the brainfuck source-code sequence specifically.

##### 1.1.2 Execution Phase
We refer to the **execution phase** as the time period during interpretation/exection of a brainfuck binary where that binary's behaviour is based entirely on the content of the brainfuck source-code sequence it is intended to interpret. 

##### 1.1.3 Cleanup Phase
The **cleanup phase** refers to the sequence of behaviour of a brainfuck binary/interpreter following the execution of a brainfuck source-code sequence. The cleanup phase may cause the binary/interpreter to exit, or may move directly into a new initialisation phase.

#### 1.2 Cell
A **cell** refers to the atomic unit of addressable memory in brainfuck runtime memory, and is subject to the following conditions:

##### 1.2.1 Size Determines Specificity
By default, all cells are assumed to be 8 bits in length. Implementations with differing cell-lengths which otherwise comply with the FloatFuck standard are said to be in compliance with FloatFuck-`x`, where `x` is the cell-length of the implementation in bits. For instance, a FloatFuck-compliant 
compiler with a cell-length of 16 bits is said to be in compliance with FloatFuck-16. 

**FloatFuck** always refers to FloatFuck-8 unless a post-script compliance is specified. An implementation may support any number of post-script compliance options.

##### 1.2.2 Cells must implement `unsigned` integer arithmetic
While a cell may conceptually represent any type of value, on overflow/underflow a cell must conform strictly to the expectations of unsigned integer arithmetic.

#### 1.3 Brainfuck Memory
**Brainfuck memory** (also **brainfuck memory space**, **brainfuck runtime memory**) refers to the region(s) of runtime memory used by brainfuck binaries/interpreters for all read/write operations relating solely to the execution of brainfuck source code sequences. It is subject to the following conditions:

##### 1.3.1 Minimum allocation of 30,000 cells
Brainfuck memory must be allocated a minimum of 30,000 cells.

##### 1.3.2 Read/Write Availability
Brainfuck memory must be both readable and writable for the entire duration of the execution phase.

##### 1.3.3 Allocation must outlast execution
Brainfuck memory must be allocated before entering the execution phase, and must remain allocated until the end of the exeuction phase.

#### 1.4 Data Pointer
There must exist a minimum of one **data pointer** with read/write access to brainfuck memory for the entirety of the execution phase.

#### 1.5 Instruction Pointer

### Memory Model
Traditionally, brainfuck runtime memory is a contiguous 'ticker-tape' of cells (usually 8-bit words), accessable through a single read-write pointer called the "data pointer" <cite>[<sup>1</sup>][1]</cite>. 
```
                            Ticker Tape Memory

                                    |          |
---+----------+----------+----------|----------|----------+----------+
 0 | 01110101 | 10100011 | 00110010 | 00000000 | 11111111 | 01101011 |
---+----------+----------+----------|----------|----------+----------+
                    ^         ^     |          |
                    |         |           ^
     Cells --------------------           |
                                    Data Pointer
```
The data pointer is the only way to read or write program data, meaning that data manipulation within a brainfuck program occurs one cell at at a time.
In order to write to other data locations, the data pointer can be shifted left or right using the Move-Left `<` and Move-Right `>` commands <cite>[<sup>2</sup>][2]</cite>. 

#### Move-Left `<`
A Move-Left will cause the data pointer to point at the cell immediately preceeding the cell it currently points at.
For example, if we apply a Move-Left to the above ticker-tape we get the following:

```
                               <-------- Move Direction

                         |          |
---+----------+----------|----------|----------+----------+----------+
 0 | 01110101 | 10100011 | 00110010 | 00000000 | 11111111 | 01101011 |
---+----------+----------|----------|----------+----------+----------+
                         |          |
                               ^          ^
                               |          \ -------- Old Position
                         Data Pointer
                        (New Position)
```
#### Move-Right `>`
A Move-Right will cause the data pointer to point at the cell immediately following the cell it currently points at.
As an example, if we apply a Move-Right to our original ticker-tape we get this:

```
                              Move Direction -------->
        
                                               |          |
---+----------+----------+----------+----------|----------|----------+
 0 | 01110101 | 10100011 | 00110010 | 00000000 | 11111111 | 01101011 |
---+----------+----------+----------+----------|----------|----------+
                                               |          |
                                         ^           ^
                   Old Position -------- /           |
                                               Data Pointer
                                              (New Position)
```
#### Addressing The Edge-Case
A problem can arise when the data pointer is pointing to the first or last cell of the ticker tape, where a Move operation in the wrong direction would cause the pointer to go out-of-bounds. For example, a Move-Left when the data pointer points to the first cell results in the following situation where the data pointer is now addressing past the ticker-tape entirely and into memory from the shadow-realm<cite>[<sup>3</sup>][3]</cite>:
```
Cell index: |     ?    |     0          1          2          3          4      
            |----------|----------+----------+----------+----------+----------+----
            | 01010101 | 01110101 | 10100011 | 00110010 | 00000000 | 11111111 | ...
            |----------|----------+----------+----------+----------+----------+----
            |          |
                  ^
                  \ ------- Data Pointer
```
FloatFuck considers dereferencing the data pointer in this situation to be undefined behaviour. Compilers following the spec may allow for recovery or utilise out-of-bounds addresses for optimisation. However, users must not assume the currently executing program is recoverable if the data pointer is dereferenced while out of bounds.


### Data Values
Brainfuck provides exactly two commands for changing data values, Increment '+' and Decrement '-', which affect the cell addressed by data pointer.

#### Increment `+`
**Technical Description:** Performs an arithmetic add modulo 256 between 1 and the value obtained by dereferencing the data pointer, then stores the result in the cell referenced by the data pointer.

**Human Description:** Adds 1 to the current cell.

For example, here is the result of performing consecutive increments on a cell starting with value `0`:
```
+----------+  Increment  +----------+  Increment  +----------+  Increment  +----------+  Increment
| 00000000 | ----------> | 00000001 | ----------> | 00000010 | ----------> | 00000011 | ----------> ...
+----------+             +----------+             +----------+             +----------+
```
#### Decrement `-`
**Technical Description:** Performs an arithmetic subtract modulo 256 of 1 from the value obtained by dereferencing the data pointer, then stores the result in the cell referenced by the data pointer.

**Human Description:** Subtracts 1 from the current cell.

For example, here is the result of performing consecutive decrements on a cell starting with value `5`:
```
+----------+  Decrement  +----------+  Decrement  +----------+  Decrement  +----------+  Decrement
| 00000100 | ----------> | 00000011 | ----------> | 00000010 | ----------> | 00000001 | ----------> ...
+----------+             +----------+             +----------+             +----------+
```
#### Overflow and underflow

There are implementations of brainfuck in which 'wrapping' a cell (ie, performing a mod 256 following an increment or decrement) is not a feature, meaning that overflow or underflow of the cell causes a runtime error. In most implementations, wrapping is the default behaviour. Because non-wrapping logic is a strict subset of wrapping logic, any wrapping implementation can also perform non-wrapping logic. In order to capture as much compatibility as possible, FloatFuck therefore requires wrapping logic in all implmentations.

### I/O

There are two commands for I/O in traditional brainfuck, Input ',' and Output '.'

#### Input `,`
**Technical Description:** Receives a single byte from stdin and stores the result in the cell referenced by data pointer.
**Human Description:** Copies a character from stdin to the current cell.

#### Output `.`
**Technical Description:** Sends the value obtained by dereferencing the data pointer to stdout.
**Human Description:** Outputs the character at the current cell.

### Control Flow

### Jump If Zero `[`

### Jump Not Zero `]`

[1]: https://en.wikipedia.org/wiki/Brainfuck#Language_design
[2]: https://esolangs.org/wiki/Brainfuck#Language_overview
[3]: https://yugioh.fandom.com/wiki/Shadow_Realm
