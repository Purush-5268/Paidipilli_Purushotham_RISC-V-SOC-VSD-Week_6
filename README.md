# 🚀 VSDIAT Week 6, Day 1: Inception of OpenLANE and Sky130

## 📝 Overview

This document details the learnings and lab work from Day 1 of Week 6, focusing on the foundational concepts of open-source EDA tools. This session provides a comprehensive introduction to the **OpenLANE** automated RTL-to-GDSII flow, the **Sky130 PDK**, and the fundamental components of modern System-on-Chip (SoC) design.

-----

## 1\. 💡 Inception of Open Source EDA, OpenLane, and Sky130 PDK

### 1.1 🖥️ How to Talk to Computers

This section covered the hierarchy of translating human-readable software applications into physical hardware operations. A high-level language like C++ or Java is first compiled into **assembly language**, which is a low-level, human-readable representation of machine instructions. An **assembler** then translates this into binary **machine code** (the 1s and 0s) that the hardware can directly execute.

> [Image: Flow.jpg]

### 1.2 📦 Introduction to QFN-48 Package, Chip, Pads, Core, Die, and IPs

A chip is not just the silicon die but a complete system integrated into a package. We explored the physical anatomy of a chip.

  * **Package:** The protective casing for the silicon die. We examined the **QFN-48 (Quad Flat No-leads)** package, a common 7x7mm standard.
  * **Die:** The small, square piece of silicon containing the entire integrated circuit.
  * **Core:** The central area of the die where the primary logic (like the CPU) is placed.
  * **Pads:** Conductive points on the die's periphery that act as the interface between the internal signals of the die and the external pins of the package.
  * **IPs (Intellectual Properties):** Reusable blocks of logic or circuitry.
      * **Foundry IPs:** Provided by the fabrication plant (e.g., ADC, DAC, SRAMs).
      * **Macros:** IPs that you design or acquire (e.g., RISC-V SoC, SPI controller).

> [Image: Day - 1 Arduino proccesor.jpg]
>
> [Image: SOC Day-1.jpg]

### 1.3 🏛️ Introduction to RISC-V

**RISC-V** is an open-source **Instruction Set Architecture (ISA)**. Unlike proprietary ISAs like ARM, RISC-V is free to use, modify, and extend.

  * It acts as the fundamental "architecture" of a computer, defining the set of instructions the processor can execute.
  * This open standard is a perfect match for an open-source EDA flow, allowing for truly open-source hardware design from the ISA all the way to the physical layout.

> [Image: Stop watch - Riscv.jpg]
>
> [Image: Flow Riscv.jpg]

### 1.4 🔄 From Software Applications to Hardware

We traced the path from a C code application (like a stopwatch) down to the hardware that executes it:

1.  **Application Software (C Code)**
2.  **Compiler:** Converts C code into platform-specific assembly language.
3.  **Assembler:** Converts assembly language into binary machine code (the 1s and 0s).
4.  **Hardware:** The physical RISC-V processor, built from logic gates, fetches and executes these binary instructions.

-----

## 2\. 🧱 SoC Design and OpenLane

### 2.1 🧩 Introduction to Components of Open-Source Digital ASIC Design

A successful open-source ASIC design flow relies on three key components working together:

1.  **RTL IPs:** The Verilog/VHDL code describing the chip's logic.
2.  **EDA Tools:** The software suite that synthesizes, places, and routes the design (e.g., OpenLANE, Yosys, OpenROAD, Magic).
3.  **PDK (Process Design Kit):** The "rulebook" from the foundry (e.g., SkyWater) that bridges the design and the physical fabrication process.

💡 **Process Design Kit (PDK) Explained**

The PDK is the critical interface between the designer and the fabrication plant (FAB). It contains:

  * **Process Design Rules:** Rules for DRC (Design Rule Check), LVS (Layout vs. Schematic), and PEX (Parasitic Extraction).
  * **Device Models:** Digital standard cell libraries (SCL), I/O libraries, and SPICE models that describe the electrical behavior of transistors.
  * **Technology Files:** Information on metal layers, densities, etc., used by EDA tools.

The open-source `github.com/google/skywater-pdk` is the foundation for this flow.

### 2.2 🗺️ Simplified RTL2GDS Flow

The **RTL-to-GDSII** flow is the entire automated process of turning Verilog code into a final layout file (GDSII) ready for manufacturing.

The high-level steps are:
**RTL -\> Synthesis -\> Floor & Power Planning -\> Placement -\> Clock Tree Synthesis -\> Routing -\> Sign-Off -\> GDSII**

#### a) Synthesis

Converts the abstract RTL (Verilog) into a circuit of components (logic gates, flip-flops) from the PDK's **Standard Cell Library (SCL)**.

> [Image: Synthesis.jpg]

#### b) Floor & Power Planning

  * **Floorplanning:** Partitions the chip die, places the I/O pads, and defines the core boundaries. It also involves placing large macros.
  * **Power Planning:** Creates the **Power Distribution Network (PDN)**. This involves adding power rings and straps (VDD, VSS) to deliver power to all cells with minimal resistance.

#### c) Placement

Places all the standard cells from the synthesized netlist onto the floorplan rows.

  * **Global Placement:** Finds an optimal, but not necessarily legal (i.e., cells can overlap), position for all cells.
  * **Detailed Placement:** Adjusts cell positions to ensure they are legally placed (no overlaps) while honoring the global placement logic.

#### d) Clock Tree Synthesis (CTS)

Builds a dedicated network (a "tree") to deliver the clock signal to all sequential elements (like flip-flops) with minimum and balanced delay (**skew**). This is often done using **H-Trees** or **X-Trees**.

> [Image: CTS - Design Flow.png]

#### e) Routing

Implements the physical wire interconnects between all the cells using the available metal layers (Sky130 has 6 metal layers).

  * **Global Routing:** Generates a high-level guide for the routing paths.
  * **Detailed Routing:** Implements the actual metal wires and vias based on the guide.

#### f) Sign-Off

The final verification stage to ensure the design is manufacturable and functional.

  * **Physical Verification:**
      * **DRC (Design Rule Check):** Ensures the layout doesn't violate any foundry rules (e.g., wire spacing, width).
      * **LVS (Layout vs. Schematic):** Confirms that the physical layout perfectly matches the original synthesized netlist.
  * **Timing Verification:**
      * **STA (Static Timing Analysis):** Verifies that the design meets all timing requirements (setup and hold times) and will run at the target clock speed.

### 2.3 🏎️ Introduction to OpenLANE and Strive Chipsets

**OpenLANE** is an automated script-based flow that integrates various open-source EDA tools (like Yosys, OpenROAD, Magic, Netgen) to perform the complete RTL-to-GDSII flow.

The **strive SoC Family** (strive, strive2, strive3, etc.) are real-world examples of chips that were successfully taped out using OpenLANE and the Sky130 PDK, proving the viability of this open-source flow.

> [Image: Strive Family Soc.jpg]

### 2.4 📋 Introduction to OpenLANE Detailed ASIC Design Flow

We examined the detailed OpenLANE flow chart, which shows the interaction between tools.

> [Image: Opwn Lane Asic Flow.jpg]

Key concepts discussed:

  * **Modes:** OpenLANE can be run in two modes:
      * **Autonomous:** A "push-button" flow that runs all steps automatically.
      * **Interactive:** Allows the user to run one command at a time, inspect the results, and then proceed, offering more granular control.
  * **Challenges:** We also discussed real-world fabrication challenges like the **Antenna Effect**, where long metal wires can accumulate charge during fabrication and damage transistor gates. OpenLANE inserts "Antenna Diodes" to safely discharge this static buildup.

-----

## 3\. 🔬 Getting Familiar with Open Source EDA Tools (Lab)

This section covers the hands-on lab work, where we set up and ran the OpenLANE flow for the first time.

### 3.1 🛠️ Lab Setup: Fixing the PDK Path

A common setup issue is OpenLANE failing to find the Process Design Kit (PDK) files. The following steps were used to correct the environment variable and point OpenLANE to the correct PDK directory.

First, navigate to the `openlane_working_dir`:

```bash
# We were in .../designs/picorv32a, so we go up three levels
cd ../../..
# Now we are in ~/Desktop/work/tools/openlane_working_dir
```

Next, set the `PDK_ROOT` environment variable to the correct path:

```bash
export PDK_ROOT=$(pwd)/pdks
```

This command tells the system that the PDKs are located in the `pdks` folder inside the current working directory.

### 3.2 🐳 Running the OpenLANE Flow

With the path corrected, we can now enter the OpenLANE Docker container and start the flow.

```bash
# Navigate into the openlane directory
cd openlane

# Start the Docker container
docker
```

> [Image: Open lane.jpg]

Once inside the Docker container (prompt changes to `bash-4.2$`), we start the OpenLANE flow in **interactive mode**.

```bash
./flow.tcl -interactive
```

This starts the Tcl shell (prompt changes to `%`).

### 3.3 ⚙️ Design Preparation Step

Inside the Tcl shell, we first load the OpenLANE package and then prepare our design, `picorv32a`.

```tcl
# Load the OpenLANE package
package require openlane 0.9

# Prepare the design
prep -design picorv32a
```

This `prep` command:

1.  Finds the design's `config.tcl` file (in `designs/picorv32a/`).
2.  Merges it with the default PDK and flow-level configurations.
3.  Creates a new `run` directory (e.g., `runs/RUN_...`) where all outputs for this specific run will be stored.

### 3.4 📊 Running and Characterizing Synthesis

With the design prepared, we run the first major step: **synthesis**.

```tcl
run_synthesis
```

This command automatically:

1.  Uses **Yosys** to synthesize the Verilog code into a gate-level netlist.
2.  Performs **Static Timing Analysis (STA)** with OpenSTA to get an initial timing estimate.
3.  Generates the gate-level netlist (`.v` file) and various reports in the `runs/<run_name>/results/synthesis` directory.

After the run, we inspected the synthesis reports (`*.synthesis.rpt`) and found key metrics for the `picorv32a` design, such as the total count of D-flip-flops (DFFs) used, which was **1634**.
