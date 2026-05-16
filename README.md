# Floatfuck - A Brainfuck Specification

## Quick Specification 

Below is a table specifying the attributes most commonly altered between brainfuck variations, and the values for these attribues that are compliant with this spec.

| Specification           | Value                      |
|-------------------------|----------------------------|
| Spec Version            | 1.0.0                      |
| Minimum Cell Count      | 30,000 (30k)               |
| Cell Pointer Wrapping   | None (Undefined Behaviour) |
| Cell Overflow Behaviour | Wrapping (Mod 256)         |
| Cell Format             | Unsigned                   |
| Cell Size               | 8 bits                     |

## Justification

The closest thing to an official brainfuck spec is probably the [esolang article on brainfuck](https://esolangs.org/wiki/Brainfuck). It contains an excellent writeup on the history and conventions of 
brainfuck, its variants, and its many compilers/interpreters. However, for the most aggressive optimisations and analysis we need to be clear about the behavioural expectations of brainfuck-related 
software, so we have to draw a line in the sand about which of the countless conventions from the many brainfuck interpreters/compilers it makes sense to bring forward into future projects. 
This is not _the_ brainfuck standard; the intent of this spec is just to nail down a _specific_ model of brainfuck that maintains its traditional mental model and attempts to match the expectations of modern 
brainfuck enthusiasts.

## The Floatfuck Standard

### Memory Model
Traditionally, brainfuck runtime memory is a contiguous 'ticker-tape' of cells (usually -but not always- 8-bit words), accessable through a single read-write pointer. 
