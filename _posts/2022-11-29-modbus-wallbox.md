---
layout: post
title: Controlling a Wallbox via Modbus-RTU
image: "/img/pvcar.jpg"
---

When operating a [photovoltaic (PV) system](https://en.wikipedia.org/wiki/Photovoltaic_system) at home in 2022, we usually want to make good use of the electrical energy harvested by the system -- instead of feeding too much excess energy back into the grid. While this is definitely true for the setup we're operating here (in Germany), the situation might be different for others: it depends on how much the grid operator pays you for energy fed back and other factors.

<p align="center">
   <img alt="" src="/img/pvcar.jpg" width="80%" />
   <br />
   Image <a href="http://www.freepik.com">designed by macrovector / Freepik</a>
</p>

One way to consume more energy yourself is to charge electric vehicles at home. Ideally, the vehicle would dynamically adjust its charging parameters based on excess energy available in the system, i.e. **consume exactly the amount of energy available**. A Wallbox like the Heidelberg Energy Control can help approach this goal, as we will see in this article.

This box offers a serial [RS-485](https://en.wikipedia.org/wiki/RS-485) interface that supports [Modbus RTU](https://en.wikipedia.org/wiki/Modbus). I think this is a great opportunity to learn more about these protocols - so let's get some bus connector hardware and start experimenting!

## Modbus Register Map
The first step is to take a look at the vendor's docs, to find out what readings and settings are available for us to use. You can google for `Heidelberg Energy Control Externes Lastmanagement` to find the PDF containing the register map and lots of other instructions, or take a look at [this script](https://github.com/ronalterde/wallbox/blob/main/wallbox/core.py).

Modbus defines several *object types* and their corresponding *function codes*. According to the register map, a subset of each is implemented for this particular device.

### Object types
- Input Register (read-only), 16 bits
- Holding Register (read-write), 16 bits

### Function codes
- 3: read multiple holding registers
- 4: read multiple input registers
- 6: write a single holding register

The register map reveals the address of the most important register for our use case: the one for **setting maximum current**. It is a holding register at address #261, with a range between 60 and 160, corresponding to a max. [RMS](https://en.wikipedia.org/wiki/Root_mean_square#Voltage) current of 6-16A.

Note the PDF also contains some information about which SW version implements what subset of registers. It might be worth checking the SW version of your device first (register #4). For instance, it looks like the max. current command (address #261) got added with version 1.0.7 while others have been present since version 1.0.0.

## Hardware Setup
Before attempting to establish a Modbus connection we want to make sure to satisfy a few hardware prerequisites.

The PDF mentioned above contains descriptions of various micro switches the box offers, and their possible configurations (you have to turn off power and open the box to get to these). I've set mine as follows:

|Switch|Setting|Description|
|------|-------|-----------|
|S1|5|Max. current 16A|
|S2|0000|(relevant for bus ID #16 only)|
|S3|0|Min. current 6A|
|S4|0001|Bus ID #1|
|S5|0000|Front LED turns off after 5 minutes; box acts as a server|
|S6|0100|Termination resistor enabled|

This switch configuration is suitable for a single box acting as a Modbus server (formerly called *slave*), utilizing the full current range of 6 through 16A.

With the desired switch configuration set up, make sure to wire up your USB-to-RS485 adapter correctly (signals A, B, and GND) and ideally terminate the bus on the side of the USB adapter using a 120 Ohm resistor across A and B. I'm sure there are lots of USB adapters suitable for this job; I've got a `DSD TECH SH-U10L USB` from Amazon that has been working well so far.

## First connection to the box

Now that hardware is ready to go, and we are equipped with knowledge about object types and registers, an initial sanity check would be nice. Let's first query some of the read-only registers, to make sure the box responds as expected. The input register at address #10 is a good candidate: it tells us the **RMS voltage between L1 and N**, in volts.

There are various tools and libraries out there supporting Modbus-RTU over a USB-RS485 adapter. I can recommend [QModBus](https://github.com/ed-chemnitz/qmodbus/), a graphical program. I found it pretty useful when exploring the register map of the box. That said, for the following examples we will be using [MinimalModbus](https://minimalmodbus.readthedocs.io/en/stable/), a Python package that works just as fine, and helps to automate things.

The following Python snippet is enough to establish a connection with the box (please double-check the device name assigned to your USB adapter; is it `ttyUSB0`?):
```python
import minimalmodbus
instrument = minimalmodbus.Instrument(port=/dev/ttyUSB0, slaveaddress=1)
instrument.serial.parity = minimalmodbus.serial.PARITY_ODD
instrument.serial.timeout = 1 # in seconds
```

Now let's issue a read request to the voltage register mentioned above:

```python
voltage_L1 = instrument.read_register(10, functioncode=4)
```

When I initially tried this, surprisingly there was **no response at all**. Some research shows that the device features a standby mode that apparently even applies to the communications interface: the box would shut down the serial interface if there's no vehicle connected for some time! In case you run into that issue, for now, I suggest to just **power-cycle the box** and try again.

Now, the box should respond with a value of around 230 volts, which sounds plausible as an RMS voltage between phase and neutral.

## Disabling Standby & Watchdog
To permanently disable standby, at least for the debugging phase we're in right now, write a value of *4* to register #258:
```python
instrument.write_register(258, 4, functioncode=6)
```

From what I can tell this keeps standby mode disabled, even across power cycles.

In addition to Standby, there's yet another feature we might want to disable while exploring communications to the box: **the watchdog**. It is enabled by default and makes the box blink after a while (of no Modbus traffic? I haven't really dug into that, to be honest).

To disable the watchdog altogether, write a timeout value of *zero* to register #257:
```python
instrument.write_register(257, 0, functioncode=6) 
```

Now that we have tackled standby and watchdog, we can start focusing on what the original intent of this was: setting maximum current.

## Setting maximum current
The following snippet sets the max. current to 6A, which is the smallest value possible.
```python
self.instrument.write_register(261, 60, functioncode=6) 
```

I *think* (but haven't tried) the allowed range for this register is defined by how we've set the micro switches before, in our case 6A - 16A. What I *have* observed is that the register value would drop to zero upon writing any value less than 60, in that hardware configuration.

For experimenting with this register, we can also read back its value, like so:
```python
self.instrument.read_register(261, functioncode=3) 
```

So that's exactly what we need: we can dynamically adjust the maximum current supplied to a vehicle, in a range between 6 and 16 Ampere. This is equivalent to a power range of 4 through 11 kVA, assuming we're dealing with a three-phase system.  

## Phase Switching
Thinking about that power range, this illustrates a drawback of this particular wall box. While it makes sense to operate it at three phases to be able to reach the maximum charging power available, this won't allow for adjusting below 4 kVA (6A * 230V * 3).

This is quite a limiting restriction for the combined Wallbox-PV use case: we would need at least 4 kVA of excessive power in the system to even command the box to start charging.

To bridge the gap between 0 and 4 kVA, it would help to operate the device at a single phase, which would result in a range between 1.4 kVA and 3.6 kVA. While there are other boxes available that implement switching between one- and three-phase operation, that feature is not included with this box. I've seen some ideas related to this on the [WBEC project on Github](https://github.com/steff393/wbec/issues/7).

## Summary

We have shown this box can be used for limiting the maximum power supplied to a vehicle while charging, within the limits of 4 kVA and 11 kVA. There is a single register that allows us to modify this value dynamically from a Python script. We found a limitation of that particular box in the sense that it does not support dynamic phase switching -- but realized there might be ways to add that capability externally.

I've created a tiny Python package that allows you to easily accomplish the tasks mentioned in this article. You can find it at [github/ronalterde/wallbox](https://github.com/ronalterde/wallbox/), and here's how to use it:
```python
from wallbox import Wallbox, registers

# Note: instrument is meant to be an instance of minimalmodbus.Instrument
wb = Wallbox(instrument, registers)

wb.read_register(10)
wb.disable_watchdog()
wb.enable_standby(False)                                                               wb.set_max_current(65)       
```

## References
- [MinimalModbus (Python library)](https://minimalmodbus.readthedocs.io/en/stable/)
- [Discussion on Github about Standby mode](https://github.com/ioBroker/AdapterRequests/issues/559) [German]
- [Discussion on Github about automatic phase switching](https://github.com/steff393/wbec/issues/7) [German]
- [WBEC project: WiFi interface for Heidelberg Energy Control](https://github.com/steff393/wbec) [German]
