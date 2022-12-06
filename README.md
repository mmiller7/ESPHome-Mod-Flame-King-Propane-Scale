# ESPHome-Mod-Flame-King-Propane-Scale
High level reverse-engineering and modification of Flame King YSN-PS1 Bluetooth propane scale to ESPHome WiFi

-

Overview:

This project started because I wanted a propane tank scale so I could track (and remind me to refill)
the propane tank running our gas fireplace.  This was made harder because I use 40lb tanks and many
scales only support 20lb tanks.

-

Recommended Modification Tools/Parts:

Tools:
* Tweasers
* Soldering Iron
* Hot Air Reflow gun (~600F)
* Knife or rotary cutting tool (to cut/grind thru a PCB trace)

Parts:
* ESP8266 module (suggest D1 Mini)
* DS18B20 Temperature Sensor
* 4.7K Resistor (suggest 1/4 watt or smaller size)
* Large value capacitor rated 5V or higher (junk bin, big one that fits - mine was 2200uF 25V)
* Wire (~28 AWG) for connecting ESP8266 board
* Wire (~22 AWG) for connecting large capacitor
* Shrink-tubing or tape (to insulate connections at temperature sensor pins)

Information:
* Empty weight of your tank (this is the number marked "TW" stamped on the handle of your tank)
* Max allowed fill of your tank when full (this is the traditional named-size like 20lb for a BBQ)

-

Background:

After trying many things unsuccessfully I determined that none of the off-the-shelf solutions did
what I wanted, most requiring a phone-app and some (such as the Flame King scale) wanting
additional purchases to enable different size tanks in the app.  Even if I purchased it, the
battery life was rather dismal according to reviews and required turning on and off manually
which was not something I wanted to do in cold, snowy, or icy weather and I would have to go
in range of it and wait to update rather than be notified of the low level automagically.

The solution seemed to be building my own scale that I could make run for a longer time, and
have it integrate into Home Assistant.  Unfortuniately how to make the scale reasonably weather
resistant and finding load cells that would be easy to mount sufficiently off the ground to not
be flooded, filled by bugs, etc. was difficult.  So I decided to buy the Flame King scale again
and use it as a base for modification to build my own guts using just the frame and load cells.

The first issue I encountered is most guides have 2 or 4 load cells where this has 3.  I wasn't
sure how to handle that but a friend suggested the existing board probably had the proper
amplifier for the load cells wired correctly, and that I could simply intercept it before the
signal went out over Bluetooth.  After a modest amount of investigation, we determined that
there was a separate Bluetooth radio and microcontroller brain on the board.  I attempted to
trace the wires and eventually had to desolder the microcontroller to see some traces under it.

Once the traces were all visible it became far easier to tell approximately where the analog
signals went into the microcontroller, and a multimeter helped me quickly verify that I had
a valid weight signal around 2-3 volts and varied with pressure.

Armed with this knowledge, I could then tack off wires for the analog-input, LED indicator,
power, and ground to connect the ESP8266.  With a bit of YAML programming I then had the
ESPHome firmware sending raw values, and just had to calibrate it!

Unfortuniately, the load cell output seems to vary by temperature.  This meant that I had to
add in a temperature sensor so I could compensate for the temperature of the scale to get
reasonably accurate readings.

At the same time I was adding the temperature sensor, I also made a modification to the board
cutting a power-trace and connecting the sensor-power rail back to the ESP8266 GPIO pin so
that it can switch off the extra circuitry while in deep-sleep and improve battery life by
approximately double.  There seemed to be an issue with the start-up sequence not letting
power stabilize before attempting to detect the temperature sensor but this can be worked
around by adding a short delay in the YAML boot code.

An easy way to further increase the runtime would be putting a 2x D-cell holder under the
middle of the tank stand and run wires up to the AA battery compartment.  It may also be
possible to use a small solar cell and rechargable batteries to top-up the batteries and
extend runtime as well.  Although I have not tried it, I believe there is also sufficient
space that a micro-USB connector could be plugged into the ESP8266 and carefully routed out
thru the battery compartment hole or by cutting a tiny notch in the cover plate to power from
a mains USB adapter, but my appication is not near a mains outlet (remember, propane you want
away from sources of ignition and away from the house to the extent practical for safety).

-

In the process, I have learned a few things noteworthy:
* The factory board has a voltage regulator that holds 3.3v from input range 1.75-3.35 volts.
* ~150-170mA Power consumption with the ESP8266 fully-active
* ~4mA Power consumption with the ESP8266 deep-sleep and always-on stripped down PCB is
* ~2mA Power consumption with the ESP8266 deep-sleep and modified to shut off sensor power
* The ESP8266 introduces some instability in the 3.3v line affecting readings which results
  in a higher than intended reading, I don't know the voltage regulator specs.  Adding
  a large capacitor across the power rail where the ESP8266 taps off seems to help.
* The load cell calibration drifts with temperature.  Results varied by 6-7lb over 30F.

-

GitHub Files:
Board Pics - Front/Back scaled to identical size and flpped to match positionally.  You can
display both files in a photo-viewer and "toggle" between them to follow traces between the
sides on your computer monitor easily.
* back_scaled.jpg
* back_scaled-nochip-composite.jpg
* front_scaled-flipped-composite.jpg

Board Pics - Original unmodified board, and close-up of microprocessor
* microprocessor_closeup.jpg
* original_board.jpg

Example YAML file
* example ESPHome - fireplace_propane.yaml
* example Home Assistant package - propane_scale_fireplace.yaml

Modification Pics - installation in housing
* Capacitor Added.jpg
* ESP8266 Connected.JPG
* ESP8266 Fit in scale.JPG

Modification Pics - Wiring Reference
* Example Trace Cut.jpg
* (ppt export) Slide 1 - Load Cell Original Wiring.JPG
* (ppt export) Slide 2 - Bottom Signal and Power Tap Points.JPG
* (ppt export) Slide 3 - Top Modification and Power Tap Points.JPG
* (ppt export) Slide 4 - Pinouts.JPG
* wiring.pptx

-

The Modification Process:

1. Take apart the Flame King scale (remove all the bottom screws - be careful not to loose
   the screws, battery door, or small rubber grey fake-button-plug thing)

2. Remove the PCB (remove 2 screws, gently pry board out of place)
   NOTE: Mine was stuck/glued to one of the blue alignment pins.  Use care not to break
         the PCB and try not to bend the LED indicator too badly.

3. Optionally (but recommended) note the position of and desolder each wire to remove the PCB
   NOTE: If you break a wire or lose track of where it goes, the load-cell factory wiring
         diagram is in the file "Modification Pics/Slide 1 - Load Cell Original Wiring.JPG"

4. Carefully remove (desolder or otherwise) the microprocessor from the PCB.  Use care not to
   damage traces or other SMD components.  I used tweasers and a hot-air reflow gun at 600F.

5. Carefully remove (desolder or otherwise) the Bluetooth module from the PCB.  Use care not to
   damage traces or other SMD components.  I used tweasers and a hot-air reflow gun at 600F.
   NOTE: This is probably optional, but it is just wasting power if connected to the board.
         Another option would be identifying and disconnecting the power-pins and leave it.

6. Use a knife or other tool (I recommend a Dremel cutoff wheel to "grind" away) to cut thru
   the PCB trace next to the internal slide-switch to isolate the sensor-power rail from the
   always-on 3.3v regulated power rail.
	 Reference diagram: "Modification Pics/Slide 2 - Bottom Signal and Power Tap Points.JPG"
	 Example Photo: "Modification Pics/Example Trace Cut.jpg"

7. Use small wire, connect each of the power and signal wires to the ESP8266 board on the
   side that the components are populated on (where you removed the microcontroller).

     Reference diagram: "Modification Pics/Slide 2 - Bottom Signal and Power Tap Points.JPG"
     * 3.3V power from large tab of UR1 (to 3.3V rail of ESP8266)
     * Status LED to bottom of R17 near the removed U1 processor (to ESP8266 GPIO)
     * Analog scale reading at corner of C5 near near the corner of the PCB (to ESP8266 GPIO)
     * Switched sensor power to P4 pin 1 (from ESP8266 GPIO)
     * Ground to P4 pin 3

     Reference diagram: "Slide 4 - Pinouts.JPG"
     * 3.3V power rail
     * Ground
     * Jumper wake-timer GPIO to Reset for deep-sleep wakeup operation
     * Connect GPIO pins to appropriate points on the board and sensors

8. Use large wire, connect the power and ground wires.  I recommend tacking them to the
   opposite side of the P4 header (the side with the switches) and running them towards the
   LED indicator where they can drop into a large-ish space in the housing to the capacitor

9. Use small wire, connect the DS18B20 temperature sensor.  I recommend tapping power off
   where the large capacitor was connected, data-wire back to the ESP8266, and the pull-up
   resistor soldered across the power/data pins of the DS18B20 itself seemed the easiest
   way to solder everything free-hand.  Remember to use shrink-tubing or tape to insulate
   the pins of the DS18B20 from shorting to each other or any other components.

10. Program the ESP8266 with ESPHome if you have not already done so.

11. Carefully reinstall the DS18B20 sensor, capacitor, PCB, and ESP8266 (see modification
    photos for ideas) such that they are clear of all housing pertrusions.  I recommend
    test-fitting cover plate before installing the screws.
    NOTE: Since the WiFi module gets slightly warm, I suggest trying to place the
          temperature sensor farther away, near the battery compartment.  I also used a
          small piece of 1/4" thick insulation foam to hold the ESP8266 slightly away
          from the adjacent load-cell foot, but this is optional.

12. Test the scale for operation, ensure that the raw weight ADC value changes with
    weight placed on the scale, and that the temperature sensor is detected and reports
    a reasonable value for the temperature.  Note units in log are C not F.

13. Reinstall the bottom cover (don't forget fake-button-plug-thing in the hole) and screws

14. Calibrate your scale, see ESPHome guide for detals how to calibrate sensor filters
    by modifying the YAML values based on debug log output.

    a) At a steady temperature (such as room temperature), calibrate the weight sensor

       1. Set the scale on it's feet but empty (no load sitting on it)

       2. Power on and review logfile for "*CALIBRATE Weight ADC: adc_raw=" lines, this
          will be the left-side value for your zero-calibration in the YAML

       3. Place a large known weight (suggest 60-100lb) on the scale.  I used one of my
          spare full 40lb propane tanks (~68 lb), and weighed it with a bathroom scale to
          get the actual known weight.

       4. Power on and review logfile for "*CALIBRATE Weight ADC: adc_raw=" lines, this
          will be the left-side value for your large-weight-calibration in the YAML.
          The right-side value is your measured known weight from step 3.

       5. OTA update the ESP8266 firmware with your new YAML file to fix calibartion
     
    b) With a constant known medium weight (say around 20lb), calibrate the temperature
       correction factor.  You will need to do this in multiple temperatures, after
       waiting for the scale to fully aclimate to the temperature so the internals are
       all stabilized to the same temperatured.
       
       1. Start at the same temperature you performed the part-A calibration.

       2. Power on and review logfile for "*CALIBRATE Temp LB-Correction: temp_raw=" lines,
          this will be the left-side value for your weight-calibration "zero compensation"

       3. Allow the scale (and ideally the sample-weight) to cool down to a moderately
          low temperature.  You can do this outdoors in winter, or using a fridge/freezer.
          I recommend 20-30F or whatever is near your winter temperatures that you care
          about having an accurate low-fuel reading at.  If you are using it for a summer
          BBQ grill, you may wish to go the other direction and get a correction offset
          at a high temperature such as 90-100F instead.

       2. Power on and review logfile for "*CALIBRATE Temp LB-Correction: temp_raw=" lines,
          this will be the left-side value for your weight-calibration "zero compensation"

       4. Place the known weight on the scale

       5. Power on and review logfile for "*CALIBRATE Temp LB-Correction: temp_raw=" lines,
          this will be the left-side value for your calibration "cold/hot compensation".

       6. At the same time, also note logfile output for the reported weight reading from
          the line "*CALIBRATE Weight [...] / output="

       7. Take the known-weight, subtract the output reading from step 6.  This is your
          right-side value for the "cold/hot compensation" filter adjustment.

       8. OTA update the ESP8266 firmware with your new YAML file to fix calibartion

-

Bonus Knowledge:

For those who don't already know, some useful knwoledge about propane tanks and how to
identify and calculate capacities and weights...

There are various pieces of information stamped on your tank handle (at least in USA),
here are a few examples from a 20lb tank I have:
```
WC 21.6L   -> 100% full volume Water Capacity in Liters
T 8.0KG    -> Empty Tare Weight in KG
DT 101.6MM -> (unknown)
DOT4BA240  -> DOT tank type
<a serial number>
TW 17.1    -> Empty Tare Weight in Pounds
WC 47.6    -> 100% full volume Water Capacity in Pounds
DT 4.0     -> Type of valve?
M4014      -> Manufacturer code
SJ         -> (unknown)
02 22      -> February 2022 Manufacture Date (most new tanks are certified for 12 years in the USA)
```

Of these, there are only really 3 that are of interest:
```
TW 17.1 -> Empty weight, we need to know for computations
WC 47.6 -> How much it can hold at 100% full
02 22   -> How old it is, so you know if it will be rejected when you get it refilled
```

Rules require portable tanks never be filled over 80% for safety.  This means there is a maximum
capacity that you can calculate based on the information on the tank.

Let's compute the maximum allowable quantity of Propane for this tank...

WC 47.6 -> 47.6 pounds of water; Water weighs 8.345 pounds
```
Maths: 47.6 / 8.345 = 5.70401438 gallons
```

Now we can only legally put 80% of that in it for safety:
```
Maths: 5.70401438 * 0.8 = 4.563211504 gallons
```

* Reputable filling stations will charge you by-the-gallon for how much they pump into your tank.
  This also means you can take a mostly-empty tank and have it filled, only paying for what they
	need to put in to reach the legal fill limit.

* Most places I have been wil "round up" 4.56 to 4.6 gallons of fill in a totally empty tank

But for useful stats to measure quantity of fuel, we need some more conversions to pounds.
At room temperature "77F" propane weighs 4.11 pounds per gallon.
```
Maths: 4.563211504 * 4.11 = 18.754799281 pounds
```
or if you rounded off...
```
Maths: 4.6 * 4.11 = 18.906 pounds
```

Now we need to know what a scale would read, so we go find the empty weight marked "TW"
TW 17.1 lb

So to know the tank weight on a scale what it ought to be when filled, we do more maths
```
Maths: 17.1 lb empty + 18.9 lb fuel = 36 lb total weight
```

The reverse is also possible, which is how my automation works...
Weight the whole thing - suppose you put it on a scale and measure 25 lb
```
Maths: 25 lb measured total - 17.1 lb empty = 7.9 lb fuel remaining
```

An annoying thing once you learn all this, you may notice the swap-stations don't add up.
It seems all the ones I've gone to take "80% of 80%" for reasons I don't understand.
```
Maths: 18.9lb * 0.8 = 15.1 lb
```
And that matches up with my experience of tank-swaps when I put them on a scale I measure
about 14.5-15.0 pounds of gas after subtracting the empty-weight.

Which is really annoying because at least in my area the tank-swap stations for a BBQ tank
I have to pay around $25 for a tank nearly 4 pounds short of the legal fill limit!  By comparison,
when I take it to a filling station I find they often fill it to what my math indicates is
about 19.5 lb of fuel, which may be slightly over the 18.9 we computed was allowed but
ends up being about 2/3 the cost of the tank-swap for a tank that is actually maxed out.

Happy fueling!
