# CSE206 - Computer Organization
## Assignment #1: CPU Emulator Implementation Report

**Prepared by:** Yahya Efe Kuruçay (Student ID: 20220808005)
**Date:** [11/05/2025]

This report explains how I built the CPU emulator for Assignment #1. This is an updated version that I made after the first time I submitted it. It got a lot better because I looked at it more closely, talked about it with others, and worked closely with my roommate, Burak Yalçın (Student ID: [20220808069]).

## 1. Introduction

This report is about how I designed and built a simple CPU emulator. This was for Assignment #1 in the CSE206 Computer Organization course. The main idea of this assignment was to make a pretend version of a basic computer. This computer needed a CPU that could understand certain instructions, a main memory, and a simple type of fast memory called a direct-mapped cache. In this report, I’ll talk about the design choices I made, explain how the main parts work, discuss problems I ran into, and explain why I made the final design choices.

## 2. System Architecture Overview

The emulator I built pretends to be a system with these features, as the assignment described:

* **CPU:** It has an Accumulator (AC) and a Program Counter (PC). These are special storage spots in the CPU. It also has one Comparison Flag, which is like a simple yes/no indicator. All these start at zero.
* **Main Memory:** It has 64 Kilobytes (which is 65,536 bytes) of memory. You can access each byte by its own address.
* **Cache:** This is a small, fast memory of 16 bytes. It's set up as a "direct-mapped" cache with 8 blocks, and each block holds 2 bytes. When the CPU writes data, it writes it to both the cache and the main memory at the same time (this is called "write-through"). If the CPU tries to write to a spot that's not in the cache, space is made for it in the cache first (this is "write-allocate").
* **Instruction Format:** Instructions are 16 bits long. The first 4 bits tell the CPU what to do (opcode), and the next 12 bits give it more information (operand). Instructions are stored in memory and take up 2 bytes each.
* **Byte Ordering:** The system uses "little-endian" byte order. This is important for putting two bytes together to make a 16-bit word, especially when the CPU is getting instructions from memory.

## 3. Core Components Implementation

The emulator is made of a few main Java classes:

* **Memory.java:** This class acts like the 64 KB main memory. It has simple functions to `read(address)` and `write(address, value)` for single bytes and checks to make sure the address is valid. It also loads the program (from a file called `program.txt`) into memory, starting at a specific `loadAddress`.
* **Cache.java:** This class handles the direct-mapped cache. It keeps track of the cache blocks, including whether a block is valid, its tag (to identify it), and the data bytes. All memory access goes through its `read(address, memory)` and `write(address, value, memory)` functions. This class figures out the tag, index, and offset for addresses, knows if data is in the cache (a "hit") or not (a "miss"), replaces blocks when there's a miss, and follows the "write-through" and "write-allocate" rules. It also counts the total number of cache hits and misses.
* **CPUEmulator.java:** This class has the main CPU logic. It holds the PC, AC, and Flag registers. It manages the fetch-decode-execute cycle (how the CPU runs instructions). It talks to the Memory and Cache classes to get instructions and data. It knows how to perform each of the 15 instructions defined for the CPU. It reads settings like `loadAddress` and `initialPC` from a file called `config.txt` and starts the program.
* **Main.java:** This is the starting point of the program. It deals with any commands given when the program starts, sets up the other parts (Memory, Cache, CPUEmulator), and then tells the CPUEmulator to start running.

## 4. Memory and Program Loading

The 64 KB main memory is like a list of bytes inside `Memory.java`. Program instructions, which are given as 16-bit binary numbers in `program.txt`, are loaded into this memory. Each 16-bit instruction is treated as 2 bytes.

When loading the program (which `CPUEmulator.loadProgram` does), each binary number is turned into an integer, then split into two bytes (a low byte and a high byte). These bytes are stored in memory one after the other, starting from `loadAddress`. Because it's "little-endian," the low byte of the instruction is stored at the first address, and the high byte is stored at the next address (address + 1). When the program is first loaded, it's written directly to the Memory, skipping the Cache.

## 5. Instruction Handling and Addressing Modes

Instructions are 16 bits long, with a 4-bit opcode (what to do) and a 12-bit operand (data or address). The emulator can handle all 15 instructions from the assignment. A key part is figuring out what the operand means based on the instruction's "addressing mode":

* **Immediate Addressing:** For instructions like `LOAD`, `ADD`, `SUB`, `MUL`, the 12-bit operand is used directly as a number. Based on sample programs, I treated this number as an unsigned integer (from 0 to 4095).
* **Relative Addressing (for Memory Operands):** For instructions that use memory (like `LOADM`, `STOREM`, `CMPM`, `ADDM`, `SUBM`, `MULM`), the 12-bit operand is an offset from `loadAddress`. The actual memory address is `loadAddress + operand`. Looking at sample programs, it seemed these operations usually work on single bytes (8 bits). So, my program reads or writes only one byte at that address, using the cache's `read()` and `write()` functions.
* **Addressing for PC and Jumps (JMP, CJMP):** The assignment said to use "Absolute (for PC and jumps)" with a 12-bit operand. At first, it was hard to use the 12-bit operand as a direct address in the 64KB memory. The operand values in the sample program (like 6 or 15) didn't make sense as absolute addresses where the program was loaded (around address 0x2000) and often caused the program to go wrong or get stuck in loops.
    After debugging and looking at comments in the sample program (which listed jump targets as small numbers, like line numbers), I figured out that for `JMP` and `CJMP`, the 12-bit operand is an instruction number relative to where the program starts. So, the actual byte address for the PC is calculated as `loadAddress + operand * 2` (because each instruction is 2 bytes long). This way, the sample programs worked correctly.

## 6. Fetch-Decode-Execute Cycle and PC Management

The CPU's main loop, handled by `CPUEmulator.run()`, does the following for each instruction:

* **Fetch:** In each cycle, the CPU gets the 16-bit instruction from the memory location pointed to by the PC. It does this by reading two bytes in a row (at PC and PC+1) using the cache's `read()` method. These two bytes are then combined into one 16-bit instruction, using little-endian order. This makes sure that getting instructions uses the cache and counts towards cache statistics.
* **Decode:** The 16-bit instruction is split into its 4-bit opcode and 12-bit operand.
* **Execute:** A `switch` statement looks at the opcode and runs the code for that specific instruction (like doing math, accessing memory, jumping, input/output, or stopping).

**PC Management:** The PC is updated after each instruction:

* For most instructions (all except `JMP`, `CJMP`, and `HALT`), the PC is increased by 2 (`pc+=2`). This makes it point to the start of the next 16-bit instruction in memory.
* For jump instructions (`JMP`, `CJMP`), the PC is set directly to the calculated target address (`loadAddress + operand * 2`). The usual PC increase is skipped.
* The `HALT` instruction sets a flag that stops the execution loop.

## 7. Cache Implementation Details

* **Organization:** Direct-Mapped, 8 blocks, with 2 bytes in each block.
* **Policies:** Write-Through and Write-Allocate.
* **Address Mapping:** A 16-bit address is divided into a 12-bit Tag, a 3-bit Index, and a 1-bit Offset.
* **Accesses and Statistics:** All times the memory is used – both for getting instructions (reading 2 bytes) and for data (reading/writing 1 byte for `LOADM`, `STOREM`, etc.) – go through the `Cache.read()` and `Cache.write()` methods. So, every access like this is counted for cache hit and miss statistics. The `getHitRatio()` method tells you the percentage of hits out of all these counted accesses.

## 8. Development Process and Challenges Faced

Making the emulator involved understanding what the assignment wanted and then building and testing it bit by bit. Working with Burak was very helpful during this time.

* **Initial Interpretation of Addressing:** At first, I thought jump addresses were simple offsets from `loadAddress` (like, `pc = loadAddress + operand`), and the PC would increase by 1. I tried this because I was having trouble getting the sample program to work correctly with the standard PC increase. This early idea gave the right AC output for the sample program, but I realized it wasn't how a standard CPU works. It also might have broken the rule that "all memory accesses go through the cache," because I might have handled instruction fetching differently at first. The cache hit ratio I got then (which only counted data accesses) was very high (around 98%), which seemed strange.
* **Implementing Standard PC and Fetch:** I realized I needed a more standard and correct emulator. So, I changed the code to increase the PC by 2, calculate jump targets as `loadAddress + operand * 2`, and get instructions from memory through the cache. This was a very important step to make the simulation more accurate.
* **Debugging Incorrect Program Output:** When I first implemented the standard fetch and PC logic, a more complex factorial test program gave the wrong output (AC=0), even though the main CPU logic seemed right. I had to debug step-by-step very carefully, using lots of print statements to see what was happening. This showed that the problem was in the test program itself (the operand for a `MULM` instruction was wrong, making it multiply by zero), not in my emulator's CPU or cache. Fixing the test program solved this.
* **Cache Hit Ratio Interpretation:** Once instruction fetches were correctly going through the cache and being counted, the cache hit ratio for test programs (like the sample sum program or the factorial program) was much lower than the first high value (for example, I saw values around 55-75%, depending on the program). This lower ratio is a more accurate picture of how well the cache performs when all memory use is considered. The exact number really depends on how a specific program uses memory and how the cache interacts with that (including possible clashes when fetching instructions). The final version of my emulator makes sure all memory accesses go through the cache for statistics.

## 9. Conclusion

The CPU emulator I built successfully simulates the system parts and instructions that were required. The tricky parts, especially understanding addressing modes and making sure the cache worked correctly, were solved by working on it step-by-step, debugging carefully, and having helpful talks with my roommate.

The final emulator correctly does the fetch-decode-execute cycle with a standard PC increase. It understands jump targets as instruction numbers relative to the load address. It handles memory operations through the cache, and it accurately calculates cache hit ratios based on all memory accesses (both instructions and data). The emulator successfully runs the sample sum program and other test cases, like the factorial program, and gets the correct math results.

This assignment was hard but very rewarding. It helped me understand computer organization principles much better.