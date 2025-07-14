# TCL Workshop 

## Overview  
This repository contains course materials, lab scripts, and hands-on exercises from the **VSD TCL & Linux Workshop**. The goal is to build strong proficiency in **TCL scripting** and **Linux-based automation** for semiconductor digital design flows.

The content focuses on automating design constraints, integrating with open-source tools like **Yosys** and **OpenTimer**, and performing **Quality of Results (QOR)** analysis using TCL procedures.

---

## Key Learning Outcomes
- Master the fundamentals and advanced concepts of **TCL scripting**
- Automate generation of constraint files (CSV/SDC to OpenTimer)
- Integrate TCL with **Yosys** for RTL synthesis and memory module generation
- Perform hierarchical design checks and error detection
- Analyze QOR metrics including:
  - **WNS** (Worst Negative Slack)  
  - **FEP** (False Endpoint Path)

---

## Curriculum

### Module 1: TCL Basics & VSDSYNTH Toolbox
- Introduction to TCL scripting and task automation
<img width="1040" height="516" alt="image" src="https://github.com/user-attachments/assets/2977a467-2dfc-4298-adb2-91838f4a2357" />
- VSDSYNTH toolbox usage and user input handling
Task is to create vsdsynth and vsdsynth.tcl files. The basic structure of bash code used for the implementation of general conditions such as not giving the csv file or giving an non exsitient csv fileor giving the correct file
<img width="923" height="697" alt="image" src="https://github.com/user-attachments/assets/e9a2c229-2592-4fe9-87fe-d5170f752dec" />

```bash
if ($#argv != 1) then
	echo "Info: Please provide the csv file"
	exit 1
endif

if (! -f $argv[1] || $argv[1] == "-help") then
	if ($argv[1] != "-help") then
		echo "Error: Cannot find csv file $argv[1]. Exiting..."
		exit 1
	else
		echo USAGE: ./vsdsynth \<csv file\>
		echo
		echo        where \csv file\> consists of 2 columns, below keyword being in 1st column and is Case Sensitive. PLease request PS for sample csv file
		echo
		echo        \<Design Name\> is the name of the top level module
                echo
                echo        \<Output Directory\> is the name of the output directory where you want to dump synthesis script, synthesized netlist and timing reports
                echo
                echo        \<Netlist Directory\> is the name of  directory where all the RTL netlist are present
                echo
                echo        \<Early Library Path\> is the file path of the early cell library to be used for STA
                echo
                echo        \<Late Library Path\> is the file path of the late cell library to be used for STA
                echo
                echo        \<Constraints file\> is csv file path of contraints to be used for STA
		echo
		exit 1
	endif
else tclsh ./vsdsynth.tcl $argv[1]
endif
```
Working of the above script in the user interface 
<img width="1036" height="507" alt="image" src="https://github.com/user-attachments/assets/cdb88f67-6859-4164-aa93-acf490ef0058" />
In the above picture, we can see scenarios such as providing no input file to the command, providing an incorrect number of inputs, using a wrong .csv file, and seeking help to understand the command's capabilities and requirements.
### Module 2: Variable Creation & Constraint Processing
Module 2 involves writing the TCL script in vsdsynth.tcl to handle variable creation, check for file and directory existence, and process the constraints CSV file. The script should convert the constraints into both Format 1 (required by the Yosys tool) and the standard SDC (Synopsys Design Constraints) format used in the industry.
<img width="950" height="520" alt="image" src="https://github.com/user-attachments/assets/b03effdb-2b03-4922-8faf-ddf3651e4a2f" />
- Working with arrays, matrices, and loop constructs
- Parsing and validating CSV/SDC constraint files
Inside the openMSP430_design_constraints.csv file
<img width="1037" height="503" alt="image" src="https://github.com/user-attachments/assets/62995540-72d8-4630-b2a9-7da8b7164e77" />
vsdsynth.tcl (using the matric package and creating a matrix from the details.csv file.

```bash
set filename [lindex $argv 0]
package require csv
package require struct::matrix
struct::matrix m
set f [open $filename]
csv::read2matrix $f m , auto
close $f
set columns [m columns]
#m add columns $columns
m link my_arr
set num_of_rows [m rows]
```
Variables were automatically created by converting the .csv file into a matrix, determining the number of rows and columns, and dynamically generating variables for each entry using the matrix structure (my_arr).

```bash
set i 0
	while {$i < $num_of_rows} {
		puts "\nInfo: Setting $my_arr(0,$i) as '$my_arr(1,$i)'"
		if {$i == 0} {
			set [string map {" " ""} $my_arr(0,$i)] $my_arr(1,$i)
		} else {
			set [string map {" " ""} $my_arr(0,$i)] [file normalize $my_arr(1,$i)]
		}
		set i [expr {$i+1}]
	}
}

puts "\nInfo: Below are the list of initial variables and their values. User can use these variables for further debug. Use 'puts <variable name>' command to query value of below variables"
puts "DesignName = $DesignName"
puts "OutputDirectory = $OutputDirectory"
puts "NetlistDirectory = $NetlistDirectory"
puts "EarlyLibraryPath = $EarlyLibraryPath"
puts "LateLibraryPath = $LateLibraryPath"
puts "ConstraintsFile = $ConstraintsFile"
```

The names were converted into variable identifiers by removing spaces, and the corresponding paths were assigned to these variables.
<img width="1037" height="352" alt="image" src="https://github.com/user-attachments/assets/237a231d-fba2-4ff0-806f-13c5a8835fe6" />
Directory Existence Checking:
The script includes logic to verify the existence of all required files and directories. If any critical file or directory is missing, the program terminates with an appropriate error message to prevent further execution. The only exception is the output directory, which is automatically created if it does not already exist. Below are the corresponding code snippets and terminal screenshots demonstrating this functionality—one showing the creation of a new output directory, and another showing a case where the output directory exists but the constraints file is missing.

```bash
if {![file isdirectory $OutputDirectory]} {
	puts "\nInfo: Cannot find output directory $OutputDirectory. Creating $OutputDirectory"
	file mkdir $OutputDirectory
} else {
	puts "\nInfo: Output directory found in path $OutputDirectory"
}
if {![file isdirectory $NetlistDirectory]} {
        puts "\nInfo: Cannot find RTL netlist directory $NetlistDirectory. Exiting..."
        exit
} else {
        puts "\nInfo: RTL Netlist directory found in path $NetlistDirectory"
}
if {![file exists $EarlyLibraryPath]} {
        puts "\nInfo: Cannot find early cell library in path $EarlyLibraryPath. Exiting..."
        exit
} else {
        puts "\nInfo: Early cell library found in path $EarlyLibraryPath"
}
if {![file exists $LateLibraryPath]} {
        puts "\nInfo: Cannot find late cell library in path $LateLibraryPath. Exiting..."
        exit
} else {
        puts "\nInfo: Late cell library found in path $LateLibraryPath"
}
if {![file exists $ConstraintsFile]} {
        puts "\nInfo: Cannot find constraints file in path $ConstraintsFile. Exiting..."
        exit
} else {
        puts "\nInfo: Contraints file found in path $ConstraintsFile"
}
```

<img width="1030" height="341" alt="image" src="https://github.com/user-attachments/assets/a47bc594-0a7f-47e7-b271-37a797a74ffb" />
<img width="1022" height="375" alt="image" src="https://github.com/user-attachments/assets/ae7d14cc-80d5-478e-846b-9e0d091a690f" />
Constraint File Processing (openMSP430_design_constraints.csv):
The openMSP430_design_constraints.csv file was successfully read and converted into a matrix structure. The script extracted the total number of rows and columns, along with identifying the starting indices for clock, input, and output constraints. Below is the core TCL code used for processing, along with a terminal screenshot displaying variable values printed using puts for verification.

```bash
puts "\nInfo: Dumping SDC contraints file for $DesignName"
::struct::matrix constraints
set  chan [open $ConstraintsFile]
csv::read2matrix $chan constraints  , auto
close $chan
set number_of_rows [constraints rows]
puts "number_of_rows are $number_of_rows"
set number_of_columns [constraints columns]
puts "number_of_columns are $number_of_columns"
set clock_start [lindex [lindex [constraints search all CLOCKS] 0 ] 1]
puts "clock_start = $clock_start"
set clock_start_column [lindex [lindex [constraints search all CLOCKS] 0 ] 0]
puts "clock_start_column = $clock_start_column"
#----check row number for "inputs" section in constraints.csv------------#
set input_ports_start [lindex [lindex [constraints search all INPUTS] 0 ] 1]
puts "input_ports_start = $input_ports_start"
#----check row number for "inputs" section in constraints.csv------------#
set output_ports_start [lindex [lindex [constraints search all OUTPUTS] 0 ] 1]
puts "output_ports_start = $output_ports_start"
```
<img width="1035" height="470" alt="image" src="https://github.com/user-attachments/assets/fac15699-cc0d-4955-8cb1-183512a72bee" />

### Module 3: Clock & Input Constraint Scripting
The objective of Module 3 was to process design constraints from a .csv file, specifically targeting clock and input signal definitions, and to generate corresponding SDC (Synopsys Design Constraints) commands. This task involved the application of matrix-based search algorithms and logic to differentiate between bus signals and individual bits. I have successfully completed the implementation for Module 3. The process included parsing the constraint .csv file, extracting clock and input-related data, identifying input types (bus or bit), and generating the appropriate SDC commands. These commands were then written into a .sdc file for use in subsequent stages of the design flow. For the clock-related constraints, the relevant entries were accurately extracted from the .csv file, and suitable SDC commands were generated and dumped into the output .sdc file. The implementation includes several debug outputs using puts statements to monitor variable states and confirm processing logic. The core script, terminal snapshots showing the debug prints, and the resulting .sdc content are provided below.
- Writing clock constraints (period, duty cycle)
- Classifying input ports using regular expressions
```bash
#----------------------clock constraints-------------------------------------#
#----------------------clock latency constraints-----------------------------#

set clock_early_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] early_rise_delay] 0] 0]
set clock_early_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] early_fall_delay] 0] 0]
set clock_late_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] late_rise_delay] 0] 0]
set clock_late_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] late_fall_delay] 0] 0]
#----------------clock transition contraints---------------------------------#
set clock_early_rise_slew_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] early_rise_slew] 0] 0]
set clock_early_fall_slew_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] early_fall_slew] 0] 0]
set clock_late_rise_slew_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] late_rise_slew] 0] 0]
set clock_late_fall_slew_start [lindex [lindex [constraints search rect $clock_start_column $clock_start [expr {$number_of_columns-1}] [expr {$input_ports_start-1}] late_fall_slew] 0] 0]
set sdc_file [open $OutputDirectory/$DesignName.sdc "w"]
set i [expr {$clock_start+1}]
set end_of_ports [expr {$input_ports_start-1}]
puts "\nInfo-SDC: Working on clock constraints....."
while {$i < $end_of_ports} {
	puts "Working on clock [constraints get cell 0 $i]"
	puts -nonewline $sdc_file "\ncreate_clock -name [constraints get cell 0 $i] -period [constraints get cell 1 $i] -waveform \{0 [expr {[constraints get cell 1 $i]*[constraints get cell 2 $i]/100}]\} \[get_ports [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -rise -min  [constraints get cell $clock_early_rise_slew_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -fall -min  [constraints get cell $clock_early_fall_slew_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -rise -max  [constraints get cell $clock_late_rise_slew_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_transition -fall -max  [constraints get cell $clock_late_fall_slew_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -early -rise  [constraints get cell $clock_early_rise_delay_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -early -fall  [constraints get cell $clock_early_fall_delay_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -late -rise  [constraints get cell $clock_late_rise_delay_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	puts -nonewline $sdc_file "\nset_clock_latency -source -late -fall  [constraints get cell $clock_late_fall_delay_start $i] \[get_clocks [constraints get cell 0 $i]\]"
	
	set i [expr {$i+1}]
}
```
<img width="1033" height="493" alt="image" src="https://github.com/user-attachments/assets/9fe95ab8-cdd4-4f39-8384-980bbe6937f1" />
<img width="1037" height="252" alt="image" src="https://github.com/user-attachments/assets/a1439a1f-efb0-4d5f-9c08-25a82c9bfa37" />
Processing of the .csv file for input constraints and generation of corresponding SDC commands has been successfully completed. The implementation involved identifying and differentiating between bit-level and bus-type input signals. Based on the processed data, appropriate input-related SDC commands were generated and written into the .sdc file. The core script, along with terminal screenshots displaying variable values through puts statements for debugging, and the resulting .sdc output, are shown below.

```bash
#--------------------input constraints-------------------------------#
set input_early_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] early_rise_delay] 0] 0]
set input_early_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] early_fall_delay] 0] 0]
set input_late_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] late_rise_delay] 0] 0]
set input_late_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] late_fall_delay] 0] 0]

set input_early_rise_slew_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] early_rise_slew] 0] 0]
set input_early_fall_slew_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] early_fall_slew] 0] 0]
set input_late_rise_slew_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] late_rise_slew] 0] 0]
set input_late_fall_slew_start [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] late_fall_slew] 0] 0]

set related_clock [lindex [lindex [constraints search rect $clock_start_column $input_ports_start [expr {$number_of_columns-1}] [expr {$output_ports_start-1}] clocks] 0 ] 0]
set i [expr {$input_ports_start+1}]
set end_of_ports [expr {$output_ports_start-1}]
puts "\nInfo-SDC: WOrking on IO constraints......."
puts "\nInfo-SDC: Categorizing input ports as bits and bussed"

while {$i < $end_of_ports} {
#------------------------------------optional----------differentiating input ports as bussed and bits-------------------#
set netlist [glob -dir $NetlistDirectory *.v]
set tmp_file [open /tmp/1 w]
foreach f $netlist {
	set fd [open $f]
	puts "reading file $f"
	while {[gets $fd line] != -1} {
		set pattern1 " [constraints get cell 0 $i];"
		if {[regexp -all -- $pattern1 $line]} {
			puts "pattern1 \"$pattern1\" found and matching in verilog file \"$f\" is \"$line\""
			set pattern2 [lindex [split $line ";"] 0]
			puts "creatng pattern2 by splitting pattern1 using semi-colon as delimiter => \"$pattern2\""
			if {[regexp -all {input} [lindex [split $pattern2 "\S+"] 0]]} {
				puts "out of all patterns, \"$pattern2\"has matching string \"input\". Sp preserving this line and ignoring others"
				set s1 "[lindex [split $pattern2 "\S+"] 0] [lindex [split $pattern2 "\S+"] 1] [lindex [split $pattern2 "\S+"] 2]"
				puts "printing first 3 elements of pattern2 as \"$s1\" using space as delimiter"
				puts -nonewline $tmp_file "\n[regsub -all {\s+} $s1 " "]"
				puts "replace mutiple spaces in s1 by single space and reformant as \"[regsub -all {\s+} $s1 " "]\""
			        }
			}
		
		}
	

close $fd
}
close $tmp_file

set tmp_file [open /tmp/1 r]
#puts "reading [read $tmp_file]"
#puts "reading /tmp/1 file as [split [read $tmp_file] \n]"
#puts "sorting /tmp/1 contents as [lsort -unique [split [read $tmp_file] \n ]]"
#puts "joining /tmp/1 as [join [lsort -unique [split [read $tmp_file] \n ]] \n]"
set tmp2_file [open /tmp/2 w]
puts -nonewline $tmp2_file "[join [lsort -unique [split [read $tmp_file] \n]] \n]"
close $tmp_file
close $tmp2_file

set tmp2_file [open /tmp/2 r]
#puts "Count is [llength [read $tmp2_file]]"
set count [llength [read $tmp2_file]]
#puts "Splitting content of tmp_2 using space and counting number of elements as $count"

if {$count > 2} {
	set inp_ports [concat [constraints get cell 0 $i]*]
	puts "bussed"
} else {
	set inp_ports [constraints get cell 0 $i]
	puts "not bussed"
}
	puts "input port name is $inp_ports since count is $count\n"
	puts -nonewline $sdc_file "\nset_input_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -rise -source_latency_included [constraints get cell $input_early_rise_delay_start $i] \[get_ports $inp_ports\]"
	puts -nonewline $sdc_file "\nset_input_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -fall -source_latency_included [constraints get cell $input_early_fall_delay_start $i] \[get_ports $inp_ports\]"
	puts -nonewline $sdc_file "\nset_input_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -rise -source_latency_included [constraints get cell $input_late_rise_delay_start $i] \[get_ports $inp_ports\]"
	puts -nonewline $sdc_file "\nset_input_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -fall -source_latency_included [constraints get cell $input_late_fall_delay_start $i] \[get_ports $inp_ports\]"

	puts -nonewline $sdc_file "\nset_input_transition -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -rise -source_latency_included [constraints get cell $input_early_rise_slew_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_transition -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -fall -source_latency_included [constraints get cell $input_early_fall_slew_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_transition -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -rise -source_latency_included [constraints get cell $input_late_rise_slew_start $i] \[get_ports $inp_ports\]"
        puts -nonewline $sdc_file "\nset_input_transition -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -fall -source_latency_included [constraints get cell $input_late_fall_slew_start $i] \[get_ports $inp_ports\]"

	set i [expr {$i+1}]
}
close $tmp2_file
```
<img width="1037" height="706" alt="image" src="https://github.com/user-attachments/assets/178a4605-a5f3-4474-b430-45e34685e05e" />
<img width="1041" height="413" alt="image" src="https://github.com/user-attachments/assets/b4cdf5dc-2e9f-42ee-b7ad-5972c95e25da" />
<img width="1033" height="502" alt="image" src="https://github.com/user-attachments/assets/6fce74be-2897-4d62-8758-8a19c0a9e8e3" />

### Module 4: RTL Synthesis & Yosys Integration
Module 4 focused on multiple tasks: processing the output section of the constraints .csv file and generating the corresponding SDC commands, performing a sample synthesis using Yosys with an example memory block, checking the design hierarchy in Yosys, and implementing error handling for hierarchy-related issues. I successfully completed all the objectives of Module 4. This included parsing the .csv file to extract output-related constraints, distinguishing between bit-level and bus-type outputs, and generating the appropriate SDC commands which were written to the .sdc file. Additionally, I explored Yosys-based synthesis for a sample memory design, understanding its read and write mechanisms. A Yosys script was developed to perform a hierarchy check, and error-handling code was implemented to identify and manage hierarchy-related issues during synthesis. The implementation includes the core script for output constraint processing, terminal snapshots with puts debug statements displaying variable values, and the resulting .sdc file. These are provided below as supporting evidence of successful completion
- Developing complete synthesis scripts
- Memory module synthesis and TCL error handling with Yosys
```bash
set output_early_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] early_rise_delay] 0] 0]
set output_early_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] early_fall_delay] 0] 0]
set output_late_rise_delay_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] late_rise_delay] 0] 0]
set output_late_fall_delay_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] late_fall_delay] 0] 0]
set output_load_start [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] load] 0 ] 0]

set related_clock [lindex [lindex [constraints search rect $clock_start_column $output_ports_start [expr {$number_of_columns-1}] [expr {$number_of_rows-1}] clocks] 0 ] 0]
set i [expr {$output_ports_start+1}]
set end_of_ports [expr {$number_of_rows-1}]
puts "\nInfo-SDC: Working on IO constraints......."
puts "\nInfo-SDC: Categorizing output ports as bits and bussed"

while {$i < $end_of_ports} {
set netlist [glob -dir $NetlistDirectory *.v]
set tmp_file [open /tmp/1 w]
foreach f $netlist {
        set fd [open $f]
        #puts "reading file $f"
        while {[gets $fd line] != -1} {
                set pattern1 " [constraints get cell 0 $i];"
                if {[regexp -all -- $pattern1 $line]} {
                        #puts "pattern1 \"$pattern1\" found and matching in verilog file \"$f\" is \"$line\""
                        set pattern2 [lindex [split $line ";"] 0]
                        #puts "creatng pattern2 by splitting pattern1 using semi-colon as delimiter => \"$pattern2\""
                        if {[regexp -all {input} [lindex [split $pattern2 "\S+"] 0]]} {
                                #puts "out of all patterns, \"$pattern2\"has matching string \"input\". Sp preserving this line and ignoring others"
                                set s1 "[lindex [split $pattern2 "\S+"] 0] [lindex [split $pattern2 "\S+"] 1] [lindex [split $pattern2 "\S+"] 2]"
                                #puts "printing first 3 elements of pattern2 as \"$s1\" using space as delimiter"
                                puts -nonewline $tmp_file "\n[regsub -all {\s+} $s1 " "]"
                                #puts "replace mutiple spaces in s1 by single space and reformant as \"[regsub -all {\s+} $s1 " "]\""
                                }
                        }

                }
close $fd
}
close $tmp_file

set tmp_file [open /tmp/1 r]
#puts "reading [read $tmp_file]"
#puts "reading /tmp/1 file as [split [read $tmp_file] \n]"
#puts "sorting /tmp/1 contents as [lsort -unique [split [read $tmp_file] \n ]]"
#puts "joining /tmp/1 as [join [lsort -unique [split [read $tmp_file] \n ]] \n]"
set tmp2_file [open /tmp/2 w]
puts -nonewline $tmp2_file "[join [lsort -unique [split [read $tmp_file] \n]] \n]"
close $tmp_file
close $tmp2_file

set tmp2_file [open /tmp/2 r]
#puts "Count is [llength [read $tmp2_file]]"
set count [llength [read $tmp2_file]]
#puts "Splitting content of tmp_2 using space and counting number of elements as $count"

if {$count > 2} {
        set op_ports [concat [constraints get cell 0 $i]*]
        #puts "bussed"
} else {
        set op_ports [constraints get cell 0 $i]
        #puts "not bussed"
}
        #puts "output port name is $op_ports since count is $count\n"
        puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -rise -source_latency_included [constraints get cell $output_early_rise_delay_start $i] \[get_ports $op_ports\]"
        puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -min -fall -source_latency_included [constraints get cell $output_early_fall_delay_start $i] \[get_ports $op_ports\]"
        puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -rise -source_latency_included [constraints get cell $output_late_rise_delay_start $i] \[get_ports $op_ports\]"
        puts -nonewline $sdc_file "\nset_output_delay -clock \[get_clocks [constraints get  cell $related_clock $i]\] -max -fall -source_latency_included [constraints get cell $output_late_fall_delay_start $i] \[get_ports $op_ports\]"
        puts -nonewline $sdc_file "\nset_load [constraints get cell $output_load_start $i] \[get_ports $op_ports\]"

        set i [expr {$i+1}]

}

close $tmp2_file
#return
close $sdc_file

puts "\nInfo: SDC created. PLease use constraints in path $OutputDirectory/$DesignName.sdc"
```
<img width="1036" height="66" alt="image" src="https://github.com/user-attachments/assets/90642a20-2447-424f-a11e-6f38e2060396" />
<img width="1033" height="506" alt="image" src="https://github.com/user-attachments/assets/587e56b7-d827-42af-ba1e-a30bce900513" />
The /tmp/1 and /tmp/2 files will have similar formats for both input and output ports.

For the memory module, a sample Yosys synthesis was performed using the Verilog file memory.v, which defines a memory unit with a single-bit address and single-bit data. The code and its synthesis flow are explained below.
```bash
module memory (CLK, ADDR, DIN, DOUT);

parameter wordSize = 1;
parameter addressSize = 1;

input ADDR, CLK;
input [wordSize-1:0] DIN;
output reg [wordSize-1:0] DOUT;
reg [wordSize:0] mem [0:(1<<addressSize)-1];

always @(posedge CLK) begin
	mem[ADDR] <= DIN;
	DOUT <= mem[ADDR];
	end

endmodule
```
The basic Yosys script memory.ys, used to synthesize the design and generate both the gate-level netlist and a 2D gate-level representation of the memory module, is provided below.
```bash
read_liberty -lib -ignore_miss_dir -setattr blackbox /home/kunalg/Desktop/work/openmsp430/openmsp430/osu018_stdcells.lib
read_verilog memory.v
synth top memory
splitnets -ports -format ___
dfflibmap -liberty /home/kunalg/Desktop/work/openmsp430/openmsp430/osu018_stdcells.lib
opt
abc -liberty /home/kunalg/Desktop/work/openmsp430/openmsp430/osu018_stdcells.lib
flatten
clean -purge
opt
clean
write_verilog memory_synth.v
```
The output view of netlist from the code is shown below.
<img width="950" height="537" alt="image" src="https://github.com/user-attachments/assets/bf099d02-9269-4d1e-a7a4-a1698f459c63" />
The memory write process is illustrated in the following images using a truth table, providing a basic visual explanation of how data is written to memory.
<img width="947" height="691" alt="image" src="https://github.com/user-attachments/assets/c7fd87bf-b763-44f8-a67e-ac7fdd9db363" />
Before first rising edge of the clock
<img width="948" height="536" alt="image" src="https://github.com/user-attachments/assets/79471398-7b35-4877-9569-889278f94bf6" />
After first rising edge of the clock - write process done
<img width="945" height="526" alt="image" src="https://github.com/user-attachments/assets/fcb81c0f-66c4-4c77-95c2-24cc14412930" />
The memory read process is demonstrated in the following images through a truth table, offering a basic illustration of how data is retrieved from memory.
<img width="952" height="682" alt="image" src="https://github.com/user-attachments/assets/938da74d-9335-498c-b3b0-be183ef19699" />
After first rising edge and before second rising edge of the clock
<img width="940" height="533" alt="image" src="https://github.com/user-attachments/assets/6bc33280-92d8-4b4f-ab04-bb866c28c4e8" />
After second rising edge of the clock - read process done
<img width="952" height="523" alt="image" src="https://github.com/user-attachments/assets/6dbd5b74-037b-4f64-a4ec-13717c0fe8f7" />
The hierarchy check script was successfully implemented. The developed code generates the required script for performing hierarchy verification. Below are the core script, terminal screenshots displaying variable values through puts statements for debugging, and the resulting output.hier.ys file.
```bash
#Hierarchy Check
puts "\nInfo: Creating hierarchy check script to be used by Yosys"
set data "read_liberty -lib -ignore_miss_dir -setattr blackbox ${LateLibraryPath}"
puts "data is \"$data\""
set filename "$DesignName.hier.ys"
puts "Filename is \"$filename\""
set fileId [open $OutputDirectory/$filename "w"]
puts -nonewline $fileId $dat
set netlist [glob -dir $NetlistDirectory *.v]
puts "Netlist is \"$netlist\""
foreach f $netlist {
	set data $f
	puts "Data is \"$f\""	
	puts -nonewline $fileId "\nread_verilog $f"
}
puts -nonewline $fileId "\nhierarchy -check"
close $fileId
```
<img width="1035" height="493" alt="image" src="https://github.com/user-attachments/assets/cd1382b1-d0ac-4b45-b31a-53bbb0d9e375" />
<img width="1041" height="321" alt="image" src="https://github.com/user-attachments/assets/28cd9c21-cd99-4e44-a107-066cd25edd17" />

Hierarchy Check and Error Handling
I have successfully implemented error handling for the hierarchy check process in Yosys. The script is designed to detect and respond to any errors that occur during the hierarchy check, terminating execution if a failure is encountered. The core code, along with terminal screenshots showing puts statements for variable values and debugging, is provided below.

```bash
set my_err [catch {exec yosys -s $OutputDirectory/$DesignName.hier.ys >& $OutputDirectory/$DesignName.hierarchy_check.log} msg]
puts "error flag is $my_err"
if { $my_err } {
set filename "$OutputDirectory/$DesignName.hierarchy_check.log"
	puts "Log file name is $filename"
	set pattern {referenced in module}
	puts "Pattern is $pattern"
	set count 0
	set fid [open $filename r]
	while {[gets $fid line] != -1} {
		incr count [regexp -all -- $pattern $line]
		if {[regexp -all -- $pattern $line]} {
			puts "\nError: Module [lindex $line 2] is not part of design $DesignName. Please correct RTL in the path '$NetlistDirectory'"
			puts "\nInfo: Hierarchy check has FAILED"
		}
	}
	close $fid
} else {
	puts "\nInfo: Hierarchy check is PASS"
}
puts "\nInfo: PLease find hierarchy check details in [file normalize $OutputDirectory/$DesignName.hierarchy_check.log] for more info"
cd $working_dir
```
<img width="1036" height="111" alt="image" src="https://github.com/user-attachments/assets/248bc85a-9888-4614-99f4-8719b22a4849" />
<img width="1037" height="506" alt="image" src="https://github.com/user-attachments/assets/f2002fa5-5663-46ef-87f3-f9fec95c6b02" />
<img width="1036" height="505" alt="image" src="https://github.com/user-attachments/assets/24639870-f85b-486c-a990-e8cb0b4dd7ce" />

### Module 5: QOR Report Generation
Module 5 focused on executing the main synthesis flow using Yosys, gaining a working understanding of procedural blocks (procs) and applying them at the application level. The tasks also included generating essential files for OpenTimer such as .conf, .spef, and .timing, writing a complete OpenTimer script, performing a static timing analysis (STA) run, and extracting relevant Quality of Results (QoR) data from the .results file. The final step involved formatting and displaying the collected QoR data in a standard tool-compatible format. I have successfully implemented all components required for Module 5. This includes writing and dumping the main Yosys synthesis script (.ys file), creating the necessary OpenTimer input files, executing the STA run, and parsing the results to present QoR data in the expected format. The implementation details, along with the core scripts and terminal outputs using puts statements for variable monitoring and debugging, are shown below.
- Runtime and delay extraction using TCL procedures
- Conversion of constraints to OpenTimer format
- Bit-blasting bussed signals
- 
```bash
# Main Synthesis Script
puts "\nInfo: Creating main synthesis script to be used by Yosys"
set data "read_liberty -lib -ignore_miss_dir -setattr blackbox ${LateLibraryPath}"
set filename "$DesignName.ys"
set fileId [open $OutputDirectory/$filename "w"]
puts -nonewline $fileId $data

set netlist [glob -dir $NetlistDirectory *.v]
foreach f $netlist {
	set data $f
	puts -nonewline $fileId "\nread_verilog $f"
}
puts -nonewline $fileId "\nhierarchy -top $DesignName"
puts -nonewline $fileId "\nsynth -top $DesignName"
puts -nonewline $fileId "\nsplitnets -ports -format ___\ndfflibmap -liberty ${LateLibraryPath}\nopt"
puts -nonewline $fileId "\nabc -liberty ${LateLibraryPath}"
puts -nonewline $fileId "\nflatten"
puts -nonewline $fileId "\nclean -purge\niopadmap -outpad BUFX2 A:Y -bits\nopt\nclean"
puts -nonewline $fileId "\nwrite_verilog $OutputDirectory/$DesignName.synth.v"
close $fileId
puts "\nInfo: Synthesis script created and can be accessed from path $OutputDirectory/$DesignName.ys"

puts "\nInfo: Running synthesis................"
```
<img width="1035" height="481" alt="image" src="https://github.com/user-attachments/assets/e920757c-b72d-443d-a2c5-064ca221d5f2" />
<img width="1036" height="426" alt="image" src="https://github.com/user-attachments/assets/406c158a-a13a-49c9-aeda-c2a0d0168ed3" />

Running Main Synthesis Script and Error Handling
The code for executing the main Yosys synthesis script has been successfully implemented, including error handling to terminate the process if any issues are encountered during execution. The core script and terminal screenshots are provided below.
```bash
if {[catch {exec yosys -s $OutputDirectory/$DesignName.ys >& $OutputDirectory/$DesignName.synthesis.log} msg]} {
	puts "\nError: Synthesis failed due to errors. Please refer to log $OutputDirectory/$DesignName.synthesis.log for errors"
	exit
} else {
	puts "\nInfo: Synthesis finished successfully"
}
puts "Please refer to log $OutputDirectory/$DesignName.synthesis.log"
```

<img width="1037" height="190" alt="image" src="https://github.com/user-attachments/assets/081af310-03da-4055-9fb7-dd9bd84b3c11" />
<img width="1035" height="491" alt="image" src="https://github.com/user-attachments/assets/6ecd4445-6050-4027-b8ee-bb8949817e44" />
Editing .synth.v for OpenTimer Compatibility
I have successfully developed a script to modify the main synthesis output netlist (.synth.v) to ensure compatibility with OpenTimer and other STA/PnR tools. This involved replacing instances of "*" with the word form and removing quotation marks (") from all relevant lines. The core script, along with terminal screenshots displaying variable states and debug information via puts statements, is shown below.

```bash
#Editing synth.v to be usable by Opentimer
set fileId [open /tmp/1 "w"]
puts -nonewline $fileId [exec grep -v -w "*" $OutputDirectory/$DesignName.synth.v]
close $fileId

set output [open $OutputDirectory/$DesignName.final.synth.v "w"]

set filename "/tmp/1"
set fid [open $filename r]
	while {[gets $fid line] != -1} {
		puts -nonewline $output [string map {"\\" ""} $line]
		puts -nonewline $output "\n"
	}
	close $fid
	close $output

	puts "\nInfo: Please find the synthesized netlist for $DesignName at below path. You can use this netlist for STA or PNR"
puts "\n$OutputDirectory/$DesignName.final.synth.v"
```
<img width="1038" height="503" alt="image" src="https://github.com/user-attachments/assets/488442c1-eaf4-4b09-bd0f-c11ae7fd7b09" />
<img width="1045" height="513" alt="image" src="https://github.com/user-attachments/assets/111abf56-e7ca-4ee2-b514-7c359c1ff71b" />
Exploring the Use of Procs
Procs (procedures) can be used to define custom commands for enhancing script flexibility and reusability. I have successfully implemented multiple user-defined procs, including set_multi_cpu_usage and read_sdc, among others. The corresponding code for each proc, along with terminal screenshots displaying variable outputs and debug messages via puts statements, is provided below.
reopenStdout.proc
This proc is designed to redirect the standard output (stdout) to a file specified as an argument. The implementation is written in Tcl as shown in the snippet below.
Code:

```bash
#!/bin/tclsh
proc reopenStdout {file} {
    close stdout
    open $file w       
}
```

set_multi_cpu_usage.proc
This proc configures and outputs the command to enable multi-threaded CPU usage, as required by the OpenTimer tool.

```bash
#!/bin/tclsh

proc set_multi_cpu_usage {args} {
        array set options {-localCpu <num_of_threads> -help "" }
        while {[llength $args]} {
                switch -glob -- [lindex $args 0] {
                	-localCpu {
				set args [lassign $args - options(-localCpu)]
				puts "set_num_threads $options(-localCpu)"
			}
                	-help {
				set args [lassign $args - options(-help) ]
				puts "Usage: set_multi_cpu_usage -localCpu <num_of_threads> -help"
				puts "\t-localCpu - To limit CPU threads used"
				puts "\t-help - To print usage"
                      	}
                }
        }
}
```

<img width="497" height="466" alt="image" src="https://github.com/user-attachments/assets/3965ea1c-ccc2-4606-9747-28718ab37abc" />
read_lib.proc
This proc generates the commands needed to load both early and late timing libraries required by the OpenTimer tool.

```bash
#!/bin/tclsh

proc read_lib args {
	# Setting command parameter options and its values
	array set options {-late <late_lib_path> -early <early_lib_path> -help ""}
	while {[llength $args]} {
		switch -glob -- [lindex $args 0] {
		-late {
			set args [lassign $args - options(-late) ]
			puts "set_late_celllib_fpath $options(-late)"
		      }
		-early {
			set args [lassign $args - options(-early) ]
			puts "set_early_celllib_fpath $options(-early)"
		       }
		-help {
			set args [lassign $args - options(-help) ]
			puts "Usage: read_lib -late <late_lib_path> -early <early_lib_path>"
			puts "-late <provide late library path>"
			puts "-early <provide early library path>"
			puts "-help - Provides user deatails on how to use the command"
		      }	
		default break
		}
	}
}
```
read_verilog.proc
This proc generates the commands to load the synthesized netlist, which is essential for the OpenTimer tool.

```bash
#!/bin/tclsh

# Proc to convert read_verilog to OpenTimer format
proc read_verilog {arg1} {
	puts "set_verilog_fpath $arg1"
}
```

read_sdc.proc
This proc generates the necessary commands to load the .timing constraints file required by the OpenTimer tool. It performs a conversion of SDC file contents into a .timing file format compatible with OpenTimer. The conversion process is explained step-by-step with accompanying screenshots for clarity.

Converting create_clock Constraints
The proc begins by accepting the SDC file as an input parameter and processes the create_clock constraints section, translating it into the appropriate .timing format.

```bash
#!/bin/tclsh

proc read_sdc {arg1} {

# 'file dirname <>' to get directory path only from full path
set sdc_dirname [file dirname $arg1]
# 'file tail <>' to get last element
set sdc_filename [lindex [split [file tail $arg1] .] 0 ]
set sdc [open $arg1 r]
set tmp_file [open /tmp/1 w]

# Removing "[" & "]" from SDC for further processing the data with 'lindex'
# 'read <>' to read entire file
puts -nonewline $tmp_file [string map {"\[" "" "\]" " "} [read $sdc]]     
close $tmp_file

# Opening tmp file to write constraints converted from generated SDC
set timing_file [open /tmp/3 w]

# Converting create_clock constraints
# -----------------------------------
set tmp_file [open /tmp/1 r]
set lines [split [read $tmp_file] "\n"]
# 'lsearch -all -inline' to search list for pattern and retain elementas with pattern only
set find_clocks [lsearch -all -inline $lines "create_clock*"]
foreach elem $find_clocks {
	set clock_port_name [lindex $elem [expr {[lsearch $elem "get_ports"]+1}]]
	set clock_period [lindex $elem [expr {[lsearch $elem "-period"]+1}]]
	set duty_cycle [expr {100 - [expr {[lindex [lindex $elem [expr {[lsearch $elem "-waveform"]+1}]] 1]*100/$clock_period}]}]
	puts $timing_file "\nclock $clock_port_name $clock_period $duty_cycle"
}
close $tmp_file
```

Code and Results for STA

```bash
puts "\nInfo: Timing Analysis Started....."
puts "\nInfo: Initializing number of threads, libraries, sdc, verilog netlist path...."
source /home/vsduser/vsdsynth/procs/reopenStdout.proc
source /home/vsduser/vsdsynth/procs/set_num_threads.proc
reopenStdout $OutputDirectory/$DesignName.conf
set_multi_cpu_usage -localCpu 2
#return

source /home/vsduser/vsdsynth/procs/read_lib.proc
read_lib -early /home/vsduser/vsdsynth/osu018_stdcells.lib

read_lib -late /home/vsduser/vsdsynth/osu018_stdcells.lib

source /home/vsduser/vsdsynth/procs/read_verilog.proc
read_verilog $OutputDirectory/$DesignName.final.synth.v

source /home/vsduser/vsdsynth/procs/read_sdc.proc
read_sdc $OutputDirectory/$DesignName.sdc
reopenStdout /dev/tty
#return
if {$enable_prelayout_timing == 1} {
	puts "\nInfo: enable_prelayout_timing is $enable_prelayout_timing. Enabling zero-wire load parasitics"
	set spef_file [open $OutputDirectory/$DesignName.spef w]
puts $spef_file "*SPEF \"IEEE 1481-1998\" "
puts $spef_file "*DESIGN \"$DesignName\" "
puts $spef_file "*DATE \"Sun May 11 20:51:50 2025\" "
puts $spef_file "*VENDOR \"PS 2025 Hackathon\" "
puts $spef_file "*PROGRAM \"Benchmark Parasitic Generator\" "
puts $spef_file "*VERSION \"0.0\" "
puts $spef_file "*DESIGN_FLOW \"NETLIST_TYPE_VERILOG\" "
puts $spef_file "*DIVIDER / "
puts $spef_file "*DELIMITER : "
puts $spef_file "*BUS_DELIMITER [ ] "
puts $spef_file "*T_UNIT 1 PS "
puts $spef_file "*C_UNIT 1 FF "
puts $spef_file "*R_UNIT 1 KOHM "
puts $spef_file "*L_UNIT 1 UH "
}

close $spef_file

set conf_file [open $OutputDirectory/$DesignName.conf a]
puts $conf_file "set_spef_fpath $OutputDirectory/$DesignName.spef"
puts $conf_file "init_timer "
puts $conf_file "report_timer "
puts $conf_file "report_wns "
puts $conf_file "report_worst_paths -numPaths 10000 "
close $conf_file

set tcl_precision 3

set time_elapsed_in_us [time {exec /home/vsduser/OpenTimer-1.0.5/bin/OpenTimer < $OutputDirectory/$DesignName.conf >& $OutputDirectory/$DesignName.results} 1]
set time_elapsed_in_sec "[expr {[lindex $time_elapsed_in_us 0]/100000}] sec"
puts "\nInfo: STA finished in $time_elapsed_in_sec seconds"
puts "\nInfo: Refer to $OutputDirectory/$DesignName.results for warning and errors"

puts "tcl_precision is $tcl_precision
```

<img width="567" height="1067" alt="image" src="https://github.com/user-attachments/assets/342f7bc2-ea70-4964-a3f9-7c243123d8f8" />
<img width="1021" height="941" alt="image" src="https://github.com/user-attachments/assets/949638cc-13c1-41dd-aad2-7e7295092af7" />
<img width="1007" height="895" alt="image" src="https://github.com/user-attachments/assets/ba13cc78-225a-4fe4-a651-c76e7887ef3a" />
<img width="1108" height="949" alt="image" src="https://github.com/user-attachments/assets/75997d84-bbfb-403b-8be2-9321530810dc" />
<img width="1054" height="934" alt="image" src="https://github.com/user-attachments/assets/b759fd14-44db-4aec-9e96-1d89aef37b95" /> 
<img width="1067" height="440" alt="image" src="https://github.com/user-attachments/assets/3d5a7562-b46e-483d-8470-03dfeda0e857" />
<img width="1027" height="187" alt="image" src="https://github.com/user-attachments/assets/47459fa8-97db-457a-b2d0-3b55b47a4e94" />
<img width="1058" height="345" alt="image" src="https://github.com/user-attachments/assets/0f0e5b8e-a9be-4c5e-ace8-efccd9e5b98e" />
 
Data Collection from .results File and Additional Resources for QoR
I have successfully developed the code to extract all necessary data points from the .results file and other relevant sources for Quality of Results (QoR) analysis. This includes capturing key metrics as well as the total runtime of the complete .tcl script. The core script, along with terminal screenshots showing variable outputs and debugging information via puts statements, is provided below.
```bash
#---------------find WNS using RAT-------------------------#
set worst_RAT_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r]
set pattern {RAT}
puts "pattern is $pattern"
while {[gets $report_file line] != -1} {
	if {[regexp $pattern $line]} {
		set worst_RAT_slack "[expr {[lindex $line 3]/1000}]ns"
		puts "worst_RAT_slack is $worst_RAT_slack"
		break
	} else {
		continue
	}
}
close $report_file
#return

#--------------------------fine number of output violation------------#
set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
                incr count [regexp -all -- $pattern $line]
}
set Number_of_output_violations $count
puts "Number of output violations $Number_of_output_violations"
close $report_file


#---------------find WNS setup violation-------------------------#
set worst_negative_setup_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r]
set pattern {Setup}
puts "pattern is $pattern"
while {[gets $report_file line] != -1} {
        if {[regexp $pattern $line]} {
                set worst_negative_setup_slack "[expr {[lindex $line 3]/1000}]ns"
                break
        } else {
                continue
        }
}
close $report_file

#---------------find number of setup violation-------------------------#

set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
                incr count [regexp -all -- $pattern $line]
}
set Number_of_setup_violations $count
close $report_file

#---------------find WNS hold violation-------------------------#
set worst_negative_hold_slack "-"
set report_file [open $OutputDirectory/$DesignName.results r]
set pattern {Hold}
puts "pattern is $pattern"
while {[gets $report_file line] != -1} {
        if {[regexp $pattern $line]} {
                set worst_negative_hold_slack "[expr {[lindex $line 3]/1000}]ns"
                break
        } else {
                continue
        }
}
close $report_file

#---------------find number of hold violation-------------------------#

set report_file [open $OutputDirectory/$DesignName.results r]
set count 0
while {[gets $report_file line] != -1} {
                incr count [regexp -all -- $pattern $line]
}
set Number_of_hold_violations $count
close $report_file


#---------------find number of instance---------------------------#
set pattern {Num of gates}
puts "pattern is $pattern"
set report_file [open $OutputDirectory/$DesignName.results r]
while {[gets $report_file line] != -1} {
        if {[regexp $pattern $line]} {
                set Instance_count [lindex [join $line " "] 4 ]
                break
        } else {
                continue
        }
}
close $report_file

set Instance_count "$Instance_count PS"
set time_elapsed_in_sec "$time_elapsed_in_sec PS"
```
<img width="1032" height="501" alt="image" src="https://github.com/user-attachments/assets/dc059b36-fcb2-4bb7-b6f4-2f0d1d8d2920" />
QoR Generation
The code for generating the Quality of Results (QoR) report has been successfully implemented. The core script, along with terminal screenshots, is provided below.

```bash
set formatStr {%15s%15s%15s%15s%15s%15s%15s%15s%15s}

puts [format $formatStr "-----------" "-------" "--------------" "-----------" "-----------" "----------" "----------" "-------" "-------"]
puts [format $formatStr "Design Name" "Runtime" "Instance Count" " WNS Setup " " FEP Setup " " WNS Hold " " FEP Hold " "WNS RAT" "FEP RAT"]
puts [format $formatStr "-----------" "-------" "--------------" "-----------" "-----------" "----------" "----------" "-------" "-------"]
foreach design_name $DesignName runtime $time_elapsed_in_sec instance_count $Instance_count wns_setup $worst_negative_setup_slack fep_setup $Number_of_setup_violations wns_hold $worst_negative_hold_slack fep_hold $Number_of_hold_violations wns_rat $worst_RAT_slack fep_rat $Number_of_output_violations {
	puts [format $formatStr $design_name $runtime $instance_count $wns_setup $fep_setup $wns_hold $fep_hold $wns_rat $fep_rat]
}

puts [format $formatStr "-----------" "-------" "--------------" "-----------" "-----------" "----------" "----------" "-------" "-------"]
puts "\n"
```
<img width="1030" height="461" alt="image" src="https://github.com/user-attachments/assets/318183a7-2964-4473-85a7-03e3835664d0" />
<img width="1036" height="461" alt="image" src="https://github.com/user-attachments/assets/17865828-6376-47de-9b98-bb42ae78cc7a" />
<img width="1037" height="346" alt="image" src="https://github.com/user-attachments/assets/20b3a0fb-c9a3-4a0c-b7eb-5ec8f07acabe" />
<img width="1037" height="482" alt="image" src="https://github.com/user-attachments/assets/63c84515-bfd5-42ec-ada8-b9252ce1dd3d" />

---

## Tools Used
- TCL Development Suite
- [Yosys](https://yosyshq.net/yosys/) – Open-source RTL synthesis
- [OpenTimer](https://github.com/OpenTimer/OpenTimer) – Static timing analysis
- VSD custom libraries 

---

## Projects & Exercises
- TCL scripts for automating constraint generation
- Yosys-based memory synthesis and QOR report generation
- Parsing and bit-blasting signal constraints
- End-to-end QOR reporting using TCL

---

## Prerequisites
- Basic knowledge of digital design
- Familiarity with Verilog and Linux CLI

---

## Instructor
**Kunal Ghosh** – Founder, VLSI System Design Corp (VSD)

---

