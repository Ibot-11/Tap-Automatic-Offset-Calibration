# **Readme and Macros are still Work in Progress!**  
# **Paused due to other projects.**

## <ins>How does Auto-Offset work?</ins>
The endstop is connected to GND and the nozzle is connected to a endstop pin.
If the nozzle touches the endstop pin the loop closes and the endstop triggers.
It's just a really simple switch. The real advantage is that there is no switch pre travel (like with AutoZ for Klicky)
  
#### **Okay, but how can I calibrate my Z-offset with this?**
First you home your nozzle on the endstop. Next you do some probes on top of the endstop. With this relative distance between the endstop trigger point and the probe trigger point (+ an additional offset) you can calculate the Z-offset.  
The additional offset is required since you want some distance between the nozzle and bed when you are at Z0. Otherwise your nozzle would touch your bed when at Z0  
The adjustment is done via GCODE_OFFSET since you cannot edit the probe offset without restarting Klipper. (The same way AutoZ for Klicky does it)  
  
#### **Is this a polished mod?**
**NO!** This is still a proof of concept and not something you should or can use as your default setup. There are still reliability issues which can’t be easily solved.  

## <ins>Demo Video</ins>
[![Video](https://i.imgur.com/ZFL8DYg.png)](https://www.youtube.com/watch?v=vMMA7r7aXLY)

## <ins>Features</ins>
+ Automatic calibration of the first layer once configured correctly
+ Multiple “offset correction” values for different print surfaces (textured, smooth) which you can easily select via a macro
+ Dynamic probing temperature. Example: PLA -> 150°C; ABS -> 180°C (Increases accuracy and compensates for thermal expansion)
+ Adds a z-endstop, but still homes with tap. No reference index required

**Safety Features**  
+ Crash detection if you miss the endstop or something is not connected right.
+ Maximal adjustment (+/-0.5mm default)
+ Maximal deviation between samples (0.008mm default)
+ Retry with extra nozzle cleaning if crash protection triggers
+ You can only use AUTO_OFFSET and AUTO_OFFSET_START. Submacros for calculations can’t be used directly
+ Unable to home via endstop if hotend is below 150°C (to squish ooze away)
  
  
## <ins>Installation</ins>
**ATTENTION! Use at your own RISK!**
  
### <ins>Installing the endstop</ins>
This depends on the endstop you are using. Currently there are 2 options:
  
**Modified Voron Endstop:** (Coming soon)  
**Machined hex bolt:** (Currently only available for beta testers)  
![1698243237961](https://github.com/Ibot-11/Tap-Automatic-Offset-Calibration/assets/84148069/6081fa7c-d344-4299-bbc5-879f96ad33d3)

  
### <ins>Connecting the endstop</ins>
Crimp a connector to the wire and connect the endstop to any GND endstop connection.
  
### <ins>Connecting the Nozzle/Hotend to a Endstop Pin</ins>
This is a bit more complicated and depends on your hotend. A **Dragon HF** for example needs some “scratching” where the spacers go into the heatsink in order to get a proper connection.
I just wrapped some wire around one of the mounting bolts. Simple, but it works.
![1688649731782](https://github.com/Ibot-11/Tap-Automatic-Offset-Calibration/assets/84148069/01879875-e808-4daa-a37d-068a120869a4)
![1688649731812](https://github.com/Ibot-11/Tap-Automatic-Offset-Calibration/assets/84148069/61e5e405-81f7-46bd-9643-6f6df0ef1417)
![1688649731772](https://github.com/Ibot-11/Tap-Automatic-Offset-Calibration/assets/84148069/700c585f-636e-4b11-bc20-4e8c8a5133b6)
  
For the **Bamabu Hotend** I just used a M2,5 screw on my printhead.

**DO NOT ATTACH ANY WIRE DIRECTLY TO THE HEATER BLOCK!** At first glance this will work just fine, but the wire will oxidize over time which causes poor conductivity and a lot of problems.
![1688649731762](https://github.com/Ibot-11/Tap-Automatic-Offset-Calibration/assets/84148069/523a02fc-d4ee-4753-8a59-a0fd66557f14)
  
  
**Attention!**
Don't let any positive wire touch your hotend. If a 24V wire touches your hotend (5V endstop) you may short out your board!  
Also make sure your endstop GND is isolated from the bed/frame Ground. While theoretically nothing should happen I still don’t recommend connecting them.  
  
**Check if your endstop works** by using a wire before running any macros/commands.  
You may have to clean your nozzle to get connection with your wire. Use QUERY_ENDSTOPS to get the endstop state.  
  
  
  
## <ins>Macro Installation</ins>
#### **Create a backup of your configs before changing anything!**
  
### <ins>[z_stepper] Changes</ins>
+ Change your endstop pin to the one connected to the hotend/nozzle
+ Change your homing speed to 0.5mm/s or slower.
+ Change your second homing speed to 0.05mm/s or slower
+ Change your retract distance to 0.1mm
##### **Example:**
```
[stepper_z]
step_pin: PF11
dir_pin: PG3
enable_pin: !PG5
rotation_distance: 40
gear_ratio: 80:16
microsteps: 64
endstop_pin: !sb2040:gpio29
position_max: 210
position_min: -5
position_endstop: 0
homing_speed: 0.5
second_homing_speed: 0.05
homing_retract_dist: 0.1
```
  
**Why 0.05mm/s?**
In my testing I found that you have to home slow to get really accurate results (around +/-0,003). The hotend just needs some time to squish away any material left on the nozzle.
For default homing we’ll use Tap instead of the endstop. So the slow speeds don't really matter.
  
  
  
## <ins>Setting up Configs</ins>
  
### Include Auto_Offset.cfg
Copy Auto_Offset.cfg to your config folder.
Add ```[include Auto_Offset.cfg]``` to your printer.cfg.
  
### <ins>Homing Override</ins>
In order to use AutoZ you have to add a homing override.  
This adds the function to either home with tap or your endstop.  
**G28 Z & G28 -> Tap**  
**G28 P -> Tap (P for Probe)**  
**G28 E -> Endstop (E for Endstop)**  
  
The homing override calls some macros which should work for sensorless homing and normal sensor homing. These are based on the recommended sensorless homing macros by the Voron team.  
**You have to remove [safe_z_home] if defined in your config**  
[homing_override] is included in the Auto-Offset.cfg. No changes needed.
  
  
  
### <ins>Auto_Offset.cfg Settings</ins>
  
#### <ins>Endstop Position</ins>
You also have to set your endstop position in Z_ENDSTOP_POSITION! Similar as with the default Voron endstop. You can set this at the beginning of the Auto-Offset.cfg
##### **Example:**
```
[gcode_macro POSITION_Z_ENDSTOP]
description: Dummy macro to set the endstop position
variable_endstop_x : 202.5
variable_endstop_y : 252.5
gcode:

```
  
#### <ins>Clean Nozzle Macro</ins>
It’s important to set your nozzle cleaning macro (Default:”CLEAN_NOZZLE”)
This will be used when running ```AUTO_OFFSET```  
**It’s important that your nozzle cleaning macro lifts the nozzle after cleaning by at least 5mm!**
  
#### <ins>More Settings…</ins>
All other variables/settings are explained in the Auto_Offset.cfg
  
  
  
### <ins>PRINT_START</ins>
The last step is to add AUTO_OFFSET to your PRINT_START macro. There are two options:
  
#### Option 1: AUTO_OFFSET_START (Recommended)
AUTO_OFFSET_START is a macro package which you can insert into your PRINT_START macro. This package includes:
+ Printer homing
+ Heating to probe temperature
+ Nozzle cleaning
+ Quad gantry level
+ Auto-Offset
+ Bed mesh calibrate
+ Heting to printing temperature
+ Nozzle cleaning
  
Requried parameters: EXTRUDER_TEMP and BED_TEMP
  
##### **Example:**
```
[gcode_macro PRINT_START]
description: Perform calibration and get ready to print
gcode:
    {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|float %}
	  {% set BED_TEMP = params.BED_TEMP|float %}

    CLEAR_PAUSE # clear any pause... again
    BED_MESH_CLEAR # clear bed mesh

    M107 # turn off part fan (heatsoak turned it on before)

    AUTO_OFFSET_START EXTRUDER_TEMP={EXTRUDER_TEMP} BED_TEMP={BED_TEMP} # Parameters must be defined. This is a module of AUTO_OFFSET

    LINE_PURGE # KAMP Purge

    M117 # Clear status messages
```
  
  
#### Option 2: AUTO_OFFSET
  
Just add it between your quad gantry level and bed mesh leveling. You also have to specify the hotend temperature for the dynamic probe temperature.
##### **Example:**
```
[gcode_macro PRINT_START]
description: Perform calibration and get ready to print
gcode:
    {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(200)|float %}
    {% set BED_TEMP = params.BED_TEMP|default(60)|float %}
    {% set PROBE_TEMP = 150 %}
    {% set th = printer.toolhead %}

    CLEAR_PAUSE
    BED_MESH_CLEAR

    M107

    M117 Homing..
    G28

    G90
    G0 X{th.axis_maximum.x//2} Y{th.axis_maximum.y - 10} Z30 F30000
    SET_HEATER_TEMPERATURE HEATER=extruder TARGET={PROBE_TEMP}
    SET_HEATER_TEMPERATURE HEATER=heater_bed TARGET={BED_TEMP} 
    M109 S{PROBE_TEMP}               ; wait for extruder temperature
    M190 S{BED_TEMP}                    ; wait for bed temprature

    CLEAN_NOZZLE

    M117 Gantry adjust..
    QUAD_GANTRY_LEVEL
    G28 Z

    M117 Calibrating first layer..
    AUTO_OFFSET EXTRUDER_TEMP={EXTRUDER_TEMP}

    M117 Bed Mesh calibrate..
    BED_MESH_CALIBRATE

    G90
    G0 X{th.axis_maximum.x//2} Y{th.axis_maximum.y - 10} Z30 F30000

    M109 S{EXTRUDER_TEMP}

    M117 Cleaning..
    CLEAN_NOZZLE

    LINE_PURGE # KAMP Purge

    M117
```
  
  
  
## <ins>Getting started</ins>
First I recommend doing a PROBE_CALIBRATE if you never calibrated your offset before.
  
#### <ins>Auto Offset</ins>
For  Auto Offset you may have to fine tune the values specified in the plate macros. Different plates and first layer settings may need different first layer squish. You can also fine tune them to your personal preference.
You can also create a new plate preset by copy & paste an existing one. Don’t forget to rename everything! You can create as many plates as you like.
  
#### <ins>First Print</ins>
Just start your first print as usual. Check that your AUTO_OFFSET or AUTO_OFFSET_START macro is added to the PRINT_START macro correctly. Be ready to press the emergency button at any time. Just in case. You can still live adjust your offset. But don’t forget to apply any +/- changes to the print plate specific correction value.
  
  
  
## <ins>Macro Parameters</ins>
### <ins>Auto_Offset</ins>
+ EXTRUDER_TEMP - Used for dynamic probe temperature. Must be defined
+ SAMPLES - Sample amount to probe
  
### <ins>Auto_Offset_Start</ins>
+ EXTRUDER_TEMP - Used to set temperatures. Must be defined
+ BED_TEMP - Used to set temperatures. Must be defined
  
  
  
## <ins>Misc (FAQ)</ins>
##### Should I use a reference index for my bed mehs?  
NO! This can cause serious problems or even damage!
  
**Why do you use a machined hex bolt?**  
Theoretically a endstop pin like the default Voron endstop should work. But I really hate centering my nozzle on this tiny pin. The hex bolt just gives you much more surface to work with.
  
**Does this work with adaptive mesh like KAMP?**  
Of course! I also use it with KAMP.
  
**Which nozzles / hotends work?**  
Pretty much every hotend / nozzle which is conductive. Just check the conductivity with a multimeter and also ensure your endstop triggers by bridging the endstop and nozzle with a wire. I currently use a Bambu Hotend (Clone) with CHT steel nozzle
  
**Does this work with a Klicky probe?**  
No. Only nozzle probes like TAP are supported
  
**Why is this a macro and not a python script?**  
Because I have no clue how to code python. Also my macro skills are limited. Putting together the macros already was a challenge for me.
  
**Why is this not a single macro?**  
Because Klipper macros can be really weird where lines get executed before the ones above them aren’t finished. This results in wrong calculations and values. By splitting the macro you can prevent this behavior. At this size it’s also easier to work with single macro snippets instead of one single big macro.
