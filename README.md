# 🚀 Day 1: Inception of OpenLANE and Sky130

## 📝 Overview

This document details the learnings and lab work from Day 1 of Week 6, focusing on the foundational concepts of open-source EDA tools. This session provides a comprehensive introduction to the **OpenLANE** automated RTL-to-GDSII flow, the **Sky130 PDK**, and the fundamental components of modern System-on-Chip (SoC) design.

-----

## 1\. 💡 Inception of Open Source EDA, OpenLane, and Sky130 PDK

### 1.1 🖥️ How to Talk to Computers

This section covered the hierarchy of translating human-readable software applications into physical hardware operations. A high-level language like C++ or Java is first compiled into **assembly language**, which is a low-level, human-readable representation of machine instructions. An **assembler** then translates this into binary **machine code** (the 1s and 0s) that the hardware can directly execute.

> <img width="1920" height="1080" alt="Flow" src="https://github.com/user-attachments/assets/7edad297-3a9e-48db-9bc7-0a07afe1ea28" />

### 1.2 📦 Introduction to QFN-48 Package, Chip, Pads, Core, Die, and IPs

A chip is not just the silicon die but a complete system integrated into a package. We explored the physical anatomy of a chip.

  * **Package:** The protective casing for the silicon die. We examined the **QFN-48 (Quad Flat No-leads)** package, a common 7x7mm standard.
  * **Die:** The small, square piece of silicon containing the entire integrated circuit.
  * **Core:** The central area of the die where the primary logic (like the CPU) is placed.
  * **Pads:** Conductive points on the die's periphery that act as the interface between the internal signals of the die and the external pins of the package.
  * **IPs (Intellectual Properties):** Reusable blocks of logic or circuitry.
      * **Foundry IPs:** Provided by the fabrication plant (e.g., ADC, DAC, SRAMs).
      * **Macros:** IPs that you design or acquire (e.g., RISC-V SoC, SPI controller).

> <img width="1546" height="846" alt="Day - 1 Arduino proccesor" src="https://github.com/user-attachments/assets/6bd81ec0-c622-462f-badb-08f7cf23ac9a" />
>
> <img width="1522" height="835" alt="SOC Day-1" src="https://github.com/user-attachments/assets/c73946ac-d9bd-4f76-877d-db1abd043dce" />

### 1.3 🏛️ Introduction to RISC-V

**RISC-V** is an open-source **Instruction Set Architecture (ISA)**. Unlike proprietary ISAs like ARM, RISC-V is free to use, modify, and extend.

  * It acts as the fundamental "architecture" of a computer, defining the set of instructions the processor can execute.
  * This open standard is a perfect match for an open-source EDA flow, allowing for truly open-source hardware design from the ISA all the way to the physical layout.

> <img width="1920" height="1080" alt="Stop watch - Riscv" src="https://github.com/user-attachments/assets/d7e64044-ee8a-4655-a2ce-f5ad56bcbaaa" />
>
> <img width="1920" height="1080" alt="Flow Riscv" src="https://github.com/user-attachments/assets/42cd3571-c9f6-4aeb-988b-d590e5a0d20f" />

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

><img width="1856" height="1006" alt="Synthesis" src="https://github.com/user-attachments/assets/77572e69-092c-4c8f-aa18-733e82e69421" />

#### b) Floor & Power Planning

  * **Floorplanning:** Partitions the chip die, places the I/O pads, and defines the core boundaries. It also involves placing large macros.
  * **Power Planning:** Creates the **Power Distribution Network (PDN)**. This involves adding power rings and straps (VDD, VSS) to deliver power to all cells with minimal resistance.

#### c) Placement

Places all the standard cells from the synthesized netlist onto the floorplan rows.

  * **Global Placement:** Finds an optimal, but not necessarily legal (i.e., cells can overlap), position for all cells.
  * **Detailed Placement:** Adjusts cell positions to ensure they are legally placed (no overlaps) while honoring the global placement logic.

#### d) Clock Tree Synthesis (CTS)

Builds a dedicated network (a "tree") to deliver the clock signal to all sequential elements (like flip-flops) with minimum and balanced delay (**skew**). This is often done using **H-Trees** or **X-Trees**.

> <img width="1170" height="412" alt="CTS - Design Flow" src="https://github.com/user-attachments/assets/395f1cb3-1789-4220-a68a-6a8ae6ed8dd5" />

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

><img width="1721" height="881" alt="Strive Family Soc" src="https://github.com/user-attachments/assets/cdc6e2cb-8267-4aec-8519-08c23e2fde27" />

### 2.4 📋 Introduction to OpenLANE Detailed ASIC Design Flow

We examined the detailed OpenLANE flow chart, which shows the interaction between tools.

><img width="1777" height="948" alt="Opwn Lane Asic Flow" src="https://github.com/user-attachments/assets/0b984916-5f7e-4cb9-ad2e-d5db4e0ac1c9" />

Key concepts discussed:

  * **Modes:** OpenLANE can be run in two modes:
      * **Autonomous:** A "push-button" flow that runs all steps automatically.
      * **Interactive:** Allows the user to run one command at a time, inspect the results, and then proceed, offering more granular control.
  * **Challenges:** We also discussed real-world fabrication challenges like the **Antenna Effect**, where long metal wires can accumulate charge during fabrication and damage transistor gates. OpenLANE inserts "Antenna Diodes" to safely discharge this static buildup.

-----
## 3\. 🔬 Getting Familiar with Open Source EDA Tools (Lab)

This section covers the hands-on lab work, where we set up and ran the OpenLANE flow for the first time, exploring the interactive mode and the synthesis stage.

### 3.1 🛠️ Lab Setup: Setting the PDK Path

To begin, we start from a fresh terminal. The first step is to navigate to the correct working directory and set the `PDK_ROOT` environment variable. This variable is critical as it tells OpenLANE where to find all the foundry-specific files (libraries, models, etc.).

```bash
# Navigate to the openlane working directory from home
cd ~/Desktop/work/tools/openlane_working_dir

# Set the PDK_ROOT environment variable
export PDK_ROOT=$(pwd)/pdks
```

This command tells the system that the PDKs are located in the `pdks` folder inside our current working directory.

### 3.2 🐳 Running the OpenLANE Flow

With the path corrected, we can now enter the OpenLANE Docker container, which provides a consistent environment with all the necessary EDA tools pre-installed.

```bash
# Navigate into the openlane directory
cd openlane

# Start the Docker container
docker
```

Once inside the Docker container (prompt changes to `bash-4.2$`), we start the OpenLANE flow in **interactive mode**.

```bash
./flow.tcl -interactive
```

### 3.3 ⚙️ Design Preparation Step

Inside the Tcl shell (prompt changes to `%`), we first load the OpenLANE package and then prepare our design, `picorv32a`.

```tcl
# Load the OpenLANE package
package require openlane 0.9

# Prepare the design
prep -design picorv32a
```

> <img width="1920" height="1080" alt="Prep of picorev" src="https://github.com/user-attachments/assets/54b8fe6e-cbe7-4d33-af23-7dcd58d3636a" />

This `prep` command:

1.  Finds the design's `config.tcl` file (in `designs/picorv32a/`).
2.  Merges it with the default PDK and flow-level configurations.
3.  Creates a new `run` directory (e.g., `runs/RUN_2025-10-31_18-12-59`) where all outputs for this specific run will be stored. We can see the creation of `logs`, `results`, `tmp`, and other directories.

> <img width="1920" height="1080" alt="After prep results" src="https://github.com/user-attachments/assets/ce0edb24-44c3-46ec-8e96-a564182c544c" />

### 3.4 📊 Running and Characterizing Synthesis

With the design prepared, we run the first major step: **synthesis**.

```tcl
run_synthesis
```

This command automatically:

1.  Uses **Yosys** to synthesize the Verilog code into a gate-level netlist.
2.  Performs **Static Timing Analysis (STA)** with OpenSTA to get an initial timing estimate.
3.  Generates the gate-level netlist (`.v` file) and various reports.

The screenshot below shows the `run_synthesis` command completing successfully.

><img width="1920" height="1080" alt="Synthesis" src="https://github.com/user-attachments/assets/e16fff64-ba63-4b7a-b788-30abbd2e120c" />

After the run, we inspected the generated files. We navigated to the `results/synthesis` directory to find the synthesized netlist (`picorv32a.synthesis.v`).

```bash
# To check the results
cd runs/RUN_2025-10-31_18-12-59/results/synthesis
ls -ltr
```

> <img width="1920" height="1080" alt="Resulsts" src="https://github.com/user-attachments/assets/328b6682-3c1e-4987-b10e-f67f981d3774" />

We can also check the `reports/synthesis` directory for synthesis statistics and logs.

```bash
# To check the reports
cd runs/RUN_2025-10-31_18-12-59/reports/synthesis
ls -ltr
```

><img width="1920" height="1080" alt="Screenshot from 2025-10-31 19-12-38" src="https://github.com/user-attachments/assets/b8c708e0-a4b9-453b-8593-e1a2117dced4" />

By examining the `*.synthesis.rpt` files, we found key metrics for the `picorv32a` design, such as the total count of D-flip-flops (DFFs) used, which was **1634**.
