# C Two-Pass Assembler (12-bit Architecture)

> **⚠️ Academic Integrity Disclaimer:**
> *The source code for this academic project is kept private. This document outlines the system architecture, underlying data structures, and engineering design implemented in the project. Access to the source code may be granted individually by invitation.*

## 📌 Project Overview

This project is a Two-Pass Assembler and Pre-Assembler developed in **ANSI C (C90-compatible)** and built and tested in a **Linux (Ubuntu)** environment. It translates assembly instructions for a custom **12-bit machine architecture** with eight registers and a memory space of 4,096 words.

The software processes assembly source files (`.as`), expands macros, parses instructions, resolves symbols and memory addresses, and generates machine-code and symbol files.

## 🗂️ Modules & File Structure

To maintain separation of responsibilities, the software is divided into dedicated functional modules. The accompanying header files declare the interfaces shared between these modules:

- **main.c**: The entry point that coordinates the assembly pipeline for each input file.
- **pre_assembler.c & macro_utils.c**: Identify macro definitions, validate names, and expand macro calls inline.
- **first_pass.c & first_pass_utils.c**: Handle line parsing, directives, label collection, memory mapping, and initial instruction encoding.
- **second_pass.c**: Resolves references and generates `.ob` output and optional `.ent` and `.ext` files.
- **assembler.c & macro_table.c**: Implement workspace management, dynamic symbol/code/data arrays, and linked-list macro storage.
- **instruction_utils.c & utils.c**: Provide operand parsing, addressing validation, bitwise encoding, and shared string utilities.
- **errors.c**: Provides shared functions for reporting errors and warnings using the workspace's current line counter.
- **assembler.h & globals.h**: Define core data structures, architecture constants, and shared encoding definitions.

## ⚙️ System Architecture (The Assembly Pipeline)

The assembly process is divided into three phases to handle macro expansion, forward references, and memory mapping.

### 1. Pre-Assembler (Macro Deployment)

- **Operation:** Scans source code for macro definitions delimited by `mcro` and `mcroend`.
- **Implementation:** Uses a dynamic **Macro Table** to store names and instruction blocks. Calls to previously defined macros are expanded inline, generating an `.am` file.

### 2. First Pass (Symbol Mapping & Initial Encoding)

- **Operation:** Reads the `.am` file line by line to construct the code and data images.
- **Implementation:**
  - Identifies labels and calculates addresses using the Instruction Counter (IC) and Data Counter (DC), relocating data after the code section. Code starts at address 100.
  - Validates syntax, addressing modes, and operand counts, and processes data, string, entry, and extern directives.
  - Populates the **Symbol Table**, encodes known instruction and operand words, and reserves placeholders for unresolved references.

### 3. Second Pass (Symbol Resolution & Output Generation)

- **Operation:** Completes unresolved references in the in-memory code image.
- **Implementation:**
  - Resolves forward references by looking up addresses in the Symbol Table.
  - Calculates relative offsets, handles external-symbol uses, and checks entry declarations.
  - Completes machine-word values and A/R/E markers before generating the output files.

## 🧠 Data Structures & Memory Management

The assembler combines dynamic arrays and linked lists with explicit memory allocation. The workspace keeps each input file's counters and allocated storage together independently:

- **Dynamic Arrays:** Store the Symbol Table and code/data images. Capacities double through `realloc` when additional storage is required. Each label record tracks its address, symbol type, and entry status.
- **Linked Lists:** Implement the Macro Table, with each node storing a name, dynamically allocated macro body, and pointer to the next node.
- **Memory Allocation & Cleanup:** Uses `malloc`, `realloc`, and `free`. Dedicated cleanup functions release macro storage after preprocessing and the assembly workspace after file processing.

## 🛠️ Key Engineering Implementations

- **Bitwise Manipulation:** Uses `<<`, `|`, and `&` to build instruction fields and retain 12-bit values, including negative values represented in Two's Complement.
- **Error Handling:** Checks invalid addressing modes, duplicate or undefined labels, numeric ranges, and memory limits, and reports syntax and semantic errors through diagnostic messages. The first pass continues scanning after ordinary line errors to collect further diagnostics.
- **Custom String Parsing:** Uses string-processing helpers to tokenize lines, handle whitespace, extract operands, and validate comma placement and label names.

## 📁 Output Files

Upon successful assembly, the system generates the following files when applicable:

1. `*.ob` (Object File): Contains decimal addresses, three-digit hexadecimal machine words, and A/R/E markers.
2. `*.ent` (Entries): Lists exported symbols and their resolved addresses.
3. `*.ext` (Externals): Lists external symbols and the addresses where they are used.
