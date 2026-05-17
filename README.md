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

The closest thing to an official brainfuck spec is probably the [esolang article on brainfuck](https://esolangs.org/wiki/Brainfuck). It contains an excellent writeup on the history and conventions of 
brainfuck, its variants, and its many compilers/interpreters. However, for the most aggressive optimisations and analysis we need to be clear about the behavioural expectations of brainfuck-related 
software, so we have to draw a line in the sand about which of the countless conventions from the many brainfuck interpreters/compilers it makes sense to bring forward into future projects. 
This is not _the_ brainfuck standard; the intent of this spec is just to nail down a _specific_ model of brainfuck that maintains its traditional turing-machine mental model and attempts to match the expectations of modern brainfuck enthusiasts.

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
The data pointer is the only way to read or write program data, meaning that data manipulation within a brainfuck program occurs one byte at at a time.
In order to write to other data locations, the data pointer can be shifted left or right using the Move-Left `<` and Move-Right `>` commands <cite>[<sup>2</sup>][2]</cite>. 

#### Move-Left
If we apply a Move-Left to the above ticker-tape, we get the following:

```
                            <-------- Shift Direction

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
#### Move-Right
Alternatively, if we apply a Move-Left to our original ticker-tape, we get this:

```
                            Shift Direction -------->
        
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
#### What happens at the edges?
Consider the following situation, where the data pointer is pointing at the first cell of a ticker tape:
```
Cell index: |     0    |     1          2          3          4          5      
            |----------|----------+----------+----------+----------+----------+----
            | 01110101 | 10100011 | 00110010 | 00000000 | 11111111 | 01101011 | ...
            |----------|----------+----------+----------+----------+----------+----
            |          |
                  ^
                  \ ------- Data Pointer
```
A Move-Left would result in this:
```
Cell index: |     ?    |     0          1          2          3          4      
            |----------|----------+----------+----------+----------+----------+----
            | 01010101 | 01110101 | 10100011 | 00110010 | 00000000 | 11111111 | ...
            |----------|----------+----------+----------+----------+----------+----
            |          |
                  ^
                  \ ------- Data Pointer
```


[1]: https://en.wikipedia.org/wiki/Brainfuck#Language_design
[2]: https://esolangs.org/wiki/Brainfuck#Language_overview
