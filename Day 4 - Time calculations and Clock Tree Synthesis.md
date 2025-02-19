Section 1: Timing Modelling using Delay Tables
Creation of lef file
We need only certain things like oundaries, power and ground rails, and the inputs and outputs datab for the PnR step that is performed by OpenLANE. The mag file has a lot of information. Most of this information is not needed for placement and routing. therefore we create a .lef file containing only the things we need for the PnR step (boundaries, power and ground rails, and the input and output data).

However, there are certain things we need to take care of.

The input and output ports must lie at the intersection of the horizontal and vertical tracks.
The width of the standard cell should be in odd multiples of the horizontal track pitch.
The height of the standard cell should be in even multiples of the vertical track pitch.
The track info will be present in Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/openlane/sky130_fd_sc_hd/tracks.info

(Image of tracks.info)

image

Tracks are used in the routing stage. Routs are the leftovers of metal layers.

grid command syntax in tkcon

grid [xSpacing [ySpacing [xOrigin yOrigin]]]
Info from tracks.info

li1 X 0.23 0.46
li1 Y 0.17 0.34
Adjust grid according to info from tracks.info

 grid 0.46um 0.34um 0.23um 0.17um
New grid

image

There is at least 1 intersection for a port

image image

So the 1st requirement is satisfied (The input and output ports must lie at the intersection of the horizontal and vertical tracks).

The dimensions of the standard cell is shown as the white box.

image

It is clearly seen that the width of the standard cell is 3 grid boxes. So the 2nd requirement is also satisfied (The width of the standard cell should be in odd multiples of the horizontal track pitch).

image

It is observed that the height of the standard cell is 8 grid boxes. So the 3rd requirement is also satisfied (The height of the standard cell should be in odd multiples of the vertical track pitch).

Ports are defined as pins while extracting the lef file.

We insert text to name the ports by clicking text under the edit tab in magic.

image

Set ports following the below diagram (it is already done for us)

(Lecture Videos)

image

Save the file and give it a name

save sky130_vsdinv.mag
image

# Open the created file
magic -T sky130A.tech sky130_vsdinv.mag &
image

Create lef file

lef write
image

On opening lef file

image

image

Inserting Inverter lef file in Picorv32a flow
Copy the lef file to the picorv32a src folder. This is done to ensure that all our design files are in the same folder.

cd Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign
image

cp sky130_vsdinv.lef /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src
image

# Copy library files to src folder
cp libs/sky130_fd_sc_hd__* ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
image

Modify  /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/config.tcl from this..

image

to this..

image

by adding the following.

set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
# This tells the flow to include an extra lef
set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
Now we run the OpenLANE flow with these variables

cd Desktop/work/tools/openlane_working_dir/openlane
image

docker
image

./flow.tcl -interactive
# The -interactive flag opens flow.tcl in interactive mode (step by step)
image

package require openlane 0.9
image

prep -design picorv32a -tag 31-01_18-27 -overwrite
# This command makes sure that future commands follow the design specifications as set in picorv32a
# The tags at the end keep all your work in a single directory instead of creating multiple directories each time you run the OpenLANE flow
image

# Include lefs. This isfrom the vsdstdcelldesign repo
set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
image

set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
image

add_lefs -src $lefs
image

run_synthesis
# runs the synthesis step in OpenLANE
image

Instances of our custom cell

image

image

tns -711.59 wns -23.89 Chip area for module '\picorv32a': 147712.918400

image

View value of $::env(SYNTH_STRATEGY)

echo $::env(SYNTH_STRATEGY) 
image

Change variables hpoing to improve slack.

 set ::env(SYNTH_STRATEGY) 1
image

set ::env(SYNTH_SIZING) 1
image

run_synthesis
# Running synthesis withchanges
image

image

tns -711.59 wns -23.89

run_floorplan
# Runs the floorplan stepD in OpenLANE
image

** ERROR!!! **

image

image

Since we are facing unexpected un-explainable error while using run_floorplan command, we can instead use the following set of commands available based on information from Desktop/work/tools/openlane_working_dir/openlane/scripts/tcl_commands/floorplan.tcl and also based on Floorplan Commands section in Desktop/work/tools/openlane_working_dir/openlane/docs/source/OpenLANE_commands.md

image

image

# Follwing commands are alltogather sourced in `run_floorplan` command according to `OpenLANE_commands.md`. So we wil now run them directly without making use of the `run_floorplan` command
init_floorplan
place_io
tap_decap_or
image

image

image

run_placement
# Runs the placement step in OpenLANE
image

image

Now we have to check weather or cell has been included in the flow.

cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/31-01_18-27/results/placement/
image

Load it in magic

magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
image

image

Our cell

image

After running expand in tkcon

image

Running STA and minimising Slack
Create new file in OpenLANE directory named pre_sta.conf

set_cmd_units -time ns -capacitance pF -current mA -voltage V -resistance kOhm -distance um

read_liberty -max /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib

read_liberty -min /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib

read_verilog /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/31-01_18-27/results/synthesis/picorv32a.synthesis.v

link_design picorv32a

read_sdc /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/my_base.sdc

report_checks -path_delay min_max -fields {slew trans net cap input_pin}
report_tns
report_wns
Create my_base.sdc for STA analysis in openlane/designs/picorv32a/src directory based on the file openlane/scripts/base.sdc

set ::env(CLOCK_PORT) clk
set ::env(CLOCK_PERIOD) 24.73
#set ::env(SYNTH_DRIVING_CELL) sky130_vsdinv
set ::env(SYNTH_DRIVING_CELL) sky130_fd_sc_hd__inv_8
set ::env(SYNTH_DRIVING_CELL_PIN) Y
set ::env(SYNTH_CAP_LOAD) 17.653
set ::env(IO_PCT) 0.2
set ::env(SYNTH_MAX_FANOUT) 6

create_clock [get_ports $::env(CLOCK_PORT)]  -name $::env(CLOCK_PORT)  -period $::env(CLOCK_PERIOD)
set input_delay_value [expr $::env(CLOCK_PERIOD) * $::env(IO_PCT)]
set output_delay_value [expr $::env(CLOCK_PERIOD) * $::env(IO_PCT)]
puts "\[INFO\]: Setting output delay to: $output_delay_value"
puts "\[INFO\]: Setting input delay to: $input_delay_value"

set_max_fanout $::env(SYNTH_MAX_FANOUT) [current_design]

set clk_indx [lsearch [all_inputs] [get_port $::env(CLOCK_PORT)]]
#set rst_indx [lsearch [all_inputs] [get_port resetn]]
set all_inputs_wo_clk [lreplace [all_inputs] $clk_indx $clk_indx]
#set all_inputs_wo_clk_rst [lreplace $all_inputs_wo_clk $rst_indx $rst_indx]
set all_inputs_wo_clk_rst $all_inputs_wo_clk

# correct resetn
set_input_delay $input_delay_value  -clock [get_clocks $::env(CLOCK_PORT)] $all_inputs_wo_clk_rst
#set_input_delay 0.0 -clock [get_clocks $::env(CLOCK_PORT)] {resetn}
set_output_delay $output_delay_value  -clock [get_clocks $::env(CLOCK_PORT)] [all_outputs]

# TODO set this as parameter
set_driving_cell -lib_cell $::env(SYNTH_DRIVING_CELL) -pin $::env(SYNTH_DRIVING_CELL_PIN) [all_inputs]
set cap_load [expr $::env(SYNTH_CAP_LOAD) / 1000.0]
puts "\[INFO\]: Setting load to: $cap_load"
set_load  $cap_load [all_outputs]
Now with these 2 files run STA

Open a new terminal

cd Desktop/work/tools/openlane_working_dir/openlane
Run the sta according to the details given in the file pre_sta.conf

sta pre_sta.conf
image

image

It is observed tha more fanout causes more delay. So we set a maximum value for this veiable env(SYNTH_MAX_FANOUT) to limit the maximum fanout.

We do so by re-running the flow, editing the variable and then running synthesis.

prep -design picorv32a -tag 31-01_18-27 -overwrite
image

set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
image

add_lefs -src $lefs
image

set ::env(SYNTH_SIZING) 1
image

# Editing the max fanout variable
set ::env(SYNTH_MAX_FANOUT) 4
image

echo $::env(SYNTH_DRIVING_CELL)
image

run_synthesis
image

Open the STA file and run it to perform the STA after editing the max fanout setting.

cd Desktop/work/tools/openlane_working_dir/openlane
image

sta pre_sta.conf
image

Now we take timing ECO fixes to remove all violations What this means is that we find overworked cells and replace them with stronger cells. Suppose we have a cell of strength 1 driving 4 fanouts. This cell is overloaded and causes delay. Se we replace this cell with a cell that has more strength (like strength 4).

Our STA results-

Fanout     Cap    Slew   Delay    Time   Description
-----------------------------------------------------------------------------
                  0.00    0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock network delay (ideal)
                  0.00    0.00    0.00 ^ _29052_/CLK (sky130_fd_sc_hd__dfxtp_2)
                  0.06    0.58    0.58 v _29052_/Q (sky130_fd_sc_hd__dfxtp_2)
     4    0.01                           irq_pending[3] (net)
                  0.06    0.00    0.58 v _14460_/A (sky130_vsdinv)
                  0.19    0.17    0.75 ^ _14460_/Y (sky130_vsdinv)
     3    0.01                           _11622_ (net)
                  0.19    0.00    0.75 ^ _14461_/B2 (sky130_fd_sc_hd__o22ai_2)
                  0.09    0.14    0.89 v _14461_/Y (sky130_fd_sc_hd__o22ai_2)
     1    0.00                           _11623_ (net)
                  0.09    0.00    0.89 v _14462_/C1 (sky130_fd_sc_hd__a221o_2)
                  0.08    0.55    1.44 v _14462_/X (sky130_fd_sc_hd__a221o_2)
     1    0.00                           _11624_ (net)
                  0.08    0.00    1.44 v _14481_/A (sky130_fd_sc_hd__or4_2)

 # OR gate of strength 2 driving OA gate has more delay
 
 **                  0.21    1.53    2.97 v _14481_/X (sky130_fd_sc_hd__or4_2)
     1    0.00                           _11643_ (net)
                  0.21    0.00    2.97 v _14509_/A1 (sky130_fd_sc_hd__o2111a_2)
**                  0.07    0.54    3.51 v _14509_/X (sky130_fd_sc_hd__o2111a_2)
     2    0.00                           _11671_ (net)
                  0.07    0.00    3.51 v _14510_/C (sky130_fd_sc_hd__or3_2)
   **               0.21    1.04    4.55 v _14510_/X (sky130_fd_sc_hd__or3_2)
     4    0.01                           _11672_ (net)**
                  0.21    0.00    4.55 v _14513_/A (sky130_fd_sc_hd__or2_2)
                  0.11    0.71    5.26 v _14513_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _11674_ (net)
                  0.11    0.00    5.26 v _14514_/C (sky130_fd_sc_hd__or3_2)
    **              0.20    1.03    6.29 v _14514_/X (sky130_fd_sc_hd__or3_2)
     4    0.01                           _11675_ (net)**
                  0.20    0.00    6.29 v _15166_/B (sky130_fd_sc_hd__or2_2)
                  0.11    0.67    6.96 v _15166_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12148_ (net)
                  0.11    0.00    6.96 v _15167_/C (sky130_fd_sc_hd__or3_2)
                  0.17    0.98    7.94 v _15167_/X (sky130_fd_sc_hd__or3_2)
     2    0.00                           _12149_ (net)
                  0.17    0.00    7.94 v _15168_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.70    8.65 v _15168_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12150_ (net)
                  0.14    0.00    8.65 v _15169_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65    9.29 v _15169_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12151_ (net)
                  0.12    0.00    9.29 v _15170_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68    9.98 v _15170_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12152_ (net)
                  0.14    0.00    9.98 v _15171_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   10.63 v _15171_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12153_ (net)
                  0.12    0.00   10.63 v _15172_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   11.31 v _15172_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12154_ (net)
                  0.14    0.00   11.31 v _15173_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   11.96 v _15173_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12155_ (net)
                  0.12    0.00   11.96 v _15174_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   12.64 v _15174_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12156_ (net)
                  0.14    0.00   12.64 v _15175_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   13.29 v _15175_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12157_ (net)
                  0.12    0.00   13.29 v _15176_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   13.97 v _15176_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12158_ (net)
                  0.14    0.00   13.97 v _15177_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   14.62 v _15177_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12159_ (net)
                  0.12    0.00   14.62 v _15178_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   15.30 v _15178_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12160_ (net)
                  0.14    0.00   15.30 v _15179_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   15.95 v _15179_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12161_ (net)
                  0.12    0.00   15.95 v _15180_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   16.63 v _15180_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12162_ (net)
                  0.14    0.00   16.63 v _15181_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   17.28 v _15181_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12163_ (net)
                  0.12    0.00   17.28 v _15182_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   17.96 v _15182_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12164_ (net)
                  0.14    0.00   17.96 v _15183_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   18.61 v _15183_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12165_ (net)
                  0.12    0.00   18.61 v _15184_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   19.29 v _15184_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12166_ (net)
                  0.14    0.00   19.29 v _15185_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   19.94 v _15185_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12167_ (net)
                  0.12    0.00   19.94 v _15186_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   20.62 v _15186_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12168_ (net)
                  0.14    0.00   20.62 v _15187_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   21.27 v _15187_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12169_ (net)
                  0.12    0.00   21.27 v _15188_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   21.95 v _15188_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12170_ (net)
                  0.14    0.00   21.95 v _15189_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   22.60 v _15189_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12171_ (net)
                  0.12    0.00   22.60 v _15190_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   23.28 v _15190_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12172_ (net)
                  0.14    0.00   23.28 v _15191_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   23.93 v _15191_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12173_ (net)
                  0.12    0.00   23.93 v _15192_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   24.61 v _15192_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12174_ (net)
                  0.14    0.00   24.61 v _15193_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   25.26 v _15193_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12175_ (net)
                  0.12    0.00   25.26 v _15194_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   25.94 v _15194_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12176_ (net)
                  0.14    0.00   25.94 v _15195_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   26.59 v _15195_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12177_ (net)
                  0.12    0.00   26.59 v _15196_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   27.27 v _15196_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12178_ (net)
                  0.14    0.00   27.27 v _15197_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   27.92 v _15197_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12179_ (net)
                  0.12    0.00   27.92 v _15198_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   28.60 v _15198_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12180_ (net)
                  0.14    0.00   28.60 v _15199_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   29.25 v _15199_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12181_ (net)
                  0.12    0.00   29.25 v _15200_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   29.93 v _15200_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12182_ (net)
                  0.14    0.00   29.93 v _15201_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   30.58 v _15201_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12183_ (net)
                  0.12    0.00   30.58 v _15202_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   31.27 v _15202_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12184_ (net)
                  0.14    0.00   31.27 v _15203_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   31.91 v _15203_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12185_ (net)
                  0.12    0.00   31.91 v _15204_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   32.60 v _15204_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12186_ (net)
                  0.14    0.00   32.60 v _15205_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   33.25 v _15205_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12187_ (net)
                  0.12    0.00   33.25 v _15206_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   33.93 v _15206_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12188_ (net)
                  0.14    0.00   33.93 v _15207_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   34.58 v _15207_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12189_ (net)
                  0.12    0.00   34.58 v _15208_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   35.26 v _15208_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12190_ (net)
                  0.14    0.00   35.26 v _15209_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   35.91 v _15209_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12191_ (net)
                  0.12    0.00   35.91 v _15210_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   36.59 v _15210_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12192_ (net)
                  0.14    0.00   36.59 v _15211_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   37.24 v _15211_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12193_ (net)
                  0.12    0.00   37.24 v _15212_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   37.92 v _15212_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12194_ (net)
                  0.14    0.00   37.92 v _15213_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   38.57 v _15213_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12195_ (net)
                  0.12    0.00   38.57 v _15214_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   39.25 v _15214_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12196_ (net)
                  0.14    0.00   39.25 v _15215_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   39.90 v _15215_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12197_ (net)
                  0.12    0.00   39.90 v _15216_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   40.58 v _15216_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12198_ (net)
                  0.14    0.00   40.58 v _15217_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   41.23 v _15217_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12199_ (net)
                  0.12    0.00   41.23 v _15218_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   41.91 v _15218_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12200_ (net)
                  0.14    0.00   41.91 v _15219_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   42.56 v _15219_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12201_ (net)
                  0.12    0.00   42.56 v _15220_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   43.24 v _15220_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12202_ (net)
                  0.14    0.00   43.24 v _15221_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   43.89 v _15221_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12203_ (net)
                  0.12    0.00   43.89 v _15222_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   44.57 v _15222_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12204_ (net)
                  0.14    0.00   44.57 v _15223_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   45.22 v _15223_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12205_ (net)
                  0.12    0.00   45.22 v _15224_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   45.90 v _15224_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12206_ (net)
                  0.14    0.00   45.90 v _15225_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.65   46.55 v _15225_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12207_ (net)
                  0.12    0.00   46.55 v _15226_/B (sky130_fd_sc_hd__or2_2)
                  0.14    0.68   47.23 v _15226_/X (sky130_fd_sc_hd__or2_2)
     3    0.01                           _12208_ (net)
                  0.14    0.00   47.23 v _15227_/B (sky130_fd_sc_hd__or2_2)
                  0.12    0.66   47.89 v _15227_/X (sky130_fd_sc_hd__or2_2)
     2    0.00                           _12209_ (net)
                  0.12    0.00   47.89 v _15230_/B2 (sky130_fd_sc_hd__o221a_2)
                  0.07    0.44   48.33 v _15230_/X (sky130_fd_sc_hd__o221a_2)
     1    0.00                           _03928_ (net)
                  0.07    0.00   48.33 v _30440_/D (sky130_fd_sc_hd__dfxtp_2)
                                 48.33   data arrival time

                  0.00   24.73   24.73   clock clk (rise edge)
                          0.00   24.73   clock network delay (ideal)
                          0.00   24.73   clock reconvergence pessimism
                                 24.73 ^ _30440_/CLK (sky130_fd_sc_hd__dfxtp_2)
                         -0.29   24.44   library setup time
                                 24.44   data required time
-----------------------------------------------------------------------------
                                 24.44   data required time
                                -48.33   data arrival time
-----------------------------------------------------------------------------
                                -23.90   slack (VIOLATED)
Commands to replace cells.

# Reports all the connections to a net
report_net -connections _11672_
image

# Checks command syntax
help replace_cell
image

# Replaces cell
replace_cell _14510_ sky130_fd_sc_hd__or3_4
image

Image of replaced cell

image

# Generating custom timing report
report_checks -fields {net cap slew input_pins} -digits 4
image

Now we do this for all the cells that have been highlighted above in the STA

2nd cell replacement

report_net -connections _11675_
image

replace_cell _14514_ sky130_fd_sc_hd__or3_4
image

report_checks -fields {net cap slew input_pins} -digits 4
image

Image of replaced cell

image

3rd cell replacement

report_net -connections _11643_
image

replace_cell _14481_ sky130_fd_sc_hd__or4_4
image

report_checks -fields {net cap slew input_pins} -digits 4
image

STA Results

Fanout       Cap      Slew     Delay      Time   Description
-------------------------------------------------------------------------------------
                    0.0000    0.0000    0.0000   clock clk (rise edge)
                              0.0000    0.0000   clock network delay (ideal)
                    0.0000    0.0000    0.0000 ^ _29043_/CLK (sky130_fd_sc_hd__dfxtp_2)
                    0.0581    0.5838    0.5838 v _29043_/Q (sky130_fd_sc_hd__dfxtp_2)
     4    0.0069                                 irq_pending[12] (net)
                    0.0581    0.0000    0.5838 v _14484_/A (sky130_vsdinv)
                    0.1864    0.1676    0.7515 ^ _14484_/Y (sky130_vsdinv)
     3    0.0139                                 _11646_ (net)
                    0.1864    0.0000    0.7515 ^ _14486_/A2 (sky130_fd_sc_hd__o22ai_2)
                    0.0878    0.1674    0.9189 v _14486_/Y (sky130_fd_sc_hd__o22ai_2)
     1    0.0021                                 _11648_ (net)
                    0.0878    0.0000    0.9189 v _14487_/C1 (sky130_fd_sc_hd__a221o_2)
                    0.0784    0.5469    1.4658 v _14487_/X (sky130_fd_sc_hd__a221o_2)
     1    0.0013                                 _11649_ (net)
                    0.0784    0.0000    1.4658 v _14506_/A (sky130_fd_sc_hd__or4_2)
                    0.2092    1.5317    2.9975 v _14506_/X (sky130_fd_sc_hd__or4_2)
     1    0.0023                                 _11668_ (net)
                    0.2092    0.0000    2.9975 v _14509_/A2 (sky130_fd_sc_hd__o2111a_2)
                    0.0792    0.5193    3.5168 v _14509_/X (sky130_fd_sc_hd__o2111a_2)
     2    0.0044                                 _11671_ (net)
                    0.0792    0.0000    3.5168 v _14510_/C (sky130_fd_sc_hd__or3_4)
                    0.1349    0.6755    4.1923 v _14510_/X (sky130_fd_sc_hd__or3_4)
     4    0.0089                                 _11672_ (net)
                    0.1349    0.0000    4.1923 v _14513_/A (sky130_fd_sc_hd__or2_2)
                    0.1182    0.6880    4.8803 v _14513_/X (sky130_fd_sc_hd__or2_2)
     2    0.0034                                 _11674_ (net)
                    0.1182    0.0000    4.8803 v _14514_/C (sky130_fd_sc_hd__or3_4)
                    0.1290    0.6794    5.5597 v _14514_/X (sky130_fd_sc_hd__or3_4)
     4    0.0070                                 _11675_ (net)
                    0.1290    0.0000    5.5597 v _15166_/B (sky130_fd_sc_hd__or2_2)
                    0.1148    0.6414    6.2011 v _15166_/X (sky130_fd_sc_hd__or2_2)
     2    0.0032                                 _12148_ (net)
                    0.1148    0.0000    6.2011 v _15167_/C (sky130_fd_sc_hd__or3_2)
                    0.1692    0.9831    7.1842 v _15167_/X (sky130_fd_sc_hd__or3_2)
     2    0.0035                                 _12149_ (net)
                    0.1692    0.0000    7.1842 v _15168_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.7026    7.8868 v _15168_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12150_ (net)
                    0.1422    0.0000    7.8868 v _15169_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492    8.5360 v _15169_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12151_ (net)
                    0.1162    0.0000    8.5360 v _15170_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813    9.2174 v _15170_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12152_ (net)
                    0.1422    0.0000    9.2174 v _15171_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492    9.8666 v _15171_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12153_ (net)
                    0.1162    0.0000    9.8666 v _15172_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   10.5480 v _15172_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12154_ (net)
                    0.1422    0.0000   10.5480 v _15173_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   11.1972 v _15173_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12155_ (net)
                    0.1162    0.0000   11.1972 v _15174_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   11.8785 v _15174_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12156_ (net)
                    0.1422    0.0000   11.8785 v _15175_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   12.5278 v _15175_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12157_ (net)
                    0.1162    0.0000   12.5278 v _15176_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   13.2091 v _15176_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12158_ (net)
                    0.1422    0.0000   13.2091 v _15177_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   13.8584 v _15177_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12159_ (net)
                    0.1162    0.0000   13.8584 v _15178_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   14.5397 v _15178_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12160_ (net)
                    0.1422    0.0000   14.5397 v _15179_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   15.1889 v _15179_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12161_ (net)
                    0.1162    0.0000   15.1889 v _15180_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   15.8703 v _15180_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12162_ (net)
                    0.1422    0.0000   15.8703 v _15181_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   16.5195 v _15181_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12163_ (net)
                    0.1162    0.0000   16.5195 v _15182_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   17.2009 v _15182_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12164_ (net)
                    0.1422    0.0000   17.2009 v _15183_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   17.8501 v _15183_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12165_ (net)
                    0.1162    0.0000   17.8501 v _15184_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   18.5314 v _15184_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12166_ (net)
                    0.1422    0.0000   18.5314 v _15185_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   19.1807 v _15185_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12167_ (net)
                    0.1162    0.0000   19.1807 v _15186_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   19.8620 v _15186_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12168_ (net)
                    0.1422    0.0000   19.8620 v _15187_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   20.5113 v _15187_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12169_ (net)
                    0.1162    0.0000   20.5113 v _15188_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   21.1926 v _15188_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12170_ (net)
                    0.1422    0.0000   21.1926 v _15189_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   21.8419 v _15189_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12171_ (net)
                    0.1162    0.0000   21.8419 v _15190_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   22.5232 v _15190_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12172_ (net)
                    0.1422    0.0000   22.5232 v _15191_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   23.1724 v _15191_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12173_ (net)
                    0.1162    0.0000   23.1724 v _15192_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   23.8538 v _15192_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12174_ (net)
                    0.1422    0.0000   23.8538 v _15193_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   24.5030 v _15193_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12175_ (net)
                    0.1162    0.0000   24.5030 v _15194_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   25.1843 v _15194_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12176_ (net)
                    0.1422    0.0000   25.1843 v _15195_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   25.8336 v _15195_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12177_ (net)
                    0.1162    0.0000   25.8336 v _15196_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   26.5149 v _15196_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12178_ (net)
                    0.1422    0.0000   26.5149 v _15197_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   27.1642 v _15197_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12179_ (net)
                    0.1162    0.0000   27.1642 v _15198_/B (sky130_fd_sc_hd__or2_2)
                    0.1422    0.6813   27.8455 v _15198_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12180_ (net)
                    0.1422    0.0000   27.8455 v _15199_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   28.4947 v _15199_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12181_ (net)
                    0.1162    0.0000   28.4947 v _15200_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   29.1761 v _15200_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12182_ (net)
                    0.1421    0.0000   29.1761 v _15201_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   29.8253 v _15201_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12183_ (net)
                    0.1162    0.0000   29.8253 v _15202_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   30.5066 v _15202_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12184_ (net)
                    0.1421    0.0000   30.5066 v _15203_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   31.1558 v _15203_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12185_ (net)
                    0.1162    0.0000   31.1558 v _15204_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   31.8372 v _15204_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12186_ (net)
                    0.1421    0.0000   31.8372 v _15205_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   32.4864 v _15205_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12187_ (net)
                    0.1162    0.0000   32.4864 v _15206_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   33.1677 v _15206_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12188_ (net)
                    0.1421    0.0000   33.1677 v _15207_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   33.8169 v _15207_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12189_ (net)
                    0.1162    0.0000   33.8169 v _15208_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   34.4982 v _15208_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12190_ (net)
                    0.1421    0.0000   34.4982 v _15209_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   35.1475 v _15209_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12191_ (net)
                    0.1162    0.0000   35.1475 v _15210_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   35.8288 v _15210_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12192_ (net)
                    0.1421    0.0000   35.8288 v _15211_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   36.4780 v _15211_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12193_ (net)
                    0.1162    0.0000   36.4780 v _15212_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   37.1593 v _15212_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12194_ (net)
                    0.1421    0.0000   37.1593 v _15213_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   37.8085 v _15213_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12195_ (net)
                    0.1162    0.0000   37.8085 v _15214_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   38.4899 v _15214_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12196_ (net)
                    0.1421    0.0000   38.4899 v _15215_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   39.1391 v _15215_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12197_ (net)
                    0.1162    0.0000   39.1391 v _15216_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   39.8204 v _15216_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12198_ (net)
                    0.1421    0.0000   39.8204 v _15217_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   40.4696 v _15217_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12199_ (net)
                    0.1162    0.0000   40.4696 v _15218_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   41.1510 v _15218_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12200_ (net)
                    0.1421    0.0000   41.1510 v _15219_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   41.8002 v _15219_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12201_ (net)
                    0.1162    0.0000   41.8002 v _15220_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   42.4815 v _15220_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12202_ (net)
                    0.1421    0.0000   42.4815 v _15221_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   43.1307 v _15221_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12203_ (net)
                    0.1162    0.0000   43.1307 v _15222_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   43.8120 v _15222_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12204_ (net)
                    0.1421    0.0000   43.8120 v _15223_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   44.4612 v _15223_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12205_ (net)
                    0.1162    0.0000   44.4612 v _15224_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   45.1426 v _15224_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12206_ (net)
                    0.1421    0.0000   45.1426 v _15225_/B (sky130_fd_sc_hd__or2_2)
                    0.1162    0.6492   45.7918 v _15225_/X (sky130_fd_sc_hd__or2_2)
     2    0.0035                                 _12207_ (net)
                    0.1162    0.0000   45.7918 v _15226_/B (sky130_fd_sc_hd__or2_2)
                    0.1421    0.6813   46.4731 v _15226_/X (sky130_fd_sc_hd__or2_2)
     3    0.0079                                 _12208_ (net)
                    0.1421    0.0000   46.4731 v _15227_/B (sky130_fd_sc_hd__or2_2)
                    0.1228    0.6610   47.1341 v _15227_/X (sky130_fd_sc_hd__or2_2)
     2    0.0044                                 _12209_ (net)
                    0.1228    0.0000   47.1341 v _15230_/B2 (sky130_fd_sc_hd__o221a_2)
                    0.0713    0.4381   47.5723 v _15230_/X (sky130_fd_sc_hd__o221a_2)
     1    0.0016                                 _03928_ (net)
                    0.0713    0.0000   47.5723 v _30440_/D (sky130_fd_sc_hd__dfxtp_2)
                                       47.5723   data arrival time

                    0.0000   24.7300   24.7300   clock clk (rise edge)
                              0.0000   24.7300   clock network delay (ideal)
                              0.0000   24.7300   clock reconvergence pessimism
                                       24.7300 ^ _30440_/CLK (sky130_fd_sc_hd__dfxtp_2)
                             -0.2939   24.4361   library setup time
                                       24.4361   data required time
-------------------------------------------------------------------------------------
                                       24.4361   data required time
                                      -47.5723   data arrival time
-------------------------------------------------------------------------------------
 # Reduced slack (from -23.1405 to -23.1306, closer to 0)
 **                              -23.1362   slack (VIOLATED)**
OR gate of strength 2 driving OA gate has more delay.

image

Commands to replace OR gate with another OR gate with higher strength

report_net -connections _11668_
image

replace_cell _14506_ sky130_fd_sc_hd__or4_4
image

report_checks -fields {net cap slew input_pins} -digits 4
image

Slack has beeen reduced even more to -22.6173 (by 1.2827 ns from the origional slack)

Command to verify instance _14506_ has been replaced with sky130_fd_sc_hd__or4_4

report_checks -from _29043_ -to _30440_ -through _14506_
image

Replacing the old netlist with the new netlist generated after timing ECO fix and implement the floorplan, placement and CTS.
Commands to make a copy of the old netlist.

cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/31-01_18-27/results/synthesis/
image

ls
image

cp picorv32a.synthesis.v picorv32a.synthesis_old.v
image

Commands to write verilog

help write_verilog
image

write_verilog /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/31-01_18-27/results/synthesis/picorv32a.synthesis.v
image

exit
image

The rest of this documentation has been taken from my other documentation on this same course done a little earlier.(https://github.com/Sushil15-ai/OpenLANE-Sky130)

Commands load the design and run necessary stages

prep -design picorv32a -tag 21-01_15-14 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
set ::env(SYNTH_STRATEGY) "DELAY 3"
set ::env(SYNTH_SIZING) 1
Running synthesis

run_synthesis
Running floorplan (alternative steps)

init_floorplan
place_io
tap_decap_or
Running placement

run_placement
Runnning command to synthesis the clock tree.

run_cts
image

# run OpenROAD
openroad
# Read lef file
read_lef /openLANE_flow/designs/picorv32a/runs/21-01_16-03/tmp/merged.lef
# Read def file
read_def /openLANE_flow/designs/picorv32a/runs/21-01_16-03/results/cts/picorv32a.cts.def
# Create OpenROAD database
write_db pico_cts.db
image

# Loading the created database
read_db pico_cts.db
image

# Read netlistS
read_verilog /openLANE_flow/designs/picorv32a/runs/21-01_15-14/results/synthesis/picorv32a.synthesis_cts.v
# Read library
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a
# Read the custom sdc
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
image

# Set all cloks as propagated clocks
set_propagated_clock [all_clocks]
help report_checks
# Generating timing report
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4
image

exit
Post CTS Openroad timing analysis Commands for executing OpenROAD timing analysis. Exploration by removing 'sky130_fd_sc_hd__clkbuf_1' cell from clock buffer list variable 'CTS_CLK_BUFFER_LIST'.
Commands to be run in OpenLANE flow to do OpenROAD timing analysis after changing CTS_CLK_BUFFER_LIST

echo $::env(CTS_CLK_BUFFER_LIST)
set ::env(CTS_CLK_BUFFER_LIST) [lreplace $::env(CTS_CLK_BUFFER_LIST) 0 0]
echo $::env(CTS_CLK_BUFFER_LIST)
echo $::env(CURRENT_DEF)
set ::env(CURRENT_DEF) /openLANE_flow/designs/picorv32a/runs/21-01_15-14/results/placement/picorv32a.placement.def
image

run_cts
echo $::env(CTS_CLK_BUFFER_LIST)
openroad
read_lef /openLANE_flow/designs/picorv32a/runs/21-01_15-14/tmp/merged.lef
read_def /openLANE_flow/designs/picorv32a/runs/21-01_15-14/results/cts/picorv32a.cts.def
image

write_db pico_cts1.db
read_db pico_cts.db
read_verilog /openLANE_flow/designs/picorv32a/runs/21-01_15-14/results/synthesis/picorv32a.synthesis_cts.v
image

read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a
Read the .sdc file we created

read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
image

set_propagated_clock [all_clocks]
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4
image

report_clock_skew -hold
report_clock_skew -setup
image

exit
echo $::env(CTS_CLK_BUFFER_LIST)
# Inserting 'sky130_fd_sc_hd__clkbuf_1' to first index of list
set ::env(CTS_CLK_BUFFER_LIST) [linsert $::env(CTS_CLK_BUFFER_LIST) 0 sky130_fd_sc_hd__clkbuf_1]
echo $::env(CTS_CLK_BUFFER_LIST)
