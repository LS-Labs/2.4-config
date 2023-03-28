# 2.4-config
Voron 2.4 config for the LSLabs Full Kit

This config file is provided as a reference only. Please read the voron docs and configure your printer carefully to avoid any mishaps 

Warning:- Do not just copy this over and then click print. At best it might just fail with an error and at worst it may crash your new printer toolhead into the bed or overshoot on heating. Follow the Voron Guide on initial setup and testing using this as a template for your values before using it.

On your first homing and any tests it is worth hovering your mouse over the emergency stop button just in case. 

Everything in this config is based on the amazing work done by the Voron team and the amazing support from the members of the Voron Discord. Without whom I would not be printing today.

You will need to follow some steps to configure your printer in order for this to work correctly.

This config uses seperate folders and files for different configs allowing easy updates without having to go through a single very long printer.cfg file

This works by using as an example ```[include machine/*.cfg]``` in the printer.cfg file. This instructs klipper to include all files in the machine folder relative to the printer.cfg file.

This is used extensively throughout this config so please familiarise yourself with it now.

# What file contains what

## Hardware Definitions

All hardware based configs are in the /machine/ folder
    adxl.cfg contains the adxl information
    displays.cfg contains the config for the standard lcd display, the colour for the display and also contains the Beeper configuration for audible notifications
    extruder.cfg contains the extruder config. including the rotation distance value which you will set below.
    fans.cfg contains the config for the hot end fan, parts cooling fan and controller/electronics bay fan. There is a second controller fan definition you can use if you plugged them in seperately instead of wiring them together.
    heaters.cfg contains the bed heater configuration.
    nevermore.cfg contains the config for the nevermore. Note the additions to your print start macro / start gcode for your slicer that is detailed at the top of the file. The delayed gcode macro to tunr the fan off after a specified time is in this file and not in the Macros folder.
    stealthburnerleds.cfg contains the config for the toolhead LED's
    steppers.cfg contains the stepper motor configs for the X,Y and Z steppers
    temps.cfg contains the config for additional temp readouts. This has the chamber temp MCU and Rpi. The Extruder and Bed temp definitions are in Heaters.cfg and extruder.cfg

## Macros

Macros are contained in the /macros folder and are included in printer.cfg on line 10 [include macros/macros.cfg]
the main /macros/macro.cfg then includes the following.
    line 2 includes all macros in the /macros/main folder. This contains our print start and print end macros. 
    line 5 includes the test speed macro. for usage see here https://ellis3dp.com/Print-Tuning-Guide/articles/determining_max_speeds_accels.html#usage-of-the-test_speed-macro
    line 9 includes the G32 macro which homes all axis then QGL and then another home in prep for printing.
    line 10 includes a very basic M600 macro which currently just pauses the printer and beeps to notify you.
    line 11 includes an unload and load filament macro for us when swapping filaments.
    line 12 is the modified nozzle scrub macro. This is for use with the decontaminator nozzle mod. As we had the nozzle crash into the brush when starting a print we modified this to just purge on a single side and then move in front of the brush before resuming. see https://github.com/VoronDesign/VoronUsers/tree/master/orphaned_mods/printer_mods/edwardyeeks/Decontaminator_Purge_Bucket_%26_Nozzle_Scrubber for more and comparison to the standard file

The /macros/main folder has all files included and contains
    beeper.cfg is a basic macro to accept arguments to make some beeps. this is used for example by the M600 macro to notify you that it is paused and by the print end macro.
    pauseresume.cfg makes some more sensible pause and resume macros to park the toolhead at the front left corner during a pause so it is easier to swap filaments and some checks to confirm it can extrude or restart filament safely.
    printend.cfg contains the PRINT_END macro which you will use in your slicer it also clears the currently set bed mesh
    prinstart.cfg contains everything for the start print including the nozzle scrub. it also checks if the printer is already homed and QGL has been run so it doesnt need to do it again.

for these macros to work well your superslicer config should contain the following

start g-code
```
M190 S0
M109 S0 ; uncomment to remove set&wait temp gcode added automatically after this start gcode
print_start EXTRUDER=[first_layer_temperature[initial_tool]] BED=[first_layer_bed_temperature]
SET_FAN_SPEED FAN=Nevermore SPEED=1
```


This passes the extruder and bed temps to the system and enables the nevermore fan when starting. 

End G-Code

```
UPDATE_DELAYED_GCODE ID=filter_off DURATION=600
PRINT_END
```


This passes a delay of 600 seconds (ten mins) to the nevermore delayed gcode which will turn off the nevermore after 10 mins has passed after the print ends.


Confirm you know and understand the names of the pinouts of the Octopus v1.1 board. https://www.lslabs.co.uk/wp-content/uploads/2023/03/btt_octopus_1.1_pins.png 

The config file contains the pin names and numbering used on the Octopus v1.1 board and may well be different if you have not followed our wiring guide precisely or added other options to your kit.

This config includes an ADXL section in /machine/adxl.cfg 
This requires you to perform some additional software setup detailed here https://www.klipper3d.org/Measuring_Resonances.html

In the https://www.klipper3d.org/Measuring_Resonances.html#software-installation section you will need to run the commands shown to update klipper and add some additional python libraries.

It is not always obvious reading through the klipper docs but you will also need to run through the following https://www.klipper3d.org/RPi_microcontroller.html to enable the RPi to be a secondary MCU so that it can recieve and process the adxl information.

If you do not want to run the adxl tuning for now and just want to start printing. you can just comment out the contents of the adxl file by adding a # to the start of each line and saving the file. 

Make a backup of your current config (if you ahve one) and then replace the entire of the /home/pi/printer_data/config folder with the above /config/ folder.

You will need to check/change the following.
    mcu path is in the printer.cfg file
    Confirm your Stepper Motors are correctly wired to the right connectors/pins and check the pins in /machine/steppers.cfg https://docs.vorondesign.com/build/startup/#stepper-motor-check
    Check your extruder motor pins in /machine/extruder.cfg
    Check that your fans turn on when expected 

Run the stepper Buzz and confirm direction is correct for the steppers.


Unklicky / Klicky
All the files required for the Unklicky probe are in the /klicky folder which is included in the main printer.cfg on line 12 using [include klicky/klicky-probe.cfg] 
This file in turn then includes all the required files for the Voron 2.4 build 
In /klicky/klicky-variables.cfg on lines 40 and 41 is the definitions for the X and Y coordinates for the klicky probe mount. This should be roughly correct if your probe mount is as far to the left as it can go on the gantry however once you have homed the X and Y for the first time move the toolhead across to position it directly in front of the probe and then make a note of this X position. update that in the klicky variables file and save and restart.
The Y position should always be at 350 as the probe should be in line with the toolhead when the Y is homed.
All of the homing functions are overwritten by the klicky macros so after you have confirmed the X position you should then be able to home the Z using the klicky just by clicking Home Z


Last steps

Run PID Tuning https://github.com/Klipper3d/klipper/blob/master/docs/Config_checks.md#calibrate-pid-settings
for the extruder run PID_CALIBRATE HEATER=extruder TARGET=170 with the nozzle at 175,175,20
for the bed run PID_CALIBRATE HEATER=heater_bed TARGET=60 
Save config after each PID tuning to save the config to the printer.cfg file

Z Offset.
When you have homed the printer you will need to set your Z offset. We do this by centering the toolhead to 175 175 on the X and Y and then move the z down gradually using the controls in mainsail until it is at Z 0
Then below the main controls you have the Z offset. Move the toolhead down gradually using these microsteps using a piece of paper to confirm the height is just off the bed.
Once this is completed you then use the Z offset save button and "Save to probe" This will write the offset to the probe offset value which is saved in the printer.cfg file. 



Run extruder motor calibration and update /machine/extruder.cfg with the correct rotation_distance https://ellis3dp.com/Print-Tuning-Guide/articles/extruder_calibration.html 


Home the printer. on X and Then Y check https://docs.vorondesign.com/build/startup/#stepper-motor-check if it is moving in the wrong direction at any point.

If you need to modify the stepper motor directions then the direction pins are in
    X Y and Z stepper motors are in /machine/steppers.cfg 
    Stepper x line 10
    stepper Y line 42
    Z0 (front left) line 86
    Z1 (back left) line 122
    Z2 (back right) line 140
    Z3 (front right) line 158

When testing the extruder the direction pin is in /machine/extruder.cfg line 10


As always please be careful when first using and testing your printer.  


