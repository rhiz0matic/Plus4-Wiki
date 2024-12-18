# Chamber Temperature Manager

## Add the following to your `gcode_macro.cfg` file

```
[gcode_macro MANAGE_CHAMBER_TEMP]
gcode:
    {% set target = printer['heater_generic chamber'].target %}
    {% set temperature = printer['heater_generic chamber'].temperature %}
    {% if temperature > 70 %}
        M106 P3 S255                      # Too hot! Set the exhaust fan to 100%
    {% else %}
        # Allow for 3C of "grace" before we start ramping the exhaust fan speed
        # This prevents the macro from fighting with the chamber heater PID algorithm
        {% set diff = temperature - (target + 3) %}
        {% if diff < 0 %}
            M106 P3 S0                    # Disable Exhaust Fan
        {% else %}
            {% set speed = ([(diff * 50), 255] | min) | int %}
            M106 P3 S{speed}
        {% endif %}
    {% endif %}
```

Make sure that this macro gets called on each layer change by adding it to the layer change machine g-code section in the Printer defition in your slicer.

For example:

![Layer Change Macro Addition](./image.png)

## What it does

The point of this macro is to keep ramping up the chamber exhaust fan speed if the chamber temperature is above the target chamber temperature.

The further above the target the chamber temperature is, the faster it runs the chamber exhaust fan.  This is an attempt to keep the chamber temperature from overheating

- It turns off the exhaust fan is the chamber is below the target temperature
- If the chamber is above 70C, then it turns on the exhaust fan full blast, to prevent overheating (or try to)
- If the chamber is more than 3C warmer than the target then it slowly ramps the exhaust fan speed

For example:

- If the chamber temperature is more than 8C above the target, then the exhaust fan is set to 100%
- If the chamber temperature is 4C above the target, the exhaust fan is at 20%.  At 5C above, the fan is set 40%, and so on.

