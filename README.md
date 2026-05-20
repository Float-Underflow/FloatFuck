# Floatfuck - A Brainfuck Specification

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

## Justification

The closest thing to an official brainfuck spec is probably the [esolang article on brainfuck](https://esolangs.org/wiki/Brainfuck). It contains an excellent writeup on the history and conventions of brainfuck, its variants, its many compilers/interpreters, and it provides a spec of it. However, for the most aggressive optimisations and analysis we need to be clear about the behavioural expectations of brainfuck-related software, so we have to draw a line in the sand about which of the countless conventions from the many brainfuck interpreters/compilers it makes sense to bring forward into future projects. 
This is not _the_ brainfuck standard; the intent of this spec is just to nail down a _specific_ model of brainfuck that maintains its traditional mental model and attempts to match the expectations of modern brainfuck enthusiasts while allowing compilers and interpreters implementing the spec to set a clear expecation for behaviour.

## The Floatfuck Standard

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
**Technical:** Increment performs an arithmetic add modulo 256 between 1 and the value obtained by dereferencing the data pointer, then stores the result in the cell referenced by the data pointer.

**Human:** Adds 1 to the current cell.

For example, here is the result of performing consecutive increments on a zero-cell:
```
+----------+  Increment  +----------+  Increment   +----------+  Increment  +----------+  Increment
| 00000000 | ----------> | 00000001 | -----------> | 00000010 | ----------> | 00000011 | -----------> ...
+----------+             +----------+              +----------+             +----------+
```
#### Decrement `-`
Decrement performs identically to Increment except that it subtracts 1 instead of adding.
For example, here is the result of performing consecutive decrements on a cell with value `5`:
```
+----------+  Decrement  +----------+  Decrement   +----------+  Decrement  +----------+  Decrement
| 00000100 | ----------> | 00000011 | -----------> | 00000010 | ----------> | 00000001 | -----------> ...
+----------+             +----------+              +----------+             +----------+
```
#### Overflow and underflow



[1]: https://en.wikipedia.org/wiki/Brainfuck#Language_design
[2]: https://esolangs.org/wiki/Brainfuck#Language_overview
[3]: https://yugioh.fandom.com/wiki/Shadow_Realm
