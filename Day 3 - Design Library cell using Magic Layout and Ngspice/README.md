## 📅 Day 3: Cell Design, Layout, & Characterization

This day is broken into three parts:

1.  **Part 1:** Revisiting IO placement, understanding SPICE simulation, and cloning the `vsdstdcelldesign` lab.
2.  **Part 2:** Inspecting the inverter layout, understanding the CMOS fabrication process, and extracting a SPICE netlist *from* the layout.
3.  **Part 3:** Building a testbench for the extracted netlist, running the post-layout SPICE simulation, and introducing Magic's DRC (Design Rule Check) system.

-----

### Part 1: IO Placer & SPICE Simulation Setup

<details> <summary><h3>💡 Concept: CMOS Inverter Simulation</h3></summary>

-----

#### 📜 SPICE Deck Creation for CMOS Inverter

A **SPICE deck** (or netlist) is the text-based description of a circuit. To characterize a cell (like an inverter), we must create a testbench. This involves:

1.  **Instantiating the Inverter:** Calling the inverter "subcircuit" (`.subckt`).
2.  **Adding Power:** Providing VDD and GND voltage sources.
3.  **Adding Stimulus:** Attaching a `VPULSE` source to the input to create a rising/falling signal.
4.  **Adding a Load:** Attaching a capacitor (`Cload`) to the output to simulate the load of the next gates and wires.
5.  **Adding Commands:** Telling SPICE what to do (`.tran` for transient analysis, `.measure` to calculate delays).

#### ⚡ Switching Threshold ($V_m$)

The **Switching Threshold ($V_m$)** is a critical metric for a CMOS inverter. It is the input voltage ($V_{in}$) at which the output voltage ($V_{out}$) is equal.

  * $V_{in} = V_{out}$
  * Theoretically, for a perfectly symmetrical inverter, this is $V_{DD} / 2$.
  * In practice, it depends on the PMOS-to-NMOS size ratio ($\beta_p / \beta_n$). We simulate this using a `.dc` analysis, sweeping $V_{in}$ from 0 to VDD and plotting $V_{out}$ vs. $V_{in}$ to find the crossover point.

#### 📊 Static and Dynamic Simulation

  * **Static Simulation (`.dc`):** Used to find DC properties like $V_m$ or to check for power leakage. It does not involve time.
  * **Dynamic Simulation (`.tran`):** A **transient** analysis that simulates the circuit's behavior *over time*. This is what we use to measure propagation delay, rise time, and fall time, as it shows us the actual waveforms.

</details>

<br>

<details> <summary><h3>🔬 Lab: IO Placer Revision & vsdstdcelldesign Clone</h3></summary>
-----

#### 🔄 Lab: IO Placer Revision

In OpenLANE, we can control how I/O pins are placed. The default is equidistant (`set ::env(FP_IO_MODE) 1`). We will change this to `2` to see the effect.

1.  **Change the configuration:**

    ```bash
    # Navigate to the configurations directory
    cd Desktop/work/tools/openlane_working_dir/openlane/configurations
    # Edit the floorplan.tcl file (e.g., using vim)
    less floorplan.tcl
    ```

    > <img width="1920" height="1080" alt="less floorplantcl" src="https://github.com/user-attachments/assets/380b50ff-47d3-4877-8d47-d48183425fdd" />

2.  **Inside the OpenLANE interactive shell:**

    ```tcl
    # Set the IO mode to 2 
    set ::env(FP_IO_MODE) 2

    # Re-run the floorplan
    run_floorplan
    ```

    > <img width="1920" height="1080" alt="Set 2 floorplan" src="https://github.com/user-attachments/assets/b4077c80-fabd-46cb-8644-04c2a7418bb6" />

3.  **View the new floorplan:** We can now open the new `picorv32a.floorplan.def` file in Magic and see that the pins are no longer equidistant; they are clustered on the left side.

    ```bash
    # Navigate to the new results directory and open Magic
    cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/<NEW_RUN_DIR>/results/floorplan
    magic -T ... lef read ... def read picorv32a.floorplan.def &
    ```

    > <img width="1920" height="1080" alt="Magic after set 2 " src="https://github.com/user-attachments/assets/d484ba17-f382-4ea5-8ddf-bfb015d233fa" />

-----

#### 📦 Lab: Cloning `vsdstdcelldesign`

Next, we clone the repository containing the inverter layout (`.mag` file) that we will use for the rest of the labs.

```bash
# Change directory to openlane
cd Desktop/work/tools/openlane_working_dir/openlane

# Clone the repository with custom inverter design
git clone https://github.com/nickson-jose/vsdstdcelldesign
```

> <img width="1920" height="1080" alt="gitclone" src="https://github.com/user-attachments/assets/c9c18c99-2680-4ef6-a980-5b413aaf8c76" />

```bash
# Change into repository directory
cd vsdstdcelldesign

# Copy magic tech file to the repo directory for easy access
cp /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech .

# Check contents whether everything is present
ls

# Command to open custom inverter layout in magic
magic -T sky130A.tech sky130_inv.mag &
```

><img width="1920" height="1080" alt="Magic command to open inv" src="https://github.com/user-attachments/assets/ba6379a3-92ad-4685-9609-21d5a8422b4a" />

</details>

-----

-----

### Part 2: Layout Inspection & Spice Extraction

<details> <summary><h3>💡 Theory: CMOS Fabrication Process</h3></summary>
A 2D layout in Magic is just a set of colored masks. These masks correspond to the physical steps of the **CMOS fabrication process** that builds the 3D transistor structures on the silicon wafer.

1.  **Create Active Regions:** The process starts with a base P-type silicon wafer. A mask (corresponding to the `diff` or `active` layer) is used to define the areas where transistors will be built.
2.  **Form N-well and P-well:** To build PMOS transistors (which require an N-type substrate), an "N-well" is created by implanting N-type ions into the P-substrate. The P-well (for NMOS) is the P-substrate itself.
3.  **Form Gate Terminal:** A very thin layer of silicon dioxide (the "gate oxide") is grown. On top of this, a layer of **polysilicon** is deposited. This layer is then masked and etched (using the `poly` layer mask) to form the gate terminal of the transistors.
4.  **Lightly Doped Drain (LDD) Formation:** A light "LDD" implant is performed, which helps to create shallow extensions for the source and drain. This improves the transistor's reliability and prevents hot-electron effects.
5.  **Source & Drain Formation:** A heavier implant of N-type ions (for NMOS) and P-type ions (for PMOS) forms the final **source and drain** terminals. The polysilicon gate itself acts as a mask, "self-aligning" the source and drain implants.
6.  **Local Interconnect Formation:** A dielectric (insulating) layer is added. "Contacts" (vias) are etched and filled with metal (like `locint` or `li`) to connect to the polysilicon and active regions.
7.  **Higher Level Metal Formation:** The process is repeated for multiple metal layers (Metal 1, Metal 2, Metal 3, etc.), separated by insulators. Vias are used to connect the different metal layers, wiring the entire circuit together.

</details>
<br>
<details> <summary><h3>🔬 Lab: Layout Inspection & Spice Extraction</h3></summary>
-----

#### 🔍 Lab: Introduction to Sky130 Layers

We use Magic to explore the `sky130_inv.mag` layout, which we opened in the previous lab.

> <img width="1920" height="1080" alt="Layout" src="https://github.com/user-attachments/assets/d380e3dd-7905-4cd5-93d1-730c5bd80cd4" />

We can identify all the components of the CMOS inverter by their layers. We can also use the `what` command in the `tkcon` window after selecting a component.

  * **Identify PMOS:** A PMOS transistor is formed where **polysilicon** (red) crosses **P-diffusion** (orange-brown) inside an **N-well** (grayish-hatched).
     > <img width="1920" height="1080" alt="pmos identified by using what" src="https://github.com/user-attachments/assets/c5cdb891-e17c-4870-8dba-e1ea856cfd95" />
  * **Identify NMOS:** An NMOS transistor is formed where **polysilicon** (red) crosses **N-diffusion** (green) in the P-substrate.
    > <img width="1920" height="1080" alt="nmos define what" src="https://github.com/user-attachments/assets/9fffc156-a516-4109-a2c8-46bf4728e577" />
   

  * **Trace Connections:** We can visually trace the wires:
      * **Input (A):** The single polysilicon line (red) acts as the gate for both transistors.
      * **Output (Y):** `Met1` (red-hatched) connects the drain of the PMOS and the drain of the NMOS.
      * **Power (VPWR/VGND):** `Met1` connects the PMOS source to the `VPWR` rail and the NMOS source to the `VGND` rail.
    > <img width="1920" height="1080" alt="Source connections" src="https://github.com/user-attachments/assets/4cc50523-0ad8-46a9-bb48-558f682b3ce3" />

-----

#### ⚡ Lab: Extract Spice Netlist from Layout

Now that we have a layout, we can use Magic to perform a "layout extraction." This reads the layout geometry (including all the "unwanted" parasitic resistances and capacitances) and generates a **post-layout SPICE netlist**.

1.  In the `tkcon` window in Magic, run the `extract` command. This reads the layout and creates a `.ext` file.

    ```tcl
    # Extraction command to extract to .ext format
    extract all
    ```

    > <img width="1920" height="1080" alt="Extract command" src="https://github.com/user-attachments/assets/d47f93bb-9b19-42e5-943a-b7b1029bc929" />

2.  Next, we convert the extracted file to a `.spice` file. We include the `cthresh 0 rthresh 0` options to ensure that **all** parasitic capacitors and resistors are extracted, which is critical for an accurate simulation.

    ```tcl
    # Enable parasitic resistance (R) and capacitance (C) extraction
    ext2spice cthresh 0 rthresh 0

    # Convert the .ext file to a .spice file
    ext2spice
    ```

    This generates the file `sky130_inv.spice`. This file is a subcircuit (`.subckt`) that accurately represents our layout, complete with all its real-world parasitic effects.
    <img width="1920" height="1080" alt="ext2spice " src="https://github.com/user-attachments/assets/df0e3ace-7c9a-4953-a3bb-e5b6b133e8ed" />

</details>
-----

### Part 3: Post-Layout Characterization & DRC

<details> <summary><h3>🔬 Lab: Post-Layout SPICE Characterization</h3></summary>

-----

#### 🔌 Lab: Create Final SPICE Deck

The `sky130_inv.spice` file created by Magic (in Part 2) is just a subcircuit. It describes the transistors and parasitic\_s. To simulate it, we must create a **testbench** by editing the file.

1.  **Open the file:**

    ```bash
    vim sky130_inv.spice
    ```

    This is the initial extracted file. It defines the `.subckt` and lists all the transistors (M0, M1) and the parasitic capacitors (C1, C2, etc.) based on the layout geometry.

    > <img width="1920" height="1080" alt="vim sky130_inv spice" src="https://github.com/user-attachments/assets/f512ed2b-85b5-482e-9618-6a2908c86d34" />


2.  **Edit the file:** We add the necessary SPICE commands to create a full testbench:

      * `.include` statements for the Sky130 transistor models (`nshort.lib`, `pshort.lib`).
      * Voltage sources for power (`VDD`, `VSS`).
      * A `VPULSE` source (`Va`) for the input `a` to create a rising and falling signal.
      * A `.control` block with a `.tran` command to run a transient analysis.
      * `plot` commands to automatically plot the input `a` and output `y`.

    > <img width="1920" height="1080" alt="CODE AFTER MODIFY" src="https://github.com/user-attachments/assets/9ce359b9-6000-420c-9a18-1c9eaeeabf5e" />

-----

#### 📈 Lab: Characterize Inverter with `ngspice`

Now we run the simulation using our complete testbench.

1.  **Run `ngspice`:**

    ```bash
    # Command to directly load spice file for simulation
    ngspice sky130_inv.spice
    ```

    > [<img width="1920" height="1080" alt="Spice Output2" src="https://github.com/user-attachments/assets/86275149-babe-4008-a37b-67927594d94b" />


2.  **Plot Waveforms:** The `plot` command in our SPICE deck runs automatically, generating a plot of the input `a` (blue) and the inverted output `y` (red). This confirms the inverter works.

    > [<img width="1920" height="1080" alt="spice Output" src="https://github.com/user-attachments/assets/9cf60f46-399f-45e0-90d6-ea6c377ea04b" />

3.  **Calculate Transition Times & Delay:** We use the plot to find the exact values at the 20%, 50%, and 80% thresholds. (Note: $V_{DD}$ = 3.3V)

      * **20% VDD** = 0.66V
      * **50% VDD** = 1.65V
      * **80% VDD** = 2.64V

    **1. Rise Transition Time (Output):**

      * `Time(80%) - Time(20%)`
      * `2.2465 ns - 2.1824 ns = 0.0641 ns` = **64.1 ps**

    > [Image: `ngspice` terminal measuring rise time values (rise Time.png)]

    **2. Fall Transition Time (Output):**

      * `Time(20%) - Time(80%)`
      * `4.09548 ns - 4.05295 ns = 0.0425 ns` = **42.5 ps**

    > <img width="1920" height="1080" alt="rise Time" src="https://github.com/user-attachments/assets/35570b21-3ebc-4117-9be3-746725d1f377" />


    **3. Rise Cell Delay ($t_{pLH}$):**

      * `Time(output_rises_50%) - Time(input_falls_50%)`
      * `2.21157 ns - 2.1498 ns = 0.06177 ns` = **61.77 ps**

    > <img width="1920" height="1080" alt="rise Time" src="https://github.com/user-attachments/assets/35570b21-3ebc-4117-9be3-746725d1f377" />

    **4. Fall Cell Delay ($t_{pHL}$):**

      * `Time(output_falls_50%) - Time(input_rises_50%)`
      * `4.07784 ns - 4.04983 ns = 0.02801 ns` = **28.01 ps**

    > <img width="1920" height="1080" alt="Rise delay  or propagation delay" src="https://github.com/user-attachments/assets/0b91296c-552d-4060-bac2-53e6c7df7383" />

</details>

<br>

<details> <summary><h3>📐 Lab: Sky130 Tech File & DRC Rules</h3></summary>

-----

#### 📚 Lab: Introduction to Magic & DRC Rules

**DRC (Design Rule Check)** is a system that ensures our layout is manufacturable. The rules are defined in the technology file (`.tech`). These rules are provided by the foundry.

  * **Magic VLSI Tool:** [http://opencircuitdesign.com/magic/](http://opencircuitdesign.com/magic/)
  * **Sky130 PDK Rules:** [https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html)

> <img width="1912" height="1018" alt="image" src="https://github.com/user-attachments/assets/318fe9ce-f8a1-44fb-a379-b2683e35fde0" />

> <img width="1918" height="1020" alt="image" src="https://github.com/user-attachments/assets/b79f2ce7-b8c3-4062-9378-60e7fdc31f78" />

><img width="1918" height="1017" alt="image" src="https://github.com/user-attachments/assets/d561ce58-9bdf-4af0-80dc-61ad0ba27cc8" />


-----

#### 📦 Lab: Download `drc_tests`

We download a set of layouts (`drc_tests.tgz`) that are known to have specific DRC errors, which we will use for testing.

```bash
# Change to home directory
cd

# Command to download the lab files
wget http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz

# Since lab file is compressed command to extract it
tar xfz drc_tests.tgz

# Change directory into the lab folder
cd drc_tests

# List all files
ls -al
```

> <img width="1920" height="1080" alt="magic -d xr" src="https://github.com/user-attachments/assets/13eb6e42-af93-4c41-8afc-b396db303c45" />


-----

#### 🪄 Lab: Loading Sky130 Tech Rules

The `.magicrc` file is a startup script that tells Magic where to find the tech file. We use `gvim` to view it and `magic -d XR` to open the tool in a high-compatibility graphics mode.

```bash
# Command to view .magicrc file
gvim .magicrc

# Command to open magic tool in better graphics
magic -d XR &
```

><img width="1920" height="1080" alt="vim magicrc" src="https://github.com/user-attachments/assets/a9ca4f9c-91f7-4302-8a72-40d41c84e221" />

-----

#### 🐞 Lab: Checking DRC Errors

We open `met3.mag`, a file known to have errors, to learn the DRC commands.

1.  **Open the file:** In the `drc_tests` directory, run `magic -d XR &` and open `met3.mag`.
2.  **Run DRC:** In the `tkcon` window, we use these commands to find errors:
    ```tcl
    # To select the area
    select area

    # Must re-run drc check to see updated drc errors
    drc check

    # Selecting region displaying the new errors and getting the error messages
    drc why
    ```
    The `drc why` command explains the specific rule violation, such as "Metal3 spacing \< 0.3um".
    > <img width="1155" height="915" alt="image" src="https://github.com/user-attachments/assets/8c705fc6-5483-41d7-903f-884b2a4d4a6b" />

-----

#### 🔧 Lab: Fixing Tech File DRC Rules

The `.tech` file itself can have bugs. We will fix a few known errors in our local `sky130A.tech` file (the one in the `vsdstdcelldesign` directory).

1.  **Fix `poly.9` (Poly Resistor Spacing):**

      * Open `poly.mag` to see all the poly rules. We are interested in `poly.9`.

    > <img width="993" height="908" alt="image" src="https://github.com/user-attachments/assets/eb522a66-4fb4-46e1-a873-756a6f3e3ea0" />

      * Open `poly.9.mag`. The layout is incorrect, but Magic does not show a DRC error, meaning the rule is broken in the tech file.

    > <img width="1178" height="912" alt="image" src="https://github.com/user-attachments/assets/02ef487d-0324-4fd7-a1c2-c116f2dc693b" />

      * Open `sky130A.tech` and insert the new, corrected DRC commands.
      * In the `tkcon` window, load the fixed tech file and re-check:

    <!-- end list -->

    ```tcl
    #To load the .tech file
    tech load sky130A.tech
    drc check
    drc why
    ```

    After loading the fix, `drc why` correctly identifies the spacing error.

    > <img width="1107" height="907" alt="image" src="https://github.com/user-attachments/assets/33de2f5f-166e-4d92-a19e-a0f822d59e06" />


2.  **Fix `difftap.2` (Diff to Tap Spacing):**

      * Open `difftap.mag`. We can see the `difftap.2` layout is incorrect.

    ><img width="1145" height="917" alt="image" src="https://github.com/user-attachments/assets/4a19bdc2-5573-4dae-b070-faee339de2bc" />

      * Add the new DRC commands for `difftap.2` to the `sky130A.tech` file.
      * Reload the tech file (`tech load sky130A.tech`) and re-check to confirm the fix.

3.  **Fix `nwell.4` (N-well Complex Rule):**

      * Open the `nwell.mag` file.
      * Add the complex DRC rule logic to the `sky130A.tech` file.
      * In the `tkcon` window, load the file and set the style to `drc(full)` to enable all checks.

    <!-- end list -->

    ```tcl
    # Loading updated tech file
    tech load sky130A.tech
    # Change drc style to drc full
    drc style drc(full)
    # Must re-run drc check to see updated drc errors
    drc check
    drc why
    ```

</details>

<br>

-----

### 🏁 Conclusion

After successfully creating the layout using the Magic VLSI tool and performing DRC (Design Rule Check) through Tkcon, we can conclude:

  * The final inverter layout is **DRC clean**, meaning it adheres to all the fabrication design rules defined by the `sky130A.tech` file. This ensures the layout is manufacturable.
  * The process demonstrated the importance of geometric design constraints, such as the minimum width/spacing of metal and poly, proper via placements, and correct layer overlaps.
  * The use of Tkcon commands (like `drc check`, `drc why`, `tech load`) is critical for debugging and interactively correcting DRC errors in both the layout and the technology file itself.
</details>
