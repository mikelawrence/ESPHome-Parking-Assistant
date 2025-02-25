# ESPHome Parking Assistant
From early on I learned that parking the car in the garage was a problem the needed to be solved. My Dad solved his problem with a piece of styrofoam hanging from the ceiling. When it touched the windshield you knew to stop. This was very important because the lawn mowing equipment was in front of the car. Crunch time was inevitable, don't ask me how I know. 

I recently moved and the problem became an issue again. Not because there was a crunch time incident but because we needed as much room as possible in front of the car. Having to get out of the car multiple times to make sure the door won't hit the car was a pain. So I did some searching on Youtube and found this [video](https://www.youtube.com/watch?v=HqqlY4_3kQ8) from ResinChem Tech. I really liked his use of a RGB LED strip to indicate position and when to stop. There is a ton of information so watch the whole video.1

***So yes this project is a complete ripoff of his idea!*** But I wanted to do several things different. First I wanted to use ESPHome. Second I really like designing my own hardware. And third I want to keep learning how to use Fusion 360 to design 3D printed stuff.

## Design Decisions
### ESP32 and ESPHome
ESPHome is closely aligned with Home Assistant. In fact they are the same company. Home Assistant uses ESPHome for some of their hardware like the [Home Assistant Voice Preview Edition](https://www.home-assistant.io/voice-pe/) or [Bluetooth Proxy](https://esphome.io/components/bluetooth_proxy.html). I chose a newer ESP32-S3 for this design which also has Bluetooth support. The newer ESP32's work better with the ESP-IDF framework as opposed to the default Arduino framework. Not everything is supported in the ESP-IDF, particularly several of the RGB LED librariesbut the ESP32 RMT LED Strip library works well once you make sure have enough RMT memory allocated. Sometimes I add 

### USB-C Power Delivery
I liked the idea of using RGB LED Pixels since ESPHome supports these directly without custom code. I wanted a flexible power supply for driving the RGB LED Pixels both in the power capacity and voltage. I've been using a MagWLED driver for driving RGB LED pixels for a while now and I really like it's approach to using USB-C PD (Power Delivery) to select between 5V and 12V. The USB-C PD 2.0/3.0 specification supports currents greater than 3A but will need a USB-C Power Cable with E-Marker and a USB-C PD Power Source that supports these higher currents. I went with the simpler and more supported max 3A. The USB-C PD 2.0/3.0 specification will negotiate several fixed voltages at 3A: 5V-15W, 9V-27W, 12V-36W, 15V-45W and 20V-60W. 12V-36W PD is less common in chargers but 9V-27W is always available. 12V RGB LED Pixels will usually run on 9V, the difference in LED brightness is not that noticeable and may make your LED live longer. To handle the USB-C PD negotiation I chose the STUSB4500 from ST Microelectronics. This guy will support negotiate all of the USB-C PD 2.0/3.0 specification but this design will support only the following: 5V-15W, 9V-27W or 12V-36W. The STUSB4500 will be configured to support 12V-36W with a fallback of 9V-27W or just 5V-15W

The STUSB45000 will negotiate 20V by default if the USB-C PD Power Source supports it so our power supplies must support an input voltage range of 5V - 20V. The STUSB4500 uses an external pass transistor to enable V<sub>BUS</sub> at higher currents and voltages when PD is properly negotiated. Until the configured PD is properly negotiated this pass transistor is turned off. Using USB-C PD to provide a variable input voltage from USB-C PD is elegant but if 9V-27W or 12V-36W PD is configured 3.3V is still needed for the ESP32 and 5V for the TFMini. This means two DC/DC converters are necessary, one 3.3V @ 600mA to support primarily the ESP32-S3 chip and 5V @ 1.0A primarily for the TFMini Plus sensor. 

The 3.3V power supply will always have an input voltage greater than 3.3V. I could use a linear regulator but I wanted to get more experience designing efficient DC/DC converters so I chose the TPS560430 SIMPLE SWITCHER® 4-V to 36-V, 600-mA Synchronous Step-Down Converter. The 3.3V power supply is connected directly to the USB connector and will power the ESP32-S3 immediately. If you connect this board to a older USB 2.0 power source you may only get 5V at 500mA. This is enough to get the ESP32-S3 running without violating the 500mA max requirement. 

For the 5V @ 1A power supply it's input voltage can be 5V so a regular step-down will not work. I chose the TPS552872, 36-V, 4-A, Fully Integrated Buck-boost Converter also from TI. This converter will switch to boost mode when the input voltage is 5V and still operate correctly. This power supply must be connected to the V<sub>BUS</sub> pass transistor and not directly to the V<sub>BUS</sub> on the USB connector. Doing so would exceed the 500mA maximum when connected to a USB 2.0 power source. Some USB 2.0 Power Sources support more than 500mA but the specification states 500mA maximum. So until PD is properly negotiated the V<sub>BUS</sub> to the RGB LED pixels and the 5.V power supply are not enabled.

An INA228 85-V, 20-Bit, Ultra-Precise Power/Energy/Charge Monitor With I2C Interface device is used to monitor both energy and power consumption. Energy consumption is a better metric than instantaneous power for understanding actual device usage. Just look at what the power company uses to bill you (kWh).

### TFMini Plus
ResinChem Tech compared multiple distance sensors in his video. He settled on the TFMini-S sensor. I also chose a TFMini sensor but went with the TFMini Plus because it was in a more durable package with a lens cover. The spot size is different but in my tests this sensor worked just fine. This sensor can use a UART or I2C interface. The default UART interface can be set to automatically send measurements at a specified rate while the I2C must be setup with a UART command first and requires polling. The TFMini Plus requires 5V power but uses 3.3V logic levels.

### RGB LED pixel interface
This board only supports RGB LED pixels that have a single data line for communication. SPI pixels are NOT supported. RGB LED pixels requires 5V logic level on the data and while many will tell you that it will work just fine with 3.3V logic I prefer to be more conservative. After doing some [research](https://electricfiredesign.com/2021/03/12/logic-level-shifters-for-driving-led-strips/) I chose the TC4427V MOSFET Driver. It can source/sink 1.5A; definatley help long cable runs. Note the V temperature range option. The V option does more that expand the temperature range it also lowers V<sub>IH</sub> from 2.4V to 2.0V. This give plenty of margin for 3.3V ESP32 with a worst case V<sub>OH</sub> of 0.8\*VCC or 0.8\*3.0=2.4V.


## Enclosure

The enclosure is designed to mount on [Elfa](https://www.containerstore.com/s/elfa/garage-plus-by-elfa/garage-plus-by-elfa-predesigned-spaces/garage-plus-7-tier-6-wall-shelving-solution/123d?productId=11023646) wire shelving. I used [Autodesk Fusion](https://www.autodesk.com/products/fusion-360/overview) to design this enclosure. I'm a novice with this tool but I'm always looking to improve. For this design I tried out a few new features of Fusion, namely Rendering and Animation.

<p align="center">
    <a href="enclosure"><img src="enclosure/meta/ESPHome%20Parking%20Assistant%20Enclosure%20Exploded.gif" width="70%"></a> <br />
    Exploded View of Enclosure
</p>

## PCB

This 4-layer board is designed with [KiCad](https://www.kicad.org/). I been using this free tool for a while and it just keeps getting better and better.

<p align="center">
    <a href="pcb"><img src="pcb/meta/ESPHome-Parking-Assistant-Render.png" width="70%"></a> <br />
    3D Render of ESPHome Parking Assistant PCB
</p>

## Operation

### USB-C Power Delivery
By USB-C PD 2.0 we make it easy to change the voltage for the RGB LED pixel but you have to have the right charger. 

### LED status
There are six LEDs on the PCB to indicate status.
* **Status:** Will be green when operating conditions are normal and red when there has been an error.
* **+5V:** Will be green when the +5V power supply is on. Note that +5V will only work when PD has been negotiated.
* **+3.3V:** Will be green when the +3.3V power supply is on. This power supply is always on when connected to USB-C or USB 2.0 power sources. If the LED is off then the USB is not connected or there is a failure.
* **PD5V:** Will be green when the PDO1 (+5V) has been negotiated.
* **PD9V:** Will be green when the PDO2 (+9V) has been negotiated.
* **PD12V:** Will be green when the PDO3 (+12V) has been negotiated.

> [!NOTE]
> Only one of the three PD LEDs will be lit at a time. If no PD LEDs are lit then no PD was negotiated.

### Connecting RGB LED strips
The mate to the on-board terminal strip connector is Phoenix Contact 1847071.

The LED connector has 4 wires.
* **PD**: This is the LED voltage wire, often labeled 12V+ or 5V. This PCB supports up to 3A at 5V, 9V or 12V as long as the correct USB-C PD Power Source is connected.
* **33Ω**: This is data line with a 33Ω series resistor. Use this for longer wire runs over 10ft.
* **249Ω**: This is the same data line but with a 249Ω series resistor. Use this for shorter wire runs.
* **GND**: This is minus side of the LED voltage.

> [!NOTE]
> Connect only one of 33Ω or 249Ω to your LED Strip.

> [!NOTE]
> Many 12V RGB LEDs run just fine on 9V. In fact running at 9V may extend their life expectancy.

## Configuration
### STUSB4500 Configuration
> [!CAUTION]
> When configuring the STUSB4500 for the first time you should realize its default configuration will allow up to 20V on V<sub>BUS</sub>. This PCB works fine with 20V but this voltage is passed to the LED connector. Make sure your LEDs are disconnected before configuring the STUSB4500.

1. Uncomment either the 12V LED section or the 5V LED section in the ```config.yaml``` file.
2. Uncomment ```flash_nvm: true``` in the ```config.yaml``` file
   You should see the following messages in your log.
     ```log
     [00:10:03][C][stusb4500:112]: STUSB4500:
     [00:10:03][E][stusb4500:119]:   NVM has been flashed, power cycle the device to reload NVM
     ```
   If you forgot to uncomment ```flash_nvm: true``` to the configuration you will see an error in the log.
     ```log
     [00:06:27][C][stusb4500:112]: STUSB4500:
     [00:06:27][E][stusb4500:114]:   NVM does not match current settings, you should set flash_nvm: true for one boot
     [00:06:27][C][stusb4500:130]:   PDO3 negotiated 20.00V @ 1.00A, 20.00W
     ```
3. Power cycle your board. Make sure ```NVM matches settings``` is present in the log and that one of your PDOs was negotiated. The example below shows the result of the 12V LED configuration.
   ```log
   [00:00:19][C][stusb4500:112]: STUSB4500:
   [00:00:19][C][stusb4500:116]:   NVM matches settings
   [00:00:19][C][stusb4500:130]:   PDO3 negotiated 12.00V @ 3.00A, 36.00W
   ```
4. If you selected the 12V LED config section you should see either the PD12V LED or PD9V LED lit. PD5V should be lit if you selected the 5V LED section. 

5. The STUSB4500 component does check to see that the NVM is indeed different before flashing the NVM but it is prudent to remove the ```NVM matches settings``` after it is clear the STUSB4500 is working as intended. You don't want to wear out your NVM.

### Installation
Place the enclosure so the the sensor hits as flat an area as possible. For my car this is the license plate. You may have to play different options for best performance. My experience was pretty much place and forget. It just

Now the sensor is placed, close the garage door with no car present. Home Assistant will show the measured distance. Convert to cm if necessary and subtract half of ```door_tolerance``` set ```door_distance``` to this value.

Next we set ```start_distance``` to ```door_distance``` - 25 and install the current config.

For the final setting, open the garage door and park your car where you want it. Set ```stop_distance``` using the converted distance, if necessary, from Home Assistant and again install the current config.

You now have a basic setup. Testing is in order. Back the car first and then start moving back in. Watch the LED strip light turn on and start movig toward the center as you move the car in. Stop as soon as the LED strip light turn green. Check the parking position and adjust ```stop_distance``` a little bit. As you repeat this test you will get a feel for how much to change ```stop_distance``` to get the car to stop in the right place.

## Status
  * PCB Rev B: Is currently being fabricated and assembled by JLCPCB. Rev A worked well enough to write the code for ESPHome including the new STB4500 and TFMini components. The main problem with Rev A was the 5V DC/DC converter won't start at 5V. This was technically in the datasheet but there was also a minimum operating voltage the was well below 5V input. I didn't realize that you need to apply more than 5V for the regulator to start working and then you can lower the input to 5V. Not exactly what I needed.
  * Enclosure: Printed and successfully tested on Rev A. Rev B is ready to go when the PCB comes in.
