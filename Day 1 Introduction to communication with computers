## Section 1: Communication with computers

### Location of the package in an arduino board.

(lecture videos)

![image](https://github.com/user-attachments/assets/27238e24-41b2-4cfd-a8c5-2ea7f9e36c36)

The above picture is a picture of an aurdino board. The circled area is known as a **package** and is what this course is going to be about.

(lecture videos)

![image](https://github.com/user-attachments/assets/107a06fc-c5e6-42c4-8bfd-16b8e210aea1)

This picture is a simplified version of the aurdino board.

![image](https://github.com/user-attachments/assets/42229c9d-ec19-4dce-889d-1f8c69a5e697)

This image shows what we get if we zoom into the circled area (package) shown in the first diagram.
* The **pins** are what connects the logic in the chip to the external world. 
* The **pads** are what the connections have to pass through to get to the chip. 
* The **foundry IP's** are components made in a foundry.

### Conversion of programs to actual hardware that can perform the code written in the programs.

(lecture videos)

![image](https://github.com/user-attachments/assets/2239ea74-59be-46d4-a74e-d06d561708fc)

1. The C-language code is first compiled into assembly language which in our case is RISC-V
2. This assembly language is implemented uding **Hardware Description Language (HDL)** which is picorv32a in our case.
3. This is then converted to machine language which is binary in the real world.

### Coordination between Hardware and Software

(lecture videos)

![image](https://github.com/user-attachments/assets/e4c64801-13b4-49c0-b8f4-96c57a9d7c5a)

1) All your applications are installed on your **Operating System (OS)**. The applications are essentially code that is in a high level language like java or C.
2) The **compiler** is a part of your sysem software that converts the highlevel languages our applications are programmed in to a set of simpler instructions in an *executable (.exe)* file.
3) The syntax of these instructions are dependent on your hardware.
4) The **assembler** then converts these instructions to assembly level language aka. binary (1's and 0's) that the hardware can then implement.
5) All the compilers and assemblers etc. that help convert the C program to binary is known to be the **Instruction Set Architecture (ISA)** of the computer.

## Section 2: SoC design and OpenLANE

### Requirements of ***Open Source*** ASIC Design

* Hardware Description Language and RTL IP's (github.com, opencores.org, librecores.org)
* Electron Design Automation (EDA) tools. (qflow, OpenROAD, OpenLANE)
* **PDK** data (Google and Skywaters open source *[130nm PDK](https://github.com/google/skywater-pdk)*)
* In 1979, chip making was separated into design and fabrication. So Pure Play Fabs (only fabrication no design) and Fabless (only design nd no fabrication) emerged.
* The interface bbetween the fabricators and the designers was knwn as the **PDK**.

(lecture videos)

![image](https://github.com/user-attachments/assets/4e3898f3-abbe-4700-9cf7-4539fca1877d)

### Process Design Kit (PDK)

* A **Process Design Kit (PDK)** is a collection of files used to model the specific fabrication process for the EDA tools used to design an integrated circuit (IC).
* The PDK contains information about the available logic cells, routing rules and device parameters for a particular chip fabrication process.

(lecture videos)

![image](https://github.com/user-attachments/assets/a33b6ed0-2544-4d52-86ea-490c3e1d09cb)

### Simplified ASIC (RTL to GDSII) Flow

1. **Synthesis:** This stage converts the HDL code (e.g., Verilog) into a gate-level netlist. The netlist represents the circuit's functionality using basic logic gates.

(lecture videos)

![image](https://github.com/user-attachments/assets/edd36bad-aade-4703-aaa4-bb7537adbc59)

2. **Floor and Power Planning:** In this stage, the chip layout is planned. The die area is divided into sections for different functional blocks and I/O pads. The power supply network is also designed to ensure proper power distribution throughout the chip.
 
3. **Placement:** This stage involves positioning the logic cells from the netlist onto the chip layout. The goal is to find optimal locations for the cells considering factors like timing and area constraints. There are two main placement steps:

    * **Global placement:** This provides an initial placement for all cells, aiming for optimal positions but not necessarily following all layout rules. The placement in this step may not always be legal.
    * **Detailed placement:** This refines the initial placement to ensure all cells adhere to the design rules.

(lecture videos)

![image](https://github.com/user-attachments/assets/d7eba5be-7b87-4aef-a5ca-50e0b81ae6c8)

(lecture videos)

![image](https://github.com/user-attachments/assets/9fe021aa-a7c2-42d2-bbdb-a10760e62990)

4. **Clock Tree Synthesis (CTS):** A clock tree is a network of buffers and wires that distributes the clock signal to all sequential elements (flip-flops) in the design. CTS aims to minimize clock skew, which is the variation in arrival time of the clock signal at different flip-flops.

5. **Routing:** This stage involves creating connections between the placed cells using metal layers available in the fabrication process. Routing is typically done in two steps:

  * **Global routing:** This defines the general paths for the connections between cells.
  * **Detailed routing:** This implements the actual wiring connections based on the global routing plan.

6. **Sign-off:** This stage involves various checks to ensure the design meets the desired functionality and manufacturing requirements. Here are some common sign-off checks:

  * **Design Rule Checking (DRC):** This verifies if the layout adheres to the design rules specified by the PDK for the fabrication process.
  * **Layout vs Schematic (LVS):** This compares the final layout with the original gate-level netlist to ensure they are equivalent.
  * **Static Timing Analysis (STA):** This analyzes the timing delays in the design to ensure all signals meet the timing constraints and the circuit operates at the required frequency.

(lecture videos)

![image](https://github.com/user-attachments/assets/feb256ed-4dfc-4569-9fc5-f883e1552900)

* **Introduction to OpenLANE and StriVe**

* OpenLANE is an open-source design flow that aims to automate the entire process of generating a GDSII file from an RTL design, without requiring manual intervention.
* StriVe is a collection of open-source resources for SoC design, including open PDKs, EDA tools, and RTL examples.
* It provides features like Design Space Exploration to help find optimal design configurations and offers a growing collection of design examples (43 and growing).

Versions:

(lecture videos)

![image](https://github.com/user-attachments/assets/2f3d96a3-2617-4adf-a3d6-46fe1ba06da9)

### OpenLANE ASIC Design Flow:

(lecture videos)

![image](https://github.com/user-attachments/assets/39b1dfd1-20c0-435f-b80e-d04265f0689f)

## Section 3: Introduction to Opem Source EDA tools

### General Linux Commands

`cd`- Change directory

`ls`- Lists files and directories present in working directory.

`pwd`- Prints the current working directory.

`mkdir`- Makes a new directory.

`cp`- Copy.

`tar`- Archive file commands.

```bash
# Change directory to the openlane folder that contains the pdk's designs's etc.
cd Desktop/work/tools/openlane_working_dir/openlane/
```
![image](https://github.com/user-attachments/assets/a1a4fbec-fd49-427e-a949-1ee5fb6b4d22)

```bash
docker
```
![image](https://github.com/user-attachments/assets/cac7890f-bab7-436b-bf2f-b90007a811ff)

```bash
# The flow uses the script in flow.tcl
# The -interactive tag lest the flow run step by step instead of all at once as it normally would.
./flow.tcl -interactive
```
![image](https://github.com/user-attachments/assets/70712b6e-6a59-4532-b6b3-1c27e19a1cfb)

```bash
# Load the required packages required for the flow
package require openlane 0.9
```
![image](https://github.com/user-attachments/assets/12a5656b-ddef-43b5-ac8a-6278f00a91b6)

```bash
# prepare the picorv32a design for synthesis
# This step also merges lef files
prep -design picorv32a
```
![image](https://github.com/user-attachments/assets/0b903227-76ce-4274-9e81-4d60e0de678d)

*merged.lef* created by our previous command in `/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/30-01_17-13/tmp`

![image](https://github.com/user-attachments/assets/eb696fcb-18cc-4975-b0ac-2ac834c7881a)

Open by running the following commands in a new tab
```bash
cd /Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/30-01_17-13/tmp
```
```bash
# Opening merged.lef file
less merged.leff
```
![image](https://github.com/user-attachments/assets/d62fbce1-0fae-4f80-a445-5c8e909f4e02)

```bash
# Run synthesis
run_synthesis
```
![image](https://github.com/user-attachments/assets/73a24b7b-1f16-4860-9443-cfb9e4e050b7)

Screenshot of synthesis executed sucessfully.

![image](https://github.com/user-attachments/assets/8e0557d0-c091-4114-ab64-93d100fe4c33)

> [OpenLANE Docs](https://openlane.readthedocs.io/en/latest/)

Our next step is to calculate the flop ratio which is The `no. of D Flip Flops` / `Total no. of cells`

![image](https://github.com/user-attachments/assets/e47a45b1-b12e-4e5c-b3d5-aa7a8ca23731)

No. of D Flip Flops = `sky130_fd_sc_hd__dfxtp_2` = 1613

Total no of cells = `Number of cells:` = 14876

Flop ratio = 1613 / 14876 = 0.10842968539

In % Flop ratio = 10.842968539

Synthesised netlist in `/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/30-01_17-13/results/synthesis`
```bash
cd /Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/30-01_17-13/results/synthesis
```
![image](https://github.com/user-attachments/assets/4fb9bb8f-e8a7-452f-8472-6804aad52fae)

```bash
less picorv32a.synthesis.v
```
![image](https://github.com/user-attachments/assets/6b635520-08b7-4333-b598-8c19471612e9)

Screenshot of report in `/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/30-01_17-13/reports/synthesis` folder.
```bash
cd /Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/30-01_17-13/reports/synthesis
# After this use the `less` commandto view the reports.
```

![image](https://github.com/user-attachments/assets/ea9c97e4-ad12-4641-a867-92bd0d3d38de)
