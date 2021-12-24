# Handling 6502 interrupts in py65
Py65 (https://github.com/mnaberez/py65) is a great simulator for the 6502.  It doesn't handle interrupts though, so if you use interrupt driven I/O in your 6502 project you'll have to modify your code to simulate it in py65.  This could be as simple as defining new getc and putc routines to map py65 I/O to the same addresses as used in your interrupt I/O.  However, maintaining a separate version of your code for simulation may be a hassle if you add more interrupts or if they get too complex.  Luckily, py65 is open-source and modifying it to handle interrupts isn't very difficult.
This repository provides a framework for handling interrupts in py65.  Ideally, you'd specify the interrupt and it's address on the command-line.  I'm not there yet and may never be.  I've included routines for a basic VIA shift register interrupt that I use to interface with a PS/2 keyboard, an ACIA receiver data register full interrupt and ACIA transmitter data register functionality.  A few modifications need to be made to core py65 modules as well.

# Contents

* `interrupts.py` sets up the interrupts
* `via65c02.py` handles VIA interrupts
* `acia65c02.py` handles ACIA interrupts

# Modifications to core py65 modules

1. `monitor.py`

2. `mpu6502.py`

