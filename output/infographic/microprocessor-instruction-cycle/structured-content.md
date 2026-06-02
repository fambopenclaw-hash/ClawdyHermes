# How a Microprocessor Executes Instructions

## Overview
A step-by-step breakdown of the 4-stage Instruction Cycle — Fetch → Decode → Execute → Writeback — that a microprocessor performs billions of times per second to run programs.

## Learning Objectives
The viewer will understand:
1. The four stages of the instruction cycle and what happens at each stage
2. How CPU components (PC, IR, Control Unit, ALU, Registers) work together
3. How the cycle loops to execute a complete program, and how pipelining boosts speed

---

## Section 1: Overview — The Instruction Cycle

**Key Concept**: The microprocessor executes instructions through a repeating 4-stage cycle: Fetch → Decode → Execute → Writeback.

**Content**:
- Stage 1: FETCH — Get the instruction from RAM
- Stage 2: DECODE — Understand what the instruction means
- Stage 3: EXECUTE — Do the math or logic
- Stage 4: WRITEBACK — Save the result
- The cycle then loops back to Fetch for the next instruction

**Visual Element**: A 4-step flow diagram arranged horizontally, with arrows pointing from each stage to the next, and a curved arrow looping from Writeback back to Fetch.

**Text Labels**:
- Headline: "THE INSTRUCTION CYCLE"
- Subhead: "4 Stages — Repeated Billions of Times Per Second"
- Stage labels: "① FETCH", "② DECODE", "③ EXECUTE", "④ WRITEBACK"

---

## Section 2: Stage 1 — FETCH

**Key Concept**: The CPU reads the next instruction from memory (RAM).

**Content**:
- The Program Counter (PC) holds the memory address of the *next* instruction
- That address is sent to RAM via the address bus
- RAM returns the instruction data (a string of 0s and 1s) via the data bus
- The instruction is stored in the Instruction Register (IR)
- The PC increments to point to the next instruction (PC = PC + 4)

**Visual Element**: Diagram showing the PC sending an address to RAM via the address bus, and the instruction flowing back via the data bus into the IR.

**Text Labels**:
- Headline: "① FETCH"
- Subhead: "Read from Memory"
- Labels: "Program Counter (PC)", "Address Bus", "RAM", "Data Bus", "Instruction Register (IR)"

---

## Section 3: Stage 2 — DECODE

**Key Concept**: The Control Unit interprets the instruction to determine what operation to perform and which data to use.

**Content**:
- The Control Unit (CU) looks at the bits in the Instruction Register
- The instruction has two parts: an opcode (what to do) and operands (what to do it on)
- For `ADD R1, R2, R3` — opcode = ADD, operands = registers R2, R3 as inputs, R1 as destination
- The CU activates the appropriate circuits and routes data to the ALU
- The CU tells the ALU: "Get ready to add these two values"

**Visual Element**: The Control Unit examining bits from the IR, then routing register values R2 and R3 toward the ALU.

**Text Labels**:
- Headline: "② DECODE"
- Subhead: "Interpret the Instruction"
- Labels: "Control Unit (CU)", "Instruction Register (IR)", "Opcode + Operands", "Route to ALU"

---

## Section 4: Stage 3 — EXECUTE

**Key Concept**: The Arithmetic Logic Unit (ALU) performs the actual computation using transistor circuits.

**Content**:
- The ALU receives the values from registers R2 and R3
- Transistors inside the ALU form an adder circuit (half-adders and full-adders made from logic gates)
- The addition happens in one clock cycle (or a few, depending on complexity)
- The result (e.g., R2=5, R3=7 → result=12) comes out of the ALU's output
- The ALU also sets flag bits (overflow? zero result?)

**Visual Element**: The ALU with inputs R2 and R3 flowing in, and the result flowing out. A magnified callout shows transistors forming logic gates inside the ALU.

**Text Labels**:
- Headline: "③ EXECUTE"
- Subhead: "Do the Computation"
- Labels: "Arithmetic Logic Unit (ALU)", "Inputs: R2, R3", "Output: Result", "Transistor Circuits"

---

## Section 5: Stage 4 — WRITEBACK

**Key Concept**: The computed result is saved to the destination register.

**Content**:
- The result (12) is written back into register R1
- The Control Unit marks this instruction as complete
- The CPU loops back to Stage 1 (Fetch) for the next instruction

**Visual Element**: The result value flowing from the ALU output into register R1. An arrow loops back from this stage to Stage 1 (Fetch) to show the cycle.

**Text Labels**:
- Headline: "④ WRITEBACK"
- Subhead: "Save the Result"
- Labels: "Destination: Register R1", "Instruction Complete", "← Loop back to FETCH"

---

## Section 6: Real-World Speed & Pipelining

**Key Concept**: Modern CPUs execute instructions at astonishing speeds using pipelining.

**Content**:
- A 3 GHz CPU ticks 3 billion times per second
- A single instruction might take 1 to ~30 cycles depending on complexity
- Modern CPUs use pipelining — while one instruction is executing, the next is already being decoded, and the one after that is being fetched. Multiple instructions are in different stages simultaneously, like an assembly line.

**Visual Element**: A pipeline diagram showing 4 instructions at different stages simultaneously — Instruction A at Writeback, B at Execute, C at Decode, D at Fetch.

**Text Labels**:
- Headline: "SPEED & PIPELINING"
- Subhead: "Assembly-Line Efficiency"
- Stats: "3 GHz = 3 Billion Cycles/Second", "1–30 Cycles Per Instruction"
- Concept: "Pipeline: Parallel Processing"

---

## Data Points (Verbatim)

### Statistics
- "A 3 GHz CPU ticks 3 billion times per second"
- "A single instruction might take 1 to ~30 cycles depending on complexity"
- "3 billion times per second"

### Key Terms
- **Program Counter (PC)**: Holds the memory address of the next instruction
- **Instruction Register (IR)**: Stores the current instruction being executed
- **Control Unit (CU)**: Decodes instructions and coordinates components
- **ALU (Arithmetic Logic Unit)**: Performs arithmetic and logic operations
- **Pipeline**: Multiple instructions in different stages simultaneously, like an assembly line

---

## Design Instructions

### Style Preferences
- Technical-schematic: Blueprint aesthetic with white-on-blue or dark background with grid
- Engineering precision, clean geometry

### Layout Preferences
- Linear-progression: Horizontal flow from left to right with numbered stages
- Clear start and end points with directional arrows

### Other Requirements
- Educational, clear, technically accurate
- All text in English
- Font: Clean sans-serif, technical stencil feel
- Color palette: Blues (#2563EB), teals, grays, white lines, deep blue background (#1E3A5F), amber highlights (#F59E0B)