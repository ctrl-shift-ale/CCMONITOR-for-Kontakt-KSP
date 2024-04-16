# CCMONITOR for Kontakt KSP

############ 1. WHAT IS CCMONITOR?

CCMONITOR is a  tool for KSP scripters, offering real-time visual feedback by displaying MIDI CC values directly within your DAW's MIDI track. This enables precise tracking of condition triggers and variable changes synchronized with your Kontakt instrument's audio output. Additionally, CCMONITOR features a  bus routing tool, configurable via its UI, which allows effortless monitoring of individual groups within your instrument. 

The primary functions of CCMONITOR are as follows:

A. Variable Monitoring: Allows for the monitoring of selected variables of a target script, visualized as CC envelopes directly within a MIDI track in your DAW.
B. Event and Condition Monitoring: Provides real-time monitoring of event and condition triggers in the target script, represented as clicks in CC channels within a MIDI track in your DAW.
C. Effortless Group Audio Routing: Simplifies the process of monitoring individual groups through Kontakt auxiliary outputs.

The release also includes:
1. An instrument template file featuring an ad-hoc Instrument Buses configuration set to work with a template configuration .ksp file (refer to paragraph 2.2) 
2. A demo consisting of a Kontakt instrument and all the pre-compiled scripts loaded in it (refer to paragraph 2.3)

############ 

############ 2. RELEASED CONTENT

2.1. CCMONITOR core scripts (Sublime KSP)

CCmonitor.Functions.ksp

CCmonitor.Interface.ksp

CCmonitor.UI.ksp


The  files above must be located in the same folder where the target script is located.
Please note that these scripts have been written using SublimeKSP v. 1.17.2.


2.2. CCMONITOR  templates

CCmonitor.ScriptTemplate.ksp contains all the lines of code required to interface a target script with CCMONITOR.

CCmonitor_routingTemplate.nki (for Kontakt 6) is an empty instrument with a configured setup meant to be used by CCMONITOR to route groups to be monitored to Kontakt’s auxiliary outputs.


2.3. CCMONITOR demo scripts and .nki

CCmonitor_DEMOscript.ksp

CCmonitor_DEMOscript.Config.ksp

CCmonitor_DEMOscript.UI.ksp

CCmonitor_DEMO.nki


CCmonitor_DEMOscript is a basic script  which  modifies the volume of two Kontakt groups of the instrument CCmonitor_DEMO.nki according to a random function inside a NOTE_HELD while loop.

CCmonitor_DEMOscript.Config contains the macro CCMONITOR.Configuration which includes the information about the variables, groups and output to be monitored / used by CCMONITOR (see paragraph 3.1).

CCmonitor_DEMOscript.UI is CCMONITOR’s UI script already configured to be compiled on Sublime Text and loaded in the instrument CCmonitor_DEMO.nki (see paragraph 3.2).

CCmonitor_DEMO.nki (for Kontakt 6) already contains the compiled versions of CCmonitor_DEMOscript and CCmonitor_DEMOscript.UI. 

############ 

############ 3. CONFIGURING CCMONITOR

To configure CCMONITOR for use with your script, follow this guide.


3.1. Create the Macro CCMONITOR.Configuration

Write the configuration lines within a macro named CCMONITOR.Configuration. 

Use the commands below inside macro CCMONITOR.Configuration in order to indicate the variables to be monitored, the groups to be routed, and the buses to be used for the routing.

CCMONITOR.CONFIG.ADD.Variable( var name, UI name , scaling bool , min val , max val )
 
	‘var name’ : The name of the variable as declared in the monitored script

	‘UIname’ (string): The name of the variable as it will be displayed on CCMONITOR UI menus

	‘scaling bool’ : Set to 1 to scale the variable's value to the MIDI range 0-127, or set to 0 to output  the actual 	variable value (values < 0 will be displayed as 0, and values > 127 will be displayed as 127)

	‘min val’: The minimum input value for the scaling operation. The value of this argument is irrelevant if scal	ing is off

	‘max val’: The maximum input value for the scaling operation. The value of this argument is irrelevant if scal	ing is off

Please note: The scaling mode, as well as the minimum and maximum scalable values, can be modified at any 	time from the target script using dedicated commands (refer to paragraph 3.4).


CCMONITOR.CONFIG.ADD.Group( group name ,  default bus number )

	‘group name’ (string) : the name of the group as set in the Kontakt instrument

	‘default bus number’ :  the number of the bus to which the group will be routed back when the group is no 	longer selected for monitoring in CCMONITOR


CCMONITOR.CONFIG.ADD.Bus( UI bus name , bus number )

	‘UI bus name’ (string) : the name of the bus as it will be displayed on CCMONITOR UI menus
	‘bus number’ :  the number of the bus 

Here below an example of CCMONITOR.Configuration:

macro CCMONITOR.Configuration

	CCMONITOR.CONFIG.ADD.Variable(rnd, "Random" , 1, 0, 999) // scaling is on (as rnd max value > 127)
	CCMONITOR.CONFIG.ADD.Variable(attDur_ms , "Attack" , 1 , 0, 0) // no scaling as attDur is always < 127
	CCMONITOR.CONFIG.ADD.Variable(relDur_ms , "Release" , 1, 200, 1500) 
	
	CCMONITOR.CONFIG.ADD.Group("Mezzoforte",0)
	CCMONITOR.CONFIG.ADD.Group("Forte",0)

	CCMONITOR.CONFIG.ADD.Bus("AUX 1" , 1) 
	CCMONITOR.CONFIG.ADD.Bus("AUX 2" , 2) 

end macro


Save the macro inside a .ksp file which  has to be imported at the beginning of both the script CCmonitor.UI.ksp and the script to be monitored (refer to paragraphs 2.2 and 2.3 below).


3.2. Configure and compile the script CCmonitor.UI.ksp

Open CCmonitor.UI.ksp and add the following line at the beginning of the script:

import configuration_ksp_file

where configuration_ksp_file is the .ksp file where the macro CCMONITOR.Configuration (see paragraph 3.1) is located.

Paste the compiled version into a free script tab in your Kontakt instrument.


3.3. Configure the target script 

To integrate your script with CCMONITOR:

1. add the following lines at the beginning of the script:

	import configuration_ksp_file

	where configuration_ksp_file is the  file containing the macro CCMONITOR.Configuration (see paragraph 3.1)

2. import "CCmonitor.Interface.ksp"

3. add the following line at the beginning of the init callback:

	CCMONITOR.init

4. add the following line at the beginning of the note callback:

	CCMONITOR.noteOn

5. add the following line at the end of the NOTE_HELD while loop inside the note callback:

	CCMONITOR.update

6. add the following line at the beginning of the pgs_changed callback:

	CCMONITOR.pgs

7. add the following line at the end of your script:

	CCMONITOR.functions


3.4. Code-specific commands

1. Each time a monitored variable undergoes an assignment statement  in your script, the command

	CCMONITOR.setVarVal( variable  )

must be called. Failure to invoke this command will result in CCMONITOR not being notified of the variable's 	value change.


2. The command 

	CCMONITOR.click( cc channel ) 

will trigger a spike ( = a sudden change of the CC value to 127) in the selected CC channel. This can be useful to check if and when a condition in the target script has been triggered. The value of the CC channel will move to 0 at the next iteration of the command CCMONITOR.update


3. The scaling of a variable prior to be put out as MIDI cc value can be re-configured anytime in the target script with the following commands:

	to activate/disactivate the scale function:

		CCMONITOR.setVarScaled( variable , scaling bool )

	to change the min / max value of the input for the scale function:

		CCMONITOR.setVarMin( value ) 

		and 

		CCMONITOR.setVarMax( value )


3.5. Utility Variables 

Assign 0 to CCMONITOR.CONFIG.ACTIVEin the init callback of the target script to switch CCMONITOR off. The variable’s default value is 1 (on).

Assign 1 to CCMONITOR.CONFIG.PRINT in the init callback of the target script to activate all the message() commands in CCMONITOR for debugging purposes. The variable’s default value is 0 (off).



 
