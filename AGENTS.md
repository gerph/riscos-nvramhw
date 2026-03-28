# NVRAMV hardware prototype module

This module provides a prototype implementation of the NVRAMV vector in a RISC OS module
called `NVRAMHW`.

The module has claims for the NVRAMV vector. The `c/module` file contains the module
interfaces for the vector. The `c/nvram` file contains the stub functions that would
implement the hardware interfaces within the module to retrieve or set the data.

There are comments within the `c/nvram` source file which give examples of how you might
interact with the NVRAM in an IIC PCF8583 device to perform the hardware operations.
The stub implementation in the `c/nvram` file includes debug output which reports what
is being done, and some dummy values are returned when reads are requested.

SWI calls are in the `c/os` file, not inline with the interfaces within the module.

The `nvramv.xml` describes the interface to the NVRAMV vector.

A BBC BASIC program `TestNVRAM,fd1` will check that the behaviour of the module is
correct.
