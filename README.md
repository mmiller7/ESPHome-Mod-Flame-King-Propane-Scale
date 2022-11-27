# ESPHome-Mod-Flame-King-Propane-Scale
High level reverse-engineering and modification of Flame King YSN-PS1 Bluetooth propane scale to ESPHome WiFi

This project started because I wanted a propane tank scale so I could track (and remind me to refill)
the propane tank running our gas fireplace.  This was made harder because I use 40lb tanks and many
scales only support 20lb tanks.

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

In the process, I have learned a few things noteworthy:
* The factory board has a voltage regulator that holds 3.3v from input range 1.75-3.35 volts.
* Power consumption with the ESP8266 and stripped down PCB is ~4mA deep-sleep ~150-170mA active.
* The ESP8266 introduces some instability in the 3.3v line affecting readings which results
  in a higher than intended reading, I don't know the voltage regulator specs.  Adding
  a large capacitor across the power rail where the ESP8266 taps off seems to help.

Battery life still needs improvement.  It ocurrs to me an easy "fix" to increase the
runtime would be putting a 2x D-cell holder under the middle of the tank stand and run wires
up to the AA battery compartment.  It may also be possible to use a small solar cell and
rechargable batteries to top-up the batteries and extend runtime as well.  Although I have
not tried it, I believe there is also sufficient space that a micro-USB connector could be
plugged into the ESP8266 and carefully routed out thru the battery compartment hole or by
cutting a tiny notch in the cover plate to power from a mains USB adapter, but my appication
is not near a mains outlet (remember, propane you want away from sources of ignition and
away from the house to the extent practical for safety).


Recommended Modification Tools/Supplies:

* Tweasers
* Soldering Iron
* Hot Air Reflow gun (~600F)
* Wire (~28 AWG) for connecting ESP8266 board
* Wire (~22 AWG) for connecting large capacitor
* ESP8266 module (suggest D1 Mini)
* Large value capacitor rated 5V or higher (junk bin, big one that fits - mine was 2200uF 25V)


The Modification Process:

1. Take apart the Flame King scale (remove all the bottom screws - be careful not to loose
   the screws, battery door, or small rubber grey fake-button-plug thing)

2. Remove the PCB (remove 2 screws, gently pry board out of place)
   NOTE: Mine was stuck/glued to one of the blue alignment pins.  Use care not to break
	       the PCB and try not to bend the LED indicator too badly.

3. Optionally (but recommended) note the position of and desolder each wire to remove the PCB

4. Carefully remove (desolder or otherwise) the microprocessor from the PCB.  Use care not to
   damage traces or other SMD components.  I used tweasers and a hot-air reflow gun at 600F.

5. Carefully remove (desolder or otherwise) the Bluetooth module from the PCB.  Use care not to
   damage traces or other SMD components.  I used tweasers and a hot-air reflow gun at 600F.
	 NOTE: This is probably optional, but it is just wasting power if connected to the board.
	       Another option would be identifying and disconnecting the power-pins and leave it.

6. Use small wire, connect each of the power and signal wires to the ESP8266 board on the
   side that the components are populated on (where you removed the microcontroller).

     Reference diagram: "Modification Pics/Signal Tap Points.JPG"
     * Status LED to bottom of R17 near the removed U1 processor
		 * Analog scale output reading to the corner of C5 near near the corner of the PCB
		 * 3.3V power to P4 pin 1
		 * Ground to P4 pin 3

7. Use large wire, connect the power and ground wires.  I recommend tacking them to the
   opposite side of the P4 header (the side with the switches) and running them towards the
	 LED indicator where they can drop into a large-ish space in the housing to the capacitor

8. Program the ESP8266 with ESPHome if you have not already done so.

9. Carefully reinstall the PCB, ESP8266, and capacitor (see modification photos for ideas)
   such that they are clear of all housing pertrusions.  I recommend test-fitting cover
	 plate before installing the screws.

10. Test the scale for operation, see ESPHome guide for how to calibrate load cell readings

11. Reinstall the bottom cover (don't forget fake-button-plug-thing in the hole) and screws

