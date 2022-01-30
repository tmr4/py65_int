# Status as of January 29, 2022

Now that I've learned a bit more Python, I've decided to create a package that works on top of py65 rather than modifies it.  I've incorporated interrupts into my 65816 simulation while maintaining the py65 installation intact.  See [py65816](https://github.com/tmr4/py65816) for a version with interrupts.  This is where I'll be making future updates to the simulating interrupts.

# Handling 6502 interrupts in py65
Py65 (https://github.com/mnaberez/py65) is a great simulator for the 6502.  It doesn't handle interrupts though, so if you use interrupt driven I/O in your 6502 project you'll have to modify your code to simulate it in py65.  This could be as simple as defining new getc and putc routines to map py65 I/O to the same addresses as used in your interrupt I/O.  However, maintaining a separate version of your code for simulation may be a hassle if you add more interrupts or if they get too complex.  Luckily, py65 is open-source and modifying it to handle interrupts isn't very difficult.

This repository provides a framework for handling interrupts in py65.  Ideally, you'd specify the interrupt device and it's base address on the command-line.  I'm not there yet and may never be.  I've included sample routines for a basic VIA shift register interrupt that I use to interface with a PS/2 keyboard, an ACIA receiver data register full interrupt that I use for SD card access and ACIA transmitter data register functionality for serial output to a display.  A few modifications need to be made to core py65 modules as well.

# Contents

I've added three modules, interrupts, via65c02 and acia65c02 to allow py65 to handle interrupts.  As samples, I've provided the code needed to handle the interrupts for my current 6502 build.  They should give you an idea for writing your own handlers.  Note that I'm a Python newbie and appreciate any feedback to make these better.

* `interrupts.py`

This file sets up the interrupts you want to simulate.  An instance of the Interrupts class is created in the py65 monitor.  In my sample, upon initialization, the Interrupts class pulls the MPU's IRQ pin high and creates an instance of the VIA and ACIA classes using the provided base addresses for these devices.

* `via65c02.py`

This file defines the VIA class.  In my sample, I've modeled a very simple shift register interrupt which, when enabled, creates a thread that polls the console, checking for a key press.  When it detects one it pulls the MPU IRQ pin low.  The class also sets up a callback to retreive the shift register contents when read.  To facilitate escaping to the monitor I've added code to capture `<ESC>Q` or `<ESC>q`.

* `acia65c02.py`

This file defines the ACIA class.  My sample defines callbacks for the transmitter and receiver registers.  I've also included code to simulate the raw SD card access available on my 6502 build.
  
# Modifications to core py65 modules

I've tried to minimize the changes to the core py65 modules.  The following modifications are needed for py65 to handle the interrupts above:

1. `monitor.py`

* Add a reference to interrupts class `from interrupts import Interrupts`
* Create a new instance of the interrupts class with `self.interrupts = Interrupts(self, self._mpu)` at the end of the `_reset` method.

2. `mpu6502.py`

* The py65 monitor simulates your build by steping through your code, instruction by instruction.  To handle interrupts, we need to check if any have been raised prior to each step. To do this add the following to the beginning of the the step method:

````
if (self.IRQ_pin == 0) and (self.p & self.INTERRUPT == 0):
    self.irq()
    self.IRQ_pin = 1
````

This code calls the MPU's irq method if the IRQ pin has been pulled low AND if interrupts are enabled.  It then resets the IRQ pin.  This could be done elsewhere depending on your needs.

3. `console.py`

* Though not needed to handle interrupts generally, the following method addition to the console module is needed to run my sample:

````
def kbhit():
    return msvcrt.kbhit()
````

4. `monitor.py`

* Again, not needed to handle interrupts generally, but to properly break to the monitor on `<ESC>Q` in my sample you need to correct a few bugs in the py65 monitor module.  In the `__init__`, `onecmd` and `_run` methods, add `self.unbuffered_stdin` as an argument to the call to `console.restore_mode`.

5. `mpu6502.py`

* Note that pip contains version 1.1.0 of py65.  If you have this version you will also need to copy the irq method from the mpu6502 module from version 2.0.0.dev0 on GitHub to use my sample code.  Alternatively, you can modify the development version of the py65 monitor and run it separately from your pip downloaded version.



