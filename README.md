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
vsdsynth.tcl (using matric package and creating a matrix from the details.csv file)
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
- Writing clock constraints (period, duty cycle)
- Classifying input ports using regular expressions

### Module 4: RTL Synthesis & Yosys Integration
- Developing complete synthesis scripts
- Memory module synthesis and TCL error handling with Yosys

### Module 5: QOR Report Generation
- Runtime and delay extraction using TCL procedures
- Conversion of constraints to OpenTimer format
- Bit-blasting bussed signals

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

