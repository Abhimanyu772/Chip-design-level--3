LAB TASKS

(This part of the documentation has been taken from my documentation for this same course from a little earlier

Perform generation of Power Distribution Network (PDN) and explore the PDN layout.
Commands to perform all necessary stages up until now

cd Desktop/work/tools/openlane_working_dir/openlane
docker
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
image

set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
set ::env(SYNTH_STRATEGY) "DELAY 3"
set ::env(SYNTH_SIZING) 1
image

run_synthesis
image

Alternative for running floorplan

init_floorplan
place_io
tap_decap_or
image

Running placement

run_placement
image

unset ::env(LIB_CTS)
Running Clock Tree Synthesis (CTS)

run_cts
image

Generate the PDN

gen_pdn 
image

Commands to load PDN def in magic in another terminal

# Change directory to path containing pdn def file
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/21-01_15-14/tmp/floorplan/
Open in magic

magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read 14-pdn.def &
image

image

Perfrom detailed routing using TritonRoute and explore the routed layout.
Commands to run routing

# Check value of 'CURRENT_DEF'
echo $::env(CURRENT_DEF)

# Check value of 'ROUTING_STRATEGY'
echo $::env(ROUTING_STRATEGY)

# Command for detailed route using TritonRoute
run_routing
image

image

Commands to load routed def in magic in another terminal

Change directory to the one that contains the routing .def

# Change directory to path containing routed def
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/21-01_15-14/results/routing/
Open in magic.

magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.def &
image

image

Post-Route parasitic extraction using SPEF extractor.
Commands for SPEF extraction using external tool (not nescessary in new openlane) Github link: https://github.com/HanyMoussa/SPEF_EXTRACTOR

# Change directory to the one where you have cloned the github repo
cd Desktop/work/tools/SPEF_EXTRACTOR
# Extract the spef
python3 main.py /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/21-01_15-14/tmp/merged.lef /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/21-01_15-14/results/routing/picorv32a.def
Post-Route OpenSTA timing analysis with the extracted parasitics of the route.
Commands to be run in OpenLANE flow to do OpenROAD timing analysis with integrated OpenSTA in OpenROAD

# Command to run OpenROAD tool
openroad
Read the .lef file we have created.

read_lef /openLANE_flow/designs/picorv32a/runs/21-01_15-14/tmp/merged.lef
image

read the .def file we have created.

read_def /openLANE_flow/designs/picorv32a/runs/21-01_15-14/results/routing/picorv32a.def
image

write_db pico_route.db
read_db pico_route.db
Read the verilog file created in the synthesis step.

read_verilog /openLANE_flow/designs/picorv32a/runs/21-01_15-14/results/synthesis/picorv32a.synthesis_preroute.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a
image

read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
image

set_propagated_clock [all_clocks]
Read the spef file we just extracted(not nescessary in new openlane)

read_spef /openLANE_flow/designs/picorv32a/runs/21-01_15-14/results/routing/picorv32a.spef
image

report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4
image

Exit the openlane flow.

exit
