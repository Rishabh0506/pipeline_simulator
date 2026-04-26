# Pipeline Hazard Simulator

An interactive, web-based educational simulator designed to visualize instruction scheduling and data hazards in classic RISC architectures. This tool helps visualize how 4-stage and 5-stage pipelines handle Read-After-Write (RAW) data hazards both with and without data forwarding (bypassing).

## Features

- **Dual Architecture Support**: Toggle between a standard 4-stage pipeline (IF, ID, EX, MEM/WB) and a classic 5-stage MIPS pipeline (IF, ID, EX, MEM, WB).
- **Hazard Detection & Stalls**: Automatically detects RAW hazards and inserts the correct stall cycles (bubbles) into the pipeline grid according to standard academic rules.
- **Data Forwarding**: Simulates ALU-to-ALU and MEM-to-ALU bypass paths, correctly mitigating penalties and accurately visualizing the mandatory 1-cycle stall for Load-Use hazards.
- **Textbook Accuracy**: Replicates nuanced pipeline behaviors, including sequential instruction fetching for 4-stage pipelines and synchronous fetch-decode stalls for 5-stage pipelines.
- **Interactive Stepping**: Features a "Step in Detail" mode that walks through the pipeline cycle-by-cycle, providing a narrative log that explains exactly *why* an instruction is stalled or forwarded at any given moment.
- **Modern UI**: A clean, responsive, and aesthetic dark-mode interface built entirely with a single HTML/JS/CSS file.

## Getting Started

Because this simulator is a standalone, client-side web application, there are no dependencies, servers, or build steps required.

1. Clone or download this repository to your local machine.
2. Open `pipeline_simulator.html` in any modern web browser (Google Chrome, Firefox, Safari, Edge).
3. Configure your pipeline settings, add instructions via the left panel, and click **Run** to generate the pipeline schedule!

## Supported Instructions

The simulator supports a core subset of operations commonly used to demonstrate hazard scheduling:

- **ALU Operations**: `ADD`, `SUB`
- **Memory Operations**: `LW` (Load Word), `SW` (Store Word)

---

## Simulator Assumptions & Rules

To ensure predictable and textbook-accurate results, the simulator enforces specific rules regarding pipeline timing, hardware structure, and hazard detection.

### 1. Instruction Set & Processing
*   **Linear Execution**: The simulation assumes a perfectly linear stream of instructions. **Control hazards** (Branches, Jumps) are not simulated, meaning the Program Counter (PC) always increments sequentially.
*   **Immediate Values**: Immediate values and offsets are assumed to be readily available within the instruction word without requiring extra fetch cycles or causing subsequent stalls.

### 2. Registers & Hardware
*   **Register Set**: Supported registers are limited to `R0`-`R31`.
*   **No $zero Register**: The simulator treats all registers identically; it does not simulate a hardwired `$zero` register (meaning writing to it will still trigger hazards if read subsequently).
*   **No Structural Hazards**: The pipeline assumes a Harvard architecture (separate Instruction and Data memories) and sufficient register file ports. Thus, no structural hazards ever occur (e.g., IF and MEM stages never collide over memory access).

### 3. Pipeline Timing & Stages
*   **Fixed Stage Duration**: Every stage (IF, ID, EX, MEM, WB) takes exactly 1 clock cycle. There are no cache misses, page faults, or variable-latency ALU operations.
*   **Split-Phase Register Access**: It is assumed that register writes occur in the *first half* of the clock cycle, and register reads occur in the *second half*. Therefore, an instruction in the `ID` stage can read a register without stalling if the producing instruction is simultaneously in the `WB` stage.
*   **Fetch Conventions**: 
    - The **4-stage pipeline** assumes an instruction queue where instructions are fetched sequentially every cycle, regardless of downstream pipeline stalls. 
    - The **5-stage pipeline** assumes a tightly coupled fetch-decode unit, meaning an instruction is only fetched concurrently with the cycle that the previous instruction successfully completes `ID`.

### 4. Hazard Detection & Resolution
*   **Only RAW Hazards**: The simulator strictly checks for **Read-After-Write (RAW)** data hazards. Write-After-Write (WAW) and Write-After-Read (WAR) hazards are implicitly ignored, which is architecturally correct for a purely in-order scalar pipeline.
*   **Stall Placement Conventions**: To match standard textbook architectures:
    - The **4-stage pipeline** places data hazard stalls *after* `ID` (between `ID` and `EX`). 
    - The **5-stage pipeline** asserts hazard detection early and stalls the decode stage itself, placing data hazard stalls *before* `ID`. 
    Bubble (`STALL`) cycles are dynamically inserted into the UI to accurately reflect these delays.

### 5. Data Forwarding (Bypassing)
*   **Forwarding Paths**: When data forwarding is enabled, the pipeline features bypass paths from the end of the `EX` stage (ALU output) and the end of the `MEM` stage (Data Memory output) directly into the ALU inputs of the `EX` stage for the consumer instruction.
*   **Load-Use Penalty**: Even with full forwarding enabled, a "load-use" hazard (an ALU operation depending on an immediately preceding `LW`) still incurs a mandatory 1-cycle stall. This is because the memory data is only available at the end of the `MEM` stage, which is too late to feed into the beginning of the simultaneous `EX` stage.
*   **No Register File Update Needed**: Forwarded data is consumed directly by the ALU; the pipeline does not wait for the register file to be updated before execution proceeds.

