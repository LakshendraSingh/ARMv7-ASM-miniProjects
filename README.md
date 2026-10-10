# ARMv7 Assembly Mini Projects

A collection of small, heavily commented ARMv7 (32-bit ARM) assembly programs written while learning the instruction set. Each file focuses on a single concept: registers, arithmetic, logic, branching, addressing modes, the stack, and basic hardware I/O.

## Contents

| File | Concept | Key instructions |
|------|---------|------------------|
| `learningArmAssembly.asm` | Minimal program: put a value in a register and exit via syscall | `MOV`, `SWI` |
| `arithmeticAndCPSRFlags.asm` | Arithmetic, and which instructions update the CPSR flags | `ADD`, `SUB`, `MUL`, `ADDS`, `SUBS`, `ADC` |
| `logicalOperators.asm` | Bitwise operations and masking | `AND`, `ANDS`, `ORR`, `EOR`, `MVN` |
| `logicaLShiftsAndRotations.asm` | Shifts and rotates, including rotate-left via `ROR` | `LSL`, `LSR`, `ROR` |
| `addressingModes.asm` | Direct, indirect, offset, pre-index and post-index addressing | `LDR Rx, =label`, `[R0]`, `[R0,#4]`, `[R0,#4]!`, `[R0],#4` |
| `conditionsAndBranches.asm` | Comparing values and branching on the result | `CMP`, `BGT`, `BGE`, `BLT`, `BLE`, `BEQ`, `BNE`, `BAL` |
| `conditionalInstructonExecution.asm` | Conditional execution without branching | `CMP`, `ADDLT`, `MOVGE` |
| `LoopsAndBranches.asm` | Summing a word array in a loop with a sentinel value | `LDR`, `CMP`, `BEQ`, `BAL`, `.equ`, `.word` |
| `Functions_BrancheWithLinkRegistersAndReturns.asm` | Calling a subroutine and returning | `BL`, `BX lr` |
| `preservingAndRetreivingDataFromStackMemory.asm` | Saving and restoring registers around a call | `PUSH`, `POP`, `BL`, `BX lr` |
| `hardwareInteractions.asm` | Memory-mapped I/O: read switches, drive LEDs | `LDR`, `STR`, `.equ` |
| `printingStringsToTerminal/helloworld.s` | Hello World using Linux `write` and `exit` syscalls | `SWI 0`, `.asciz` |

## Running the programs

There are two ways to run these, depending on the program.

### Option 1: CPUlator (no setup)

Most of the `.asm` files are best explored step by step in a simulator. [CPUlator](https://cpulator.01xz.net/?sys=arm-de1soc) (ARMv7 / DE1-SoC) lets you paste code, assemble it, and watch registers, flags and memory change as you single-step. `hardwareInteractions.asm` uses the DE1-SoC switch and LED addresses (`0xFF200040` and `0xFF200000`), so it needs this system to do anything visible.

### Option 2: Linux ARM toolchain + QEMU

For `helloworld.s` (and the programs that exit via `SWI 0`), assemble and link with the GNU ARM toolchain, then run under QEMU user-mode emulation.

```sh
# Debian/Ubuntu
sudo apt install binutils-arm-linux-gnueabihf qemu-user

# Build and run
arm-linux-gnueabihf-as -o helloworld.o printingStringsToTerminal/helloworld.s
arm-linux-gnueabihf-ld -o helloworld helloworld.o
qemu-arm ./helloworld
```

Expected output: `hello world`

On a native 32-bit ARM machine (such as a Raspberry Pi running a 32-bit OS), use `as` and `ld` directly and run `./helloworld`.

To step through any program, run it under `gdb-multiarch` with `qemu-arm -g 1234 ./prog` and inspect registers with `info registers`.

## Quick reference

**Syscall convention (Linux, ARM EABI):** syscall number in `R7`, arguments in `R0`–`R2`, then `SWI 0`.

| Syscall | `R7` | Arguments |
|---------|------|-----------|
| `exit`  | 1 | `R0` = exit code |
| `write` | 4 | `R0` = fd (1 = stdout), `R1` = buffer address, `R2` = length |

**Addressing modes** (from `addressingModes.asm`):

| Syntax | Meaning |
|--------|---------|
| `LDR R0, =label` | Load the address of `label` into `R0` |
| `LDR R1, [R0]` | Load the word at address `R0` |
| `LDR R2, [R0,#4]` | Load the word at `R0 + 4` (base unchanged) |
| `LDR R3, [R0,#4]!` | Pre-index: `R0 += 4`, then load from `R0` |
| `LDR R4, [R0],#4` | Post-index: load from `R0`, then `R0 += 4` |

**Rotate left by n:** `ROR Rx, #(32 - n)`

## Notes and known quirks

These are learning snippets, so a few files are intentionally minimal:

- Most programs have no exit sequence and simply end after the last instruction. This is fine for single-stepping in a simulator, but running them natively will fall through into whatever follows in memory. Add `MOV R7, #1` / `SWI 0` to terminate cleanly.
- `conditionsAndBranches.asm` has no branch after each label, so execution falls through the labels in order (as the note at the bottom of the file points out). `R2` ends at `7` regardless of the branch taken.
- `Functions_BrancheWithLinkRegistersAndReturns.asm` has no exit after `MOV R3, #4`, so execution continues into `add2` a second time.
- `LoopsAndBranches.asm` stops when it reads the sentinel `0xAAAAAAAA`, but the data list (`1`–`10`) does not include it. Append `, 0xaaaaaaaa` to the `.word` list so the loop terminates at the end of the array.
- In `addressingModes.asm`, the comments for pre- and post-increment describe when the base register is updated, relative to the load. Pre-index updates it before the load, and post-index updates it after.

## Repository layout

```
.
├── README.md
├── addressingModes.asm
├── arithmeticAndCPSRFlags.asm
├── conditionalInstructonExecution.asm
├── conditionsAndBranches.asm
├── Functions_BrancheWithLinkRegistersAndReturns.asm
├── hardwareInteractions.asm
├── learningArmAssembly.asm
├── logicalOperators.asm
├── logicaLShiftsAndRotations.asm
├── LoopsAndBranches.asm
├── preservingAndRetreivingDataFromStackMemory.asm
└── printingStringsToTerminal/
    └── helloworld.s
```

## Resources

- [ARM Architecture Reference Manual, ARMv7-A/R](https://developer.arm.com/documentation/ddi0406/latest)
- [CPUlator ARMv7 simulator](https://cpulator.01xz.net/?sys=arm-de1soc)
- [Linux ARM syscall table](https://arm.syscall.sh/)
