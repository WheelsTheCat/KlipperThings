 The fake chamber heater is a method of controlling your bed fans to regulate chamber temperatures. It is capable of maintaining a set temperature without the use of macros by treating the fans themselves as heaters.

## How it works

- We will be creating a generic heater in Klipper where the heater_pin is the pin assigned to your bed fans, the sensor_pin is the chamber thermistor, and the sensor_type is set to match.
- M141 will be added to the printer configuration to set the chamber temperature
- M191 will be added to provide a temperature wait for the chamber (to allow for heat soaking)

## Backup your configuration files!

As with any change to your machines, you should ensure you have a good backup of the configuration. This can be as simple as downloading the printer.cfg file to your local machine, or you can (and should) automate backups. Eric Zimmerman has an excellent write-up here: [Backing up printer configuration files to GitHub](https://github.com/EricZimmerman/Voron-Documentation/blob/main/community/howto/EricZimmerman/BackupConfigToGithub.md)

## Configuration in Klipper

1. Add the following two macros to your configuration. **These can be copy/paste with no changes**
    ```ini
    [gcode_macro M141]
    description: Set bed_fans temperature
    gcode:
    {% if 'S' in params %}
        SET_HEATER_TEMPERATURE HEATER=bed_fans TARGET={params.S|float}
    {% else %}
        RESPOND PREFIX="M141" MSG="Missing S parameter"
    {% endif %}
     
    [gcode_macro M191]
    description: Set bed_fans temperature and wait
    gcode:
    {% if 'S' in params %}
        TEMPERATURE_WAIT SENSOR="heater_generic bed_fans" MINIMUM={params.S|float}
    {% else %}
        RESPOND PREFIX="M191" MSG="Missing S parameter"
    {% endif %}
    ```
2. To configure the heater, you will need to replace the heater_pin, sensor_type, and sensor_pin with values appropriate for your machine. 
   ```ini
   [heater_generic bed_fans]
   heater_pin: <The pin your fans are connected to>
   sensor_type: <Generic 3950 or your thermistor type>
   sensor_pin: <The pin your thermistor is connected to>
   control: pid
   pid_Kp: 61.470837
   pid_Ki: 0.5
   pid_Kd: 0
   pwm_cycle_time: 0.3
   min_temp: -273.15
   max_temp: 90

   [verify_heater bed_fans]
   max_error: 120
   check_gain_time: 480
   hysteresis: 50
   heating_gain: 1
   ```
3. Save the configuration and restart Klipper.
4. You should now have a heater in your UI that is called Bed Fans, and you should be able to set a target temperature and have the fans ramp up and down. There should be no reason to change the PID values supplied above and you definitely don't want to attempt to run a PID tune on the bed_fans heater. 
5. You do, however, need to run a PID tune for your bed with the bed_fans running, so at this time we will turn the bed fans to 100% by setting them to 70C and then doing a PID tune of the bed to your normal printing temperature.

## PRINT_START

If you are not using Jontek's "[A Better Print Start Macro](https://github.com/jontek2/A-better-print_start-macro)", I highly recommend it. It covers most of what people need in a print_start. It is the only one discussed here, and there are two required changes from the current version (as of this writing).
1. Find the line 
   ```ini 
   TEMPERATURE_WAIT SENSOR="temperature_sensor chamber" MINIMUM={target_chamber_wait} 
   ```
    and comment it out by prefixing it with a "#".

2. Add a new line below it with 
   ```ini
   M191 S{target_chamber_wait}
   ```
3. Now you need to decide where to place your `M141 S{target_chamber}`. This command will immediately start the bed fans running in an attempt to reach the requested temperature, but will not cause the machine to wait for it to be reached. If you have a high wattage bed and good insulation, you will probably want to place it as the first line of the print_start. If you have issues with the bed reaching temperature, you may want to place it immediately after the heat soak and before the line that reads
   ``` 
   # Heat hotend to 150c. This helps with getting a correct Z-home.
   ```
## Slicer Setup

1. If you are currently using "A Better Print Start Macro" and passing chamber temps as instructed there, you should be finished
2. If you are not, you can follow his instructions here: [Required changes in your slicer](https://github.com/jontek2/A-better-print_start-macro#warning-required-changes-in-your-slicer-warning)

## Something isn't working
In the Voron Design Discord Server, you can visit [Toasted Marshmallow -- READ THE PINS.](https://discord.com/channels/460117602945990666/1021908552702505051)

## Credits
Initial configuration example from Deutherius

