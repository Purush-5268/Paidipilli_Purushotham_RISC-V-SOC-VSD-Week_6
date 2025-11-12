## 📅 Day 3: Cell Design, Layout, & Characterization 
This day is broken into three parts:
1. **Part 1:** Revisiting IO placement, understanding SPICE simulation, and cloning the vsdstdcelldesign lab.
2. **Part 2:** Inspecting the inverter layout, understanding the CMOS fabrication process, and extracting a SPICE netlist *from* the layout.
3. **Part 3:** Building a testbench for the extracted netlist, running the post-layout SPICE simulation, and introducing Magic's DRC (Design Rule Check) system.
---
### Part 1: IO Placer & SPICE Simulation Setup
<details><summary><b>💡 Concept: CMOS Inverter Simulation</b></summary>

#### 📜 SPICE Deck Creation for CMOS Inverter
A **SPICE deck** (or netlist) describes circuit connectivity and component parameters. It’s the foundation for characterizing a cell such as a CMOS inverter.  
To simulate it correctly, we create a testbench with these key steps:
1. **Instantiating the Inverter:** Call the inverter subcircuit (`.subckt`).
2. **Adding Power Supplies:** Connect `VDD` and `GND` voltage sources (e.g., 2.5 V and 0 V).
3. **Adding Stimulus:** Use a `VPULSE` source at the input to generate a switching waveform.
4. **Adding Load:** Add a capacitor (`Cload = 10 fF`) at the output to model load of next-stage gates.
5. **Simulation Commands:** `.dc` for DC sweep, `.tran` for transient analysis.

#### ⚡ Component Connectivity & Node Naming
Each MOSFET requires correct node connections:
- **PMOS (M1):** Drain → out, Gate → in, Source + Substrate → VDD  
- **NMOS (M2):** Drain → out, Gate → in, Source + Substrate → 0 (GND)

We define nodes like **Vin, Vout, VDD, 0**. The substrate pin tunes transistor threshold voltage.

#### ⚙️ Model Files
Model files include NMOS/PMOS parameters from the Sky130 PDK for realistic simulation.

#### ⚡ Switching Threshold (Vm)
The **Switching Threshold (Vm)** occurs when `Vin = Vout`.  
- Ideally ≈ `VDD / 2` for a balanced inverter.  
- In practice, depends on PMOS/NMOS width ratio (βp / βn).  
Sweeping Vin from 0 → 2.5 V using `.dc` identifies Vm.  
If both transistors conduct near Vm, short current flows (VDD → GND).

> Increasing PMOS width (e.g., 3× NMOS) centers the transfer curve and improves noise margin.

#### 📊 Static vs Dynamic Simulation
- **Static (.dc):** Measures Vm and leakage.  
- **Dynamic (.tran):** Measures rise/fall delays and propagation time.
</details>
<details><summary><b>🔬 Lab: IO Placer Revision & vsdstdcelldesign Clone</b></summary>

#### 🔄 IO Placer Revision
In OpenLANE, IO pins’ distribution is controlled by `FP_IO_MODE`.  
Default `1` = equidistant pins.  
Setting `2` clusters them on one side for custom floorplans.

```bash
# Navigate to the configurations directory
cd Desktop/work/tools/openlane_working_dir/openlane/configurations
less floorplan.tcl
````

> <img width="1920" height="1080" alt="less floorplantcl" src="https://github.com/user-attachments/assets/380b50ff-47d3-4877-8d47-d48183425fdd" />

Inside OpenLANE interactive shell:

```tcl
set ::env(FP_IO_MODE) 2
run_floorplan
```

> <img width="1920" height="1080" alt="Set 2 floorplan" src="https://github.com/user-attachments/assets/b4077c80-fabd-46cb-8644-04c2a7418bb6" />

View floorplan in Magic:

```bash
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/<NEW_RUN_DIR>/results/floorplan
magic -T ... lef read ... def read picorv32a.floorplan.def &

# or alternatively, you can use the full path command below:
magic -T /home/purush/Desktop/work/tools/Openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &

```

> <img width="1920" height="1080" alt="Magic after set 2 " src="https://github.com/user-attachments/assets/d484ba17-f382-4ea5-8ddf-bfb015d233fa" />

Pins now appear clustered, not equidistant — confirming mode change.

#### 📦 Cloning `vsdstdcelldesign`

Clone the repository to access the CMOS inverter layout:

```bash
cd Desktop/work/tools/openlane_working_dir/openlane
git clone https://github.com/nickson-jose/vsdstdcelldesign
```

> <img width="1920" height="1080" alt="gitclone" src="https://github.com/user-attachments/assets/c9c18c99-2680-4ef6-a980-5b413aaf8c76" />

```bash
cd vsdstdcelldesign
cp /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech .
ls
magic -T sky130A.tech sky130_inv.mag &
```

> <img width="1920" height="1080" alt="Magic command to open inv" src="https://github.com/user-attachments/assets/ba6379a3-92ad-4685-9609-21d5a8422b4a" />

Opens the **custom inverter layout** with Sky130A tech file.
You can inspect layers like diffusion, poly, contacts, and metals forming the inverter.

</details>

### Part 2 – Layout Inspection & SPICE Extraction  

<details><summary><b>💡 Theory: CMOS Fabrication Process</b></summary>

### Create Active Regions  
We begin with a P-type silicon substrate (5–50 Ω·cm, (100) orientation).  
A 40 nm SiO₂ layer is first deposited for insulation, followed by an 80 nm Si₃N₄ layer to serve as an oxidation barrier.  
Then, a photoresist (~1 µm) is deposited over the wafer, and **Mask 1** is used for UV exposure to define the active transistor regions.  
After exposure, the unwanted photoresist is removed, revealing the regions for etching.  
Si₃N₄ is etched in the exposed regions, and photoresist is stripped.  
The wafer undergoes **LOCOS oxidation** (Local Oxidation of Silicon) to grow thick SiO₂, isolating PMOS and NMOS regions.  
Finally, Si₃N₄ is removed using hot phosphoric acid, leaving distinct transistor pockets.

><img width="1083" height="493" alt="image" src="https://github.com/user-attachments/assets/84ca689a-f8e5-435e-bbed-e1ed0d631dc9" />

> <img width="1105" height="562" alt="image" src="https://github.com/user-attachments/assets/fedc3932-29b1-4561-8c24-3bb3f407eb84" />
---

### Formation of N-well and P-well  
To create separate wells for PMOS and NMOS:  
Protect one region with photoresist while exposing the other using **Mask 2**.  
Boron ions (~200 keV) are implanted to form the **P-well**, followed by annealing to activate dopants.  
Similarly, Phosphorus ions are implanted using **Mask 3** to create the **N-well**.  
Both wells are then driven into the substrate using high-temperature diffusion.

> <img width="908" height="378" alt="image" src="https://github.com/user-attachments/assets/84d42137-a546-449d-a451-17c010ff09e7" />
---

### Formation of Gate Terminal  
The gate terminal controls the threshold voltage (Vth) and transistor behavior.  
Doping concentration and oxide thickness determine Vth.  
First, light ion implantation (Boron in P-substrate with Mask 4, Arsenic in N-well with Mask 5) adjusts threshold levels.  
The damaged oxide is removed with HF and regrown to obtain a high-quality oxide layer.  
Then, **polysilicon** is deposited, doped for low resistance, and patterned using **Mask 6** to form gate electrodes.

> <img width="957" height="565" alt="image" src="https://github.com/user-attachments/assets/e1b6f6aa-ff89-4b74-b9ff-900fdc3f505a" />

---

### Lightly Doped Drain (LDD) Formation  
This process reduces hot-carrier and short-channel effects.  
Using **Mask 7**, a light Phosphorus implant forms N⁻ regions for NMOS.  
Using **Mask 8**, a light Boron implant forms P⁻ regions for PMOS.  
After this, a thick SiO₂ or Si₃N₄ layer is deposited and etched anisotropically to form **side-wall spacers** around the gate.

><img width="882" height="556" alt="image" src="https://github.com/user-attachments/assets/e475d203-4d3d-4f16-86fd-06ce29aaf72b" />

---

### Source & Drain Formation  
A thin screen oxide layer is grown to prevent channeling.  
- For NMOS: Arsenic (~75 keV) ions are implanted using **Mask 9** to form N⁺ regions.  
- For PMOS: Boron (~50 keV) ions are implanted using **Mask 10** to form P⁺ regions.  
Finally, high-temperature annealing (~1000°C) activates dopants and completes source/drain formation.

> <img width="1150" height="498" alt="image" src="https://github.com/user-attachments/assets/f858faae-ed45-4f75-a276-b9b690fa0378" />
---

### Local Interconnect Formation  
To create local interconnects, first remove the screen oxide and sputter-deposit **Titanium (Ti)**.  
When heated (~650–700°C in N₂), Ti reacts with silicon to form **TiSi₂** and **TiN**.  
TiN acts as a local interconnect and barrier.  
Using **Mask 11**, unwanted TiN is etched away, forming contacts to gate, source, and drain terminals.

> <img width="1082" height="518" alt="image" src="https://github.com/user-attachments/assets/13e10698-e192-4e78-b3b3-958169906324" />
---

### Contact Hole Formation  
Planarize the wafer surface using a dielectric layer and etch holes for metal contacts.  
This enables electrical connections between different interconnect layers.

><img width="937" height="533" alt="image" src="https://github.com/user-attachments/assets/e54f454a-1dfd-408b-ac1c-8b4c3e84e6e3" />

---

### Higher Level Metal Formation  
To connect devices globally, planarize the wafer surface with a thick SiO₂ layer via **CMP (Chemical Mechanical Polishing)**.  
Deposit a TiN (~10 nm) layer as adhesion/barrier, then deposit **Tungsten (W)** for via filling.  
After planarization, deposit **Aluminum (Al)** for the first metal layer and pattern using **Mask 12–15** to form connections.  
The top is protected with Si₃N₄ or SiO₂.

> <img width="1125" height="551" alt="image" src="https://github.com/user-attachments/assets/1a7c42ed-376e-4a9d-a1c4-9c14750c1780" />

---

### Final CMOS Structure  
After completing all metal layers, a final protective Si₃N₄ layer is deposited.  
This completes the CMOS fabrication with all interconnects and metal routing.

> <img width="981" height="691" alt="image" src="https://github.com/user-attachments/assets/134c8aeb-7c34-4d76-9117-a81e0c26327b" />

</details>

<details><summary><b>🔬 Lab: Layout Inspection, Extraction & Characterization</b></summary>

### Lab Introduction to Sky130 Basic Layers Layout and LEF using Inverter  
We inspect `sky130_inv.mag` using Magic to understand the layer mapping of CMOS transistors.  
> <img width="1920" height="1080" alt="Layout" src="https://github.com/user-attachments/assets/d380e3dd-7905-4cd5-93d1-730c5bd80cd4" />

Each color in Magic corresponds to a layer in Sky130:  
- **Red (poly):** Gate terminal  
- **Green (n-diff):** NMOS active region  
- **Orange-brown (p-diff):** PMOS active region  
- **Gray hatch (n-well):** PMOS substrate  
- **Purple hatch (met1):** Interconnect  

Use the `what` command in Magic’s tkcon to verify connections.  
> <img width="1920" height="1080" alt="pmos identified by using what" src="https://github.com/user-attachments/assets/c5cdb891-e17c-4870-8dba-e1ea856cfd95" />  
> <img width="1920" height="1080" alt="nmos define what" src="https://github.com/user-attachments/assets/9fffc156-a516-4109-a2c8-46bf4728e577" />  
> <img width="1920" height="1080" alt="Source connections" src="https://github.com/user-attachments/assets/4cc50523-0ad8-46a9-bb48-558f682b3ce3" />

---

### Lab Steps to Create Std Cell Layout and Extract SPICE Netlist  

Now that we have a layout, we perform **layout extraction** in Magic.  
This generates a `.ext` file containing parasitic resistances and capacitances, which can then be converted to a `.spice` file for simulation.

1. **Extract Layout (.ext format)**  
```tcl
extract all
````

> <img width="1920" height="1080" alt="Extract command" src="https://github.com/user-attachments/assets/d47f93bb-9b19-42e5-943a-b7b1029bc929" />

2. **Generate SPICE File with Parasitics**

```tcl
ext2spice cthresh 0 rthresh 0
ext2spice
```

This creates `sky130_inv.spice`, a subcircuit representing the inverter with real-world parasitics.

> <img width="1920" height="1080" alt="ext2spice " src="https://github.com/user-attachments/assets/df0e3ace-7c9a-4953-a3bb-e5b6b133e8ed" />

</details>

---

### Part 3: Post-Layout Characterization & DRC
<details> <summary><b>🔬 Lab: Post-Layout SPICE Characterization</b></summary>

----

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
----

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

<details> <summary><b>📐 Lab: Sky130 Tech File & DRC Rules</b></summary>

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
