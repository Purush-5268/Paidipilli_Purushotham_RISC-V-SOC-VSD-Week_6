## 📅 Day 2: Good vs. Bad Floorplan & Library Cells 

This section covers the core concepts of **chip floorplanning**, the critical first step in physical design. The objective is to define the chip's "blueprint": its size, the location of major blocks (like IPs and memory), and the power delivery network.

> **Why is this important?** A good floorplan is the foundation for a successful chip. It directly impacts **timing (setup/hold)**, **routing congestion**, and **power integrity (IR drop)**. A bad floorplan, where blocks are placed inefficiently, can make it impossible for the router to connect the design or for the chip to meet its performance targets. This lab bridges the gap from a "logical" netlist (from synthesis) to a "physical" layout.
<br>

## 1 Core Floorplanning Concepts
<details> <summary>💡 Concept: Core Floorplanning Concepts</summary>

#### 📐 Utilization Factor and Aspect Ratio

Before placing anything, we must define the **core area** inside the **die**. This is controlled by two key parameters:

  * **Utilization Factor (UF):** This is the ratio of the area occupied by all your standard cells (from synthesis) to the total available core area.

      * **Why not 100%?** A 100% utilization (a "packed" core) would leave **zero free space**. This is disastrous because the placement tool has no room to move cells for optimization, and the routing tool has no space to draw the metal wires between cells. This leads to massive congestion and timing failures.
      * **Typical Target:** A good starting point is often **50-70%**. This leaves 30-50% of the core area "open" for routing channels and for the tool to add optimization cells (like buffers) later. It's a trade-off: higher utilization saves silicon cost but makes timing and routing much harder.
      * `Utilization Factor = (Area of Standard Cells) / (Total Core Area)`

    ><img width="1078" height="797" alt="image" src="https://github.com/user-attachments/assets/d54d68f1-676a-4964-b732-f9df957131e3" />

  * **Aspect Ratio (AR):** This is simply the ratio of the core's height to its width.

      * `Aspect Ratio = Core Height / Core Width`
      * An AR of **1.0** creates a **square** core. This is often the most efficient shape for routing, as signals have similar maximum distances to travel in both x and y directions.
      * An AR of **0.5** would create a **wide** rectangle, while an AR of **2.0** would create a **tall** rectangle. These shapes might be necessary if you need to fit the chip into a specific package or place many pins on the top and bottom edges.

    > <img width="955" height="835" alt="image" src="https://github.com/user-attachments/assets/42783b15-67e2-495d-b4eb-a3e0e724ff7f" />


#### 📦 Concept of Pre-Placed Cells

While the placement tool will automatically arrange most of the 14,000+ standard cells, some special components must be placed manually **before** this step. These are called "pre-placed cells" or "macros."

  * **What are they?** These are large, complex blocks like **SRAMs (memories)**, **analog blocks (PLLs, ADCs)**, or **purchased IP (like a USB controller)**.
  * **Why pre-place them?**
    1.  **Fixed Pins:** They have fixed pin locations, and their connection to the main I/O pads is critical. We place them on the edge of the core to get the shortest, most direct path to the outside world.
    2.  **Sensitive Bypassing:** Analog blocks often need their own special, clean power supplies and must be kept away from noisy digital logic.
    3.  **Timing:** We place them strategically to minimize the wire delay to the logic blocks they "talk" to the most.
  * The automated placement tool is instructed to treat these macros as "black boxes" and arrange the smaller standard cells around them.
    > <img width="1560" height="722" alt="image" src="https://github.com/user-attachments/assets/6aa9ce42-fc44-45eb-b9d4-b068c15f801e" />

-----

#### ⚡ Decoupling Capacitors (Decaps)

When thousands of transistors switch states (e.g., at the rising edge of a clock), they all try to draw a large, sudden "spike" of current from the power grid at the exact same instant.

  * **The Problem (IR Drop):** This sudden current spike causes the local voltage to "droop" or "sag" because the power grid wires have resistance (Ohm's Law: $V = IR$). If the local $V_{dd}$ drops from 1.8V to 1.5V, the transistors get "weaker," their switching speed slows down, and the circuit can fail its timing.
    > <img width="1367" height="801" alt="image" src="https://github.com/user-attachments/assets/7fb9c1ad-4c71-4e32-b7a5-6580cf2a4775" />
  * **The Solution (Decaps):** We place **decoupling capacitors** *everywhere* inside the design. These are tiny on-chip capacitors that act as small, local charge reservoirs.
      * **How they work:** During steady state, they are charged up to 1.8V. When the transistors switch, they provide the instantaneous spike of current *locally*, preventing the voltage from drooping. They are "decoupling" the logic from the main power supply. After the spike, they slowly recharge from the main grid, ready for the next clock cycle.
    ><img width="1328" height="829" alt="image" src="https://github.com/user-attachments/assets/b8f1ad2e-2c26-43c1-8f3e-048eddbd0daa" />

-----

#### 🔌 Power Planning

We must build a robust "power grid" or **Power Delivery Network (PDN)** to deliver VDD (power) and VSS (ground) to every single cell in the design. A bad PDN is a primary cause of chip failure.

  * **The Goal:** Deliver a stable, reliable voltage to all 14,000+ cells with minimal voltage loss (IR drop) and noise.
  * **The Method (Power Mesh):** We use multiple metal layers to create a grid or mesh.
      * **Global Grid:** The top, thickest metal layers (which have low resistance) are used to create a coarse, chip-wide grid of VDD and VSS lines.
      * **Local Grid:** Lower metal layers run horizontally (in "power rails") within the standard cell rows, connecting every single cell to the global grid.
  * This mesh structure ensures there are many parallel paths for current to flow, which lowers the overall resistance and makes the power delivery stable.
    > <img width="1073" height="837" alt="image" src="https://github.com/user-attachments/assets/e74a69fc-6afb-4083-8bb7-3c380155ee54" />
    > <img width="1127" height="810" alt="image" src="https://github.com/user-attachments/assets/116885f7-8c80-4f72-a981-3997da60a442" />

-----

#### 📍 Pin Placement and Logical Cell Placement Blockage

  * **Pin Placement:** This defines where all the I/O signals (inputs, outputs, clocks) enter and leave the **core area**. The placement of these pins is critical. Placing the `clock` pin in the bottom-left corner when the clock-tree logic is in the top-right would create a very long, high-delay wire that could make timing impossible. Pins are placed to be as close as possible to the logic they connect to.
  * **Placement Blockages:** These are "keep-out" zones where the tool is **forbidden** from placing standard cells.
      * **Why?** We create blockages around **pre-placed cells (macros)** to ensure no standard cells get too close, which could block routing access. We also create blockages **under the power grid straps** or **near critical pins** to reserve that area exclusively for routing, preventing congestion.
    > <img width="1221" height="758" alt="image" src="https://github.com/user-attachments/assets/d5fcc22c-4398-49b3-92bf-7745de45d20a" />

<br>

</details>

<details> <summary><h3>🔬 Lab: Running the picorv32a Floorplan</h3></summary>

This section details the practical steps to run the floorplan stage for the `picorv32a` design in OpenLANE.

#### 1\. Reviewing Configuration Files

First, we inspect the configuration files that control the floorplan. These files define the UF, AR, and other settings.

```bash
# Navigate to the configuration folder
cd Desktop/work/tools/openlane_working_dir/openlane/configuration

# Open the configuration README file to see all variables
less README.md
```

> <img width="1920" height="1080" alt="Less Readme" src="https://github.com/user-attachments/assets/54c1daf2-fdc6-4f91-9b17-15079549664f" />

```bash
# Open the floorplan configuration script to see defaults
less floorplan.tcl
```

> <img width="1920" height="1080" alt="less floor plan" src="https://github.com/user-attachments/assets/8c6f747e-0963-464d-9812-25d4623ed9ed" />

-----

#### 2\. Running the Floorplan

We use the same interactive OpenLANE flow as Day 1, but this time we continue to the `run_floorplan` command.

**This is for Understanding**

```tcl
# --- Inside the OpenLANE interactive shell (./flow.tcl -interactive) ---

# Load OpenLANE packages
package require openlane 0.9

# Prepare the design workspace for 'picorv32a'
prep -design picorv32a

# Run synthesis (must be done before floorplan)
run_synthesis

# Execute the floorplan stage
run_floorplan
```

The terminal shows the successful completion of the floorplan, including the creation of the power grid (PDN) and the placement of tap and decap cells.

> <img width="1920" height="1080" alt="Floorplan Success" src="https://github.com/user-attachments/assets/e966a3e2-fe24-4524-911b-bcc1fc01a4b0" />

-----

#### 3\. Reviewing Floorplan Files & Calculating Area

The most important output is the `.def` (Design Exchange Format) file, which contains the geometric description of the floorplan.

```bash
# Navigate to the floorplan results directory (your run folder name will vary)
cd designs/picorv32a/runs/<YOUR_RUN_FOLDER>/results/floorplan

# Open the generated floorplan DEF file
less picorv32a.floorplan.def
```

Inside this file, we find the `DIEAREA` definition:

> `DIEAREA ( 0 0 ) ( 660685 671405 ) ;`
> `UNITS DISTANCE MICRONS 1000 ;`

This tells us:

  * **Die Area:** The chip's corners are at (0, 0) and (660685, 671405).
  * **Database Units:** 1000 units = 1 micron.

**Die Area Calculation:**
| Parameter | Database Units | Conversion | Result (µm) |
| :--- | :--- | :--- | :--- |
| **Width** | 660685 | `660685 / 1000` | **660.685 µm** |
| **Height** | 671405 | `671405 / 1000` | **671.405 µm** |

$$Area = 660.685 \ \mu m \times 671.405 \ \mu m = 443587.21 \ \mu m^2$$

-----

#### 4\. Visualizing the Floorplan in Magic

Finally, we use the Magic layout tool to visually inspect the `.def` file.

```bash
# In the .../results/floorplan directory, run:
magic -T /home/Purush/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
```

> <img width="1920" height="1080" alt="flloorplan and placement commands magic" src="https://github.com/user-attachments/assets/c691c611-43ac-47cc-b6cb-b74a2e450d00" />

**Full Floorplan View:**
The layout shows the **core area** (inner box) and the **die area** (outer box). The vertical dotted lines are the rows of tap cells and decap cells for the power grid.

> <img width="1920" height="1080" alt="magic layout" src="https://github.com/user-attachments/assets/dd482b0e-74f7-4359-bf89-4c23bcb1a291" />

**Zoomed-in View:**
Zooming in on the rows reveals the individual standard cells. To identify *what* these cells are, we use the Magic interface itself:
1.  Place the **cursor box** (the white rectangle) over one of the cells.
2.  **Left-click** to select it.
3.  Read the output in the **`tkcon 2.3 Main`** window (the console).

The `tkcon` window confirms the cell type. In this case, the cells are `sky130_fd_sc_hd__decap_3`, which are the **decoupling capacitors** we learned about in the theory section. The floorplanner automatically inserts these to stabilize the power grid.

> <img width="1920" height="1080" alt="maich what" src="https://github.com/user-attachments/assets/37bd0ee3-fca3-402a-b5c1-a34c95146c35" />
<br>
</details>

##  2 Library Binding & Placement
<details>
<summary>💡 Concept: Library Binding & Placement</summary>

---

#### 1. Netlist Binding and Initial Place Design

After floorplanning, the design is still just a "netlist" (a logical text file) and a "floorplan" (an empty core area with a power grid). The **placement** stage is where we physically build the circuit.

* **Netlist Binding:** This is the first step, where the tool "binds" or "links" the logical gates from the synthesis netlist (e.g., `AND2_X1`) to their corresponding physical "master" cells in the standard cell library.
* **Physical Cells:** A cell in the library (`.lef` file) is not just a symbol; it's a physical rectangle with a defined width, height, and pin locations. A library may have multiple versions of an AND gate (e.g., `and2_x1`, `and2_x2`, `and2_x4`) with different drive strengths and sizes.
* **Initial Placement:** The tool takes all 14,000+ of these physical cells and places them into the "site rows" created during floorplanning. This initial pass tries to place cells that are logically connected close to each other to minimize wire length.

> <img width="1218" height="795" alt="image" src="https://github.com/user-attachments/assets/c5d0e8b3-ba88-4adb-9c20-b060dfbf3c3b" />

---

#### 2. Optimize Placement (Wire-length & Capacitance)

An initial placement is never optimal. Cells connected in a long timing path (like `FF1 -> AND -> OR -> FF2`) might be placed far apart, resulting in a very long wire.

> **Why are long wires bad?** A long wire acts like a capacitor and a resistor. This **RC delay** slows down the signal, which can cause the circuit to miss its timing (a "setup violation").

The tool's main goal during optimization is to minimize **"timing-driven" wire length**. It identifies the most critical timing paths and moves those cells closer together.

> <img width="1221" height="599" alt="image" src="https://github.com/user-attachments/assets/a48b350f-e0d2-4577-910c-e1f2ab12b581" />

To fix paths that are still too long, the tool inserts **buffers** (or "repeaters"). A buffer doesn't change the logic, but it "re-powers" or "regenerates" the signal, allowing it to travel a longer distance, much like a relay station.

> <img width="1024" height="494" alt="image" src="https://github.com/user-attachments/assets/992cd921-e572-477f-a61d-053a45ea7fcb" />

---

#### 3. Final Placement Optimization

After the main optimization, the tool performs a "final" pass. It re-checks the timing (now including the delays from the newly added buffers) and makes minor adjustments. This step ensures that all cells are **"legalized"**—meaning every cell is snapped perfectly to the placement grid (the "sites") and there are no overlaps.

---

#### 4. Need for Libraries and Characterization

The placement tool isn't just guessing. How does it know the delay of a wire? Or the delay of a cell? It gets all this information from the **Standard Cell Library** (`.lib` file).

> A **`.lib` file** is a "data book" that contains all the timing (delay, setup/hold times), power, and capacitance information for every single cell. This data is "characterized" by the foundry (like Skywater) using thousands of SPICE simulations.

Every single stage of the flow—Synthesis, Floorplan, Placement, Clock Tree Synthesis (CTS), and Routing—relies on this library to make intelligent decisions.

> <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8d54b52c-7126-4b10-9937-80879e3e7ef0" />

</details>

<br>

<details>
<summary>🔬 Lab: Congestion-Aware Placement (RePlAce)</summary>

This section details running the **placement** stage for the `picorv32a` design in OpenLANE. OpenLANE uses a tool called RePlAce for congestion-aware placement.

#### 1. Running Placement

We continue in the same OpenLANE interactive shell and execute the `run_placement` command.

**This is for Understanding**
```tcl
# --- Inside the OpenLANE interactive shell ---

# Execute the placement stage
run_placement
````

The tool will first run a "global placement" to find a rough position for all cells, then run "detailed placement" to legalize and snap them to the grid.

-----

#### 2\. Visualizing the Placed Design in Magic

After placement finishes, a new `.def` file is generated in the `results/placement` directory. We use Magic to view it.

```bash
# In the .../results/placement directory, run:
magic -T /home/Purush/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```

><img width="1920" height="1080" alt="flloorplan and placement commands magic" src="https://github.com/user-attachments/assets/b3c1bec1-719a-42c2-b9ec-611a4298a9ba" />

**Full Placement View:**
The view looks dramatically different from the floorplan. The core area is no longer empty; it is now densely populated with all 14,876 standard cells. The tool has legally placed them into the site rows, leaving empty spaces (channels) for the router.

> <img width="1920" height="1080" alt="Placement" src="https://github.com/user-attachments/assets/01a57243-18e7-420d-92ee-b385d71568af" />

**Zoomed-in View:**
Zooming in, we can see the individual standard cells (e.g., `sky130_fd_sc_hd__buf_2`, `sky130_fd_sc_hd__clkbuf_1`, etc.) all perfectly aligned and snapped to the rows. This is the "final" placed layout before routing.

><img width="1920" height="1080" alt="Placement zoom" src="https://github.com/user-attachments/assets/e97a0cd2-dc81-417f-a276-90a47bb9c188" />


</details>
<br>

## 3 Cell Design & Characterization Flow
<details> <summary>💡 Concept: Cell Design & Characterization Flow</summary>
-----

#### 📥 1. Inputs for Cell Design Flow

A **Standard Cell Library** is the foundational "Lego set" for all modern chip design. It's a collection of pre-designed logic gates (AND, OR, INV, Flip-Flops) that the tools use to build the chip. Each cell exists in multiple variants, such as different drive strengths (sizes) or threshold voltages (Vt), to help optimize for speed or power.

> <img width="1223" height="776" alt="image" src="https://github.com/user-attachments/assets/98a914a0-f136-4db4-833b-b3fbf266b84e" />

The "Cell Design Flow" is the process of creating these "Lego bricks." It requires several key inputs:

  * **PDKs (Process Design Kits):** Provided by the foundry (like Skywater), containing the core technology files.
  * **DRC and LVS Rules:** Rule decks that define the "physics" of the design—how far apart metal lines must be (DRC - Design Rule Check) and how to verify the layout matches the schematic (LVS - Layout vs. Schematic).
  * **SPICE Models:** Transistor-level behavior models that allow for accurate circuit simulation.
  * **Library Specifications:** The "goals" for the cell, defining its function (e.g., "2-input NAND"), timing, and power targets.

> <img width="1223" height="776" alt="image" src="https://github.com/user-attachments/assets/bd115bad-533a-4349-b1bc-b9f6149e2f40" />

-----

#### ✍️ 2. Circuit Design Step

This is where the "logic" of a cell is first created. It involves taking the functional specification (e.g., "Inverter") and building a transistor-level circuit schematic.

  * **Functional Implementation:** The circuit is built using PMOS (pull-up) and NMOS (pull-down) transistors to achieve the desired logic.
  * **Transistor Sizing:** This is the most critical part. Designers must calculate the precise **Width-to-Length (W/L) ratio** of the transistors. This is done to meet performance targets, such as the **switching threshold (Vm)**, and to ensure the cell has the correct drive strength.

> <img width="1224" height="727" alt="image" src="https://github.com/user-attachments/assets/4f7a912e-ad9f-45a5-b805-831556f8ff3d" />


-----

#### 🎨 3. Layout Design Step

The layout step translates the "paper" schematic from circuit design into a "physical" blueprint for fabrication. This is the "art" of cell design.

  * **Euler's Path:** To create the most compact layout, designers first create **network graphs** for the PMOS and NMOS transistors. They then find the shortest **Euler's Path** through these graphs, which defines a single, unbroken line of diffusion for all transistors, minimizing parasitic capacitance.
  * **Stick Diagram:** The Euler path is converted into a **stick diagram**, a simplified layout showing the relative placement of transistors, VDD/GND rails, and metal connections.
  * **Final Layout:** The stick diagram is translated into a full, physical layout, where all components adhere to the foundry's DRC rules.
  * **Extraction:** After the layout is complete, the tool runs **parasitic extraction** to measure all the "unwanted" resistance (R) and capacitance (C) from the metal wires. This R and C data is critical for accurate characterization.

> <img width="1225" height="730" alt="image" src="https://github.com/user-attachments/assets/5741792d-9072-4834-a538-e51bddaf793c" />

-----

#### ⚙️ 4. Typical Characterization Flow

Characterization is the process of testing the cell to create its "data sheet" (the `.lib` file). We build a SPICE "testbench" to measure its performance.

Based on the lab notes, the flow is:

1.  **Read the model** files (e.g., NMOS, PMOS).
2.  **Import the extracted SPICE netlist** of the cell (the "subcircuit").
3.  **Attach power supplies** (e.g., VDD).
4.  **Apply stimulus** (e.g., a pulse input) to the cell's input pin.
5.  **Add an output load** (a capacitor) to simulate the wire and gates it will drive.
6.  **Run the simulation** to measure the cell's behavior.

This testbench setup is what allows us to measure the delay, power, and slew, as seen in the schematic below.

> <img width="1220" height="668" alt="image" src="https://github.com/user-attachments/assets/bbb6dcdd-d3d9-4b56-a56d-f120281f80da" />

This process is run hundreds of times by a tool (like **GUNA**) to generate the final timing, power, and noise models for the library.

</details>

<br>

## 4 Timing Characterization Parameters
<details> <summary>💡 Concept : Timing Characterization Parameters</summary>
-----

#### 📏 1. Timing Threshold Definitions

To measure "delay" or "slew," we must first define *where* on the waveform we are taking the measurement. These are the **timing thresholds**.

  * **Slew (Transition Time) Thresholds:**
      * `slew_low_rise_thr`: The low point for measuring a rising signal (e.g., **20%** of VDD).
      * `slew_high_rise_thr`: The high point for measuring a rising signal (e.g., **80%** of VDD).
      * The fall thresholds (`slew_low_fall_thr`, `slew_high_fall_thr`) are similar, typically 80% to 20%.
  * **Propagation Delay Thresholds:**
      * `in_rise_thr` / `in_fall_thr`: The point where we consider the *input* to have "officially" crossed (e.g., **50%** of VDD).
      * `out_rise_thr` / `out_fall_thr`: The point where we consider the *output* to have "officially" crossed (e.g., **50%** of VDD).

><img width="1220" height="583" alt="image" src="https://github.com/user-attachments/assets/e45a9300-6a2c-4377-a1cc-56a13a45ad70" />
-----

#### ⏱️ 2. Propagation Delay and Transition Time

Once the thresholds are defined, we can calculate the two most important timing parameters:

  * **Propagation Delay ($t_{pd}$):** This is the time it takes for the cell to react. It's measured from the 50% crossing of the input to the 50% crossing of the output.

      * `Delay = Time(output_crosses_50%) - Time(input_crosses_50%)`

  * **Transition Time (Slew):** This measures how "fast" or "sharp" the signal edge is, not how *delayed* it is. A slow slew is bad for timing.

      * `Rise Time = Time(output_crosses_80%) - Time(output_crosses_20%)`
      * `Fall Time = Time(output_crosses_20%) - Time(output_crosses_80%)`

As seen in the graph, the input slew might be 26ps, while the output slew (degraded by the cell) is 54ps.


</details>
