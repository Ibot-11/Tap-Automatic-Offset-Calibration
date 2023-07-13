## How does Auto-Offset work?
The endstop is connected to GND and the nozzle is connected to a endstop pin.
If the nozzle touches the endstop pin the loop closes and the endstop triggers.
It's just a really simple switch. The real advantage is that there is no switch pre travel (like with AutoZ for Klicky)

#### **Okay, but how can I calibrate my Z-offset with this?**
First you home your nozzle on the endstop. Next you do some probes on top of the endstop. With this relative distance between the endstop trigger point and the probe trigger point (+ an additional offset and thermal expansion compensation value) you can calculate the Z-offset.

The additional offset is required since you want some distance between the nozzle and bed when you are at Z0. Otherwise your nozzle would touch your bed when at Z0

The thermal expansion compensation is an additional feature, which you can use in order to compensate for different extruder temperatures. Since you do all your calibration at a probing temperature (150°C), the actual nozzle position changes a tiny bit when heated to a different printing temperature due to thermal expansion. Where talking about values around 0,02mm between PLA temperatures (215°C) and ABS temperatures (260°C)
This is not nearly enough to turn a good first layer to a bad one, but it’s enough to turn a perfect first layer into a good first layer.

The adjustment is done via GCODE_OFFSET since you cannot edit the probe offset without restarting klipper. (The same way AutoZ for Klicky does it)

#### **Is this a polished mod?**
**NO!** I use this method now for a while on my V2.4R2 with good results. I further polished it over the last months and rewrote the complete macro for V2.

**Features:**
Automatic calibration of the first layer once configured correctly
Multiple “offset correction” values for different print surfaces (textured, smooth) which you can easily select via a macro.
Automatic calibration and calculation of your hotend expansion factor between different temperatures (optional but recommended)
Adds a z-endstop, but still homes with tap. No reference index required.



## Installation:
####**ATTENTION! Use at your own RISK!**

### Installing the endstop:
This depends on the endstop you are using. Currently there are 2 options:

**Modified Voron Endstop:** (Coming soon)
**Machined hex bolt:** (Only available for beta testers)

#### Connecting the endstop**
Crimp a connector to the wire and connect the endstop to any GND endstop connection.


### Connecting the nozzle/hotend to a endstop pin
This is a bit more complicated and depends on your hotend. A **Dragon HF** for example needs some “scratching” where the spacers go into the heatsink in order to get a proper connection.
I just wrapped some wire around one of the mounting bolts. Simple, but it works.

For the **Bamabu Hotend** I just used a M2,5 screw on my printhead.

**DO NOT ATTACH ANY WIRE DIRECTLY TO THE HEATER BLOCK!** At first glance this will work just fine, but the wire will oxidize over time which causes poor conductivity and a lot of problems.

**Attention!**
Don't let any positive wire touch your hotend. If a 24V wire touches your hotend (5V endstop) you may short out your board!
Also make sure your endstop GND is isolated from the bed/frame Ground. While theoretically nothing should happen I still don’t recommend connecting them.

**Check if your endstop works** by using a wire before running any macros/commands.
You may have to clean your nozzle to get connection with your wire. Use QUERY_ENDSTOPS to get the endstop state.



## Macro Installation:
#### **Create a backup of your configs before changing anything!**

### z_stepper Changes
Change your endstop pin to the one connected to the hotend/nozzle
Change your homing speed to 0.5mm/s or slower.
Change your second homing speed to 0.05mm/s
Change your retract distance to 0.1mm
**Example:**
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



## Setting up Configs

### Include Auto-Offset.cfg
Copy Auto_Offset.cfg to your config folder.
Add ```[include Auto_Offset.cfg]``` to your printer.cfg.


### Homing Override
In order to use AutoZ you have to add a homing override.
This adds the function to either home with tap or your endstop.
##### **G28 Z & G28 -> Tap**
##### **G28 P -> Tap (P for Probe)**
##### **G28 E -> Endstop (E for Endstop)**

The homing override calls some macros which should work for sensorless homing and normal sensor homing. These are based on the recommended sensorless homing macros by the Voron team.
**You have to remove [safe_z_home] if defined in your config**
[homing_override] is included in the Auto-Offset.cfg. No changes needed.



### Auto-Offset.cfg Settings

#### Endstop Position
You also have to set your endstop position in Z_ENDSTOP_POSITION! Similar as with the default Voron endstop. You can set this at the beginning of the Auto-Offset.cfg
**Example:**
```
[gcode_macro POSITION_Z_ENDSTOP]
description: Dummy macro to set the endstop position
variable_endstop_x : 202.5
variable_endstop_y : 252.5
gcode:

```

#### Clean Nozzle Macro
It’s important to set your nozzle cleaning macro (Default:”CLEAN_NOZZLE”)
This will be used when running ```AUTO_OFFSET```

#### More Settings…
All other variables/settings are explained in the Auto_Offset.cfg



### PRINT_START
The last step is to add AUTO_OFFSET to your PRINT_START macro. Just add it between your quad gantry level and bed mesh leveling. You also have to specify the hotend temperature if thermal compensation is used. **For example:**
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
    SETUP_KAMP_MESHING DISPLAY_PARAMETERS=1 FUZZ_ENABLE=1

    SET_FAN_SPEED FAN=nevermore_fan SPEED=0

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
    AUTO_OFFSET PROBE_TEMP={PROBE_TEMP}

    M117 Bed Mesh calibrate..
    BED_MESH_CALIBRATE

    G90
    G0 X{th.axis_maximum.x//2} Y{th.axis_maximum.y - 10} Z30 F30000

    M109 S{EXTRUDER_TEMP}

    M117 Cleaning..
    CLEAN_NOZZLE

    PURGE_LINE

    M117
```



## Getting started:
First I recommend doing a PROBE_CALIBRATE if you never calibrated your offset before.

### Thermal Compensation (optional but recommended):
If you want to use thermal compensation, start the calibration with THERMAL_COMPENSATION_CALIBRATION. **It’s really important that your printer is at ambient temperature!** or your values will be off.
This can take a while, depending on your hotend heating and cooling performance.
At the end you get your hotend specific value in the console. Copy this value and add it to the ```Auto_Offset.cfg``` (variable_thermal_expansion).


### Auto Offset:
For  Auto Offset you may have to fine tune the values specified in the plate macros. Different plates and first layer settings may need different first layer squish. You can also fine tune them to your personal preference.
You can also create a new plate preset by copy&paste a existent one. Don’t forget to rename everything! You can create as many plates as you like.


### First Print:
Just start your first print as usual. Check that your AUTO_OFFSET macro is added to the PRINT_START macro correctly. Be ready to press the emergency button at any time. Just in case. You can still live adjust your offset. But don’t forget to apply any +/- changes to the print plate specific correction value.


## Current “Safety Features”:
##### - Crash detection if you miss the endstop or something is not connected right.
##### - Maximal adjustment (+/-0,5mm default)
##### - Maximal deviation between samples (0,005mm default)
##### - You can only use AUTO_OFFSET and THERMAL_COMPENSATION_CALIBRATION. Submacros for calculations can’t be used directly
##### - Unable to home via endstop if hotend is below 150°C (to squish ooze away)



## Misc (FAQ):
##### Should I use a reference index for my bed mehs?
NO! This can cause serious problems or even damage!

##### **Why do you use a machined hex bolt?**
Theoretically a endstop pin like the default Voron endstop should work. But I really hate centering my nozzle on this tiny pin. The hex bolt just gives you much more surface to work with.

##### **Does this work with adaptive mesh like KAMP?**
Of course! I also use it with KAMP.

##### **Which nozzles / hotends work?**
Pretty much every hotend / nozzle which is conductive. Just check the conductivity with a multimeter and also ensure your endstop triggers by bridging the endstop and nozzle with a wire. I currently use a Bambu Hotend (Clone) with CHT steel nozzle

##### **Does this work with a Klicky probe?**
No. Only nozzle probes like tap are supported

##### **Why is this a macro and not a python script?**
Because I have no clue how to code python. Also my macro skills are limited. Putting together the macros already was a challenge for me.

##### **Why is this not a single macro?**
Because Klipper macros can be really weird where lines get executed before the ones above them aren’t finished. This results in wrong calculations and values. By splitting the macro you can prevent this behavior. At this size it’s also easier to work with single macro snippets instead of one single big macro.
