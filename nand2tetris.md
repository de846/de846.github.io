# nand2tetris (July 5th, 2020)

This project was great! I started this blog after [completing](https://www.coursera.org/account/accomplishments/records/WQJTNU9NNA5E) part 1 of [the course](https://www.coursera.org/learn/build-a-computer), so will have to draw my experience from memory and code.

By far, the most rewarding part of this course when you actual build a [working CPU](https://github.com/de846/nand2tetris/blob/master/projects/05/CPU.hdl). I probably spent upwards of 20 hours to test and complete the CPU. Building the [ALU](https://github.com/de846/nand2tetris/blob/master/projects/02/ALU.hdl) before this was also really rewarding (and I remember thinking _that_ one was hard!)

The most interesting part for me, which fulfilled a lifelong goal, was to build an [assembler](https://github.com/de846/nand2tetris/tree/master/projects/06/HackAssembler) for the "assembly" code used in the nand2tetris course and translate it to machine code (to be run on the CPU I built earlier.) The assembler is comprised of three main classes:
1. The `HackParser` class which reads the assembly and maintains a cursor, determines the instruction type, parses the instruction based on the type, and has a method I implemented to parse and return all of the parsed assembly code.
2. The `HackTranslator` class which implements the maps from the parsed tokens to the resulting machine code sequences, and performs the actual translated statement to return.
3. The `HackSymbolTable` class which defines the given registers (R0 through R15, SCREEN, KBD, SP, LCL, ARG, THIS, and THAT). This is also the class that is injected into the Parser to create a new entry when a user-defined label (representing a memory address) in the assembly code is used. The Translator is responsible for getting/setting variables from and to the SymbolTable.

I used TDD (Test Driven Development) to develop the assembler, which really assisted in reducing the amount of bugs when I was developing this, and helped when debugging the few that popped up. One of the most frustrating bugs was a typo I made when populating the symbol table. I unintentionally used the same value for a `JEQ` and `JGE` instruction, which appeared once in a program I was trying to translate, and then looked at the diff in the machine code lines and referenced it back to the assembly code to figure that out.

I'm going to take a break on this for a while, and will probably start and complete part 2 of the course at some point in the future.
