---
layout: post
title: "Prototyping with MicroPython"
image: "/assets/micropython.png"
---

A recent toy project of mine involves controlling SPI devices through an STM32 MCU. Right from the beginning, I expected a lot of prototyping-style back-and-forth so I thought this would be a great opportunity to get to know MicroPython.

MicroPython is a Python 3 implementation targeted at microcontrollers, and therefore more lightweight compared to the original CPython distribution. A neat feature is that you can access a Python REPL on the microcontroller's serial port. In a way, this feels like a *serial command line interface on steroids*.

My setup consists of a battery-powered prototype board and a Raspberry Pi so I can control all board peripherals from a remote terminal session over WiFi, without ever re-flashing the microcontroller. That said, in this article I will focus on a specific STM32 eval board so you can relate to my experience and follow along easily.

## Building for a certain MCU or board

If you are lucky your board might be already supported by one of MicroPython's ports (see [ports](https://github.com/micropython/micropython/tree/master/ports) and [ports/stm32/boards](https://github.com/micropython/micropython/tree/master/ports/stm32/boards)). If not, there's a good chance you can take one of the existing boards as a template and create a new *board* yourself. The STM32 port is based on ST's Cube drivers (see [lib/stm32lib](https://github.com/micropython/stm32lib)) so most of the low-level pieces should be there already.

For now, let's assume a *NUCLEO F411RE* eval board. For this board, we can build a MicroPython executable like this:
```
git clone https://github.com/micropython/micropython
cd micropython
cd ports/stm32
make submodules
make BOARD=NUCLEO_F411RE
```

This will create a `firmware.elf` file under `build-NUCLEO_F411RE` you can then flash onto the board (using [st-util](https://github.com/stlink-org/stlink) or another method of your choice). After flashing, you'll end up with a Python interpreter running on the MCU -- so this might be the last time you actually needed to program machine code onto the board ;-).

## Getting a Python REPL on the serial port
When launching a *regular* Python interpreter on your computer you get a command prompt that starts with `>>>`. This command line interface is also called *Read-Eval-Print Loop (REPL)*. Amazingly, you will get the same experience on MicroPython right out of the box. To try it, connect a Serial-to-USB cable to your board and launch a terminal program, e.g.:
```
picocom /dev/ttyUSB0 -b115200
```

This will also give you a `>>>` prompt -- but this one is running on your microcontroller!

Using this REPL, you can read and write GPIOS (using [pyb.Pin](https://docs.micropython.org/en/latest/library/pyb.Pin.html)):
```
some_input = Pin('PD2', Pin.IN)
print(some_input.value())
```

You may also write datagrams to the SPI bus:
```
spi = SPI(1, SPI.MASTER, baudrate=200000, polarity=1, phase=1)
cs = Pin('CS', Pin.OUT_PP)

cs.low()
rx_data = spi.send_recv(b'some_tx_data')
cs.high()
```

You can, of course, also define Python classes and functions and make use of them later in the program:
```
class MyClass:
	pass

def foo();
	pass
```

There is a great description of all the available modules here: [MicroPython libraries](https://docs.micropython.org/en/latest/library/).

## The Raw REPL
MicroPython offers a special mode of the REPL you can use to transfer Python code: the [raw REPL](https://docs.micropython.org/en/v1.8.6/pyboard/reference/repl.html#raw-mode). Enter it by pressing `Ctrl-A` while connected (or `Ctrl-A, Ctrl-A` from inside picocom). This will make the *user-friendly* prompt disappear until you press `Ctrl-D` to "commit" your code.

This mode is very useful to transfer code in an automated way, e.g. from another Python program running on your host computer (or Raspberry Pi, for that matter). For instance, I'm using the raw REPL to load a Python module initially and then call individual methods later.

There's a little helper module to make this easier: [tools/pyboard.py](https://github.com/micropython/micropython/blob/master/tools/pyboard.py). It takes care of the protocol for us (i.e. those key sequences described above):

```py
import pyboard

# Connect. Note we're using a different serial device
# than before, just because this is to be executed
# on a Raspberry Pi
pyb = pyboard.Pyboard('/dev/ttyS0', 115200)

# Execute 'init' code; see below
pyb.enter_raw_repl(soft_reset=True)
pyb.execfile('init.py')
pyb.exit_raw_repl()

# Call foo() now.
pyb.enter_raw_repl(soft_reset=False)
pyb.exec("foo()")
pyb.exit_raw_repl()
```

```py
# init.py

def foo():
  print("foo() called!")
```

Did you notice the `soft_reset` flag in the snippet above? Setting it to False is important if you want to keep MicroPython's state over several REPL sessions.

## The Raspberry Pi's serial port
If you intend to use a Raspberry Pi to access MicroPython's prompt, like I did, there is another thing to keep in mind: By default, the serial port is utilized as a serial console by the Linux kernel. That is, you will see kernel messages coming out of the serial port when connected to your host computer -- which is certainly useful but unfortunately, this prevents us from talking to the microcontroller.

To check if this is an issue for you, run the following shell command. This will get a list of kernel arguments currently in use, and filter for relevant lines. Eventually, it will show you one or more lines starting in `console=`.
```sh
cat /proc/cmdline | sed 's/ /\n/g' | grep console
```

You can tell the serial console is enabled if `ttyS0` is part of the output, e.g.:
```sh
console=tty1
console=ttyS0,115200
```

To disable the console feature, run `sudo raspi-config`, select `Interface Options`, `Serial Port`. Choose `Would you like a login shell to be accessible over serial?` -> No `Would you like the serial port hardware to be enabled` -> Yes.

After rebooting the Pi, run the shell command again. The `ttyS0` line should be gone:
```sh
console=tty1
```

Note there is some confusion related to `ttyAMA0` vs. `ttyS0`. This is because several models of the Raspberry Pi are available; some of them have Bluetooth chips (connected via serial) and others don't. Refer to [elinux -- Prevent Linux from using the serial port](https://elinux.org/RPi_Serial_Connection#S.2FW:_Preventing_Linux_from_using_the_serial_port) for details and/or if `raspi-config` won't do the trick.

## Conclusion

Although I'm a huge fan of C++ on Embedded, MicroPython seems like a nice way for tasks like prototyping and board bringup (as an alternative to a simple command line interface). It is probably also suitable for automated hardware testing where the MCU acts as a gateway anyway, i.e. its sole purpose is to forward requests to peripherals on the board and return their responses. That said, I wouldn't consider MicroPython for production use, based on what I have seen up until now.

I like the fact that MicroPython is regular C code so I think it's possible to implement missing features, in case something is missing (which I haven't encountered yet).

I'll leave you with some ideas for now / things I haven't tried yet:
- Check out the 'unix' port
- Find out how to integrate Python scripts into the elf file
- Try the file system on a board that supports it

## References

- [pyboard](https://docs.micropython.org/en/latest/reference/pyboard.py.html)
- [Getting a REPL](https://docs.micropython.org/en/latest/esp8266/tutorial/repl.html)
- [MicroPython Github](https://github.com/micropython/micropython)
- [Title Image from Wikimedia](https://commons.wikimedia.org/wiki/File:Micropython-logo.svg)
