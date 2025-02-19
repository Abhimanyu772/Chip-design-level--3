Section 1: Labs for CMOS Inverter ngspice Simulations
Labs for CMOS Inverter NGSPICE Simulations
In this lab, you'll simulate a CMOS inverter using Ngspice to analyze its DC characteristics, transient response, and power dissipation.

🔹 Lab 1: CMOS Inverter DC Analysis
Objective
Plot the Voltage Transfer Characteristic (VTC) of the inverter.
Determine Noise Margins (NMH, NML) and Switching Threshold (V_M).
🔹 Circuit Schematic
A basic CMOS inverter consists of:

PMOS (M1) connected to Vdd
NMOS (M2) connected to GND
Input applied to both transistor gates.
markdown
Copy
Edit
         Vdd
          │
         M1
          │
     OUT ─┼──→ Vout
          │
         M2
          │
         GND
🔹 SPICE Netlist (CMOS Inverter DC Analysis)
spice
Copy
Edit
* CMOS Inverter DC Analysis

.include "sky130_fd_pr.spice"   * Include Sky130 PDK Models

Vdd Vdd 0 1.8V        * Power Supply
Vin in 0 DC 0V        * Input Voltage Sweep

M1 out in Vdd Vdd pmos w=0.5u l=0.15u  * PMOS Transistor
M2 out in 0 0 nmos w=0.5u l=0.15u      * NMOS Transistor

* DC Sweep Analysis
.DC Vin 0 1.8 0.01V
.PLOT DC V(out)  
.END
🔹 Expected Results
VTC curve: Vout vs. Vin plot
Switching threshold (V_M): Voltage at which Vout = Vin.
Noise Margins: NMH = V_OH - V_M, NML = V_M - V_OL.
🔹 Lab 2: Transient Analysis (Dynamic Response)
Objective
Analyze rise time (t_r), fall time (t_f), propagation delays (t_PLH, t_PHL).
Observe inverter behavior for a square wave input.
🔹 SPICE Netlist (CMOS Inverter Transient Response)
spice
Copy
Edit
* CMOS Inverter Transient Analysis

.include "sky130_fd_pr.spice"

Vdd Vdd 0 1.8V            * Power Supply
Vin in 0 PULSE(0 1.8 0 1n 1n 10n 20n)  * Square Wave Input

M1 out in Vdd Vdd pmos w=0.5u l=0.15u  * PMOS Transistor
M2 out in 0 0 nmos w=0.5u l=0.15u      * NMOS Transistor

* Transient Analysis
.TRAN 1n 100n
.PLOT TRAN V(in) V(out)
.END
🔹 Expected Results
Output waveform lags behind input due to propagation delay.
Measure t_PLH & t_PHL at 50% Vdd crossing.
Measure rise (10%-90%) and fall (90%-10%) times.
🔹 Lab 3: Power Analysis
Objective
Compute static (leakage) and dynamic power.
Measure short-circuit current during switching.
🔹 SPICE Netlist (Power Analysis)
spice
Copy
Edit
* CMOS Inverter Power Analysis

.include "sky130_fd_pr.spice"

Vdd Vdd 0 1.8V
Vin in 0 PULSE(0 1.8 0 1n 1n 10n 20n)

M1 out in Vdd Vdd pmos w=0.5u l=0.15u  
M2 out in 0 0 nmos w=0.5u l=0.15u      

* Measure Power
.TRAN 1n 100n
.MEASURE TRAN avg_power AVG I(Vdd)*V(Vdd)
.END
🔹 Expected Results
Static power: Very low (leakage current when Vin = 0 or Vdd).
Dynamic power: Peaks during transitions due to switching capacitance.
🔹 Conclusion
These labs provide insight into:
✅ DC analysis – Noise margins & VTC.
✅ Transient response – Delay, rise/fall times.
✅ Power analysis – Static & dynamic power consumption.
Performing NGSPICE Analysis Using OpenLANE
Objective:

Simulate CMOS circuits (like an inverter) in OpenLANE using NGSPICE.
Analyze DC, transient, and power characteristics.
Verify standard cell behavior before integrating into ASIC design.
1️⃣ Setting Up OpenLANE for NGSPICE Simulations
Step 1: Launch OpenLANE
bash
Copy
Edit
cd openlane
./flow.tcl -interactive
Step 2: Prepare the Design
tcl
Copy
Edit
prep -design my_inverter -tag spice_sim -pdk sky130A
This creates a working directory with the necessary files.

2️⃣ Running DC Analysis in OpenLANE using NGSPICE
Step 1: Create the SPICE Netlist
Inside your design directory (designs/my_inverter/), create a file my_inverter.spice:

spice
Copy
Edit
* CMOS Inverter DC Analysis using NGSPICE in OpenLANE

.include "/openlane/pdks/sky130A/libs.tech/ngspice/sky130_fd_pr.spice"  

Vdd Vdd 0 1.8V        
Vin in 0 DC 0V       

M1 out in Vdd Vdd sky130_fd_pr__pfet_01v8 w=0.5u l=0.15u  
M2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.5u l=0.15u  

* DC Sweep Input from 0V to 1.8V
.DC Vin 0 1.8 0.01V
.PLOT DC V(out)  
.END
Step 2: Run NGSPICE in OpenLANE
bash
Copy
Edit
ngspice my_inverter.spice
Expected Output
A Voltage Transfer Characteristic (VTC) plot of Vout vs. Vin.
Identify switching threshold (Vm) and noise margins.
3️⃣ Running Transient Analysis in OpenLANE
Step 1: Create the SPICE Netlist for Transient Response
spice
Copy
Edit
* CMOS Inverter Transient Analysis using NGSPICE in OpenLANE

.include "/openlane/pdks/sky130A/libs.tech/ngspice/sky130_fd_pr.spice"  

Vdd Vdd 0 1.8V  
Vin in 0 PULSE(0 1.8 0 1n 1n 10n 20n)  

M1 out in Vdd Vdd sky130_fd_pr__pfet_01v8 w=0.5u l=0.15u  
M2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.5u l=0.15u  

* Transient Analysis
.TRAN 1n 100n
.PLOT TRAN V(in) V(out)
.END
Step 2: Run NGSPICE Simulation
bash
Copy
Edit
ngspice my_inverter.spice
Expected Output
Waveform of output (Vout) vs. input (Vin).
Measure rise time, fall time, propagation delays.
4️⃣ Power Analysis in OpenLANE
Step 1: Modify SPICE Netlist for Power Calculation
spice
Copy
Edit
* Power Analysis for CMOS Inverter using NGSPICE in OpenLANE

.include "/openlane/pdks/sky130A/libs.tech/ngspice/sky130_fd_pr.spice"  

Vdd Vdd 0 1.8V  
Vin in 0 PULSE(0 1.8 0 1n 1n 10n 20n)  

M1 out in Vdd Vdd sky130_fd_pr__pfet_01v8 w=0.5u l=0.15u  
M2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.5u l=0.15u  

* Transient Simulation
.TRAN 1n 100n
.MEASURE TRAN avg_power AVG I(Vdd)*V(Vdd)
.END
Step 2: Run NGSPICE Simulation
bash
Copy
Edit
ngspice my_inverter.spice
Expected Output
Power dissipation values (static & dynamic power).
Energy per transition calculation.
5️⃣ Automating NGSPICE in OpenLANE Flow
Modify the OpenLANE configuration to integrate NGSPICE analysis:

tcl
Copy
Edit
set ::env(RUN_SPICE_SIMULATION) 1
set ::env(SPICE_SIM_NETLIST) "designs/my_inverter/my_inverter.spice"
Then, run OpenLANE:

tcl
Copy
Edit
run_synthesis
run_floorplan
run_placement
run_spice_simulation
Conclusion
🚀 Using NGSPICE in OpenLANE, you can:
✅ Simulate DC, transient, and power analysis for CMOS circuits.
✅ Verify timing characteristics (delays, noise margins, switching threshold).
✅ Integrate SPICE verification within the ASIC flow.
Section 2: 16 mask CMOS creation proscess
16-Mask CMOS Fabrication Process
The 16-mask CMOS process is a standard VLSI (Very Large Scale Integration) fabrication technique used to manufacture modern CMOS integrated circuits (ICs). It involves multiple photolithography steps using 16 masks to define different layers in the silicon wafer.

🔹 1. Overview of CMOS Fabrication Steps
The 16-mask process consists of the following key steps:

Wafer Preparation
Well Formation (N-well & P-well)
Field Oxide Formation (LOCOS or STI)
Gate Oxide Growth
Polysilicon Deposition & Patterning
Source/Drain Implantation
Sidewall Spacer Formation
Silicidation (Low Resistance Contacts)
Interlayer Dielectric (ILD) Deposition
Contact Formation
Metal Layer Deposition & Patterning
Passivation Layer Deposition
Bond Pad Opening
Final Testing & Packaging
Each step corresponds to specific masks that define transistor and interconnect layers.

🔹 2. The 16 Masks in CMOS Fabrication
Each mask defines a specific pattern for a fabrication step:

Mask #	Mask Name	Purpose
1	N-Well Mask	Defines N-well regions in a P-type substrate.
2	P-Well Mask	Defines P-well regions in an N-type substrate.
3	Shallow Trench Isolation (STI) Mask	Creates isolation between transistors.
4	Threshold Voltage Adjustment (VT) Mask	Adjusts threshold voltage via doping.
5	Gate Oxide Mask	Defines the gate oxide region.
6	Polysilicon Mask	Defines the polysilicon gate structure.
7	LDD Implant Mask	Forms Lightly Doped Drain (LDD) for better transistor performance.
8	Sidewall Spacer Mask	Controls the formation of sidewall spacers.
9	Source/Drain Mask	Defines heavily doped source & drain regions.
10	Silicide Mask	Allows low-resistance contacts using metal silicide.
11	Contact Mask	Opens vias to connect transistors to metal layers.
12	Metal-1 Mask	Defines the first metal interconnect layer.
13	Metal-2 Mask	Defines the second metal interconnect layer.
14	Via Mask	Defines vertical connections between metal layers.
15	Passivation Mask	Deposits a protective layer over the IC.
16	Bond Pad Mask	Opens bond pads for external connections.
🔹 3. CMOS Fabrication Steps in Detail
(1) Wafer Preparation
Material: High-purity silicon wafer.
Doping: P-type or N-type depending on substrate type.
(2) Well Formation (N-Well & P-Well)
Photolithography and ion implantation define wells for PMOS and NMOS transistors.
(3) Isolation (Shallow Trench Isolation - STI)
Defines isolation trenches between transistors to prevent leakage.
(4) Gate Oxide & Polysilicon Deposition
Thin SiO₂ is grown for the transistor gate oxide.
Polysilicon is deposited and patterned for the transistor gate.
(5) Lightly Doped Drain (LDD) Formation
Prevents hot carrier effects by reducing electric field near the drain.
(6) Source/Drain Implantation
Defines heavily doped regions for NMOS and PMOS transistors.
(7) Silicidation (Low Resistance Contacts)
Nickel or Titanium Silicide is deposited to improve conductivity.
(8) Metal Deposition & Patterning
Multiple layers of metal (e.g., Aluminum, Copper) are deposited to form interconnects.
(9) Passivation & Bonding
Final protective layers are added before the IC is packaged and tested.
🔹 4. Advantages of the 16-Mask CMOS Process
✅ High Density Integration – Enables compact transistor layout.
✅ Low Power Consumption – Suitable for mobile and IoT devices.
✅ Scalability – Used in sub-100nm and FinFET technologies.
✅ Better Performance – LDD, STI, and Silicidation enhance transistor speed.

🔹 5. Summary
The 16-mask CMOS process enables high-performance semiconductor manufacturing by precisely defining wells, gates, interconnects, and passivation layers. It is the backbone of modern ASIC, FPGA, and microprocessor fabrication.

