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
'''
### Module 2: Variable Creation & Constraint Processing
- Working with arrays, matrices, and loop constructs
- Parsing and validating CSV/SDC constraint files

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

## License
This repository is intended for educational purposes. Licensing details may apply based on VSD-provided content.

---

## How to Contribute
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

---

## Show Your Support
If you found this repository helpful, consider starring it!

