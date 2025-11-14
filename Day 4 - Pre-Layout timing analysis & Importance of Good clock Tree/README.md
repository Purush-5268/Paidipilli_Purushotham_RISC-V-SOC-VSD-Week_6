## 🚀 **Day-4 — : Pre-Layout timing analysis and importance of good clock tree**


### 📘 ** Part A Custom Standard Cell Creation, Track Alignment & LEF Integration in OpenLane**

#### 🎯 **Overview**

Part-A focuses on creating a custom standard cell, aligning it with Sky130 track rules, generating its LEF abstraction, and integrating it into the OpenLane flow. Each step ensures that the custom cell behaves like any other library cell during placement, routing, and timing.

#### 📐 **Section 1 — Track Configuration & Grid Setup**

**🤖 Commands to open the custom inverter layout**
```bash
# Change directory to vsdstdcelldesign
cd Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign

# Command to open custom inverter layout in magic
magic -T sky130A.tech sky130_inv.mag &
```
##### **1.1 Reading Track Information**

> 📸 <img width="1920" height="1080" src="https://github.com/user-attachments/assets/235d1f32-166b-485b-9923-6a3ce41dc55c" />

Track data defines legal routing locations. Standard cells must match these tracks so that pins align correctly during automated routing.


| Layer | Direction | Pitch (µm) | Offset (µm) |
| ----- | --------- | ---------- | ----------- |
| li1   | X         | 0.46       | 0.23        |
| li1   | Y         | 0.34       | 0.17        |

These values are used to configure the Magic editor so that cell boundaries and pin placements follow the foundry routing grid.

#### **1.2 Setting Grid in Magic**

> 📸<img width="1920" height="1080" src="https://github.com/user-attachments/assets/c9a517e5-a573-4584-b56f-d28533019b30" />

```
grid 0.46um 0.34um 0.23um 0.17um
```

Using the same pitch/offset from tracks.info ensures that the custom cell aligns with all other Sky130 HD library cells.
Cells that follow grid rules guarantee clean abutment and accurate pin connectivity.

---

### 🔌 **Section 2 — Port Definition & LEF Generation**

#### **2.1 Creating Ports in Magic**

> 📸<img width="1920" height="1080" src="https://github.com/user-attachments/assets/3b8667e8-af01-4666-9eb2-8aeb96c7c9dc" />

Pins were labeled using Magic’s *Edit → Text* interface.

Defined pins:

* **A** – Input
* **Y** – Output
* **VPWR** – Power
* **VGND** – Ground

Correct pin definitions are required so that OpenLane and OpenROAD can recognize the cell’s connectivity during synthesis and routing.

---

#### **2.2 Writing the LEF File**

The layout was saved and exported into a LEF abstraction:

```bash
save inv_rev.mag
lef write
```

> 📸 <img width="1920" height="1080" src="https://github.com/user-attachments/assets/63582286-bce8-412f-87a7-c1aca1063d4b" />

The generated LEF contains the macro boundary, pin locations, metal shapes, and routing obstructions.
LEF abstracts the internal device geometry and exposes only the details needed by PnR tools.

#### **2.3 Inspecting the LEF File**

The LEF file was reviewed to ensure:

* Correct macro dimension
* Accurate pin rectangles on li1
* VPWR / VGND defined as POWER / GROUND

A clean LEF ensures that the standard cell integrates smoothly in the physical design flow.

---

### 🔗 **Section 3 — Integrating Custom LEF into OpenLane**

#### **3.1 Copy the newly generated lef and associated required lib files to 'picorv32a' design 'src' directory.**

>📸<img width="1920" height="1080" alt="7 copy the lef file" src="https://github.com/user-attachments/assets/ad403b0e-6877-467c-941b-ac511585afc0" />

```bash
Commands to copy necessary files to 'picorv32a' design 'src' directory

# Copy lef file
cp inv_rev.lef ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

# List and check whether it's copied
ls ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

# Copy lib files
cp libs/sky130_fd_sc_hd__* ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

# List and check whether it's copied
ls ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
```

Every custom LEF must be placed inside the `src/` directory so OpenLane can automatically pick it up during preparation.

#### **3.2 Modify config.tcl**

To integrate the custom cell and ensure OpenLane uses the correct library files, the design configuration must be updated.

#### **Step 1 — Open the config.tcl file**

```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/
gedit config.tcl
```

> 📸 <img width="1920" height="1080" alt="9 add these lines in config tcl" src="https://github.com/user-attachments/assets/45f9b2dc-d7ea-4efd-b21e-1b282cdc6de1" />

#### **Step 2 — Add the required lines**

Replace the library section with the updated paths:

```tcl
set ::env(LIB_SYNTH)   "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd_typical.lib"
set ::env(LIB_MIN)     "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd_fast.lib"
set ::env(LIB_MAX)     "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd_slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd_typical.lib"

set ::env(EXTRA_LEFS)  [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

These updates ensure that OpenLane uses your custom LEF and the correct timing libraries through synthesis, floorplan, placement, and CTS.

#### **Final Expected config.tcl (after adding lines)**

Your file must now look *exactly* like this:

```tcl
# Design
set ::env(DESIGN_NAME) "picorv32a"

set ::env(VERILOG_FILES) "./designs/picorv32a/src/picorv32a.v"
set ::env(SDC_FILE)      "./designs/picorv32a/src/picorv32a.sdc"

set ::env(CLOCK_PERIOD)  "5.000"
set ::env(CLOCK_PORT)    "clk"

set ::env(CLOCK_NET) $::env(CLOCK_PORT)

set ::env(LIB_SYNTH)   "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd_typical.lib"
set ::env(LIB_MIN)     "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd_fast.lib"
set ::env(LIB_MAX)     "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd_slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd_typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]

set filename $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/$::env(PDK)_$::env(STD_CELL_LIBRARY)_config.tcl
if { [file exists $filename] == 1} {
    source $filename
}
```

### **3.3 OpenLane Preparation**

Before running `prep`, OpenLane must be started inside the Docker environment.

#### **Step 1 — Move to the OpenLane directory**

```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane/
```

#### **Step 2 — Start the OpenLane Docker environment**

```bash
docker
```

This opens the container where all OpenLane tools are available.

#### **Step 3 — Launch the OpenLane flow in interactive mode**

```bash
./flow.tcl -interactive
```

##3# **Step 4 — Load the OpenLane package**

```bash
package require openlane 0.9
```
#### **Step 5 — Prepare the design (with overwrite enabled)**

```bash
prep -design picorv32a -tag 12-11_18-33 -overwrite
```
During preparation, OpenLane:

* Collects all LEF files
* Merges them into a single `merged.lef`
* Creates design-specific folders
* Loads configuration values from `config.tcl`

The generated `merged.lef` becomes the reference for placement, routing, and all later stages.

#### **Step 6 Adiitional commands to include newly added lef to openlane flow**
```tcl
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

#This loads all TCL commands required for preparation, synthesis, and physical design.
```

> 📸 <img width="1920" height="1080" alt="10 Openlane two commands " src="https://github.com/user-attachments/assets/ce705c2b-8bad-4cef-928a-357de2613a3e" />

### ⚙️ **Section 4 — Synthesis With Custom Cell**

#### **4.1 Running Synthesis**

> 📸 <img width="1920" height="1080" alt="11 Done Synthesis" src="https://github.com/user-attachments/assets/36199335-01cd-4ef9-af08-63778f974b64" />

```
run_synthesis
```

The synthesis step confirms whether the custom inverter is recognized and mapped correctly by the tool.
Initial timing reports show the baseline WNS/TNS before optimization.

## 🧱 **Section 5 — Floorplanning**

### **5.1 Running Initial Floorplan**

> 📸 <img width="1920" height="1080" alt="16 init floorplan commands" src="https://github.com/user-attachments/assets/e3fc73c1-fde1-4f16-83e3-d5ab3b73b655" />

Executed:

```bash
# Follwing commands are alltogather sourced in "run_floorplan" command
init_floorplan
place_io
tap_decap_or
```

Outputs generated:

* die area
* core area
* IO placement
* Power grid setup

Floorplan is clean and ready for placement.

## 📦 **Section 6 — Placement**

### **6.1 Placement Execution**

> 📸 <img width="1920" height="1080" alt="18Run_placement done" src="https://github.com/user-attachments/assets/17815d61-fcac-42bf-8843-6bcf63f01046" />

Placement engine output:

* legalized placement
* HPWL optimization
* component mapping

### **6.2 Placement Output in Results Directory**

> 📸 <img width="1920" height="1080" alt="19 placement in results" src="https://github.com/user-attachments/assets/0e146a6e-cbb6-4bd7-a43b-c3b9c28cc241" />

Final files:

* `placement.def`
* `placement.png`
* `placement.rpt`

## 🧭 **Section 7 — Viewing Layout in Magic**

### **7.1 Loading Placement DEF in Magic**

To view the placed design in Magic, run the following:

```bash
# Change directory to path containing generated placement DEF
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/24-03_10-03/results/placement/

# Command to load the placement DEF in Magic tool
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech \
lef read ../../tmp/merged.lef \
def read picorv32a.placement.def &
```

> 📸 <img width="1920" height="1080" alt="20 Magic Output placement" src="https://github.com/user-attachments/assets/52660205-84d4-4dbf-ac81-0321f869e5f3" />

Magic shows:

* 📌 Auto-placed standard cells
* 🔄 Row orientation
* 📏 Proper grid alignment

After loading, run:

```
# Command to view internal connectivity layers
expand
```
> 📸 <img width="999" height="548" alt="image" src="https://github.com/user-attachments/assets/9c9e47a5-8460-4307-86bd-d894feb672a8" />

### **7.2 Zoomed View of Standard Cell Placement**

> 📸 <img width="1920" height="1080" alt="21 zoomed magic" src="https://github.com/user-attachments/assets/4f8ab6b8-4fd7-417e-863f-e8cfd53c0b9f" />

Zoom view shows:

* ✔️ Correct cell abutment
* ✔️ No overlaps
* ✔️ All cells aligned on **li1 routing tracks**

---
Below is your **clean, professional, fully-structured Part-B**, using **your images**, **your SDC**, **your STA config**, and written exactly in the style you asked for:
✔ professional
✔ compact
✔ no “why we need this” sentences
✔ only importance explained
✔ icons instead of ticks
✔ placeholders for images (you will insert)

---

## 🚀 **Part-B : Timing Analysis with Ideal Clocks & Setup-Time Optimization**

---

### 📘 **1. Timing Analysis with Ideal Clocks (OpenSTA)**

Timing analysis with ideal clocks helps verify whether the design meets its setup constraints **before** clock tree insertion.
An *ideal* clock means zero delay, zero skew — only pure logic delay is evaluated.

#### 🧩 **Concept Overview**

* Launch flop sends data at clock edge **0**
* Capture flop receives data at clock edge **T**
* Combinational delay between flops = **θ**
* For timing to pass → **θ < T**

> 📸<img width="1095" height="611" alt="image" src="https://github.com/user-attachments/assets/9dc3bdb7-709e-4588-b0a2-f4f2d30ef854" />

### ⚙️ **2. Setup Time & Capture Flop Internal Paths**

Every flip-flop contains internal MUX + latching circuitry.
The D-input must settle *before* capturing edge by time **S (setup time)**.

Thus setup condition becomes:

```
θ < (T – S)
```

> 📸 <img width="1122" height="317" alt="image" src="https://github.com/user-attachments/assets/a08fa92b-58d4-4101-9e52-95c9e8938ad8" />

##3 ⏱️ **3. Clock Jitter & Uncertainty**

Clock sources (PLL) do not always generate edges at perfect instants.
Variation around the clock edge introduces **uncertainty (U)**.

Updated setup equation:

```
θ < (T – S – U)
```

> 📸 <img width="1102" height="668" alt="image" src="https://github.com/user-attachments/assets/943b00fc-f803-4ac2-96c7-25a90f02e639" />

This uncertainty margin must always be reserved inside STA.

---

### ⚙️ **4. Running OpenSTA for Post-Synth Timing Analysis**

---

#### 📂 **4.1 First Synthesis Run (Default Settings)**

OpenLane performs an initial synthesis + STA evaluation.

> 📸 <img width="1920" height="1080" alt="13 synthesis after setting size and all values" src="https://github.com/user-attachments/assets/1f330f01-4aa5-4be2-ace4-23bce18b5bef" />

| Metric  | Value      | Meaning               |
| ------- | ---------- | --------------------- |
| **WNS** | –23.89 ns  | Worst setup slack     |
| **TNS** | –711.59 ns | Accumulated violation |

Negative slack indicates critical paths failing timing.

---

### ⚡ **4.2 Timing-Optimized Synthesis**

Synthesis settings were adjusted to prioritize delay and reduce violations.

#### 🔧 **Commands Used**

```tcl
set ::env(SYNTH_STRATEGY) "DELAY 3"
set ::env(SYNTH_SIZING) 1
set ::env(SYNTH_MAX_FANOUT) 4
run_synthesis
```

> 📸 <img width="1920" height="1080" alt="14 after setting synth to delay 1 completion" src="https://github.com/user-attachments/assets/c2d0e432-72b1-4b36-9c09-77a6409427da" />

These settings enable:

* Delay-driven optimization
* Better drive-strength selection
* Reduced fan-out per gate

---

### 📈 **4.3 Improved Synthesis Timing**

After tuning the synthesis parameters:

> 📸 *insert image: 14 after setting synth to delay 1 completion.png*

| Metric  | Value   |
| ------- | ------- |
| **WNS** | 0.00 ns |
| **TNS** | 0.00 ns |

✔ Timing clean at synthesis stage
✔ No setup violations remaining

---

### 🧭 **5. Creating STA Constraint Files**

---

#### 📘 **5.1 Create `my_base.sdc`**

This file defines:

* Clock
* Input/output delays
* Driving cell
* Output loads

#### 📌 **Commands**

```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
gedit my_base.sdc
```

Paste your SDC:

```tcl
# CLOCK SETTINGS
set ::env(CLOCK_PORT) clk
set ::env(CLOCK_PERIOD) 24.73
set ::env(IO_PCT) 0.2

set ::env(SYNTH_DRIVING_CELL) sky130_fd_sc_hd__inv_8
set ::env(SYNTH_DRIVING_CELL_PIN) Y
set ::env(SYNTH_CAP_LOAD) 17.653
set ::env(SYNTH_MAX_FANOUT) 6

create_clock -name $::env(CLOCK_PORT) \
             -period $::env(CLOCK_PERIOD) \
             [get_ports $::env(CLOCK_PORT)]

set input_delay_value  [expr $::env(CLOCK_PERIOD) * $::env(IO_PCT)]
set output_delay_value [expr $::env(CLOCK_PERIOD) * $::env(IO_PCT)]

set clk_idx [lsearch [all_inputs] [get_port $::env(CLOCK_PORT)]]
set all_inputs_wo_clk [lreplace [all_inputs] $clk_idx $clk_idx]

set_input_delay  $input_delay_value  -clock [get_clocks $::env(CLOCK_PORT)] $all_inputs_wo_clk
set_output_delay $output_delay_value -clock [get_clocks $::env(CLOCK_PORT)] [all_outputs]

set_driving_cell -lib_cell $::env(SYNTH_DRIVING_CELL) \
                 -pin $::env(SYNTH_DRIVING_CELL_PIN) \
                 [all_inputs]

set cap_load [expr $::env(SYNTH_CAP_LOAD)/1000.0]
set_load $cap_load [all_outputs]
```

---

### 📘 **5.2 Create `pre_sta.conf`**

Used by OpenSTA to read libraries, netlist and apply `my_base.sdc`.

#### 📌 **Commands**

```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane/
gedit pre_sta.conf
```

Paste:

```tcl
set_cmd_units -time ns -capacitance pF -current mA -voltage V -resistance kOhm -distance um

read_liberty -max /openLANE_flow/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib
read_liberty -min /openLANE_flow/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib

read_verilog /openLANE_flow/designs/picorv32a/runs/13-11_05-20/results/synthesis/picorv32a.synthesis.v

link_design picorv32a
source /openLANE_flow/designs/picorv32a/src/my_base.sdc

report_checks -path_delay min_max -fields {slew trans net cap input_pin}
report_tns
report_wns
```

---

Here is **ONLY the STA + Timing ECO section**, rewritten cleanly and professionally, using **your commands + your images exactly**.
This is ready to append at the end of your Part-A README.

---

### ⚡ **5. Static Timing Analysis Using OpenSTA**

After integrating the custom cell and completing synthesis, we perform **post-synthesis STA** to observe timing violations and critical paths.

To run OpenSTA inside OpenROAD:

```bash
sta
```

Then load the STA configuration:

```bash
sta pre_sta.conf
```

> 📸 <img width="1920" height="1080" alt="27 Slack Violated" src="https://github.com/user-attachments/assets/b42fbd51-914e-42bd-8076-a893d127a6b2" />

This shows:

* **Worst Negative Slack (WNS)** ≈ –23.89 ns
* **Total Negative Slack (TNS)** highly negative
* Multiple failing setup paths
* Weak driving cells on critical nets

These results confirm that timing needs correction.

---

### 🛠️ **6. Timing ECO (Engineering Change Order)**

Timing ECO focuses on strengthening cells on critical paths to reduce delay.

#### 🧩 **6.1 Identify the Worst Net**

Run:

```bash
report_net -connections _11672_
```

> 📸 <img width="1920" height="1080" alt="26 14481 or gate slew wrong" src="https://github.com/user-attachments/assets/c0020f71-a2b3-4920-9c0b-19e1df39b541" />

This shows that net ***11672*** is driven by a **sky130_fd_sc_hd__or2_2** gate, which is too weak for the load (4 fanouts).

---

#### 🔧 **6.2 Replace Weak Cells With Higher-Drive Cells**

Use:

```bash
replace_cell _14510_ sky130_fd_sc_hd__or3_4
```

Then more replacements as required:

```
replace_cell _14481_ sky130_fd_sc_hd__or4_4
replace_cell _15219_ sky130_fd_sc_hd__or2_4
replace_cell _15220_ sky130_fd_sc_hd__or2_4
replace_cell _15221_ sky130_fd_sc_hd__or2_4
replace_cell _15222_ sky130_fd_sc_hd__or2_4
replace_cell _15224_ sky130_fd_sc_hd__or2_4
replace_cell _15226_ sky130_fd_sc_hd__or2_4
replace_cell _15227_ sky130_fd_sc_hd__or2_4
```

> 📸 <img width="1920" height="1080" alt="27 some more replaces " src="https://github.com/user-attachments/assets/86269ed7-bec0-4b3f-93ae-787953322184" />

These replacements increase drive strength, reduce output slew, and reduce path delay.

---

#### 📉 **6.3 Re-run Timing to Check Improvement**

```bash
report_checks -fields {net cap slew input_pins} -digits 4
```

> 📸 *insert: 28 slack reduced to 22.png*

Slack improvement:

* **Before ECO:** –23.89 ns
* **After ECO:** ≈ –22.50 ns

Even though the path still violates timing, the slack has **reduced by more than 1 ns**, showing the ECO is effective.

> 📸 <img width="1920" height="1080" alt="28 slack reduced to 22 " src="https://github.com/user-attachments/assets/2ed22a4c-1fe2-4d6a-9559-744d2b5abb8f" />

---

### ✔️ **Summary of Part-B**

* STA identifies real timing problems after synthesis.
* Setup time, clock delay, and load determine path feasibility.
* Weak cells cause large delays → replaced using **replace_cell**.
* Slack improved after ECO.

---
