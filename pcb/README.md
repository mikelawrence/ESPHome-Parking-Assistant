# ESPHome-Parking-Assistant PCB
<p align="center">
    <img src="meta/ESPHome-Parking-Assistant-Render.png" width="70%">
    3D Render of ESPHome Parking Assistant PCB
</p>

The PCB is designed in [KiCad](https://www.kicad.org/). Fabrication and Assembly is tuned toward [JCLPCB](https://jlcpcb.com/). The production folder has everything you need for JLCPCB to send you fully assembled boards.

## Bill of Materials
<p align="center">
    <a href="https://htmlpreview.github.io/?https://github.com/mikelawrence/ESPHome-Parking-Assistant/blob/main/pcb/interactive-bom/index.html"><img src="pcb/meta/ESPHome-Parking-Assistant BOM" width="50%"></a> <br />
    Interactive BOM
</p>

## Operation

> [!CAUTION]
> When configuring the STUSB4500 for the first time you should realize the default configuration will allow up to 20V on V<sub>BUS</sub>. This PCB works fine with 20V but this volatage is passed to the LED connector. Make sure your LEDs are disconnected before configuring the STUSB4500.

### LED status
There are six LEDs on the PCB to indicate status.
* **Status:** Will be green when operating conditions are normal and red when there has been an error.
* **+5V:** Will be green when the +5V power supply is on. Note that +5V will only work when PD has been negotiated.
* **+3.3V:** Will be green when the +3.3V power supply is on. This power supply is always on when connected to USB-C or USB 2.0 power sources. If the LED is off then the USB is not connected or there is a failure.
* **PD5V:** Will be green when the PDO1 (+5V) has been negotiated.
* **PD9V:** Will be green when the PDO2 (+9V) has been negotiated.
* **PD12V:** Will be green when the PDO3 (+12V) has been negotiated.

> [!NOTE]
> Note: that only one of the three PD LEDs will be lit at a time. If no PD LEDs are lit then no PD was negotiated.


## PCB Info
* 4 layer to support an additional ground plane and signal layer to reduce plane interruptions on the outer layers.
* Dimension are 85mm X 59mm.
* Requires oven reflow or hot air to assemble some components.
