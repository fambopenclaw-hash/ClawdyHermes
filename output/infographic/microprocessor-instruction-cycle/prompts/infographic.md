Create a professional infographic following these specifications:

## Image Specifications

- **Type**: Infographic
- **Layout**: linear-progression
- **Style**: technical-schematic (Blueprint variant)
- **Aspect Ratio**: 16:9 (landscape)
- **Language**: en (English)

## Core Principles

- Follow the layout structure precisely for information architecture
- Apply style aesthetics consistently throughout
- Keep information concise, highlight keywords and core concepts
- Use ample whitespace for visual clarity
- Maintain clear visual hierarchy
- All text must be in English

## Layout Guidelines — linear-progression

Sequential progression showing steps in a process.

**Structure**:
- Linear arrangement (horizontal, left to right)
- 4 numbered nodes/markers at key points (one per stage)
- Connecting arrows between nodes showing direction
- Clear start (FETCH) and end (WRITEBACK) points
- Directional flow indicators between nodes

**Variants**: Process variant — action steps with numbered sequence and action icons.

**Best For**: Step-by-step tutorials and workflows.

**Visual Elements**:
- Numbered step markers (① ② ③ ④)
- Arrows or connectors showing direction
- Icons representing each step/event
- Consistent node spacing
- A curved return arrow from Stage 4 looping back to Stage 1

**Text Placement**:
- Main title at top
- Stage titles at each node
- Brief descriptions below nodes
- Technical labels and annotations

## Style Guidelines — technical-schematic (Blueprint variant)

Technical diagrams with engineering precision and clean geometry.

**Color Palette**:
- Primary: Blues (#2563EB), teals, grays, white lines
- Background: Deep blue (#1E3A5F) with subtle grid pattern
- Accents: Amber highlights (#F59E0B), cyan callouts

**Visual Elements**:
- Geometric precision throughout
- Grid pattern background
- Dimension lines and technical annotations
- Clean vector shapes
- Consistent stroke weights
- White-on-blue technical blueprint look

**Typography**:
- Technical stencil or clean sans-serif font
- All-caps labels for technical terms
- Measurement-style annotations where appropriate

**Variant**: Blueprint — engineering schematics, white on blue, measurements, grid

---

Generate the infographic based on the content below:

# How a Microprocessor Executes Instructions

## Section 1: Overview — The Instruction Cycle
The microprocessor executes instructions through a repeating 4-stage cycle: FETCH → DECODE → EXECUTE → WRITEBACK. The cycle loops continuously to run programs.

## Section 2: Stage 1 — FETCH (Get the Instruction)
- Program Counter (PC) holds the address of the next instruction
- Address sent to RAM via address bus
- RAM returns instruction (binary) via data bus
- Instruction stored in Instruction Register (IR)
- PC increments (PC = PC + 4)

## Section 3: Stage 2 — DECODE (Understand the Instruction)
- Control Unit (CU) examines bits in the IR
- Instruction has: opcode (what to do) + operands (what to do it on)
- Example: ADD R1, R2, R3 → add R2 and R3, store in R1
- CU activates circuits and routes data to the ALU

## Section 4: Stage 3 — EXECUTE (Do the Work)
- ALU receives values from registers
- Transistors inside ALU form adder circuits (logic gates)
- Computation happens in 1 clock cycle (or a few)
- Result comes out of ALU output
- ALU also sets flag bits (overflow, zero, etc.)

## Section 5: Stage 4 — WRITEBACK (Save the Result)
- Result written back to destination register (R1)
- Control Unit marks instruction as complete
- Cycle loops back to FETCH for next instruction

## Section 6: Speed & Pipelining
- A 3 GHz CPU ticks 3 billion times per second
- 1–30 cycles per instruction depending on complexity
- Modern pipelining: multiple instructions in different stages simultaneously, like an assembly line

Text labels (in en):
- Main Headline: "THE INSTRUCTION CYCLE"
- Subhead: "4 Stages — Repeated Billions of Times Per Second"
- Section 2 labels: "① FETCH", "Program Counter (PC)", "Address Bus", "RAM", "Data Bus", "Instruction Register (IR)"
- Section 3 labels: "② DECODE", "Control Unit (CU)", "Opcode + Operands", "Route to ALU"
- Section 4 labels: "③ EXECUTE", "Arithmetic Logic Unit (ALU)", "Inputs: R2, R3", "Output: Result", "Transistor Circuits"
- Section 5 labels: "④ WRITEBACK", "Destination: Register R1", "Instruction Complete", "← Loop back to FETCH"
- Section 6 labels: "SPEED & PIPELINING", "3 GHz = 3 Billion Cycles/Second", "1–30 Cycles Per Instruction", "Pipeline: Parallel Processing"
